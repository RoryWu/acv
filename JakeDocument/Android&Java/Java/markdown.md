[TOC]

# java 高级

## JUC



### 线程排序

#### countdownlatch

作减法 , 例如,开始启动了 6 个线程, 

countdownlatch.wait();

一次减为零时 , wait 结束



#### CyclicBarrier

作加法 , 

```java
CylicBarrier cylicBarrier = new CylicBarrier(7 , 
	()-> {System.out.print("--------结束-----")})
    
cylicBarrier.wait()
```



#### Semaphore

```java
final Semaphore semaphore = new Semaphore(3); // 模拟3个车位
        for (int i = 0; i < 6; i++) { // 6个车来枪战
            new Thread(){
                try{
                    semaphore.acquire();
                    // do something
                    sleep(3);
                }finally{
                    semaphore.release();
                }
            }
        }
```





### 阻塞队列

1. 阻塞队列概念:

    * 当队列里面是空的时候, 从队列里取的操作,将被阻塞
    * 当队列满的时候, 向队列里放的操作,将被阻塞

    

2. BlockQueue: 7 个实现类,常用的3个

    * ArrayBlockQueue
    * 

生产者, 消费者模式:

 两个线程, 一个生产, 一个消费 , 三个重点: synchronized , wait , notify

## 锁

1. 多个线程, 操作资源类, 高内聚, 低耦合
2. 判断,  干活, 通知
3. 预防虚假唤醒 , 线程的等待, 需要使用while

铁三角:

sync -------------- wait --------------- notify

lock --------------- await ------------- singal





#### 锁的分类

Synchronized  和 Lock 的区别, LOCK的优点?

> 原始构成

* Synchronized 是JVM层面的, 是java 的关键字 / Lock 是api 层面的
* Synchronized 的底层是 monitorenter 和monitorexit (Monitor 这个类)
* Synchronized 是可重入锁, 一次monitorenter , 两次 monitorexit 保证在正常和异常的情况下都能退出
* Lock 是具体的类

> 使用方法

* Synchronized 不需要用户手动的去释放 , 执行完代码块中的代码,自动释放锁
* ReentrantLock 则需要用户手动的去释放, 如果不释放可能造成死锁的情况
* 需要Lock/unLock 配合try/finally 类占用和释放

> 等待是否可中断

* Synchronized 不可中断, 除非异常退出或正常执行完成
* ReentrantLock　可中断　(优点)
    * 设置超时　tryLock
    * lockinterruptibly  

> 加锁是否公平

* Synchronized 非公平锁

* Ｒeentrant　可以自己控制

> 锁绑定多个条件

* Synchronized 没有
* ReentrantLock 用来实现分组唤醒需要的线程 , 可以精确唤醒 , 而不是像Synchronized 要么随机一个,要么所有(优点)



#### 死锁

两个或者两个以上的进程执行过程中, 因为争夺资源而造成的一种互相等待的现象, 若无外力干预的情况下, 无法推进下去

```java
public class JavaTest1 {
    public static void main(String[] A) {

        String locKa = "AAA";
        String lockB = "bbb";

        new Thread(new ThreadTest(locKa, lockB), "ThreadA").start();
        new Thread(new ThreadTest(lockB, locKa), "ThreadB").start();
    }
}

class ThreadTest implements Runnable {

    String lockA ;
    String lockB ;
    
    ThreadTest(String lockA , String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
        
    }
    
    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println("持有Lock:"+lockA + " 想要拿到Lock:"+lockB);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB) {
                System.out.print("222222");
            }
        } 
    }
}
```



   排查:

	* 查看log
	* 使用 jps , jstack 命令



## 线程池

## JVM

### 类加载器 class loader

1. 自定义加载器四种加载器(跟启动, 拓展加载器, 应用加载器)

2. 概念

3. 双亲委派机制

4. java 类加载过程中的沙箱机制

5. JVM 内存模型

    

### JVM内存模型

#### java 8

