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

> 理解Activity

> Activity 的生命周期

> Activity 的启动模式



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

①加载所有的Intent Filter列表 　　

②去掉action匹配失败的Intent Filter 　　

③去掉url匹配失败的Intent Filter 　　

④去掉Category匹配失败的Intent Filter 　　

⑤判断剩下的Intent Filter数目是否为0。如果为0查找失败返回异常；如果大于0，就按优先级排序，返回最高优先级的Intent Filter	



> 如何在短信中启动一个Activity



#### 5.1.8 常用组件之Drawable

#### 5.1.9 常用组件之Handler

#### 5.1.10 常用组件之AndroidManifests.mxl

#### 5.1.11 常用组件之Animation

#### 5.1.12 常用组件之布局文件

> 　常用ViewGroup 布局

LinearLayout

RelativeLayout

FrameLayout

ConstaintLayout



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



### 5.4 Android 中的多线程

Android 中的多线程



### 5.5 Android 中的存储方式

Android 中原生的存储方式主要有以下几种常用方式



### 5.6 Android 中的相机与相册

Android 中原生的存储方式主要有以下几种常用方式



### 5.10 Android 的细节注意点

1. 全局异常处理

2. 序列化

    





## 6.Android进阶知识点



### Android 应用的优化

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
    4. 后台进程：其中运行着执行onSto误p方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这样的进程系统一旦没了有内存就首先被杀死
    5. 空进程：不包含任何应用程序的程序组件的进程，这样的进程系统是一般不会让他存在的

    

    > **如何避免应用在后台被杀死?**

    1. 调用startForegound，让你的Service所在的线程成为前台进程
    2. Service的onStartCommond返回START_STICKY或START_REDELIVER_INTENT
    3. Service的onDestroy里面重新启动自己

    

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

