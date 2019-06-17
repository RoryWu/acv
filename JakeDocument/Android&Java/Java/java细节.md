# java 查缺补漏

## 1.关于Char/String

### char 的相关知识点

### String 的创建方式



### String.intern 方法

​		String类中加入这个方法可能是为了提升一点点性能，因为从常量池取数据比从堆里面去数据要快一些。(个人感觉)

　　API上的那几句关于这个方法，其实总结一句就是调用这个方法之后把字符串对象加入常量池中，常量池我们都知道他是存在于方法区的，他是方法区的一部分，而方法区是线程共享的，所以常量池也就是线程共享的，但是他并不是线程不安全的，他其实是线程安全的，他仅仅是让有相同值的引用指向同一个位置而已，如果引用值变化了，但是常量池中没有新的值，那么就会新开辟一个常量结果来交给新的引用，而并非像线程不同步那样，针对同一个对象，new出来的字符串和直接赋值给变量的字符串存放的位置是不一样的，前者是在堆里面，而后者在常量池里面，另外，在做字符串拼接操作，也就是字符串相"+"的时候，得出的结果是存在在常量池或者堆里面，这个是根据情况不同不一定的，我写了几行代码测试了一下。

先上结果：

1. 直接定义字符串变量的时候赋值，如果表达式右边只有字符串常量，那么就是把变量存放在常量池里面。

2. new出来的字符串是存放在堆里面。

3. 对字符串进行拼接操作，也就是做"+"运算的时候，分2中情况：

    1. 表达式右边是纯字符串常量，那么存放在栈里面。

    2. 表达式右边如果存在字符串引用，也就是字符串对象的句柄，那么就存放在堆里面。

        ```java
        String str1 = "aaa";
                String str2 = "bbb";
                String str3 = "aaabbb";
                String str4 = str1 + str2;
                String str5 = "aaa" + "bbb";
                System.out.println(str3 == str4); // false
                System.out.println(str3 == str4.intern()); // true
                System.out.println(str3 == str5);// true
        ```

        

        结果：str1、str2、str3、str5都是存在于常量池，str4由于表达式右半边有引用类型，所以str4存在于堆内存，而str5表达式右边没有引用类型，是纯字符串常量，就存放在了常量池里面。其实Integer这种包装类型的-128  ~ +127也是存放在常量池里面，比如Integer i1 = 10;Integer i2 = 10; i1 ==  i2结果是true，估计也是为了性能优化。

## 2.关于反射

## 3.关于泛型

## 4.关于线程

### 常见问题:

#### 什么是线程安全

可变资源(内存)在线程共享,修改的问题

#### 如何实现线程安全

* 不共享资源
* 共享不可变资源
* 共享可变资源
    * 可见性
    * 操作原子性
    * 禁止重排序

#### ThreadLocal 是什么

线程内的不共享给别人的资源

本质是一个绑定到线程上的map

```java
//ThreadLocal.java
public void set(T value){
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if(map != null){
        map.set(this , value); // 这个this 就是把当前线程绑定
    }else
        createMap(t , value);
}
```

ThreadLocalMap 与 WeakHashMap 很像, 之间的关系





## 5. 关于线程安全

### AtomicReference 与 AtomicReferenceFileUpdater 的作用和区别 

AtomicReference 的作用





## 异步的优雅的代码

Rxjava / lamda

lamda 的问题:就是有异常的就会抛出　

 Rxjava 的异常处理

```java
RxjavaPlugins.setErrorHandler(e -> {
    report(e instanceOf OnErrorNotImplementedException ? e.getCause() : e);
    Exceptions.throwIfFatal(e);// 严重异常要抛出
})
```

Rxjava 的取消处理

```java
//当异步返回结果时,可能当前的Activity 已经销毁, 会造成NullPointerException
//处理方式
private List<Disposable> disposables = new ArrayList<>();
...
    button.setOnclickListener(v -> {
        Disposable disposable = sendRequest(...)
            .subscribeOn(Scheduler.io())
            .observeOn(AndroidScheduler.mainThread())
            .subscribe();
        disposables.add(disposable);
    })
...
    
protected void onDestory(){
    for(Disposable disposable: disposables){
        disposable.dispose();
    }
}
```



 ```java
// 第二种方法
as(AutoDispose.autoDisposable(View))
 ```





## CPU 架构的适配

### native 开发与适配:

**根据线上监控的反馈,来针对性的提供Native库**

**兼容模式, 要么给全套, 要么一个都不给, 否则会出现问题**

* 为了兼容所有的CPU架构, 通常在性能不明感的时候可以采用保留

    > armeabi

    ​	这一套Native库

* 而考虑到当前项目所对应的大多数设备可以,提升到

    > armeabi-v7a

* 同时可以参考微信

    arm64-v8a :

    ```scheme
    libs
    └armeabi-v7a
    	├libmath.so
    	├libmath_v8a.so
    	└libui.so
    ```

    

### 优化空间:

tts.so 很大, 采用云端加载的方式

关于动态加载so库:

https://blog.csdn.net/aqi00/article/details/72763742

​    

​    

​    

































