# 深入了解IPC

## IPC机制简介

IPC是Inter-Process Communication的缩写，含义就是跨进程通信。
 首先我们要理解什么是进程，什么是线程。按操作系统的描述，进程是资源分配的最小单位，而线程是CPU调度的最小单位，一个进程可以包含多个线程（主线程、子线程）。多线程需要考虑并发问题。
 Android中的主线程是也叫UI线程，在主线程执行耗时操作会ANR

**多进程的两种情况**
 1 某个应用由于自身原因需要采用多进程模式来实现（如：某些模块由于特殊原因需要运行在独立进程）
 2 为了加大一个应用可使用的内存通过多进程来获取多份内存空间

## Android中的多进程模式

在正式讲解进程间通信前我们先了解Android中的多进程模式。
 通过给四大组件指定*android:process*属性可以轻易开启多进程（看起来简单，但是有许多需要注意的问题）

一般情况下Android中多进程是指一个应用中存在多个进程的情况。首先在Android使用多进程只有一中方式就是为四大组件指定android:process属性（特例：使用JNI在native底层fork一个新进程）

1 android:process=”:test” （私有进程其他应用组件不可在该进程）
 2 android:process=”com.test.l” （全局进程其他组件可用ShareUID方式跑在相同进程中）

UID：在Linux中的代表用户ID，android系统为每个应用分配的标识，也就是说android中的每个应用其实就相当于一个用户。两个应用如果想通过shareUID的方式跑在同一个进程中必须保证UID相同，并且签名一致。那么同进程中就可以相互访问对方的私有数据（如data，res资源，组件信息，内存数据等等）

**ShareUID扩展**
 通过shareduserid来获取系统权限
 (1)在AndroidManifest.xml中添加android:sharedUserId=”android.uid.system”
 (2)在Android.mk文件里面添加LOCAL_CERTIFICATE := platform（使用系统签名）
 (3)在源码下面进行mm编译
 这样生成的apk能够获取system权限，可以在任意system权限目录下面进行目录或者文件的创建，以及访问其他apk资源等（注意创建的文件(夹)只有创建者(比如system,root除外)拥有可读可写权限

## 运行机制与IPC基础

首先我们知道Android系统是基于JVM（Dalvik与ART）
 其次系统会为每个进程分配一个独立的虚拟机，意味着进程间的内存的相互独立的

一般来说使用多进程会有几个问题
 1 静态成员与单利模式完全失效
 2 线程同步机制完全失效
 3 SP可靠性下降
 4 Application创建多次

问：进程间的对象传递问题怎么解决？
 通过IBinder来实现进程间对象的转换

### IPC基础概念介绍

主要包含三部分：Serialiazable，Parcelable以及Binder
 序列化：将对象转化为字节的过程
 Serialiazable：Java提供的序列化接口（标记接口）
 Parcelable：android提供的序列化接口
 Serialiazable与Parcelable的区别：Serialiazable使用简单但是需要大量I/O操作，Parcelable使用较繁琐，主要用于内存序列化，效率高

**Binder**
 直观的看，Binder是Android中的一个类，实现了IBinder接口
 从不同角度理解Binder：
 1 从IPC角度，Binder是跨进程通信方式
 2 从FrameWork角度，Binder是ServiceManager连接各种Manager（如am，wm等）的桥梁
 3 从应用层角度，Binder是客户端与服务端通信的媒介

## AIDL的初步了解

1 创建Book.java实体类（实现Parcelable接口，支持内存序列化）
 2 创建Book类的aidl声明
 3 创建IBookManager的aidl接口

> 实现Parcelable接口的Book类

```
public class Book implements Parcelable{
    private int id;
    private String name;
    private double price;

    public Book(int id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public Book() {
    }

    protected Book(Parcel in) {
        id = in.readInt();
        name = in.readString();
        price = in.readDouble();
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(id);
        parcel.writeString(name);
        parcel.writeDouble(price);
    }
}
```

其实内部是通过Parcel进行对象的转换的，咱们顺便看看Parcel类的描述

