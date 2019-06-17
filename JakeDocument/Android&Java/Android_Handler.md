# Android Handler 详解

### 消息机制与Handler

#### 

#### 1. 基本概念

Android的消息机制主要包括Handler、MessageQueue和Looper。

Handler是Android中引入的一种让开发者参与处理线程中消息循环的机制。每个Handler都关联了一个线程，每个线程内部都维护了一个消息队列MessageQueue，这样Handler实际上也就关联了一个消息队列。可以通过Handler将Message和Runnable对象发送到该Handler所关联线程的MessageQueue（消息队列）中，然后该消息队列一直在循环拿出一个Message，对其进行处理，处理完之后拿出下一个Message，继续进行处理，周而复始。

#### 

#### 2. 为什么要有消息机制

Android的UI控件不是线程安全的，如果在多线程中访问UI控件则会导致不可预期的状态。那为什么不对UI控件访问加锁呢？

访问加锁缺点有两个：

1. 首先加锁会让UI控件的访问的逻辑变的复杂；
2. 其次，锁机制会降低UI的访问效率。

那我们不用线程来操作不就行了吗？但这是不可能的，因为Android的主线程不能执行耗时操作，否则会出现ANR。

所以，从各方面来说，Android消息机制是为了解决在子线程中无法访问UI的矛盾。

#### 

#### 3. Handler的工作原理