![img](https://img-blog.csdn.net/20180730200648583?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

* **方法区**

    * 方法区主要保存的信息是类的元数据。是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型的信息，常量池，域信息，方法信息。 

    * **作用：**

        * 用于存储已被虚拟机加载的类信息
        * 常量
        * 静态变量
        * 运行时常量池时方法区中的一部分，用于存放编译期生成的各种字面量和符号引用，并不是只有编译期才能产生常量，运行期间也有可能将新的常量放入常量池，因此也会有可能抛OutOfMemoryError异常，常见的有字符串常量池

        

* **java栈/虚拟机栈**

    * 与程序计数器一样，Java 虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java 方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame ）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

    * 栈帧由三部分组成：局部变量区，操作数，帧数据区

        * **局部变量表**存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress 类型（指向了一条字节码指令的地址）。局部变量区被定义一个从0开始的数字数组，byte、Char、short，boolean转换成int，long double2个字节。其中64 位长度的long 和double 类型的数据会占用2 个局部变量空间（Slot），其余的数据类型只占用1个。局部变量通过数组的下标访问。

        * **操作数栈**也被定义为一个数字数组，，不同于局部变量区的通过下标访问，而是通过栈的push和pop操作。

        * **帧数据区**主要作用为 

            - 解析常量池的数据
            - 方法执行完后处理方法返回，恢复调用方现场
            - 方法执行过程中抛出异常时异常的处理，当出现异常时虚拟机查找相应的异常表看是否有对应的catch语句，如果没有就抛出异常终止这个方法调用。

            

    * 

* **Java堆**

    * 整体结构

        ![在这里插入图片描述](https://img-blog.csdn.net/20181005183314708?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lpbmd6aWlzbWU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    * 创建机制:

        1. JAVA对象优先在Eden区分配，当Eden区没有足够的空间时触发一次Minor GC ，触发Minor GC时，Eden和from区中的存活对象会被复制到to区，然后from和to交换指针，以保证下次Minor GC时，to区还是空的，如果survival区无法容纳的对象将通过分配担保机制直接进入老年区

        2. 分配担保机制可以通过HandlePromotionFailure配置，如果不允许的话，则直接发生FULL GC

        3. 新生代(Young Generation)的最大大小将根据总堆的最大大小和NewRatio参数的值来计算。参数的“不受限制”默认值MaxNewSize意味着计算值不受限制，MaxNewSize除非MaxNewSize在命令行中指定了值

        4. 一般情况下，不允许-XX:Newratio值小于1，即Old要比Young大

        5. 大对象直接进入老年区的判断是根据PretenureSizeThreshold设置的阈值，所谓大对象时指需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串以及数组（笔者列出的例子中的byte[]数组就是典型的大对象）

        6. 发生full GC的条件是：

            > （1）调用System.gc时，系统建议执行Full GC，但是不必然执行 
            >
            > （2）老年代空间不足 
            >
            > （3）方法区空间不足 
            >
            > （4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存 
            >
            > （5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

        7. 对象存活判断

            > 引用计数：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题 - 
            >
            > 可达性分析：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的，不可达对象

    

* **程序计数器**

    * 线程私有
    * 此内存区域是唯一一个在Java 虚拟机规范中没有规定任何OutOfMemoryError 情况的区域。

    

* **本地方法栈**

    * 线程私有

    * 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java 方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native 方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot 虚拟机）直接就把本地方法栈和虚拟机栈合二为一。

    * 与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError 和OutOfMemoryError异常。

        

* **直接内存** 元空间



### 其他知识点:

#### 常识1: java 栈的小知识

​	栈的大小通常: 512k - 1024K 能够存放大量的方法

#### 常识2: JVM对象的访问

```java
Object obj = new Object();
```

假设这句代码出现在方法体中，那“Object obj”这部分的语义将会反映到Java 栈的本地变量表中，作为一个reference 类型数据出现。而“new Object()”这部分的语义将会反映到Java 堆中，形成一块存储了Object 类型所有实例数据值（Instance Data，对象中各个实例字段的数据）的结构化内存，根据具体类型以及虚拟机实现的对象内存布局（Object Memory Layout）的不同，这块内存的长度是不固定的。另外，在Java 堆中还必须包含能查找到此对象类型数据（如对象类型、父类、实现的接口、方法等）的地址信息，这些类型数据则存储在方法区中。

#### 由于reference 类型在Java 虚拟机规范里面只规定了一个指向对象的引用，并没有定义这个引用应该通过哪种方式去定位，以及访问到Java 堆中的对象的具体位置，因此不同虚拟机实现的对象访问方式会有所不同，主流的访问方式有两种：使用**句柄和直接指针**。

* 句柄

    * 如果使用**句柄访问**方式，Java 堆中将会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据和类型数据各自的具体地址信息，如下图所示。 

        ![这里写图片描述](https://img-blog.csdn.net/20180802192524563?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

* 直接指针

    * 如果使用直接指针访问方式，Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，reference 中直接存储的就是对象地址，如下图所示 

        ![这里写图片描述](https://img-blog.csdn.net/20180802192603892?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

* 区别:

    这两种对象的访问方式各有优势，使用句柄访问方式的最大好处就是reference 中存储的是稳定的句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference 本身不需要被修改。

    使用直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销，由于对象的访问在Java 中非常频繁，因此这类开销积少成多后也是一项非常可观的执行成本。就本书讨论的主要虚拟机Sun HotSpot 而言，它是使用第二种方式进行对象访问的，但从整个软件开发的范围来看，各种语言和框架使用句柄来访问的情况也十分常见。





## GC

1. 作用域
2. 常见的垃圾回收算法
    1. 引用计数(较难处理循环引用的问题)
    2. 复制(年轻代区使用 , 复制之后有交换, 谁空谁是错)





## Exception/Error

### 常见的Exception

* Runtime
* nullpointerexception
* classnotfoundexception
* ClassCastException
* ArrayIndexOutOfBoundsException
* ConcurrentModification*Exception*

### 重点VMError

* InternalError
* UnknownError

* OOM(重点)

    * java heap space(堆内存不够用)

        * new 一个大的对象

    * GC overhead limit exceeded(大量资源在做GC , 但是没有效果)

    * Direct buffer memory(分配在OS本地内存, JVM不管理这部分内存,)

    * unable to create new native thread

    * Metaspace

        

* StackOverflow(重点)

    

## 其他知识点

### String

* intern 方法

    String类中加入这个方法可能是为了提升一点点性能，因为从常量池取数据比从堆里面去数据要快一些。(个人感觉)

    　　API上的那几句关于这个方法，其实总结一句就是调用这个方法之后把字符串对象加入常量池中，常量池我们都知道他是存在于方法区的，他是方法区的一部分，而方法区是线程共享的，所以常量池也就是线程共享的，但是他并不是线程不安全的，他其实是线程安全的，他仅仅是让有相同值的引用指向同一个位置而已，如果引用值变化了，但是常量池中没有新的值，那么就会新开辟一个常量结果来交给新的引用，而并非像线程不同步那样，针对同一个对象，new出来的字符串和直接赋值给变量的字符串存放的位置是不一样的，前者是在堆里面，而后者在常量池里面，另外，在做字符串拼接操作，也就是字符串相"+"的时候，得出的结果是存在在常量池或者堆里面，这个是根据情况不同不一定的，我写了几行代码测试了一下。

    　　先上结果：

    　　　　1.直接定义字符串变量的时候赋值，如果表达式右边只有字符串常量，那么就是把变量存放在常量池里面。

    　　　　2.new出来的字符串是存放在堆里面。

    　　　　3.对字符串进行拼接操作，也就是做"+"运算的时候，分2中情况：

    　　　　　　i.表达式右边是纯字符串常量，那么存放在栈里面。

    　　　　　　ii.表达式右边如果存在字符串引用，也就是字符串对象的句柄，那么就存放在堆里面。

    ```
    　　　　String str1 = "aaa";
            String str2 = "bbb";
            String str3 = "aaabbb";
            String str4 = str1 + str2;
            String str5 = "aaa" + "bbb";
            System.out.println(str3 == str4); // false
            System.out.println(str3 == str4.intern()); // true
            System.out.println(str3 == str5);// true
    ```

    　　结果：str1、str2、str3、str5都是存在于常量池，str4由于表达式右半边有引用类型，所以str4存在于堆内存，而str5表达式右边没有引用类型，是纯字符串常量，就存放在了常量池里面。其实Integer这种包装类型的-128  ~ +127也是存放在常量池里面，比如Integer i1 = 10;Integer i2 = 10; i1 ==  i2结果是true，估计也是为了性能优化。

* 其他方法











