# Android_Handler_Looper

关于 Handler、Looper、MessageQueue，我想大家都了解的差不多了，简单来说就是一个 Handler 对应一个 Looper，一个 Looper 对应一个 Message。那么再想个问题，`一个 Handler 可以对应多个 Looper 吗？` `一个 Looper 可以对应多个 Handler 吗？`



之所以会提出上面这个问题，主要是因为在看 Looper 的源码时，发现了其中的 `getMainLooper` 这个方法，从名字可以看出是获取主线程的 Looper，那么为什么要特别提供这个方法呢？首先看一下这个方法的源码，很简单：

```
/*
 * 返回应用主线程中的 Looper
 */
public static Looper getMainLooper() {
    synchronized (Looper.class) {
        return sMainLooper;
    }
}
```

其实我们平时最常用的是无构造参数的 Handler，其实 Handler 还有构造参数的构造方法，如下：

```
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

在此注意构造函数中第一个参数是 Looper 就可以了，那么也就是说，我们可以传递一个已有的 Looper 来创建 Handler。这里先不写示例代码了，填个坑，以后有时间再写，大概是下面这样：

```
Handler handler = new Handler(Looper.getMainLooper()){
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
    }
};
```

注意上面的 Looper.getMainLooper()，将主线程中的 Looper 扔进去了，也就是说 handleMessage 会运行在主线程中，那么这样有什么用呢？这样可以在主线程中更新 UI 而不用把 Handler 定义在主线程中。

当然刚才提到的作用只是对应于主线程中的 sMainLooper 了，其实各种 Looper 都可以往 Handler 的构造方法这里扔，从而使得 handleMessage 运行在你想要的线程中，进而实现线程间通信。

那么想到另一篇文章 [HandlerThread源码解析](http://icodeyou.com/2015/10/11/2015-10-11-HandlerThread%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/) 中 HandlerThread#getLooper() 的作用了吗？

文章开头也提到了一个问题，那么答案就应该是：一个 Handler 中只能有一个 Looper，而一个 Looper 则可以对应多个 Handler，只要把 Looper 往 Handler 的构造方法里扔扔扔就好了。

今天再看了看 AsyncTask 的源码，发现其中也用到了 getMainLooper()，来更新 UI，源码如下：

```
private static class InternalHandler extends Handler {
        public InternalHandler() {
            // 使用主线程的 Looper 扔给 Handler
            super(Looper.getMainLooper());
        }
}
```