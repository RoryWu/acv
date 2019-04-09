

# Android 知识架构总结 by Jake

## 1. 数据结构与算法

[常用数据结构与算法](./数据结构与算法.md)

---

## 2. JVM的理解

---

## 3. Java基础知识

1. 线程

2. 内部类

3. 泛型

    

---

## 4. 设计模式

### 4.1 关于复杂度

### 4.2 单例模式

懒汉式

饿汉式

完整式

内部类的形式

### 4.3  Builder 模式

### 4.4  Adapter 模式

### 4.5  工厂模式

### 4.6  观察者模式

### 4.7  装饰者模式



---

## 5. Android 知识体系与基础知识

### 5.1 掌握Android自带的组件与类

#### 5.1.1 四大组件之Activity

* Activity 详解

    [Activity 详解](./Android_Activity.md)

    

#### 5.1.2 四大组件之Service

* Service 知识点

    [Service 详解](./Android_Service.md)



#### 5.1.3 四大组件之Broadcast

* BroadCast 详解

  [Android 广播的详解](./Android_Broadcast.md)

  

#### 5.1.4 四大组件之ContentProvider

* ContentProvider 详解

    [Android ContentProvider 详解](./Android_ContentProvider.md)



#### 5.1.5 常用组件之Fragment

* Fragment 详解 

    [Fragment 的详解](./Android_Fragment.md)

    

#### 5.1.6 常用组件之SharedPreference

* SharedPreference 的功能

    [Android SharedPreference 详解](./Android_SharedPreference.md)

    

#### 5.1.7 常用组件之Intent

* **Intent 与 IntentFilter**

    [Intent 详解](./Android_Intent.md)

* 常见问题

    >  如何在短信中启动一个Activity



#### 5.1.8 常用组件之Drawable

* Drawable 的相关知识点

    https://blog.csdn.net/lmj623565791/article/details/43752383

    

#### 5.1.9 常用组件之Handler

- Handler 作用与原理

    [Android Handler详解](./Android_Handler.md)

- 关于getMainLooper

    [getMainLooper](./Android_Handler_Looper.md)

    

#### 5.1.10 常用组件之AndroidManifests.mxl 与权限

- AndroidManifest 的重要标签和属性

    [Manifest详解](./Android_Manifest.md)

- Android 中的 权限问题

    [Android中的权限详解](./Android权限.md)

    

#### 5.1.11 常用组件之Animation

- Android中的动画

    [Android_Animation详解](./Android_Animation.md)

    

#### 5.1.12 常用组件之布局文件

* 常用ViewGroup 布局
    * LinearLayout

    * RelativeLayout

    * FrameLayout

    * ConstaintLayout

        https://www.jianshu.com/p/a74557359882

        https://mp.weixin.qq.com/s/gGR2itbY7hh9fo61SxaMQQ



#### 5.1.13 常用组件之Bitmap

* Bitmap 知识点

    [Android Bitmap 详解](./Android_Bitmap.md)





### 5.2 理解Android深层的类与原理

#### 5.2.1 Binder的理解

* Binder 的知识点

    [Android Binder](./Android_Binder.md)

​	

#### 5.2.2 Context

* Android Context 的理解

    [Android 中的Context](./Android_Context.md)

    

#### 5.2.3 IPC

* IPC 知识点





### 5.3 Android 的View

#### 5.3.1 Android 的自带控件

- **WebView**

    [WebView详解](./Android_WebView.md)

- **RecyclerView**

    [RecyclerView详解](./Android_RecyclerView.md)

- **Viewpager**

    [ViewPager详解](./Android_Viewpager.md)

    

#### 5.3.2 Android 的自定义View 相关

* **关于自定义View**

    [Android自定义View 的相关知识整理](./acv.md)

    

#### 5.3.3 关于SurfaceView

* **SurfaceView 的知识点**

    [Android SurfaceView 的详解](./Android_SurfaceView.md)



#### 5.3.4 View 的重要知识点

* ViewDragHelper

    https://blog.csdn.net/yanbober/article/details/50419059

* 坐标系统

    https://blog.csdn.net/yanbober/article/details/50419117

* Scroller

    https://blog.csdn.net/yanbober/article/details/49904715

* 关于View 的可见性

    https://www.jianshu.com/p/54a2af8f8e2b



#### 5.3.5 考点

>  **为什么要使用SurfaceView 来做过渡动画?**

