# Android 知识架构总结 by Jake

## 1. 数据结构

### 1.1 数组/队列

### 1.2 单链表

### 1.3 双链表

### 1.4 Hash 表

1.5 树

1.6 图



## 2. 算法

### 2.1 搜索

### 2.2 排序

> **各种排序：冒泡、选择、插入、希尔、归并、快排、堆排、桶排、基数的原理、平均时间复杂度、最坏时间复杂度、空间复杂度、是否稳定。**





## 3. Java基础知识与JVM的理解

1. 线程

2. 内部类

3. 泛型

    



## 4. 设计模式

### 4.1 关于复杂度

#### 时间复杂度

> 如何计算时间复杂度 

> 常见复杂度的计算时间级别

#### 空间复杂度

>  如何计算控件复杂度

### 4.2 单例模式

懒汉式

饿汉式

完整式

内部类的形式

### 4.3  Builder 模式

### 4.4  Adapter 模式

### 4.5  工厂模式

### 4.6  观察者模式





## 5. Android 知识体系与基础知识

### 5.1 掌握Android自带的组件与类

#### 5.1.1 四大组件之Activity

* **理解Activity**

    activity是独立平等的，用来处理用户操作。几乎所有的activity都是用来和用户交互的，所以activity类会创建了一个窗口，开发者可以通过setContentView(View)的接口把UI放到给窗口上。



* **重要方法**

    1. onConfigurationChanged

        前面说过，当 Activity 横竖屏切换的时候会导致 Activity 销毁并重建，哪有什么方法能避免呢？其实可以在  AndroidManifest 里面指定 android:configChanges="orientation/screenSize"  来避免重建，这时就会调用 onConfigurationChanged 方法。

        如果按上面的配置，当字体发生变化时，也会销毁重建，但是不会回调 onConfigurationChanged 方法，所以说想要监听的变化必须要包含之内。

        ```java
        @Override
            public void onConfigurationChanged(Configuration newConfig) {
                super.onConfigurationChanged(newConfig);
                if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT) {
                    //竖屏
                } else {
                    //横屏
                }
            }
        ```

        

    2. onTrimMemory

        当内存紧张时会回调，它在 onStop 回调之前。指导应用程序在不同的情况下进行自身的内存释放，以避免被系统直接杀掉，提高应用程序的用户体验。它和 onLowMemory 相比，它有一个 level 评级，onLowMemory 能兼容更低的版本。

        ```java
        @Override
        public void onTrimMemory(int level) {
        	super.onTrimMemory(level);
            if (level == TRIM_MEMORY_UI_HIDDEN) {
        
            }
        }
        ```

        

    

* **Activity 的生命周期**

    正常状态:

     onCreate()—>onStart()—>onResume()-> onPause()—>onStop()

    后台返回前台:

     onRestart()—>**onStart()**—>onResume()

    锁屏: 

    onPause()->onStop()

    解锁:

    onStart()->onResume()



* **Activity 的启动模式 任务栈**

    使用`android:launchMode="standard|singleInstance|singleTask|singleTop"`来控制Acivity任务栈。

    **任务栈**是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其`onDestory()`方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名.

    1. standard : 标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(*onCreate()->onStart()->onResume()*)都会执行。

    2. singleTop : 栈顶复用模式.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的`onNewIntent()`方法会被回调.如果Activity已经存在但是不在栈顶,那么作用与*standard模式*一样.

    3. singleTask: 栈内复用模式.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么这个Activity就会被调到栈顶,`onNewIntent()`,并且singleTask会清理在当前Activity上面的所有Activity.(clear top)

    4. singleInstance : 加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了

        

    Activity的堆栈管理以ActivityRecord为单位,所有的ActivityRecord都放在一个List里面.可以认为一个ActivityRecord就是一个Activity栈

    