> Container for a message (data and object references) that can
>  be sent through an IBinder. A Parcel can contain both flattened data
>  that will be unflattened on the other side of the IPC (using the various
>  methods here for writing specific types, or the general
>  {@link Parcelable} interface), and references to live {@link IBinder}
>  objects that will result in the other side receiving a proxy IBinder
>  connected with the original IBinder in the Parcel.

简单的翻译下，Parcel是IBinder发送消息（数据和对象引用）的容器，Parcel可以数据通过各个操作基础类型的方法将数据转化为扁平化传递给IPC的另一端，并且通过IBinder使得对方收到的代理IBinder对象中包裹着原始IBinder。总的说它的作用就是实现对象数据在内存中的传递

> Book的aidl声明

```
package com.example.pangyunxiao.ipcdemo.com.itszt.l;

parcelable Book;
```

我用的IDE是Android Studio，直接右键创建aidl文件。会自动帮你创建aidl目录，并且将你创建的aidl声明放置到你的项目包名下，如果的eclipse则开发者要自行注意。

> IBookManager的声明

```
package com.example.pangyunxiao.ipcdemo.com.itszt.l;

import com.example.pangyunxiao.ipcdemo.com.itszt.l.Book;
import com.example.pangyunxiao.ipcdemo.com.itszt.l.IOnNewBookAddedListener;
// Declare any non-default types here with import statements

interface IBookManager {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);

            void addBook(in Book book);

            List<Book> getBooks();

            void addBookAndNotify(in Book book);

            void registerListener(IOnNewBookAddedListener l);

            void unregisterListener(IOnNewBookAddedListener l);
}
```

**注意问题**
 1 实体类的包名与aidl文件的包名必须一致
 2 IBookManager中必须import所需实体类
 3 接口中声明的方法参数需要使用in/out/inout
 4 aidl文件以及实体类在两端都要有，并且同个包下

**IBookManager.java简析**
 接着咱们编译一下项目，会发现自动生成的IBookManager.java类。路径：eclipse中位于gen目录下，as中位于build\generated\source\aidl\debug目录下，接着咱们来简要分析下这个类（由于类比长，就不把代码全贴出来啦）

```
public interface IBookManager extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.example.pangyunxiao.ipcdemo.com.itszt.l.IBookManager {
        private static final java.lang.String DESCRIPTOR = "com.example.pangyunxiao.ipcdemo.com.itszt.l.IBookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.pangyunxiao.ipcdemo.com.itszt.l.IBookManager interface,
         * generating a proxy if needed.
         */
        public static com.example.pangyunxiao.ipcdemo.com.itszt.l.IBookManager asInterface(android.os.IBinder obj) {
           ...
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            ...
        }

        private static class Proxy implements com.example.pangyunxiao.ipcdemo.com.itszt.l.IBookManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            /**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException {
                ...
            }

            @Override
            public void addBook(com.example.pangyunxiao.ipcdemo.com.itszt.l.Book book) throws android.os.RemoteException {
                ...
            }

            @Override
            public java.util.List<com.example.pangyunxiao.ipcdemo.com.itszt.l.Book> getBooks() throws android.os.RemoteException {
                ...
            }

            @Override
            public void addBookAndNotify(com.example.pangyunxiao.ipcdemo.com.itszt.l.Book book) throws android.os.RemoteException {
                ...
            }

            @Override
            public void registerListener(com.example.pangyunxiao.ipcdemo.com.itszt.l.IOnNewBookAddedListener l) throws android.os.RemoteException {
                ...
            }

            @Override
            public void unregisterListener(com.example.pangyunxiao.ipcdemo.com.itszt.l.IOnNewBookAddedListener l) throws android.os.RemoteException {
               ...
            }
        }

        static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
        static final int TRANSACTION_getBooks = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
        static final int TRANSACTION_addBookAndNotify = (android.os.IBinder.FIRST_CALL_TRANSACTION + 3);
        static final int TRANSACTION_registerListener = (android.os.IBinder.FIRST_CALL_TRANSACTION + 4);
        static final int TRANSACTION_unregisterListener = (android.os.IBinder.FIRST_CALL_TRANSACTION + 5);
    }

    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;

    public void addBook(com.example.pangyunxiao.ipcdemo.com.itszt.l.Book book) throws android.os.RemoteException;

    public java.util.List<com.example.pangyunxiao.ipcdemo.com.itszt.l.Book> getBooks() throws android.os.RemoteException;

    public void addBookAndNotify(com.example.pangyunxiao.ipcdemo.com.itszt.l.Book book) throws android.os.RemoteException;

    public void registerListener(com.example.pangyunxiao.ipcdemo.com.itszt.l.IOnNewBookAddedListener l) throws android.os.RemoteException;

    public void unregisterListener(com.example.pangyunxiao.ipcdemo.com.itszt.l.IOnNewBookAddedListener l) throws android.os.RemoteException;
}
```