[![Android消息机制.png](https://camo.githubusercontent.com/d8b923da952762f0235f3a4f8e44b8fc474c4d6a/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d326434616363363430366332383033352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/d8b923da952762f0235f3a4f8e44b8fc474c4d6a/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d326434616363363430366332383033352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

如图所示，在主线程ActivityThread中的main方法入口中，先是创建了系统的Handler（H），创建主线程的Looper，将Looper与主线程绑定，调用了Looper的loop方法之后开启整个应用程序的主循环。Looper里面有一个消息队列，通过Handler发送消息到消息队列里面，然后通过Looper不断去循环取出消息，交给Handler去处理。通过系统的Handler，或者说Android的消息处理机制就确保了整个Android系统有条不紊地运作，这是Android系统里面的一个比较重要的机制。

我们的APP也可以创建自己的Handler，可以是在主线程里面创建，也可以在子线程里面创建，但是需要手动创建子线程的Looper并且手动启动消息循环。

#### 

#### 4. Handler的内存泄漏问题

##### 

##### 原因

非静态内部类持有外部类的匿名引用，导致Activity无法释放（生命周期不一致）

##### 

##### 解决方案

- Handler内部持有外部Activity的弱引用
- Handler改为静态内部类
- 在适当时机移除Handler的所有Callback()
- 原理：**静态类不持有外部类的对象，所以你的Activity可以随意被回收。由于Handler不再持有外部类对象的引用，导致程序不允许你在Handler中操作Activity中的对象了。所以你需要在Handler中增加一个对Activity的弱引用（WeakReference）。**

#### 

#### 5. 为什么在子线程中创建Handler会抛异常？

Handler的工作是依赖于Looper的，而Looper（与消息队列）又是属于某一个线程（ThreadLocal是线程内部的数据存储类，通过它可以在指定线程中存储数据，其他线程则无法获取到），其他线程不能访问。 因此Handler就是间接跟线程是绑定在一起了。因此要使用Handler必须要保证Handler所创建的线程中有Looper对象并且启动循环。因为子线程中默认是没有Looper的，所以会报错。

正确的在子线程中创建Handler的方法如下（可以使用HandlerThread代替）：

```java
    handler = null;
    new Thread(new Runnable() {

        private Looper mLooper;

        @Override
        public void run() {
            //必须调用Looper的prepare方法为当前线程创建一个Looper对象，然后启动循环
            //prepare方法中实质是给ThreadLocal对象创建了一个Looper对象
            //如果当前线程已经创建过Looper对象了，那么会报错
            Looper.prepare();
            handler = new Handler();
            //获取Looper对象
            mLooper = Looper.myLooper();
            //启动消息循环
            Looper.loop();

            //在适当的时候退出Looper的消息循环，防止内存泄漏
            mLooper.quit();
        }
    }).start();
```

注意：

- 主线程中默认是创建了Looper并且启动了消息的循环的，因此不会报错。
- 应用程序的入口是ActivityThread的main方法，在这个方法里面会创建Looper，并且执行Looper的loop方法来启动消息的循环，使得应用程序一直运行。
- 有时候出于业务需要，主线程可以向子线程发送消息。子线程的Handler必须按照上述方法创建，并且关联Looper。

#### 

#### 6. 为什么不能在子线程更新UI？

UI更新的时候，会对当前线程进行检验，如果不是主线程，则抛出异常：

```java
void checkThread() {
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

比较特殊的三种情况：

- 在Activity创建完成后（Activity的onResume之前ViewRootImpl实例没有建立），mThread被赋值为主线程（ViewRootImpl），所以直接在onCreate中创建子线程是可以更新UI的
- 在子线程中添加 Window，并且创建 ViewRootImpl，可以在子线程中更新view
- SurfaceView可以在其他线程更新

#### 

#### 7. 手写一个handler

**Handler**

```java
public class Handler {
    private Looper mLooper;
    private MessageQueue mQueue;

    public Handler() {
        //获取当前线程的looper
        mLooper = Looper.myLooper();
        //获取当前线程的消息列队
        mQueue = mLooper.messageQuene;
    }

    /**
     * 发送消息
     * @param message
     */
    public void sendMessage(Message message) {
        message.target = this;
        mQueue.enqueueMessage(message);
    }

    /**
     * 处理消息
     * @param message
     */
    public void handleMessage(Message message) {

    }

    /**
     * 分发消息
     * @param message
     */
    public void dispatchMessage(Message message) {
        handleMessage(message);
    }
}
```



**Looper**

```java
public class Looper {
    final MessageQueue messageQuene;
    private static ThreadLocal<Looper> threadLocal = new ThreadLocal<>();

    private Looper() {
        messageQuene = new MessageQueue();
    }

    /**
     * 为当前线程初始化一个looper副本对象
     */
    public static void prepare() {
        if (threadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        threadLocal.set(new Looper());
        System.out.println("looper初始化");
    }

    /**
     * 获取当前线程的looper副本对象
     *
     * @return
     */
    public static Looper myLooper() {
        return threadLocal.get();
    }

    /**
     * 轮询消息
     */
    public static void loop() {
        //获取当前线程的looper对象
        Looper me = myLooper();
        Message msg;
        //开始轮询消息
        for (; ; ) {
            //轮询消息，没有消息就阻塞
            msg = me.messageQuene.next();
            if (msg == null || msg.target == null) {
                System.out.println("Looper：" + "空消息");
                continue;
            }
            System.out.println("Looper：" + "looper轮询到了消息，发送消息");
            //轮询到了消息分发消息
            msg.target.dispatchMessage(msg);

        }
    }
}
```



**Message**

```java
public class Message {
    
    //发送的消息
    public Object obj;

    //目标Handler
    public Handler target;

    @Override
    public String toString() {
        return obj.toString();
    }
}
```





**MessageQueue**

```java
// 要实现生产者／消费者模型，首先的有锁，这里使用ReentrantLock主要考虑的重写入，它可以根据设定的变量来唤醒不同类型的锁，也就是说当我们队列有数据时，我们需要唤醒read锁；当队列有空间时，我们需要唤醒写锁。

public class MessageQueue {

    Message[] mItems;
    int mPutIndex;
    //队列中消息数
    private int mCount;
    private int mTakeIndex;
    //锁
    Lock mLock;
    //唤醒，沉睡某个线程操作
    Condition getCondition;//可取
    Condition addCondition;//可添加


    public MessageQueue() {
        mItems = new Message[50];
        mLock = new ReentrantLock();
        getCondition = mLock.newCondition();
        addCondition = mLock.newCondition();
    }

    /**
     * 消息队列取消息 出队
     *
     * @return
     */
    Message next() {
        Message msg = null;
        try {
            mLock.lock();
            //检查队列是否空了
            while (mCount <= 0) {
                //阻塞
                System.out.println("MessageQueue：" + "队列空了，读锁阻塞");
                getCondition.await();
            }
            msg = mItems[mTakeIndex];//可能空
            //消息被处理后，置空数组中该项
            mItems[mTakeIndex] = null;
            //处理越界，index大于数组容量时，取第一个item
            mTakeIndex = (++mTakeIndex >= mItems.length) ? 0 : mTakeIndex;
            mCount--;
            //通知生产者生产
            addCondition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            mLock.unlock();
        }

        return msg;
    }

    /**
     * 添加消息进队列
     *
     * @param message
     */

    public void enqueueMessage(Message message) {

        try {
            mLock.lock();
            //检查队列是否满了
            while (mCount >= mItems.length) {
                //阻塞
                System.out.println("MessageQueue：" + "队列空了，写锁阻塞");
                addCondition.await();
            }

            mItems[mPutIndex] = message;
            //处理越界，index大于数组容量时，替换第一个item
            mPutIndex = (++mPutIndex >= mItems.length) ? 0 : mPutIndex;
            mCount++;
            //通知消费者消费
            getCondition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            mLock.unlock();
        }
    }
}
```