​	因为View的绘图存在以下缺陷：

	1. View缺乏双缓冲机制
	2. 当程序需要更新View上的图像时，程序必须重绘View上显示的整张图片
	3. 新线程无法直接更新View组件



> **invalidate()和postInvalidate() 的区别及使用**

<http://blog.csdn.net/mars2639/article/details/6650876>











### 5.4 Android 中的多线程

* Android 中常见的多线程

    * Thread
    * HandlerThread

    * IntentService

    * AsyncTask

    * 线程池

    * Rxjava

        



### 5.5 Android 中的存储方式

Android 中原生的存储方式主要有以下几种常用方式

- SQLite：SQLite是一个轻量级的数据库，支持基本的SQL语法，是常被采用的一种数据存储方式。 Android为此数据库提供了一个名为SQLiteDatabase的类，封装了一些操作数据库的api
- SharedPreference： 除SQLite数据库外，另一种常用的数据存储方式，其本质就是一个xml文件，常用于存储较简单的参数设置。
- File： 即常说的文件（I/O）存储方法，常用语存储大数量的数据，但是缺点是更新数据将是一件困难的事情。
- ContentProvider: Android系统中能实现所有应用程序共享的一种数据存储方式，由于数据通常在各应用间的是互相私密的，所以此存储方式较少使用，但是其又是必不可少的一种存储方式。例如音频，视频，图片和通讯录，一般都可以采用此种方式进行存储。每个Content Provider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享时，就需要使用Content Provider为这些数据定义一个URI，然后其他的应用程序就通过Content Provider传入这个URI来对数据进行操作。



### 5.6 Android 中的相机与相册

* 调用Android中的照片

    [Android 相机相册调用](./Android_Camera & Gallery.md)



### 5.7 Android 中的序列化

* Android 中的序列化

    [Android 中的序列化](./Android序列化.md)

### 5.10 Android 的细节注意点

1. 全局异常处理

2. ANR

3. Lint

4. AOP

     

===

---

## 6.Android进阶知识点

### 6.1 Adroid 开发的优化

* **应用的内存优化**

* **应用的启动时间优化**

    * 冷启动的优化

        [启动问题](https://github.com/huannan/AndroidReview/blob/master/Android.md)

* **应用安装包Size 的裁剪**

* **APK 的瘦身**

* **应用的多机型适配**

* **电量的优化**

* **Android 中的内存泄露**

* **Android 进程的保活**

    [进程保活的方法](./进程保活.md)

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

    ​	



---

### 6.2 Android 的开发模式理解

* MVC

* MVP

* MVVM

    

---

### 6.3 Android 的开源框架

- 注解ICO框架

- 网络OKHttp 框架

- 图片加载框架
    * Fresco
    * Glide

- 数据库框架
    * Realm

    * GreenDao

- 消息机制框架 EventBus

- 布局框架

    * vlayout

- 内存监控

    * LeakCanery

        [LeakCanary的工作过程以及原理](https://github.com/linsir6/AndroidNote/blob/master/AndroidNote/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%9B%B8%E5%85%B3/LeakCanary%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B%E4%BB%A5%E5%8F%8A%E5%8E%9F%E7%90%86.md)

- Lottie

    



---

### 6.4 Android 的应用发布流程



---

### 6.5 Android 运行原理

* **Android 的开机过程**
* **Android 的启动流程**
* **Android 的打包流程**
* **Android 的分包原理**
* **APK 的安装流程**

---



### 6.6 Android 前沿知识点

* 热修复
* Kotlin
* Rxjava+Retrofit
* git
* gradle
* databinding

---

### 6.7 Android 推送方案

https://www.cnblogs.com/Joanna-Yan/p/6241354.html

---

### 6.8 Android 的零碎知识点

* keystore
* 实现倒计时的方式
* 实现沉浸式状态栏
* downloader (关于多线程断点续传)
* Android 5.0 6.0 7.0 8.0 9.0 的特性
* Dalvik 与 ART 的区别

---

### 6.9 Kotlin

---

## 7. 网络协议相关

* 网络协议相关知识点

    [网络协议](./网络协议.md)

## 8. 其他相关能力

### 8.1 项目总结

* 多渠道打包
* 

### 8.2 面试技巧

### 8.3 编程技巧

* 关于位运算

    [位运算](位运算.md)

* 关于正则表达式

    https://blog.csdn.net/bobo89455100/article/category/6604866/2?orderby=UpdateTime

* 关于架构设计

* 关于测试

* 