DESCRIPTOR：Binder的唯一标识
 1 这是个接口类并且集成了IInterface接口，所有可以在Binder中传输的接口都需要继承IIterface接口
 2 Stub内部类：是IBookManager类的内部类，就是一个Binder类
 asInterface 静态方法：Binder对象转换为接口类型对象
 asBinder 非静态方法：接口类型对象转换为Binder对象
 onTransact方法：当客户端发起远程调用时，在服务端会执行该方法；参数code：标识调用的方法；参数data：传递的参数；参数reply：用于存储返回值；该方法返回值是boolean表示远程调用是否成功（可做权限验证）
 3 Proxy：Stub的内部类，作用是作为IBookManager的代理类（asInterface返回的其实就是它）；在该类中主要实现了各个方法的执行流程。例如addBook：方法运行在客户端，首先获取两个Parcel对象（参数_data与返回值_reply），然后把参数信息写入_data中，调用IBinder对象的transact方法发起远程调用，此时当前线程被挂起然后服务器端的onTransact执行，当执行完毕，当前线程继续执行，如果方法有返回值的话通过_reply取得。

**注意点**
 当客户端发起请求时当前线程会被挂起直至服务端返回数据。那么如果一个远程方法是耗时的就不能在UI线程发起远程请求
 2 AIDL文件并不是必须的，只是为了方便系统根据aidl文件生存java类，我们可以抛开aidl文件直接写一个Binder
 （1创建IBookManager接口类2IBookManager实现类3 Servic中返回）
 3 Binder运行在服务端进程，如果服务器进程由于某种原因终止了，那么远程调用将失败（称为Binder死亡），可以通过Binder对象的linkToDeath方法设置死亡代理