* **Activity 的数据保存 和 恢复**
    * onSaveInstanceState

        在activity　可能被回收之前调用,用来保存自己的状态和信息，以便回收后重建时恢复数据（在onCreate()或onRestoreInstanceState()中恢复）。旋转屏幕重建activity会调用该方法，但其他情况在onpause()和onStop()状态的activity不一定会调用, 官方说明:

        > One example of when onPause and onStop is called and not this method is 
        > when a user navigates back from activity B to activity A: there is no 
        > need to call onSaveInstanceState on B because that particular instance 
        > will never be restored, so the system avoids calling it. An example when
        > onPause is called and not onSaveInstanceState is when activity B is 
        > launched in front of activity A: the system may avoid calling 
        > onSaveInstanceState on activity A if it isn't killed during the lifetime
        > of B since the state of the user interface of A will stay intact.

        也就是说，系统灵活的来决定调不调用该方法，**但是如果要调用就一定发生在onStop方法之前，但并不保证发生在onPause的前面还是后面。**

    * onRestoreInstanceState

        这个方法在onStart 和 onPostCreate之间调用，在onCreate中也可以状态恢复，但有时候需要所有布局初始化完成后再恢复状态。

        onPostCreate：一般不实现这个方法，当程序的代码开始运行时，它调用系统做最后的初始化工作。

    * 使用

    	```java
    public class MainActivity extends Activity {
    
    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
    		if(savedInstanceState!=null){ //判断是否有以前的保存状态信息
    			 savedInstanceState.get("Key"); 
    			 }
    		super.onCreate(savedInstanceState);
    		setContentView(R.layout.activity_main);
    	}
       @Override
    protected void onSaveInstanceState(Bundle outState) {
    	// TODO Auto-generated method stub
    	 //可能被回收内存前保存状态和信息，
    	   Bundle data = new Bundle(); 
    	   data.putString("key", "last words before be kill");
    	   outState.putAll(data);
    	super.onSaveInstanceState(outState);
    }
       @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
    	// TODO Auto-generated method stub
    	   if(savedInstanceState!=null){ //判断是否有以前的保存状态信息
    			 savedInstanceState.get("Key"); 
    			 }
    	super.onRestoreInstanceState(savedInstanceState);
    }
    }
    ```



> Activity 在manifest 标签中的一些注意点

* taskAffinity

    在 singleTask 启动模式中，多次提到某个 Activity 所需的任务栈，什么是 Activity  所需要的任务栈呢？这就要从一个参数说起：taskAffinity，任务相关性。这个参数标识了一个 Activity  所需要的任务栈的名字，默认情况下，所有 Activity 所需的任务栈的名字为应用的包名。当然，我们可以为每个 Activity 都单独指定  taskAffinity 属性，这个属性值必须不能和包名相同，否则相当于没有设置。taskAffinity 属性主要和 singleTask  启动模式和 allowTaskReparentiong 属性配对使用，在其他情况下没有意义。

    **taskAffinity 与 singleTask 配对使用：**

    如果启动了设置了这两个属性的 Activity，这个 Activity 就会在 taskAffinity 设置的任务栈中。

    **taskAffinity 与 allowTaskReparenting 配对使用：**

    当一个应用 A 启动了应用 B 的某个 Activity 后，如果这个 Activity 的 allowTaskReparenting  属性为 true 的话，那么当应用 B 被启动后，此 Activity 会直接从应用 A 的任务栈转移到应用 B  的任务栈中。这个属性主要作用就是将这个 Activity  转移到它所属的任务栈中，例如一个短信应用收到一个带有网络链接的短信，点击链接会跳到浏览器，这时候如果 allowTaskReparenting  设置为 true 的话，打开浏览器应用就会直接显示刚才打开的网页页面，而打开短信应用后这个浏览器界面就会消失。

    启动模式了解之后，那是如何指定启动模式的方式呢？

    有两种：一种是在 AndroidMenifet 文件设置 launchMode 属性，一种是给 Intent 设置 Flag。

    如果两者都存在，后者优先级更高。



#### 5.1.2 四大组件之Service

> 生命周期

> onStartCommand 不同返回值 的作用是什么?

> Service 的种类

#### 5.1.3 四大组件之Broadcast

#### 5.1.4 四大组件之ContentProvider

#### 5.1.5 常用组件之Fragment

> 为什么会有Fragment ?

> 生命周期？

> 和Activity 的关系

#### 5.1.6 常用组件之SharedPreference

#### 5.1.7 常用组件之Intent

**Intent知识点**

**IntentFilter知识点**

IntentFilter 的三个属性:

- ​	Action
- ​	URL
- ​	Category

IntentFilter 的匹配规则

- 加载所有的Intent Filter列表 　　
- 去掉action匹配失败的Intent Filter 　　
- 去掉url匹配失败的Intent Filter 　　
- 去掉Category匹配失败的Intent Filter 　　
- 判断剩下的Intent Filter数目是否为0。如果为0查找失败返回异常；如果大于0，就按优先级排序，返回最高优先级的Intent Filter	



