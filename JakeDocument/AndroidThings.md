

Android 知识架构总结 by Jake

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

### 4.7  装饰者模式





## 5. Android 知识体系与基础知识

### 5.1 掌握Android自带的组件与类

#### 5.1.1 四大组件之Activity

* Activity 详解

    [Activity 详解](./Android_Activity.md)

#### 5.1.2 四大组件之Service

* 生命周期

    

* **startService 和 bindService的区别:** 

​	* startService特点：

​	1. 使用这种start方式启动的Service的生命周期如下：onCreate()--->onStartCommand()（onStart()方法已过时） ---> onDestroy()

​	2. 如果服务已经开启，不会重复的执行onCreate()， 而是会调用onStart()和onStartCommand()

​	3. 一旦服务开启跟调用者(开启者)就没有任何关系了。

​	4. 开启者退出了，开启者挂了，服务还在后台长期的运行。

​	5.开启者不能调用服务里面的方法。



​	* bindService特点：

​	1. 绑定服务不会调用onStart()或者onStartCommand()方法

​	2. bind的方式开启服务，绑定服务。调用者调用unbindService解除绑定，服务也会跟着销毁。

​	3. 绑定者可以调用服务里面的方法。



* **onStartCommand 不同返回值 的作用是什么?**

    * START_STICKY  在运行onStartCommand后service进程被kill后，那将保留在开始状态，但是不保留那些传入的intent。不久后service就会再次尝试重新创建，因为保留在开始状态，在创建      service后将保证调用onstartCommand。如果没有传递任何开始命令给service，那将获取到null的intent。

    * START_NOT_STICKY  在运行onStartCommand后service进程被kill后，并且没有新的intent传递给它。Service将移出开始状态，并且直到新的明显的方法（startService）调用才重新创建。因为如果没有传递任何未决定的intent那么service是不会启动，也就是期间onstartCommand不会接收到任何null的intent。

    * START_REDELIVER_INTENT  在运行onStartCommand后service进程被kill后，系统将会再次启动service，并传入最后一个intent给onstartCommand。直到调用stopSelf(int)才停止传递intent。如果在被kill后还有未处理好的intent，那被kill后服务还是会自动启动。因此onstartCommand不会接收到任何null的intent。

    

* **使用startForgroundService 的方法**

    1. 申请 FOREGROUND_SERVICE 权限，它是普通权限
    2. 在 onStartCommand 中必须要调用 startForeground 构造一个通知栏，不然 ANR
    3. 前台服务只能是启动服务，不能是绑定服务

    

* **bug**

    ```java
    android.app.RemoteServiceException
    ```

    使用前台服务，必须提供一个通知栏，不然五秒就会 ANR。

    ```java
      public int onStartCommand(Intent intent, int flags, int startId) {
            Log.i(TAG, "onStartCommand: ");
            NotificationManager mNotificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            // 通知渠道的id
            String id = "my_channel_01";
            // 用户可以看到的通知渠道的名字.
            CharSequence name = "Demo";
            // 用户可以看到的通知渠道的描述
            String description = "Desc";
            int importance = NotificationManager.IMPORTANCE_HIGH;
            NotificationChannel mChannel = new NotificationChannel(id, name, importance);
            // 配置通知渠道的属性
            mChannel.setDescription(description);
            // 设置通知出现时的闪灯（如果 android 设备支持的话）
            mChannel.enableLights(true);
            mChannel.setLightColor(Color.RED);
            // 设置通知出现时的震动（如果 android 设备支持的话）
            mChannel.enableVibration(true);
            mChannel.setVibrationPattern(new long[]{100, 200, 300, 400, 500, 400, 300, 200, 400});
            mNotificationManager.createNotificationChannel(mChannel);
    
            // 通知渠道的id
            String CHANNEL_ID = "my_channel_01";
            // Create a notification and set the notification channel.
            Notification notification = new Notification.Builder(this, CHANNEL_ID)
                    .setContentTitle("New Message").setContentText("You've received new messages.")
                    .setSmallIcon(R.drawable.ic_launcher_foreground)
                    .build();
            startForeground(1, notification);
            return super.onStartCommand(intent, flags, startId);
        }
    ```

    



> Service 的种类



#### 5.1.3 四大组件之Broadcast

