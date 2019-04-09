# Android App Performance Optimization

## 优化介绍

### 阶段

1. 项目初期, 主要目标吸引积累用户,增加功能

    *　只关心崩溃率,不采集性能数据, 
    *　没有性能检测以及优化方案,
    *　没有排查问题的手段

2. 项目的壮大期

    * 指标采集, 不够全面
    * 接入成熟APM , 排查手段单一 (APM, Application Performance Management)
    * 线下检测,优化,方案不成熟

3. 项目成熟期

    * 重点关注性能问题，　数据丰富，手段多样化
    * 线上，线下有一套完善的解决方案
    * 自检APM ,新产品可以快速接入

4. 常见问题:

    * 线上线下

        > 不重视线上, 以为线下测试没有问题, 线上也没有问题
        >
        > 线下预防,线上监控
        >
        > 线上线下的解决方式不用, 线下可以使用黑科技

    * 关于APM

        > 　个性化　需求
        >
        > 　数据安全

### APM平台

* 分类监控BUG:

    ​	BUGLY

* APM 平台:

    ​	听云平台

* 自建解决方案

    ​	美团,携程,360 等

### 常见问题:

> 　为什么要做性能优化

 * app 体验差，影响核心指标
 * 线上问题的追查比较困难
 * 降低性能优化带来的长期开销

> 介绍一下你们的性能平台

 * 交代背景(经历了一个过程, 不同阶段的处理不同的需求)
 * 具体讲解

> 你们为什么要自建APM

 * 需求层面(启动时间的阶段分级)
 * 效率层面()
 * 数据安全

## APP 启动优化

### 背景介绍

**原因**:

 * 第一体验
 * 八秒等待

### 启动分类

**APP STARTUP TIME:**

* 冷启动

    1. 耗时最多,也是衡量标准

    2. 流程:

        ```
        click event -> IPC -> Process.start > Activity Thread > bindApplication > LifeCycle > ViewRootImpl
        ```

* 热启动

    1. 最快

    2. 流程:

        ```
        后台-> 前台
        ```

* 温启动

    1. 较快

    2. 流程:

        ```
        LifeCycle
        ```

* 整体流程

    1. 启动之前:
        1. 启动app
        2. 加载空白window
        3. 创建进程
    2. 随后任务
        1. 创建application
        2. 启动主线程
        3. 创建 MainActivity
        4. 加载布局
        5. 布置屏幕
        6. 首帧绘制



**优化方向**

* Applicaiton 和Activity 生命周期的优化,是我们可控的



### 优化方法

* **查看启动信息**

    ```shell
    adb shell am start -W 包名
    
    this time : 最后一个Activity 启动耗时
    total time : 所有Activity启动耗时
    WaitTime : AMS 启动Activity 的总耗时
    
    线下方便,线上不能用
    
    关于启动时间打点:
    在Application 中的attchBaseContext 是系统的第一个回调函数
    因此在这个方法中记录开始时间
    
    误区: onwindowFocusChanged只是首帧时间
    正解: 真是数据展示,Feed第一条展示时间
    
    因此在adatper 的onbindViewholder 第一条数据展示
    hodler.linearLayout.getViewTreeObserver()
    .addOnDrawListener()// api 16
    .addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener(){
        public boolean onPreDraw(){
            holder.linearLayout.getViewTreeObserver().removeOnPreDrawObserver();
            LaunchTimer.endRecorder();
            return true;
        }
    })
    ```

* 优化工具

    * traceview

        1. 图形的形式战术执行时间,调用栈等

        2. 信息全面,包含所有的线程

        3. 使用方式

            ```java
            Debug.startMethodTracing("");
            Debug.stopMethodTracing();
            生成文件在sd卡:Android/data/packagename/files
            ```

        4. 优缺点:

            * 优点：可以通过自己埋点查找特定代码
            * 缺点：占用内存，运行缓慢，可能带偏方向

    * systrace

        1. 结合Android内核的数据,生成HTML报告

        2. API 18以上,推荐tracecompat

        3. 使用方式

            ```python
            python systrace.py -t 10 [category]
            https://command_options
                
            TraceCompat.beginSection("name")
            TraceCompat.stopSection
            ```

    * cputime 与 walltime区别 

        * walltime 是代码执行的时间
        * cputiime 是代码消耗cpu 的时间(重点指标)
        * 举例:锁冲突



### 优雅的获取方法的耗时:

手动埋点:SystemClock.currentThreadTImeMillis()





## 内存优化

### 工具

* **Memory Profiler**
* **Memory Analyzer**
* **LeakCanary**

### 内存管理机制

java **内存管理**



java **内存分配**

- 方法区
- 虚拟机栈
- 本地方法栈
- 堆
- 程序计数器

java 内存回收

* 标记清除算法

    特点:

    对每一块内存进行标记，标记出需要清楚的部分　（效率较低）

* 

## 