> 如何在短信中启动一个Activity



#### 5.1.8 常用组件之Drawable

#### 5.1.9 常用组件之Handler

#### 5.1.10 常用组件之AndroidManifests.mxl

#### 5.1.11 常用组件之Animation

#### 5.1.12 常用组件之布局文件

> 　常用ViewGroup 布局

* LinearLayout

* RelativeLayout

* FrameLayout

* ConstaintLayout

    https://www.jianshu.com/p/a74557359882



#### 5.1.13 常用组件之Bitmap





### 5.2 理解Android深层的类与原理

#### 5.2.1 Binder的理解

​	在Android 的底层研究中, 我们会经常用到binder 的方式进行进程间通信, 那binder到底是什么? binder 的实现方式是什么样的呢?



> **Binder 是什么?**

​	首先Binder是Android系统进程间通信(IPC)方式之一。

​	Binder使用Client－Server通信方式。Binder框架定义了四个角色：Server,Client,ServiceManager以及Binder驱动。其中Server,Client,ServiceManager运行于用户空间，驱动运行于内核空间。Binder驱动程序提供设备文件/dev/binder与用户空间交互，Client、Server和Service Manager通过open和ioctl文件操作函数与Binder驱动程序进行通信。

​	Server创建了Binder实体，为其取一个字符形式，可读易记的名字，将这个Binder连同名字以数据包的形式通过Binder驱动发送给ServiceManager，通知ServiceManager注册一个名字为XX的Binder，它位于Server中。驱动为这个穿过进程边界的Binder创建位于内核中的实体结点以及ServiceManager对实体的引用，将名字以及新建的引用打包给ServiceManager。ServiceManager收数据包后，从中取出名字和引用填入一张查找表中。但是一个Server若向ServiceManager注册自己Binder就必须通过0这个引用和ServiceManager的Binder通信。Server向ServiceManager注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的引用了。Clent也利用保留的0号引用向ServiceManager请求访问某个Binder：我申请名字叫XX的Binder的引用。ServiceManager收到这个连接请求，从请求数据包里获得Binder的名字，在查找表里找到该名字对应的条目，从条目中取出Binder引用，将该引用作为回复发送给发起请求的Client。

​	当然，不是所有的Binder都需要注册给ServiceManager广而告之的。Server端可以通过已经建立的Binder连接将创建的Binder实体传给Client，当然这条已经建立的Binder连接必须是通过实名Binder实现。由于这个Binder没有向ServiceManager注册名字，所以是匿名Binder。Client将会收到这个匿名Binder的引用，通过这个引用向位于Server中的实体发送请求。匿名Binder为通信双方建立一条私密通道，只要Server没有把匿名Binder发给别的进程，别的进程就无法通过穷举或猜测等任何方式获得该Binder的引用，向该Binder发送请求。



> **Binder 的运行机制是什么样的呢?**

​	Linux内核实际上没有从一个用户空间到另一个用户空间直接拷贝的函数，需要先用copy_from_user()拷贝到内核空间，再用copy_to_user()拷贝到另一个用户空间。为了实现用户空间到用户空间的拷贝，mmap()分配的内存除了映射进了接收方进程里，还映射进了内核空间。所以调用copy_from_user()将数据拷贝进内核空间也相当于拷贝进了接收方的用户空间，这就是Binder只需一次拷贝的‘秘密’。

​	最底层的是Android的ashmen(Anonymous shared memory)机制，它负责辅助实现内存的分配，以及跨进程所需要的内存共享。AIDL(android interface definition language)对Binder的使用进行了封装，可以让开发者方便的进行方法的远程调用，后面会详细介绍。Intent是最高一层的抽象，方便开发者进行常用的跨进程调用。

​	从英文字面上意思看，Binder具有粘结剂的意思那么它是把什么东西粘接在一起呢？在Android系统的Binder机制中，由一系统组件组成，分别是Client、Server、Service Manager和Binder驱动，其中Client、Server、Service Manager运行在用户空间，Binder驱动程序运行内核空间。Binder就是一种把这四个组件粘合在一起的粘连剂了，其中，核心组件便是Binder驱动程序了，ServiceManager提供了辅助管理的功能，Client和Server正是Binder驱动和ServiceManager提供的基础设施上，进行Client-Server之间的通信。