* 作用与原理

  * ![broadcast](https://camo.githubusercontent.com/6d5fa4c7d2018338acd33def01441007559a33c0/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d366231636361323530653634653037612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

  

* 种类

  * 普通广播（Normal Broadcast）
    - 通过sendBroadcast进行发送，如果注册了Action匹配的接受者则会收到
    - 若发送广播有相应权限，那么广播接收者也需要相应权限
  * 系统广播（System Broadcast）
    - Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播
    - 每个广播都有特定的Intent - Filter（包括具体的action）
    - 系统广播由系统发送，不需要手动发送，只需要注册监听
  * 有序广播（Ordered Broadcast）
    - 通过sendOrderedBroadcast发送
    - 发送出去的广播被广播接收者按照先后顺序接收（有序是针对广播接收者而言的）
    - 广播接受者接收广播的顺序规则：Priority大的优先；动态注册的接收者优先
    - 先接收的可以对广播进行截断和修改
  * App应用内广播（本地广播、Local Broadcast）
    - 通过LocalBroadcastManager.getInstance(this).sendBroadcastSync();
    - App应用内广播可理解为一种局部广播，广播的发送者和接收者都同属于一个App
    - 相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 & 效率高（本地广播只会在APP内传播，安全性高；不允许其他APP对自己的APP发送广播，效率高）
  * 粘性广播（Sticky Broadcast）
    - 在Android5.0 & API 21中已经失效，所以不建议使用
    - 通过sendStickyBroadcast发送
    - 粘性广播在发送后就一直存在于系统的消息容器里面，等待对应的处理器去处理，如果暂时没有处理器处理这个广播则一直在消息容器里面处于等待状态
    - 粘性广播的Receiver如果被销毁，那么下次重新创建的时候会自动接收到消息数据

   

* 注意点与面试点

  > **本地广播的使用以及实现机制**

  - 基本使用：可以通过intent.setPackage(packageName)指定包名，也可以使用localBroadcastManager（常用），示例代码如下：

    ```
    //注册应用内广播接收器
    //步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
    mBroadcastReceiver = new mBroadcastReceiver();
    IntentFilter intentFilter = new IntentFilter();
    
    //步骤2：实例化LocalBroadcastManager的实例
    localBroadcastManager = LocalBroadcastManager.getInstance(this);
    
    //步骤3：设置接收广播的类型 
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
    
    //步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
    localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
    
    //取消注册应用内广播接收器
    localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
    
    //发送应用内广播
    Intent intent = new Intent();
    intent.setAction(BROADCAST_ACTION);
    localBroadcastManager.sendBroadcast(intent);
    ```

  - localBroadcastManager的实现机制

    1. LocalBroadcastManager高效的原因主要是因为它内部是通过Handler实现的，它的sendBroadcast()方法含义和我们平时所用的全局广播不一样，它的sendBroadcast()方法其实是通过handler发送一个Message实现的。
    2. 既然是它内部是通过Handler来实现广播的发送的，那么相比与系统广播通过Binder实现那肯定是更高效了，同时使用Handler来实现，别的应用无法向我们的应用发送该广播，而我们应用内发送的广播也不会离开我们的应用
    3. LocalBroadcastManager内部协作主要是靠这两个Map集合：mReceivers和mActions，当然还有一个List集合mPendingBroadcasts，这个主要就是存储待接收的广播对象

#### 5.1.4 四大组件之ContentProvider

* **概述**

    ContextProvider 为存储和获取数据提供了统一的接口，可以在不同的应用程序之间安全的共享数据。它允许把自己的应用数据根据需求开放给其他应用进行增删改查。数据的存储方式还是之前的方式，它只是提供了一个统一的接口去访问数据。

* **统一资源标识符**

    统一资源标识符即 URI，用来唯一标识 ContentProvider 其中的数据，外界进程通过 URI 找到对应的 ContentProvider 其中的数据，在进行数据操作。

    URI 分为系统预置和自定义，分别对应系统内置的数据（如通讯录等）和自定义数据库。

* **系统内置 URI**

    比如获取通讯录信息所需要的 URI：ContactsContract.CommonDataKinds.Phone.CONTENT_URI。

* **自定义 URI**

```java
格式:content://authority/path/id
authority:授权信息，用以区分不同的 ContentProvider
path:表名，用以区分 ContentProvider 中不同的数据表
id: ID号，用以区别表中的不同数据
示例:content://com.example.omooo.demoproject/User/1
上述 URI 指向的资源是：名为 com.example.omooo.demoproject 的 ContentProvider 中表名为 User 中 id 为 1 的数据。
```

​	注意，URI 也存在匹配通配符：* & #



* **MIME 数据类型**

    它是用来指定某个扩展名的文件用某种应用程序来打开。

    可以通过 ContentProvider.getType(uri) 来获得。

    每种 MIME 类型由两部分组成：类型 + 子类型。

    示例：text/html、application/pdf



* **ContentProvider的使用**

    * 组织数据方式

        ContentProvider 主要以表格的形式组织数据，同时也支持文件数据，只是表格形式用的比较多，每个表格中包含多张表，每张表包含行和列，分别对应数据。

    * 主要方法

```java
public class MyProvider extends ContentProvider {

    @Override
    public boolean onCreate() {
        return false;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        return null;
    }

    @Override
    public String getType(Uri uri) {
        return null;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        return null;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        return 0;
    }
}
```



* **辅助工具类**

  * ContentResolver

    统一管理不同的 ContentProvider 间的操作。
    ​	1. 即通过 URI 即可操作不同的 ContentProvider 中的数据
    ​	2. 外部进程通过 ContentResolver 类从而与 ContentProvider 类进行交互

    一般来说，一款应用要使用多个 ContentProvider，若需要了解每个 ContentProvider  的不同实现从而在完成数据交互，操作成本高且难度大，所以在 ContentProvider 类上多加一个 ContentResolver  类对所有的 ContentProvider 进行统一管理。

    ContentResolver 类提供了与 ContentProvider 类相同名字和作用的四个方法：

    ```java
    // 外部进程向 ContentProvider 中添加数据
    public Uri insert(Uri uri, ContentValues values)　 
    
    // 外部进程 删除 ContentProvider 中的数据
    public int delete(Uri uri, String selection, String[] selectionArgs)
    // 外部进程更新 ContentProvider 中的数据
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)　 
    
    // 外部应用 获取 ContentProvider 中的数据
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
    // 使用ContentResolver前，需要先获取ContentResolver
    // 可通过在所有继承Context的类中 通过调用getContentResolver()来获得ContentResolver
    ContentResolver resolver =  getContentResolver(); 
    
    // 设置ContentProvider的URI
    Uri uri = Uri.parse("content://cn.scu.myprovider/user"); 
    
    // 根据URI 操作 ContentProvider中的数据
    // 此处是获取ContentProvider中 user表的所有记录 
    Cursor cursor = resolver.query(uri, null, null, null, "userid desc"); 
    ```

  * ContentUris
    ​用来操作 URI 的，常用有两个方法：

    ```java
    // withAppendedId（）作用：向URI追加一个id
    Uri uri = Uri.parse("content://cn.scu.myprovider/user") 
    Uri resultUri = ContentUris.withAppendedId(uri, 7);  
    // 最终生成后的Uri为：content://cn.scu.myprovider/user/7
    // parseId（）作用：从URL中获取ID
    Uri uri = Uri.parse("content://cn.scu.myprovider/user/7") 
    long personid = ContentUris.parseId(uri); 
    //获取的结果为:7
    ```

  * UriMatcher
    在 ContentProvider 中注册 URI，根据 URI 匹配 ContentProvider 中对应的数据表。
    ```java
    // 步骤1：初始化UriMatcher对象
    UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH); 
    //常量UriMatcher.NO_MATCH  = 不匹配任何路径的返回码
    // 即初始化时不匹配任何东西
    
    // 步骤2：在ContentProvider 中注册URI（addURI（））
    int URI_CODE_a = 1；
    int URI_CODE_b = 2；
    matcher.addURI("cn.scu.myprovider", "user1", URI_CODE_a); 
    matcher.addURI("cn.scu.myprovider", "user2", URI_CODE_b); 
    // 若URI资源路径 = content://cn.scu.myprovider/user1 ，则返回注册码URI_CODE_a
    // 若URI资源路径 = content://cn.scu.myprovider/user2 ，则返回注册码URI_CODE_b
    
    // 步骤3：根据URI 匹配 URI_CODE，从而匹配ContentProvider中相应的资源（match（））
    @Override   
      public String getType(Uri uri) {   
        Uri uri = Uri.parse(" content://cn.scu.myprovider/user1");   
    
        switch(matcher.match(uri)){   
       // 根据URI匹配的返回码是URI_CODE_a
       // 即matcher.match(uri) == URI_CODE_a
        case URI_CODE_a:   
          return tableNameUser1;   
          // 如果根据URI匹配的返回码是URI_CODE_a，则返回ContentProvider中的名为tableNameUser1的表
        case URI_CODE_b:   
          return tableNameUser2;
          // 如果根据URI匹配的返回码是URI_CODE_b，则返回ContentProvider中的名为tableNameUser2的表
      }   
    }
    ```

  * ContentObserver
    内容观察者，当 ContentProvider 中的数据发生变化时，就会触发 ContentObserver 类。
    ```java
          // 步骤1：注册内容观察者ContentObserver
          getContentResolver().registerContentObserver（uri）；
          // 通过ContentResolver类进行注册，并指定需要观察的URI
    
          // 步骤2：当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）
          public class UserContentProvider extends ContentProvider { 
            public Uri insert(Uri uri, ContentValues values) { 
            db.insert("user", "userid", values); 
            getContext().getContentResolver().notifyChange(uri, null); 
            // 通知访问者
         } 
      }
       // 步骤3：解除观察者
       getContentResolver().unregisterContentObserver（uri）；
       // 同样需要通过ContentResolver类进行解除
    ```

    

  

* **实例**

  1. **获取通讯录信息**

  这里就不需要自己写 ContentProvider 的实现了，用系统已经给的 URI。
  ```java
      /**
       * 获取通讯录信息
       */
      private void getContactsInfo() {
          Cursor cursor = getContentResolver().query(
                  ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null
          );
          if (cursor != null) {
              while (cursor.moveToNext()) {
                  //联系人姓名
                  String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                  //联系人手机号
                  String phoneNumber = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                  Log.i(TAG, "getContactsInfo: name: " + name + "  phone: " + phoneNumber);
              }
              cursor.close();
          }
      }
  ```

  2. **结合 SQLite**
      1. 创建数据库
      2. 自定义 ContentProvider 并注册
      3. 进程内访问数据

          a. 创建数据库：

    ```java
    public class MySQLiteOpenHelper extends SQLiteOpenHelper {
  
        public MySQLiteOpenHelper(Context context) {
            super(context, "user.info", null, 1);
        }
  
        @Override
        public void onCreate(SQLiteDatabase db) {
            db.setPageSize(1024 * 4);
    //        db.enableWriteAheadLogging();
            db.execSQL("CREATE TABLE if not exists user (name text, age string)");
        }
  
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
  
        }
    }
    ```

  ​		b. 自定义 ContentProvider 并注册：
    ```java
        public class MyProvider extends ContentProvider {
            private static UriMatcher mUriMatcher;
  
            static {
                mUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
                mUriMatcher.addURI("com.example.omooo.demoproject.provider", "user", 1);
            }
  
            private MySQLiteOpenHelper mMySQLiteOpenHelper;
            private SQLiteDatabase mSQLiteDatabase;
            private Context mContext;
  
            @Override
            public boolean onCreate() {
                mContext = getContext();
                mMySQLiteOpenHelper = new MySQLiteOpenHelper(mContext);
                mSQLiteDatabase = mMySQLiteOpenHelper.getWritableDatabase();
                mSQLiteDatabase.execSQL("insert into user values('Omooo','18');");
                return true;
            }
  
            @Nullable
            @Override
            public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
                return mSQLiteDatabase.query("user", projection, selection, selectionArgs, null, null, sortOrder, null);
            }
  
            @Nullable
            @Override
            public String getType(@NonNull Uri uri) {
                return null;
            }
  
            @Nullable
            @Override
            public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
                mSQLiteDatabase.insert("user", null, values);
                mContext.getContentResolver().notifyChange(uri, null);
                return uri;
            }
  
            @Override
            public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
                return 0;
            }
  
            @Override
            public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
                return 0;
            }
  
        }
            <provider
                    android:exported="false"
                    android:authorities="com.example.omooo.demoproject.provider"
                    android:name=".provider.MyProvider"/>
    ```

  ​			c. 进程内访问数据：

    ```java
        private void insertTable() {
            Uri uri = Uri.parse("content://com.example.omooo.demoproject.provider/user");
            ContentValues values = new ContentValues();
            values.put("name", "Test");
            values.put("age", "21");
            ContentResolver resolver = getContentResolver();
            resolver.insert(uri, values);
            Cursor cursor = resolver.query(uri, new String[]{"name", "age"}, null, null, null);
            if (cursor != null) {
                while (cursor.moveToNext()) {
                    String name = cursor.getString(cursor.getColumnIndex("name"));
                    String age = cursor.getString(cursor.getColumnIndex("age"));
                    Log.i(TAG, "insertTable: name: " + name + " age: " + age);
                }
                cursor.close();
            }
        }
    ```

* 注意点与面试点



#### 5.1.5 常用组件之Fragment

* Fragment 详解 

    [Fragment 的详解](./Android_Fragment.md)

#### 5.1.6 常用组件之SharedPreference



#### 5.1.7 常用组件之Intent

**Intent知识点**

**IntentFilter知识点**

IntentFilter 的三个属性:

​	Action
​	URL
 ​	Category

IntentFilter 的匹配规则

- 加载所有的Intent Filter列表 　　
- 去掉action匹配失败的Intent Filter 　　
- 去掉url匹配失败的Intent Filter 　　
- 去掉Category匹配失败的Intent Filter 　　
	 判断剩下的Intent Filter数目是否为0。如果为0查找失败返回异常；如果大于0，就按优先级排序，返回最高优先级的Intent Filter	



> 如何在短信中启动一个Activity



#### 5.1.8 常用组件之Drawable

* Drawable 的相关知识点

    https://blog.csdn.net/lmj623565791/article/details/43752383

#### 5.1.9 常用组件之Handler

- Handler 作用与原理

    [Android Handler详解](./Android_Handler.md)

#### 5.1.10 常用组件之AndroidManifests.mxl

- 注意点与面试点

#### 5.1.11 常用组件之Animation

- Android中的动画

    [Android_Animation详解](./Android_Animation.md)

#### 5.1.12 常用组件之布局文件

> 　常用ViewGroup 布局

* LinearLayout

* RelativeLayout

* FrameLayout

* ConstaintLayout

    https://www.jianshu.com/p/a74557359882

    https://mp.weixin.qq.com/s/gGR2itbY7hh9fo61SxaMQQ



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

[Android 中的Context](./Android Context.md)

#### 5.2.3 IPC



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



> invalidate()和postInvalidate() 的区别及使用

<http://blog.csdn.net/mars2639/article/details/6650876>





#### 5.3.5 View 的重要知识点

ViewDragHelper

https://blog.csdn.net/yanbober/article/details/50419059

坐标系统

https://blog.csdn.net/yanbober/article/details/50419117

Scroller

https://blog.csdn.net/yanbober/article/details/49904715

关于View 的可见性

https://www.jianshu.com/p/54a2af8f8e2b



5.









### 5.4 Android 中的多线程

* Android 中常见的多线程
    * IntentService
    * Rxjava
    * 
    * 



### 5.5 Android 中的存储方式

Android 中原生的存储方式主要有以下几种常用方式

- SQLite：SQLite是一个轻量级的数据库，支持基本的SQL语法，是常被采用的一种数据存储方式。 Android为此数据库提供了一个名为SQLiteDatabase的类，封装了一些操作数据库的api
- SharedPreference： 除SQLite数据库外，另一种常用的数据存储方式，其本质就是一个xml文件，常用于存储较简单的参数设置。
- File： 即常说的文件（I/O）存储方法，常用语存储大数量的数据，但是缺点是更新数据将是一件困难的事情。
- ContentProvider: Android系统中能实现所有应用程序共享的一种数据存储方式，由于数据通常在各应用间的是互相私密的，所以此存储方式较少使用，但是其又是必不可少的一种存储方式。例如音频，视频，图片和通讯录，一般都可以采用此种方式进行存储。每个Content Provider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享时，就需要使用Content Provider为这些数据定义一个URI，然后其他的应用程序就通过Content Provider传入这个URI来对数据进行操作。



### 5.6 Android 中的相机与相册

[Android 相机相册调用](./Android_Camera & Gallery.md)



### 5.7 Android 中的序列化

* Android 中的序列化

    [Android 中的序列化](./Android序列化.md)

### 5.10 Android 的细节注意点

1. 全局异常处理

2. ANR

3. Lint

4. 

     





## 6.Android进阶知识点

### 6.1 Adroid 开发的优化

* **应用的内存优化**

* **应用的启动时间优化**

* **应用安装包Size 的裁剪**

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



### 6.2 Android 的开发模式理解

* MVC

* MVP

* MVVM

    



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

- 



### 6.4 Android 的应用发布流程



### 6.5 Android 运行原理

* Android 的开机过程

* Android 的启动流程

* Android 的打包流程

* APK 的安装流程



### 6.6 Android 前沿知识点

* 热修复
* Kotlin
* Rxjava+Retrofit
* git
* gradle
* databinding



### 6.7 Android 推送方案

https://www.cnblogs.com/Joanna-Yan/p/6241354.html



### 6.8 Android 的零碎知识点

* keystore
* 实现倒计时的方式
* 实现沉浸式状态栏
* downloader (关于多线程断点续传)
* Android 5.0 6.0 7.0 8.0 9.0 的特性



## 7. 网络协议相关

* 网络协议相关知识点

    [网络协议](./网络协议.md)

## 8. 其他相关能力

### 8.1 项目总结

### 8.2 面试技巧

### 8.3 编程技巧

* 关于位运算

    [位运算](位运算.md)

* 关于正则表达式

    https://blog.csdn.net/bobo89455100/article/category/6604866/2?orderby=UpdateTime

* 关于架构设计