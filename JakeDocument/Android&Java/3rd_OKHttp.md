# OKHttp

## 原理和流程

## 拦截器

1. 为什么要用拦截器:

    一般的request 请求 到response 没有缓存, 没有重定向, 没有特殊处理机制, 没有连接复用

2. 拦截器的模式

    责任链-> 链条上每一个拦截interceptor 只处理自己负责的部分

    ```java
     Request request = chain.request();
    　Response response = chain.proceed(request);
    ```

    处理完成后

    ```java
    return response;
    ```

    

3. 使用场景：

    什么情况下，我们需要考虑实现自定义的拦截器呢？

    * **缓存拦截器：**

        https://blog.csdn.net/gengqiquan/article/details/52200638

        ```java
        /* 
           但我们又想用缓存，这个时候怎么办呢。，得益于okhttp的Interceptor机制，一切的配置都可以变得那么简单优雅。 
           我们可以在拦截器里截获headers然后移除默认的Cache-Control
        
           但是我们知道有些API返回的数据适合缓存，而有些是不适合的，比如资讯列表，各种更新频率比较高的，是不可以缓存的，而像资讯详情这种数据是可以缓存的。所以我们不能直接统一写死。需要动态配置。
        
           同样的，我们也在header里面作文章，自定义一个header。注意这个header一定不能被其他地方使用，不然会被覆盖值。这里我们定义的header的key名字为：Cache-Time。我们在拦截器里去取这个header。如果取得了不为空的值，说明这个请求是要支持缓存的，缓存的时间就是Cache-Time对应的值。我们把他添加进去。 
        */
        
        ```

        自定义缓存Interceptor

        ```java
        public class CacheInterceptor implements Interceptor {
            @Override
            public Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                Response response = chain.proceed(request);
                String cache = request.header("Cache-Time");
                //缓存时间不为空
                if (!Util.checkNULL(cache)) {
                    Response response1 = response.newBuilder()
                            .removeHeader("Pragma")
                            .removeHeader("Cache-Control")
                            //cache for cache seconds
                            .header("Cache-Control", "max-age="+cache)
                            .build();
                    return response1;
                } else {
                    return response;
                }
            }
        }
        
        ```

        缓存拦截器定义好了，我们还需要配置缓存的路径。这里我们定义一个缓存的内容提供器

        ```java
        public class CacheProvide {
            Context mContext;
        
            public CacheProvide(Context context) {
                mContext = context;
            }
        
            public Cache provideCache() {//使用应用缓存文件路径，缓存大小为10MB
                return new Cache(mContext.getCacheDir(), 10240 * 1024);
            }
        }
        
        ```

        通过上面的代码我们可以看到我们指定了缓存的大小为10MB。这里如果缓存的数据量大于这个值，内部会使用lur规则进行删除。

        ```java
        OkHttpClient client = new OkHttpClient.Builder()
                            .addNetworkInterceptor(new CacheInterceptor())//缓存拦截器
                            .cache(new CacheProvide(mAppliactionContext).provideCache())//缓存空间提供器
                            .connectTimeout(8, TimeUnit.SECONDS)
                            .readTimeout(5, TimeUnit.SECONDS)
                            .writeTimeout(5, TimeUnit.SECONDS)
                            .build();
        ```

        

        

    * **日志拦截器**

        ```java
        public class LogInterceptor implements Interceptor {
            @Override
            public Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                Log.e("LogInterceptor", "request:" + request);
                Log.e("LogInterceptor", "System.nanoTime():" + System.nanoTime());
                Response response = chain.proceed(request);
                Log.e("LogInterceptor", "request:" + request);
                Log.e("LogInterceptor", "System.nanoTime():" + System.nanoTime());
                return response;
            }
        }
        ```

        

    * **请求头拦截器：**

        ```java
        //假设现在后台要求我们在请求 API 接口时，都在每一个接口的请求头上添加对应的 token 。使用 	Retrofit 比较多的同学肯定会条件反射出以下代码：
        
         @FormUrlEncoded
         @POST("/mobile/login.htm")
         Call<ResponseBody> login(@Header("token") String token, @Field("mobile") String phoneNumber, @Field("smsCode") String smsCode);
        
         这样的写法自然可以，无非就是每次调用 login API 接口时都把 token 传进去而已。
        ```

        

    

    

    

    

    

    



## 使用

