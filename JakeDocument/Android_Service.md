## Android_Service 详解



- 生命周期

      

- **startService 和 bindService的区别:** 

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



- **onStartCommand 不同返回值 的作用是什么?**

    - START_STICKY  在运行onStartCommand后service进程被kill后，那将保留在开始状态，但是不保留那些传入的intent。不久后service就会再次尝试重新创建，因为保留在开始状态，在创建      service后将保证调用onstartCommand。如果没有传递任何开始命令给service，那将获取到null的intent。
    - START_NOT_STICKY  在运行onStartCommand后service进程被kill后，并且没有新的intent传递给它。Service将移出开始状态，并且直到新的明显的方法（startService）调用才重新创建。因为如果没有传递任何未决定的intent那么service是不会启动，也就是期间onstartCommand不会接收到任何null的intent。
    - START_REDELIVER_INTENT  在运行onStartCommand后service进程被kill后，系统将会再次启动service，并传入最后一个intent给onstartCommand。直到调用stopSelf(int)才停止传递intent。如果在被kill后还有未处理好的intent，那被kill后服务还是会自动启动。因此onstartCommand不会接收到任何null的intent。

    

- **使用startForgroundService 的方法**

    1. 申请 FOREGROUND_SERVICE 权限，它是普通权限
    2. 在 onStartCommand 中必须要调用 startForeground 构造一个通知栏，不然 ANR
    3. 前台服务只能是启动服务，不能是绑定服务

    

- **bug**

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