1. Client、Server和ServiceManager实现在用户空间中，Binder驱动实现在内核空间中
2. Binder驱动程序和ServiceManager在Android平台中已经实现，开发者只需要在用户空间实现自己的Client和Server
3. Binder驱动程序提供设备文件/dev/binder与用户空间交互，Client、Server和ServiceManager通过open和ioctl文件操作函数与Binder驱动程序进行通信
4. Client和Server之间的进程间通信通过Binder驱动程序间接实现
5. ServiceManager是一个守护进程，用来管理Server，并向Client提供查询Server接口的能力



​	服务器端：一个Binder服务器就是一个Binder类的对象。当创建一个Binder对象后，内部就会开启一个线程，这个线程用户接收binder驱动发送的消息，收到消息后，会执行相关的服务代码。

​	Binder驱动：当服务端成功创建一个Binder对象后，Binder驱动也会相应创建一个mRemote对象，该对象的类型也是Binder类，客户就可以借助这个mRemote对象来访问远程服务。

​	客户端：客户端要想访问Binder的远程服务，就必须获取远程服务的Binder对象在binder驱动层对应的binder驱动层对应的mRemote引用。当获取到mRemote对象的引用后，就可以调用相应Binde对象的服务了。

​	在这里我们可以看到，客户是通过Binder驱动来调用服务端的相关服务。首先，在服务端创建一个Binder对象，接着客户端通过获取Binder驱动中Binder对象的引用来调用服务端的服务。在Binder机制中正是借着Binder驱动将不同进程间的组件bind(粘连)在一起，实现通信。

​	mmap将一个文件或者其他对象映射进内存。文件被映射进内存。文件被映射到多个页上，如果文件的大小不是所有页的大小之和，最后一个页不被使用的空间将会凋零。munmap执行相反的操作，删除特定地址区域的对象映射。

​	当使用mmap映射文件到进程后，就可以直接操作这段虚拟地址进行文件的读写等操作，不必再调用read,write等系统调用。但需注意，直接对该段内存写时不会写入超过当前文件大小的内容。

​	使用共享内存通信的一个显而易见的好处是效率高，因为进程可以直接读写内存，而不需要任何数据的拷贝。对于像管道和消息队列等通信方式，则需要在内核和用户空间进行四次的数据拷贝，而共享内存则只拷贝两次内存数据：一次从输入文件到共享内存区，另一次从共享内存到输出文件。实际上，进程之间在共享内存时，并不总是读写少量数据后就解除映射，有新的通信时，再重新建立共享内存区域，而是保持共享区域，直到通信完成为止，这样，数据内容一直保存在共享内存中，并没有写回文件。共享内存中的内容往往是在解除内存映射时才写回文件的。因此，采用共享内存的通信方式效率是非常高的。

​	aidl主要就帮助了我们完成了包装数据和解包的过程，并调用了transact过程，而用来传递的数据包我们就称为parcel

​	AIDL:xxx.aidl -> xxx.java ,注册service

1. 用aidl定义需要被调用方法接口
2. 实现这些方法
3. 调用这些方法



#### 5.2.2 Context

#### 5.2.3 IPC



### 5.3 Android 的View

#### 5.3.1 Android 的自带控件

#### 5.3.2 Android 的自定义View 相关

#### 5.3.3 关于View 的可见性

https://www.jianshu.com/p/54a2af8f8e2b

#### 5.3.4 关于SurfaceView

使用方式

