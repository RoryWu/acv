# Android 的运行原理

## Android的架构介绍

### 1.1 经典Android 架构图

![img](https://upload-images.jianshu.io/upload_images/4282881-3a3834c3f613dc3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/587/format/webp)

### 1.2 代码层面Android 架构图

![img](https://upload-images.jianshu.io/upload_images/4282881-475e9a126ce6dd59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/670/format/webp)





## Android 的开机过程

### 概述：

Loader  > Kernel > Native >  Framework > Application

### 细分：

BootRom > Bootloader > Kernel > Init > Zygote > SystemServer > Launcher

- Loader层主要包括Boot Rom和Boot Loader
- Kernel层主要是Android内核层
- Native层主要是包括init进程以及其fork出来的用户空间的守护进程、HAL层、开机动画等
- Framework层主要是AMS和PMS等Service的初始化
- Application层主要指SystemUI、Launcher的启动





## Android 的应用启动流程

### Android启动流程、app启动原理

Android 众多基于Linux内核的系统类似, 启动系统时, bootloader启动内核和init进程. init进程分裂出更多名为"daemons(守护进程)"的底层的Linux进程, 诸如android debug deamon, USB deamon等. 这些守护进程处理底层硬件相关的接口.随后, init进程会启动一个非常有意思的进程---"Zygote". 顾名思义, 这是一个Android平台的非常基础的进程. 这个进程初始化了第一个VM, 并且预加载了framework和众多App所需要的通用资源. 然后它开启一个Socket接口来监听请求, 根据请求孵化出新的VM来管理新的App进程. 一旦收到新的请求, Zygote会基于自身预先加载的VM来孵化出一个新的VM创建一个新的进程.

启动Zygote之后, init进程会启动runtime进程. Zygote会孵化出一个超级管理进程---System Server. SystemServer会启动所有系统核心服务, 例如Activity Manager Service, 硬件相关的Service等. 到此, 系统准备好启动它的第一个App进程---Home进程了.

##### Zygote 进程

Zygote 的中文意思是受精卵，从这个意思里也可以看出 Zygote 进程是用来分裂复制(fork)的，实际上所有的 App 进程都是通过对 Zygote 进程的 Fork 得来的。当 [app_process](https://link.jianshu.com?t=https://github.com/android/platform_frameworks_base/blob/master/cmds/app_process/app_main.cpp) 启动 Zygote 时，Zygote 会在其启动后，预加载必要的 Java Classes（相关列表查看 [预加载文件](https://link.jianshu.com?t=https://github.com/android/platform_frameworks_base/blob/master/preloaded-classes)） 和 Resources，并启动 [System Server](https://link.jianshu.com?t=http://www.woaitqs.cc/android/2016/05/26/android-binder-token.html) ，并打开 `/dev/socket/zygote` socket 去监听启动应用程序的请求，日后。在下面的代码中，显示了 Zygote 进程如何启动，和加载 System Server 的。

Android进程与Linux进程一样. 默认情况下, 每个apk运行在自己的Linux进程中. 另外, 默认一个进程里面只有一个线程---主线程. 这个主线程中有一个Looper实例, 通过调用Looper.loop()从Message队列里面取出Message来做相应的处理.

那么, 这个进程何时启动的呢?

简单的说, 进程在其需要的时候被启动. 任意时候, 当用户或者其他组件调取你的apk中的任意组件时, 如果你的apk没有运行, 系统会为其创建一个新的进程并启动. 通常, 这个进程会持续运行直到被系统杀死. 关键是: 进程是在被需要的时候才创建的.

##### 点击图标 启动应用



![img](https:////upload-images.jianshu.io/upload_images/2728246-a61300a471248e17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/683/format/webp)

app launch.png

Click事件会调用startActivity(Intent), 会通过Binder IPC机制, 最终调用到ActivityManagerService. 该Service会执行如下操作:

第一步通过PackageManager的resolveIntent()收集这个intent对象的指向信息.
 指向信息被存储在一个intent对象中.
 下面重要的一步是通过grantUriPermissionLocked()方法来验证用户是否有足够的权限去调用该intent对象指向的Activity.
 如果有权限, ActivityManagerService会检查并在新的task中启动目标activity.
 现在, 是时候检查这个进程的ProcessRecord是否存在了.
 如果ProcessRecord是null, ActivityManagerService会创建新的进程来实例化目标activity.

##### ActivityManager 架构

在我们编程过程中，涉及到许多 Activity 跳转的事情，在 Launcher 中点击 Icon 进行跳转也是同样的道理，调用 context.startActivity(intent) 方法。Launcher 出于一个线程，而启动的 App 则运行在另一个进程中，在这其中势必牵涉到跨进程 (IPC) 调用，这样复杂的过程显然需要一种中介者，或者一个系统来进行中转和管理，而这个服务就是 ActivityManagerService。

ActivityManagerService 作为一个守护进程运行在 Android Framework 中，如果让开发者直接接触这个类的话，就需要开发者自行处理 IPC 调用的问题，且这有不利于 Android 系统进行安全校验等工作。因而 Android 系统实现了 ActivityManager，通过这个 ActivityManager 作为一个入口，变相地和 ActivityManagerService 打交道。



## Android APK的打包流程

### 1. 背景

**1.1 apk修改后缀为zip。**



![img](https:////upload-images.jianshu.io/upload_images/5442499-be4542dee75b5c44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/517/format/webp)



**1.2 解压zip。**



![img](https:////upload-images.jianshu.io/upload_images/5442499-18714bf677fd5d80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/461/format/webp)



可以看到，我们一个项目经过编译和打包，形成了
 （1）assets资源。
 （2）lib不是每个apk都有的，主要看项目。
 （3）META-INF是签名文件夹，里面有三个文件。
 （4）res里面都是二进制的xml资源。
 （5）二进制的AndroidManifest.xml。
 （6）classes.dex是.dex文件，就是我们写的java代码经过处理得到的。
 （7）resources.arsc记录了所有的应用程序资源目录的信息，将其想象成是一个资源索引表，这个资源索引表在给定资源ID和设备配置信息（就是手机或其他运行apk的设备所拥有的配置信息）的情况下，能够在应用程序的资源目录中快速地找到最匹配的资源。

我们知道一个apk除了代码外，还有很多资源文件。上面七点内容中，（6）可以说就是我们平常辛辛苦苦写的代码了，而（1）和（4）是资源文件。这些资源文件是通过Android资源打包工具**aapt（Android Asset Package Tool）**打包到APK文件里面的。在打包之前，大部分**文本格式**的XML资源文件还会被编译成**二进制格式「1」**的XML资源文件。如（4）、（5）。

那为什么要将文本格式的xml转为二进制格式的xml呢？
 （1） 二进制格式的XML文件占用空间更小。
 （2） 二进制格式的XML文件解析速度更快。



### 2. 详述

接着我们来看一张图：
 



![img](https:////upload-images.jianshu.io/upload_images/5442499-e4b143ed93bb17fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/583/format/webp)

1. 从图中我们可以看出

 （1）除了assets和res/raw资源被原装不动地打包进APK之外，其它的资源都会被编译或者处理。

 （2）除了assets资源之外，其它的资源都会被赋予一个资源ID。包括res/raw也会有资源ID，即R.id.resourceId。

 （3）打包工具负责编译和打包资源，编译完成之后，会生成一个resources.arsc文件和一个R.java，前者保存的是一个资源索引表，后者定义了各个资源ID常量。



![img](https:////upload-images.jianshu.io/upload_images/5442499-b689f5f191f16da7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

R.java

   还是没明白resources.arsc和R.java存在的意义？我们可以这样想，一个项目由代码和资源文件组成，代码通过对资源的处理来展现业务流程，那我们代码中肯定要获取到某个所需要的资源吧？Android为了方便管理，就定了一个文件，里面定义了各个专属资源ID常量，来表示我们项目中存在的资源。写代码的时候，我们要用某个资源，是不是都是类似这样写：

```
tvMrchNo = (TextView) findViewById(R.id.tv_mrch_no);
```

   R.java中会有R.id.tv_mrch_no的常量值。resources.arsc中记录了所有资源信息，会根据资源ID（R.java）和你设备（手机或者其他Android系统设备）的配置信息，快速找到最匹配的资源展现。比如有时候我们资源中有不同大小的同一张图片，以便适应不同的设备。



2. 关于资源文件的打包，还有一点要说明一下。
      项目还未编译打包的时候res完整的有9种目录：
       **--animator**。这类资源以XML文件保存在res/animator目录下，用来描述属性动画。
       **--anim**。这类资源以XML文件保存在res/anim目录下，用来描述补间动画。
        **--color**。这类资源以XML文件保存在res/color目录下，用描述对象颜色状态选择子。
       **--drawable**。这类资源以XML或者Bitmap文件保存在res/drawable目录下，用来描述可绘制对象。
       **--layou**t。这类资源以XML文件保存在res/layout目录下，用来描述应用程序界面布局。
       **--menu**。这类资源以XML文件保存在res/menu目录下，用来描述应用程序菜单。
       **--raw**。这类资源以**任意格式的文件**保存在res/raw目录下。
       **--values**。这类资源以XML文件保存在res/values目录下，用来描述一些简单值，例如，数组、颜色、尺寸、字符串和样式值等。
       **--xml**。这类资源以XML文件保存在res/xml目录下，一般就是用来描述应用程序的配置信息。
       编译打包成apk后，apk文件中不包括res/values目录， 这是因为res/values目录下的资源文件的内容经过编译之后，都直接写入到资源项索引表resources.arsc去了。

    

3. apk打包过程图
     



![img](https:////upload-images.jianshu.io/upload_images/5442499-52cadc4569714633.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/536/format/webp)



 前面1、2点主要讲的是aapt资源打包的内容。从上图中可以看到。完整的打包流程应该是：

（1）打包资源文件。

（2）处理aidl文件，生成相应java 文件。对于没有使用到aidl的android工程，可以跳过此步骤。

（3）编译工程源代码，生成相应class 文件。

​		这一步调用了javac编译工程src目录下所有的java源文件，生成的class文件位于工程的bin\classes目录下，上图假定编译工程源代码时程序是基于android SDK开发的，实际开发过程中，也有可能会使用android NDK来编译native代码，因此，如果可能的话，这一步还需要使用android NDK编译C/C++代码，当然，编译C/C++代码的步骤也可以提前到第一步或第二步。通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件。

（4）转换所有class文件，生成classes.dex文件。

​		Android虚拟机的可执行文件为dex格式，所以需要此步骤。

（5）打包生成apk。

​		打包后的res文件夹（除res/raw资源被原装不动地打包进APK之外）、打包后类文件（.dex文件）、libs文件（包括.so文件，当然很多工程都没有这样的文件，如果你不使用C/C++开发的话）、resources.arsc、assets、AndroidManifest.xml打包成apk文件。

（6）对apk文件进行签名。

（7）对签名后的apk文件进行对其处理。

​		在 Android SDK 中包含一个名为 “zipalign” 的工具，它能够对打包后的 app 进行优化。 即对签名后的apk进行对齐处理。



**「1」**怎么知道xml到底是文本格式还是二进制格式呢？大家可以下载一个UltraEdit文本编辑器。然后查阅项目下的xml和解压后res里的xml，就会发现不同。
 



![img](https:////upload-images.jianshu.io/upload_images/5442499-041b13b1b1a02e28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/959/format/webp)

项目下的xml.png



![img](https:////upload-images.jianshu.io/upload_images/5442499-e1fe46e2d14e3718.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/862/format/webp)

解压apk中res里的xml.png



**「2」** 

1. res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。
2. res/raw不可以有目录结构，而assets则可以有目录结构（在其目录下可以再建文件夹）   。
3. 读取res/raw下的文件资源，通过以下方式获取输入流：
      `InputStream is=getResources().openRawResource(R.id.filename);`
     读取assets下的文件资源，通过以下方式获取输入流：
      `InputStream is =getResources()..getAssets().open("filename");`
     （因为apk安装之后会放在/data/app/**.apk目录下，以apk形式存在，assets被绑定在apk里，并不会解压到/data/app/xxx.apk目录下去，这个目录是只读不可写，所以我们无法直接获取到assets的绝对路径。好在Android系统为我们提供了一个AssetManager工具类。）



## Android 的分包原理

### MultiDex的产生背景

当Android系统安装一个应用的时候，有一步是对Dex进行优化，这个过程有一个专门的工具来处理，叫DexOpt。DexOpt的执行过程是在第一次加载Dex文件的时候执行的。这个过程会生成一个ODEX文件，即Optimised Dex。执行ODex的效率会比直接执行Dex文件的效率要高很多。

但是在早期的Android系统中，DexOpt有一个问题，DexOpt会把每一个类的方法id检索起来，存在一个链表结构里面。但是这个链表的长度是用一个short类型来保存的，导致了方法id的数目不能够超过65536个。当一个项目足够大的时候，显然这个方法数的上限是不够的。尽管在新版本的Android系统中，DexOpt修复了这个问题，但是我们仍然需要对低版本的Android系统做兼容。

为了解决方法数超限的问题，需要将该dex文件拆成两个或多个，为此谷歌官方推出了multidex兼容包，配合AndroidStudio实现了一个APK包含多个dex的功能。

### MultiDex的简要原理

我们以APK中有两个dex文件为例，第二个dex文件为classes2.dex。

1. 兼容包在Applicaion实例化之后，会检查系统版本是否支持 multidex，classes2.dex是否需要安装。
2. 如果需要安装则会从APK中解压出classes2.dex并将其拷贝到应用的沙盒目录下。
3. 通过反射将classes2.dex注入到当前的classloader中。

下面引入一下官方的文档。

[https://developer.android.com/tools/building](https://developer.android.com/tools/building/multidex.html#about)

```
Conversion to Dalvik format failed: Unable to execute dex: method ID not in [0, 0xffff]: 65536
```

当然，也有一些系统设备会出现以下log信息，不过反馈的是同一个问题：

[/multidex.html#about](https://developer.android.com/tools/building/multidex.html#about) 
笔者，针对官方文档的翻译如下：

### 构建超过65K方法的App

随着Android设备的慢慢发展，App的大小会变得越来越大。当我们在开发App的时候由于报的大小和引用库的原因，我们在编译我们项目的时候通常会遇到下面一个错误：

```
trouble writing output:
Too many field references: 131000; max is 65536.
You may try using --multi-dex option.
```

这两个错误条件显示一个共同的数字：65536。这个数字，它表示的是你在一个dex包中的函数方法超过了65535个。

如果你已经构建了一个Android App时，并收到了这个错误，那么恭喜你，你有很多代码！

下面我们就具体说说，如何解决这个问题。

 

### 关于65K方法限制

我们知道Android中的可执行伟剑都存储在dex文件中，其中包含已编译的代码来运行你的应用程序。Dalvik虚拟机对可执行dex文件的规格是有方法限制的，即一个单一的dex文件的方法总数最多为65536。

其中包括：

- 引用的Android Framework方法
- library的方法
- 我们自己书写代码的方法。

为了突破这个方法数的限制，我们就提出了一个方案——生成多个dex文件。这个多个dex文件的方案，我们又称为multidex方案配置。

**Multidex支持Android 5.0之前的版本** 
    Android5.0版本的平台之前，Android使用的是Dalvik Runtime执行的程序代码。默认情况下，限制应用到一个单一的classes.dex。

Dalvik字节码文件每APK。为了绕过这个限制，你可以使用multidex支持库，成为你的应用程序的主要部分和DEX文件进行管理，获得额外的dex文件，它们包含的代码。

**Multidex支持Android 5.0及更高版本** 
    Android 5.0和更高的Runtime 如art，本身就支持从应用的APK文件加载多个DEX文件。art支持预编译的应用程序在安装时扫描类（..）。Dex文件编译成一个单一的Android设备上执行.oat文件。

------

### 避免65K限制

当你确定使用multidex的分包策略的时候，请你先确定自己的代码中都是优秀的。你还需要做以下几步：

- 去掉一些未使用的import和library
- 使用ProGuard去掉一些未使用的代码

------

### 用Gradle配置使用Multidex

Android 的 Gradle插件在 Android Build Tool 21.1开始就支持使用multidex了。

设置你的应用程序开发项目中使用multidex配置，要求你做出一些修改您的应用程序开发项目。：

- 修改Gradle的配置，支持multidex
- 修改你的manifest。让其支持multidexapplication类

修改Gradle的build如下：

```java
android {
    compileSdkVersion 21
    buildToolsVersion "21.1.0"

    defaultConfig {
        ...
        minSdkVersion 14
        targetSdkVersion 21
        ...

        // Enabling multidex support.
        multiDexEnabled true
    }
    ...
}

dependencies {
  compile ‘com.android.support:multidex:1.0.0‘
}
```

> Tips: 你可以在Gradle配置文件中的 multiDexEnabled 在 defaultConfig、 
> buildType、productFlavor选项设置。

在manifest文件中，添加MultidexApplication Class的引用，如下所示：



```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.multidex.myapplication">
    <application
        ...
        android:name="android.support.multidex.MultiDexApplication">
        ...
    </application>
</manifest>
```



当然，如果你重写了 Application，就对自定义Application的继承方式做一个修改。

 

### Multidex的方式的局限性

虽然我们开起来multidex是一个极好的东西，但是multidex还是存在自己的局限性，我们在开发测试之前要清楚局限性是什么：

1. 如果二DEX文件太大，安装分割dex文件是一个复杂的过程，可能会导致应用程序无响应（ANR）的错误。在这种情况下，你应该尽量的减小dex文件的大小和删除无用的逻辑，而不是完全依赖于multidex。
2. 在Android 4.0设备（API Level 14）之前，由于Dalvik linearalloc bug（问题22586），multidex很可能是无法运行的。如果希望运行在Level 14之前的Android系统版本，请先确保完整的测试和使用。
3. 应用程序使用了multiedex配置的，会造成使用比较大的内存。当然，可能还会引起dalvik虚拟机的崩溃(issue 78035)。
4. 对于应用程序比较复杂的，存在较多的library的项目。multidex可能会造成不同依赖项目间的dex文件函数相互调用，找不到方法。

### 优化multidex开发和构建

一个multidex的配置，对系统apk的构建、签名、打包复杂性大大的增加。这就意味着，你每一次的构建过程都是相当耗时的。

为了加快我们的开发速度，加快构建的过程，我们可以在Gradle productFlavors新建出来一个 development flavor 和 production flavor 来满足我们不同构建需求。

下面是一个列子演示我们如何设置这些flavors在Gradle build文件中:

```
android {
    productFlavors {
        // Define separate dev and prod product flavors.
        dev {
            // dev utilizes minSDKVersion = 21 to allow the Android gradle plugin
            // to pre-dex each module and produce an APK that can be tested on
            // Android Lollipop without time consuming dex merging processes.
            minSdkVersion 21
        }
        prod {
            // The actual minSdkVersion for the application.
            minSdkVersion 14
        }
    }
          ...
    buildTypes {
        release {
            runProguard true
            proguardFiles getDefaultProguardFile(‘proguard-android.txt‘),
                                                 ‘proguard-rules.pro‘
        }
    }
}
dependencies {
  compile ‘com.android.support:multidex:1.0.0‘
}
```



在你完成了伤处的配置修改之后，你配置productFlavor 和 buildType来使用 ，devDebug 变种app。使用这些变种app，可以设置proguard disable、multidex enable方便我们测试。

这些配置需要针对Android Gradle插件做如下操作：

1. 在分包前，编译应用程序中的每一个module包括依赖项目，这个步骤称为 pre-dexing。
2. include每一个dex文件
3. 最重要的是，对于主dex文件，不会做切分。以保证计算速度。

这样设置既能够保证我们的最终报是一个使用了multidex模式的，而又不影响我们平时开发的测试效率。



### 在Android Studio中使用变种App

使用multidex工具构建变种App是非常方便的。在Android Studio允许我们选择这种变种构建方式的接口。

使用Android Studio构建 “devDebug”构建变种app需要完成两步：

- 打开变种编辑窗口，选择favorites选项。
- 点击编译不同的变种，如下图所示 

![img](https://images2015.cnblogs.com/blog/573504/201601/573504-20160126173906723-2095133998.png)





## APK 的安装流程

### Dalvik和ART是什么，有啥区别？

##### Dalivk

Dalvik是Google公司自己设计用于Android平台的虚拟机。支持已转换为** .dex格式**的Java应用程序的运行，.dex格式是专为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统。
 Dalvik 经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik 应用作为一个独立的Linux 进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。
 很长时间以来，Dalvik虚拟机一直被用户指责为拖慢安卓系统运行速度不如IOS的根源。
 2014年6月25日，Android L 正式亮相于召开的谷歌I/O大会，Android L 改动幅度较大，谷歌将直接删除Dalvik，代替它的是传闻已久的ART。

##### ART

即Android Runtime
 ART 的机制与 Dalvik 不同。在Dalvik下，应用每次运行的时候，字节码都需要通过即时编译器（just in time ，JIT）转换为机器码，这会拖慢应用的运行效率，而在ART 环境中，应用在第一次安装的时候，字节码就会预先编译成机器码，使其成为真正的本地应用。这个过程叫做预编译（AOT,Ahead-Of-Time）。这样的话，应用的启动(首次)和执行都会变得更加快速。

##### 区别：

* Dalvik是基于寄存器的，而JVM是基于栈的。
* Dalvik运行dex文件，而JVM运行java字节码
* 自Android 2.2开始，Dalvik支持JIT（just-in-time，即时编译技术）。
* 优化后的Dalvik较其他标准虚拟机存在一些不同特性:　
    1. 占用更少空间　
    2. 为简化翻译，常量池只使用32位索引　
    3. 标准Java字节码实行8位堆栈指令,Dalvik使用16位指令集直接作用于局部变量。局部变量通常来自4位的“虚拟寄存器”区。这样减少了Dalvik的指令计数，提高了翻译速度。当Android启动时，Dalvik VM 监视所有的程序（APK），并且创建依存关系树，为每个程序优化代码并存储在Dalvik缓存中。Dalvik第一次加载后会生成Cache文件，以提供下次快速加载，所以第一次会很慢。
        Dalvik解释器采用预先算好的Goto地址，每个指令对内存的访问都在64字节边界上对齐。这样可以节省一个指令后进行查表的时间。为了强化功能, Dalvik还提供了快速翻译器（Fast Interpreter）。

##### 对比

**ART有什么优缺点呢？**

优点：
 1、系统性能的显著提升。
 2、应用启动更快、运行更快、体验更流畅、触感反馈更及时。
 3、更长的电池续航能力。
 4、支持更低的硬件。
 缺点：
 1.机器码占用的存储空间更大，字节码变为机器码之后，可能会增加10%-20%
 2.应用的安装时间会变长

### 安装过程

Android应用安装有如下四种方式
 1.系统应用安装――开机时完成，没有安装界面
 2.网络下载应用安装――通过market应用完成，没有安装界面
 3.ADB工具安装――没有安装界面。
 4.第三方应用安装――通过SD卡里的APK文件安装，有安装界面，由packageinstaller.apk应用处理安装及卸载过程的界面。

#### 应用安装的流程及路径

 应用安装涉及到如下几个目录：

* system/app
     系统自带的应用程序，无法删除

* data/app
     用户程序安装的目录，有删除权限。
     安装时把apk文件复制到此目录

* data/data
     存放应用程序的数据

* Data/dalvik-cache
     将apk中的dex文件安装到dalvik-cache目录下(dex文件是dalvik虚拟机的可执行文件,其大小约为原始apk文件大小的四分之一)

```
   安装过程：复制APK安装包到data/app目录下，解压并扫描安装包，把dex文件(Dalvik字节码)保存到dalvik-cache目录，并data/data目录下创建对应的应用数据目录。
   
   卸载过程：删除安装过程中在上述三个目录下创建的文件及目录。 
```



把APK安装包保存在SD卡中，从手机里访问SD卡中的APK安装包，点击就可以启动安装界面，系统应用Packageinstaller.apk处理这种方式下的安装及卸载界面流程，如下：

* PackageInstallerActivity负责解析包，判断是否是可用的Apk文件
* 创建临时安装文件/data/data/com.android.packageinstaller/files/ApiDemos.apk并启动安装确认界面startInstallConfirm，列出解析得到的该应用基本信息。如果手机上已安装有同名应用，则需要用户确认是否要替换安装（校验签名）。

* 确认安装后，启动InstallAppProgress，调用安装接口完成安装。

* pm.installPackage(mPackageURI,observer, installFlags);

### 具体步骤

1. **将apk文件copy至data/app目录**

    1.1 installPackageAsUser 这个方法主要是判断安装来源,包括adb,shell,all_user,然后向PMS的mHandler发送INIT_COPY的消息,这个mHandler运行在一个HandlerThread中。



![img](https:////upload-images.jianshu.io/upload_images/2728246-91399d3679564042.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/854/format/webp)



​	1.2 handleMessage(INIT_COPY)&handleMessage(MCS_BOUND)
 		INIT_COPY主要是确保DefaultContainerService已bound,DefaultContainerService是一个应用服务，具体负责实现APK等相关资源文件在内部或外部存储器上的存储工作。而MCS_BOUND中则执行了



![img](https:////upload-images.jianshu.io/upload_images/2728246-6d7edc321b54759b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/752/format/webp)

params.startCopy()这句,也是最关键的开始copy文件。

 1.3  handleStartCopy()handleStartCopy的核心就是copyApk,其他的都是些存储空间检查,权限检查等等安全校验



 **2 .解析apk信息**

​	完成apk copy到data/app目录的操作后,下一步就到了 handleReturnCode,这个方法又跳转到processPendingInstall()方法,
 	解析了package包,然后做了大量签名和权限校验的工作,最终会走到覆盖安装和安装新应用对应的具体执行.

```
  if (replace) {
        replacePackageLI(pkg, parseFlags, scanFlags | SCAN_REPLACING, args.user,
                installerPackageName, res);
    } else {
        installNewPackageLI(pkg, parseFlags, scanFlags | SCAN_DELETE_DATA_ON_FAILURES,
                args.user, installerPackageName, res);
    }
```

scanPackageLI负责安装,而updateSettingLI则是完成安装后的设置信息更新

#### dexopt操作

Apk文件其实只是一个归档zip压缩包,而我们编写的代码最终都编译成了.dex文件,但为了提高运行性能,android系统并不会直接执行.dex,而是会在安装过程中执行dexopt操作来优化.dex文件,最终android系统执行的时优化后的'odex'文件(注意:这个odex文件的后缀也是.dex,其路径在data/dalvik-cache)。对于dalvik虚拟机,dexopt就是优化操作,而对于art虚拟机,dexopt执行的则是dex2oat操作,既将.dex文件翻译成oat文件。

dexopt操作执行完后,installNewPackageLI()方法就会走到updateSettingsLI()来更新设置信息,而更新设置信息主要是权限信息,所以直接来看updatePermissionsLPw();

最终会发送Intent.ACTION_PACKAGE_ADDED广播，apk的安装就到到此结束了。

####  其它：

1. PackageManagerService.java的内部类AppDirObserver实现了监听app目录的功能：当把某个APK拖到app目录下时，可以直接调用scanPackageLI完成安装。
2. 手机数据区目录“data/system/packages.xml”文件中，包含了手机上所有已安装

