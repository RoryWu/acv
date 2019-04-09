# 卡顿优化

### 相关概念



## 自动检测方案优化



### ＡＮＲ分析与实战

**分类:**

* keyDispatchTimeout , 5s
* BroadcastTimeout , 前台10s , 后台60s
* ServiceTimeout , 前台20s , 后台100s

**解决:**

* adb pull /data/anr/traces.txt　存储路径
* 详细分析: CPU,IO,锁

**线上ANR监控方案:**

* 通过FileOberver监控文件变化,高版本权限问题

* ANR-WatchDog

    1. ANR-WatchDog 导入 : https://github.com/SalomonBrys/ANR-WatchDog

    2. 添加代码:

        ### With Gradle / Android Studio

        1. In the `app/build.gradle` file, add:

            ```
            compile 'com.github.anrwatchdog:anrwatchdog:1.4.0'
            ```

        2. In your application class, in `onCreate`, add:

            ```
            new ANRWatchDog().start();
            ```

    3. 原理:

        1. 继承与Thread

        2. 在run 中, 每个一段时间post 一个runnable , runnable 会将一个变量进行+1 操作, 然后实时判断这个值有没有变化, 如果在规定时间内没有变化则说明发生了ANR

        3. ANRError 会通过

            ```java
            Thread mainThread = Looper.getMainLooper().getThread()
                
            ```

            拿到主线程 ,然后获取其堆栈,throw 出来,然后可以通过ANRErrorListener 监听该消息,上传到服务器

            

        4. 区别

          * AndroidPerformaceMonitor : 监控Msg , 会在每一个msg 前后有log

          * ANR-WatchDog : 看最终结果

          * 前者适用与卡顿, 后者适合ANR



**卡顿单点问题的监控:**

IPC问题的监控:

常规方案:

* IPC前后加埋点
* 不优雅，容易忘记
* 维护成本大

检测技巧：

* adb 命令

    * adb shell am trace-ipc start
    * adb shell am trace-ipc stop --dump-file /data/local/tmp/ipc-trace.txt
    * adb pull  /data/local/tmp/ipc-trace.txt

* 优雅方案：

    * ARTHOOK 还是AspectJ
    * ARTHook : 可以Hook 系统方法
    * AspectJ : 非系统方法

* 监控维度:

    * IPC
    * IO , DB 
    * View 的绘制

    
