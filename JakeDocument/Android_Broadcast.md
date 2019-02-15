## Android_Broadcast 详解

- 作用与原理

    - ![broadcast](https://camo.githubusercontent.com/6d5fa4c7d2018338acd33def01441007559a33c0/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d366231636361323530653634653037612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

    

- 种类

    - 普通广播（Normal Broadcast）
        - 通过sendBroadcast进行发送，如果注册了Action匹配的接受者则会收到
        - 若发送广播有相应权限，那么广播接收者也需要相应权限
    - 系统广播（System Broadcast）
        - Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播
        - 每个广播都有特定的Intent - Filter（包括具体的action）
        - 系统广播由系统发送，不需要手动发送，只需要注册监听
    - 有序广播（Ordered Broadcast）
        - 通过sendOrderedBroadcast发送
        - 发送出去的广播被广播接收者按照先后顺序接收（有序是针对广播接收者而言的）
        - 广播接受者接收广播的顺序规则：Priority大的优先；动态注册的接收者优先
        - 先接收的可以对广播进行截断和修改
    - App应用内广播（本地广播、Local Broadcast）
        - 通过LocalBroadcastManager.getInstance(this).sendBroadcastSync();
        - App应用内广播可理解为一种局部广播，广播的发送者和接收者都同属于一个App
        - 相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 & 效率高（本地广播只会在APP内传播，安全性高；不允许其他APP对自己的APP发送广播，效率高）
    - 粘性广播（Sticky Broadcast）
        - 在Android5.0 & API 21中已经失效，所以不建议使用
        - 通过sendStickyBroadcast发送
        - 粘性广播在发送后就一直存在于系统的消息容器里面，等待对应的处理器去处理，如果暂时没有处理器处理这个广播则一直在消息容器里面处于等待状态
        - 粘性广播的Receiver如果被销毁，那么下次重新创建的时候会自动接收到消息数据

     

- 注意点与面试点

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