```java
public class SurfaceViewTest extends Activity
{
	// SurfaceHolder负责维护SurfaceView上绘制的内容
	private SurfaceHolder holder;
	private Paint paint;

	@Override
	public void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		paint = new Paint();
		SurfaceView surface = (SurfaceView) findViewById(R.id.show);
		// 初始化SurfaceHolder对象
		holder = surface.getHolder();
		holder.addCallback(new Callback()
		{
			@Override
			public void surfaceChanged(SurfaceHolder arg0, int arg1, int arg2,
					int arg3)
			{
			}

			@Override
			public void surfaceCreated(SurfaceHolder holder)
			{
				// 锁定整个SurfaceView
				Canvas canvas = holder.lockCanvas();
				// 绘制背景
				Bitmap back = BitmapFactory.decodeResource(
					SurfaceViewTest.this.getResources()
					, R.drawable.sun);
				// 绘制背景
				canvas.drawBitmap(back, 0, 0, null);
				// 绘制完成，释放画布，提交修改
				holder.unlockCanvasAndPost(canvas);
				// 重新锁一次，"持久化"上次所绘制的内容
				holder.lockCanvas(new Rect(0, 0, 0, 0));
				holder.unlockCanvasAndPost(canvas);
			}

			@Override
			public void surfaceDestroyed(SurfaceHolder holder)
			{
			}
		});
		// 为surface的触摸事件绑定监听器
		surface.setOnTouchListener(new OnTouchListener()
		{
			@Override
			public boolean onTouch(View source, MotionEvent event)
			{
				// 只处理按下事件
				if (event.getAction() == MotionEvent.ACTION_DOWN)
				{
					int cx = (int) event.getX();
					int cy = (int) event.getY();
					// 锁定SurfaceView的局部区域，只更新局部内容
					Canvas canvas = holder.lockCanvas(new Rect(cx - 50,
							cy - 50, cx + 50, cy + 50));
					// 保存canvas的当前状态
					canvas.save();
					// 旋转画布
					canvas.rotate(30, cx, cy);
					paint.setColor(Color.RED);
					// 绘制红色方块
					canvas.drawRect(cx - 40, cy - 40, cx, cy, paint);
					// 恢复Canvas之前的保存状态
					canvas.restore();
					paint.setColor(Color.GREEN);
					// 绘制绿色方块
					canvas.drawRect(cx, cy, cx + 40, cy + 40, paint);
					// 绘制完成，释放画布，提交修改
					holder.unlockCanvasAndPost(canvas);
				}
				return false;
			}
		});
	}
	}
	
/**
上面的程序为SurfaceHolder添加了一个CallBack实例，该Callback中定义了如下三个方法：

void surfaceChanged(SurfaceHolder holder, int format, int width, int height):当一个surface的格式或大小发生改变时回调该方法。
void surfaceCreated(SurfaceHolder holder):当surface被创建时回调该方法
void surfaceDestroyed(SurfaceHolder holder):当surface将要被销毁时回调该方法
**/


```



> **SurfaceHolder提供了如下方法来获取Canvas对象**

1. Canvas lockCanvas():锁定整个SurfaceView对象，获取该Surface上的Canvas

2. Canvas lockCanvas(Rect dirty):锁定SurfaceView上Rect划分的区域，获取该Surface上的Canvas
3. unlockCanvasAndPost(canvas):释放绘图、提交所绘制的图形，需要注意，当调用SurfaceHolder上的unlockCanvasAndPost方法之后，该方法之前所绘制的图形还处于缓冲之中，下一次lockCanvas()方法锁定的区域可能会“遮挡”它



>  **为什么要使用SurfaceView 来做过渡动画?**

因为View的绘图存在以下缺陷：

1. View缺乏双缓冲机制
2. 当程序需要更新View上的图像时，程序必须重绘View上显示的整张图片
3. 新线程无法直接更新View组件



#### 5.3.5 自定义View 的其他知识点

ViewDragHelper

https://blog.csdn.net/yanbober/article/details/50419059

坐标系统

https://blog.csdn.net/yanbober/article/details/50419117

Scroller

https://blog.csdn.net/yanbober/article/details/49904715







### 5.4 Android 中的多线程

Android 中的多线程



### 5.5 Android 中的存储方式

Android 中原生的存储方式主要有以下几种常用方式

- SQLite：SQLite是一个轻量级的数据库，支持基本的SQL语法，是常被采用的一种数据存储方式。 Android为此数据库提供了一个名为SQLiteDatabase的类，封装了一些操作数据库的api
- SharedPreference： 除SQLite数据库外，另一种常用的数据存储方式，其本质就是一个xml文件，常用于存储较简单的参数设置。
- File： 即常说的文件（I/O）存储方法，常用语存储大数量的数据，但是缺点是更新数据将是一件困难的事情。
- ContentProvider: Android系统中能实现所有应用程序共享的一种数据存储方式，由于数据通常在各应用间的是互相私密的，所以此存储方式较少使用，但是其又是必不可少的一种存储方式。例如音频，视频，图片和通讯录，一般都可以采用此种方式进行存储。每个Content Provider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享时，就需要使用Content Provider为这些数据定义一个URI，然后其他的应用程序就通过Content Provider传入这个URI来对数据进行操作。



### 5.6 Android 中的相机与相册

Android 中原生的存储方式主要有以下几种常用方式



### 5.10 Android 的细节注意点

1. 全局异常处理

2. 序列化

    





## 6.Android进阶知识点



### Android 开发的优化

* **应用的内存优化**