![img](https:////upload-images.jianshu.io/upload_images/11156265-ce147e08abb046ac?imageMogr2/auto-orient/strip%7CimageView2/2/w/553/format/webp)

流程

## 实现IPC的方式

怎么实现IPC？其实只要完成它的本质需求，就是在进程之间传递数据，那就算实现了IPC。因此我们有很多种方式可以完成这个目标。实际开发过程中可以选择合适的方式去实现这个数据传递过程。

### 1 使用Bundle

这是最简单的方式，它就是通过Bundle在不同进程的组件之间传递数据

```
startActivity(new Intent(...).putExtra("data",bundle));
```

### 2 使用文件共享

利用多进程同时读写同个外部文件达到是数据交互的目的
 存储形式没有限制：xml，文本，对象序列化等等
 缺点：由于Linux系统对文件并发读写没有限制，会导致数据不同步问题，所以该方式只适合于对数据同步要求不高的进程间通信

### 3 使用共享参数

共享参数是android中一中轻量级存储方案，底层用实现xml文件，系统对它的读写有一定的缓存策略（内存中会有一份sp的备份）在多进程模式下，系统对它的读/写是不可靠的，高并发读/写时有可能会丢失数据

### 4 使用Messenger

Messenger(信使)：在不同的进程之间传递Message对象，是一种轻量级的IPC方案，底层实现就是AIDL
 客户端使用服务器的messenger向服务器发消息
 服务器使用客户端的messenger向客户端发消息
 （类似Handler的使用）
 注意Service的隐式启动再5.0+上的问题，需要将隐式Intent转为显式

> 服务端Handler

```
public class MessengerHandler extends Handler {

    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        if(msg.what == 1){
            Log.e("MessengerHandler", msg.getData().getString("msg"));
            Messenger client = msg.replyTo;
            Message message = Message.obtain();
            message.what = 1;
            Bundle bundle = new Bundle();
            bundle.putString("reply","shou dao le.");
            message.setData(bundle);
            try {
                client.send(message);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    }
}
```

> 服务端Service

```
public class MessengerService extends Service {

    Messenger messenger = new Messenger(new MessengerHandler());

    @Override
    public IBinder onBind(Intent intent) {
        return messenger.getBinder();
    }
}

        <service android:name=".MessengerService" android:enabled="true" android:exported="true">
            <intent-filter>
                <action android:name="com.itszt.lww"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </service>
```

> 客户端Handler

```
public class ClientMHandler extends Handler {

    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        if(msg.what == 1){
            String str = msg.getData().getString("reply");
            if (BuildConfig.DEBUG) Log.e("ClientMHandler", str);
        }
    }
}
```

> 客户端调起远程服务

```
intent = new Intent("com.itszt.lww");
        bindService(createExplicitFromImplicitIntent(this,intent),conn, Service.BIND_AUTO_CREATE);

ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            Messenger messenger = new Messenger(iBinder);
            if(messenger == null) return;
            Message message = Message.obtain();
            message.what = 1;
            Bundle bundle = new Bundle();
            bundle.putString("msg","i am a client.");
            message.setData(bundle);
            message.replyTo = new Messenger(new ClientMHandler());
            try {
                messenger.send(message);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };
```

### 使用AIDL

在上述的AIDL初了解的基础上，咱们来实现简单的AIDL通信。增加一个Service提供给客户端调起，并返回咱们定义IBookManager对应的IBinder对象

```
public class AIDLService extends Service {

    // 保证高并发问题
    List<Book> books = new CopyOnWriteArrayList<>();
    RemoteCallbackList<IOnNewBookAddedListener> listeners = new RemoteCallbackList<>();

    {
        books.add(new Book(1,"java",66.3));
        books.add(new Book(2,"php",61.3));
        books.add(new Book(3,"c++",65.3));
    }

    IBookManager bookManager = new IBookManager.Stub() {
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public void addBook(Book book) throws RemoteException {
            books.add(book);
        }

        @Override
        public List<Book> getBooks() throws RemoteException {
            // 前面我们有提到AIDL只支持ArrayList
            // 那么为什么我们这边可以返回CopyOrWriteArrayList
            // 是这样的，服务器返回的CopyOrWriteArrayList会按List规范生成一个ArrayList传给客户端
            // 类似的还有ConcurrentHashMap
            return books;
        }

        @Override
        public void addBookAndNotify(Book book) throws RemoteException {
            addBook(book);
            if(listeners == null) return;
            int count = listeners.beginBroadcast();
            for(int i = 0;i<count;i++){
                IOnNewBookAddedListener listener = listeners.getBroadcastItem(i);
                listener.onNewBook();
            }
            listeners.finishBroadcast();
        }

        @Override
        public void registerListener(IOnNewBookAddedListener l) throws RemoteException {
            if(listeners == null)
                listeners = new RemoteCallbackList<>();
            listeners.beginBroadcast();
            listeners.register(l);
            listeners.finishBroadcast();
            if (BuildConfig.DEBUG) Log.e("AIDLServiceAdd", "listeners.size():" + listeners.beginBroadcast());
            listeners.finishBroadcast();
        }

        @Override
        public void unregisterListener(IOnNewBookAddedListener l) throws RemoteException {
            if(listeners != null){
                listeners.beginBroadcast();
                listeners.unregister(l);
                listeners.finishBroadcast();
            }
            if (BuildConfig.DEBUG) Log.e("AIDLServiceRemove", "listeners.size():" + listeners.beginBroadcast());
            listeners.finishBroadcast();
        }

    };

    @Override
    public IBinder onBind(Intent intent) {
        return bookManager.asBinder();
    }
}

        <service android:name=".AIDLService" android:enabled="true" android:exported="true">
            <intent-filter>
                <action android:name="com.itszt.aidl"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </service>
```

> 客户端调起服务获得IBinder

```
public class MainActivity extends Activity {

    IBookManager bookManager;
    IOnNewBookAddedListener listener = new IOnNewBookAddedListener.Stub() {
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public void onNewBook() throws RemoteException {
            Toast.makeText(MainActivity.this, "收到新书通知啦！！", Toast.LENGTH_SHORT).show();
        }
    };

    Intent intent;
    ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            if(iBinder == null) return;
            bookManager = IBookManager.Stub.asInterface(iBinder);
            if(bookManager == null) return;
            try {
                iBinder.linkToDeath(new IBinder.DeathRecipient() {
                    @Override
                    public void binderDied() {
                        unbindService(conn);
                        bindService(intent,conn,Service.BIND_AUTO_CREATE);
                    }
                },0);
                bookManager.registerListener(listener);
                Toast.makeText(MainActivity.this, bookManager.getBooks().toString(), Toast.LENGTH_SHORT).show();
                bookManager.addBook(new Book(5,"mXx",90));
                bookManager.addBookAndNotify(new Book(5,"mXx",90));
                // Toast.makeText(MainActivity.this, bookManager.getBooks().toString(), Toast.LENGTH_SHORT).show();
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intent = new Intent("com.itszt.aidl");
        bindService(createExplicitFromImplicitIntent(this,intent),conn, Service.BIND_AUTO_CREATE);
    }

    public static Intent createExplicitFromImplicitIntent(Context context, Intent implicitIntent) {
        // Retrieve all services that can match the given intent
        PackageManager pm = context.getPackageManager();
        List<ResolveInfo> resolveInfo = pm.queryIntentServices(implicitIntent, 0);

        // Make sure only one match was found
        if (resolveInfo == null || resolveInfo.size() != 1) {
            return null;
        }

        // Get component info and create ComponentName
        ResolveInfo serviceInfo = resolveInfo.get(0);
        String packageName = serviceInfo.serviceInfo.packageName;
        String className = serviceInfo.serviceInfo.name;
        ComponentName component = new ComponentName(packageName, className);

        // Create a new intent. Use the old one for extras and such reuse
        Intent explicitIntent = new Intent(implicitIntent);

        // Set the component to be explicit
        explicitIntent.setComponent(component);

        return explicitIntent;
    }

    @Override
    protected void onDestroy() {
        if(bookManager!=null && bookManager.asBinder().isBinderAlive()){
            try {
                bookManager.unregisterListener(listener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        super.onDestroy();
    }
}
```

AIDL中可以使用的数据类型
 基本数据类型（int char double boolean long）
 String和CharSequence
 List：只支持ArrayList并且每个元素都被aidl支持
 Map：只支持HashMap并且每个元素都被aidl支持
 Parcelable：所有实现了Pacelable接口的对象
 AIDL：所有aidl接口本身也可以在aidl中使用

注意：
 1 在AIDL文件中必须显式import所使用到的Parcelable子类以及aidl
 2 所有用于AIDL的Parcelable类必须创建一个同名的aidl文件（声明它为Parcelable类型）
 3 除了基本数据类型外其他类型的方法参数都必须标识方向（in/out/inout）
 不要图方便全部使用inout，因为这在底层实现是有开销的
 Stub是服务器与Binder的中介
 Proxy是客户端与Binder的中介
 多进程间是内存独立的，进程1传递对象A给进程2，Binder会将对象A重新转换生成一个新的对象。对象是不能跨进程直接传输的，对象的跨进程传输本质是序列化
 RemoteCallbackLsit是系统专门提供用于管理跨进程Listener的，其内部封装一个Map

### 6 ContentProvider

ContentProvider 同样可以实现IPC，或者换个说法。ContentProvider 本质就是通过AIDL实现的。只不过它的职责专一，就是为其他应用提供数据的

## Binder连接池

在前面说到AIDL的使用及原理的时候，我们可以看到在服务端只是创建了一个Binder然后返回给客户端使用而已。于是我们可以想到是不是我们可以只有一个Service,对于不同可客户端我们只是去返回一个不同的Binder即可，这样就避免了创建了大量的Service。在任玉刚的《android开发艺术探索》给出了一个Binder连接池的概念，很巧妙的避免了Service的多次创建。这个Binder连接池类似于设计模式中的工厂方法模式。为每一个客户端创建他们所需要的Binder对象。那么下面我们看一下它是如何实现的

首先我定义了多个AIDL，总的就是提供BookManager与StudentManager的功能。注意，这边还有个IBinderPool，这个的功能是提供用户选择具体需要的IBinder对象



![img](https:////upload-images.jianshu.io/upload_images/11156265-5854461603314a11?imageMogr2/auto-orient/strip%7CimageView2/2/w/310/format/webp)

这里写图片描述

服务端 BinderPoolService 中返回了binderPool对象，客户端可以通过该对象的queryBinder方法获取对应的服务

```
public class BinderPoolService extends Service {

    public class BookManagerImpl extends IBookManager.Stub{

        List<Book> bs = new CopyOnWriteArrayList<>();
        {
            bs.add(new Book(1,"android",33));
        }

        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public List<Book> getBooks() throws RemoteException {
            return bs;
        }
    }

    public class StudentManagerImpl extends IStudentManager.Stub{

        List<Student> ss = new CopyOnWriteArrayList<>();

        {
            ss.add(new Student(1,"lee",19));
        }

        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public List<Student> getStudents() throws RemoteException {
            return ss;
        }
    }

    IBookManager bookManager = new BookManagerImpl();
    IStudentManager studentManager = new StudentManagerImpl();

    IBinderPool binderPool = new IBinderPool.Stub() {
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public IBinder queryBinder(int binderCode){
            switch (binderCode){
                case 1:
                    return bookManager.asBinder();
                case 2:
                    return studentManager.asBinder();
            }
            return null;
        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        return binderPool.asBinder();
    }
}
```

客户端的定义了 BinderPool，相当于提供远程服务调用工具类。这边可以考虑使用CountDownLatch将远程服务的连接这个异步操作转为同步

```
public class BinderPool {

    private Context context;
    private CountDownLatch countDownLatch;
    private static BinderPool binderPool;
    private IBinderPool iBinderPool;

    private BinderPool(Context context) {
        this.context = context;
        // 连接远程线程池服务
        connectBinderPoolService();
    }

    public static BinderPool getInstance(Context context) {
        if (binderPool == null) {
            synchronized (String.class) {
                if (binderPool == null)
                    binderPool = new BinderPool(context);
            }
        }
        return binderPool;
    }

    private synchronized void connectBinderPoolService() {
        // countDownLatch = new CountDownLatch(1);
        Intent intent = new Intent("com.itszt.lwwp");
        context.bindService(ServiceIntentUtil.createExplicitFromImplicitIntent(context, intent),
                conn, Service.BIND_AUTO_CREATE);
//        try {
//            // countDownLatch.await();
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
    }

    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            iBinderPool = IBinderPool.Stub.asInterface(iBinder);
            // countDownLatch.countDown();
            if (null != iBinderPool) {
                try {
                    iBinderPool.asBinder().linkToDeath(deathRecipient, 0);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };

    private IBinder.DeathRecipient deathRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            if (iBinderPool != null) {
                iBinderPool.asBinder().unlinkToDeath(this, 0);
                iBinderPool = null;
                // 重连
                connectBinderPoolService();
            }
        }
    };

    public IBinder queryBinder(int binderCode) {
        try {
            if (iBinderPool != null)
                return iBinderPool.queryBinder(binderCode);
            return null;
        } catch (RemoteException e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

客户端通过BinderPool调起远程服务

```
    new Thread(){
            @Override
            public void run() {
                binderPool = BinderPool.getInstance(MainActivity.this);
            }
        }.start();

    public void onTest(View view) {
        if(binderPool == null) return;
        IBookManager bookManager = IBookManager.Stub.asInterface(binderPool.queryBinder(1));
        IStudentManager studentManager = IStudentManager.Stub.asInterface(binderPool.queryBinder(2));
        try {
            Toast.makeText(this, "bookManager.getBooks():" + bookManager.getBooks().toString(), Toast.LENGTH_SHORT).show();
            Toast.makeText(this, "studentManager.getBooks():" + studentManager.getStudents().toString(), Toast.LENGTH_SHORT).show();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
```

## 总结

* IPC是什么？是跨进程通信
* AIDL是什么？是IPC方式的一种
* IPC为什么会导致那么多问题？内存独立
* 我们在什么时候需要IPC？某功能需要运行在独立进行；提升应用分配内存；应用间交互