* **应用的启动时间优化**

* **应用安装包Size 的裁剪**

* **应用的多机型适配**

* **电量的优化**

* **Android 中的内存泄露**

* **Android 进程的保活**

    理解Android 的保活,首先要理解Android 中的进程:

    

    > **Android 中有几种进程?**

    1. 前台进程：即与用户正在交互的Activity或者Activity用到的Service等，如果系统内存不足时前台进程是最后被杀死的
    2. 可见进程：可以是处于暂停状态(onPause)的Activity或者绑定在其上的Service，即被用户可见，但由于失去了焦点而不能与用户交互
    3. 服务进程：其中运行着使用startService方法启动的Service，虽然不被用户可见，但是却是用户关心的，例如用户正在非音乐界面听的音乐或者正在非下载页面自己下载的文件等；当系统要空间运行前两者进程时才会被终止
    4. 后台进程：其中运行着执行onStop方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这样的进程系统一旦没了有内存就首先被杀死
    5. 空进程：不包含任何应用程序的程序组件的进程，这样的进程系统是一般不会让他存在的

    

    > **如何避免应用在后台被杀死 Service的保活 和 进程保活一致**

    1. Service设置成START_STICKY

    	- kill 后会被重启（等待5秒左右），重传Intent，保持与重启前一样

    2. 提升service优先级

    	- 在AndroidManifest.xml文件中对于intent-filter可以通过`android:priority = "1000"`这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，**同时适用于广播**。
    	- 【结论】目前看来，priority这个属性貌似只适用于broadcast，对于Service来说可能无效

    3. 提升service进程优先级

    	- Android中的进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收
    	- 当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以在startForeground()使用startForeground()将service放到前台状态。这样在低内存时被kill的几率会低一些。
    	- 【结论】如果在极度极度低内存的压力下，该service还是会被kill掉，并且不一定会restart()

    4. onDestroy方法里重启service

    	- service +broadcast 方式，就是当service走onDestory()的时候，发送一个自定义的广播，当收到广播的时候，重新启动service
    	- 也可以直接在onDestroy()里startService
    	- 【结论】当使用类似口口管家等第三方应用或是在setting里-应用-强制停止时，APP进程可能就直接被干掉了，onDestroy方法都进不来，所以还是无法保证

    5. 监听系统广播判断Service状态

    	- 通过系统的一些广播，比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限
    	- 【结论】这也能算是一种措施，不过感觉监听多了会导致Service很混乱，带来诸多不便

    6. 在JNI层,用C代码fork一个进程出来

    	- 这样产生的进程,会被系统认为是两个不同的进程.但是Android5.0之后可能不行

    7. root之后放到system/app变成系统级应用

    **大招: 放一个像素在前台(手机QQ)**

    

* **Android 中的图片加载优化**

    图片加载优化,我们会常听到三级缓存这个方式, 那什么是三级缓存呢?

    > 三级缓存

    - 网络加载，不优先加载，速度慢，浪费流量
    - 本地缓存，次优先加载，速度快
    - 内存缓存，优先加载，速度最快

    

    > 三级缓存的原理

    - 首次加载 Android App 时，肯定要通过网络交互来获取图片，之后我们可以将图片保存至本地SD卡和内存中
    - 之后运行 App 时，优先访问内存中的图片缓存，若内存中没有，则加载本地SD卡中的图片
    - 总之，只在初次访问新内容时，才通过网络获取图片资源

    

    > 网络图片的加载 以及LRU



### Android 的开发模式理解

* MVC

* MVP

* MVVM

    



### Android 的开源框架

1. 注解ICO框架
2. 网络OKHttp 框架
3. 图片加载框架
    * Fresco
    * Glide
4. 数据库框架
    * Realm
    * GreenDao
5. 消息机制框架 EventBus
6. 布局框架
    * vlayout
7.  内存监控
    * LeakCanery



### Android 的应用发布流程



### Android 运行原理

#### *Android 的开机过程

#### * Android 的启动流程

#### * Android 的打包流程

#### * APK 的安装流程



### Android 前沿知识点

* 热修复
* Kotlin
* Rxjava+Retrofit
* git
* gradle
* databinding



### Android 推送方案

https://www.cnblogs.com/Joanna-Yan/p/6241354.html



## 7. 网络协议相关





## 8. 其他相关能力

### 8.1 项目总结

### 8.2 面试技巧

### 8.3 编程技巧

关于位运算

关于正则表达式

