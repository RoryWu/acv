# Android_Activity 详解

#### Activity

- **理解Activity**

    activity是独立平等的，用来处理用户操作。几乎所有的activity都是用来和用户交互的，所以activity类会创建了一个窗口，开发者可以通过setContentView(View)的接口把UI放到给窗口上。



- **重要方法**

    1. onConfigurationChanged

        前面说过，当 Activity 横竖屏切换的时候会导致 Activity 销毁并重建，哪有什么方法能避免呢？其实可以在  AndroidManifest 里面指定 android:configChanges="orientation/screenSize"  来避免重建，这时就会调用 onConfigurationChanged 方法。

        如果按上面的配置，当字体发生变化时，也会销毁重建，但是不会回调 onConfigurationChanged 方法，所以说想要监听的变化必须要包含之内。

        ```java
        @Override
            public void onConfigurationChanged(Configuration newConfig) {
                super.onConfigurationChanged(newConfig);
                if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT) {
                    //竖屏
                } else {
                    //横屏
                }
            }
        ```

        

    2. onTrimMemory

        当内存紧张时会回调，它在 onStop 回调之前。指导应用程序在不同的情况下进行自身的内存释放，以避免被系统直接杀掉，提高应用程序的用户体验。它和 onLowMemory 相比，它有一个 level 评级，onLowMemory 能兼容更低的版本。

        ```java
        @Override
        public void onTrimMemory(int level) {
        	super.onTrimMemory(level);
            if (level == TRIM_MEMORY_UI_HIDDEN) {
        
            }
        }
        ```

        

    

- **Activity 的生命周期**

    正常状态:

     onCreate()—>onStart()—>onResume()-> onPause()—>onStop()

    后台返回前台:

     onRestart()—>**onStart()**—>onResume()

    锁屏: 

    onPause()->onStop()

    解锁:

    onStart()->onResume()



- **Activity 的启动模式 任务栈**

    使用`android:launchMode="standard|singleInstance|singleTask|singleTop"`来控制Acivity任务栈。

    **任务栈**是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其`onDestory()`方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名.

    1. standard : 标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(*onCreate()->onStart()->onResume()*)都会执行。

    2. singleTop : 栈顶复用模式.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的`onNewIntent()`方法会被回调.如果Activity已经存在但是不在栈顶,那么作用与*standard模式*一样.

    3. singleTask: 栈内复用模式.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么这个Activity就会被调到栈顶,`onNewIntent()`,并且singleTask会清理在当前Activity上面的所有Activity.(clear top)

    4. singleInstance : 加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了

         

    Activity的堆栈管理以ActivityRecord为单位,所有的ActivityRecord都放在一个List里面.可以认为一个ActivityRecord就是一个Activity栈

    

- **Activity 的数据保存 和 恢复**

    - onSaveInstanceState

        在activity　可能被回收之前调用,用来保存自己的状态和信息，以便回收后重建时恢复数据（在onCreate()或onRestoreInstanceState()中恢复）。旋转屏幕重建activity会调用该方法，但其他情况在onpause()和onStop()状态的activity不一定会调用, 官方说明:

        > One example of when onPause and onStop is called and not this method is 
        > when a user navigates back from activity B to activity A: there is no 
        > need to call onSaveInstanceState on B because that particular instance 
        > will never be restored, so the system avoids calling it. An example when
        > onPause is called and not onSaveInstanceState is when activity B is 
        > launched in front of activity A: the system may avoid calling 
        > onSaveInstanceState on activity A if it isn't killed during the lifetime
        > of B since the state of the user interface of A will stay intact.

        也就是说，系统灵活的来决定调不调用该方法，**但是如果要调用就一定发生在onStop方法之前，但并不保证发生在onPause的前面还是后面。**

    - onRestoreInstanceState

        这个方法在onStart 和 onPostCreate之间调用，在onCreate中也可以状态恢复，但有时候需要所有布局初始化完成后再恢复状态。

        onPostCreate：一般不实现这个方法，当程序的代码开始运行时，它调用系统做最后的初始化工作。

    - 使用

        ```java
        public class MainActivity extends Activity {
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
        	if(savedInstanceState!=null){ //判断是否有以前的保存状态信息
        		 savedInstanceState.get("Key"); 
        		 }
        	super.onCreate(savedInstanceState);
        	setContentView(R.layout.activity_main);
        }
         @Override
        protected void onSaveInstanceState(Bundle outState) {
        // TODO Auto-generated method stub
         //可能被回收内存前保存状态和信息，
           Bundle data = new Bundle(); 
           data.putString("key", "last words before be kill");
           outState.putAll(data);
        super.onSaveInstanceState(outState);
        }
         @Override
        protected void onRestoreInstanceState(Bundle savedInstanceState) {
        // TODO Auto-generated method stub
           if(savedInstanceState!=null){ //判断是否有以前的保存状态信息
        		 savedInstanceState.get("Key"); 
        		 }
        super.onRestoreInstanceState(savedInstanceState);
        }
        }
        ```

    



- **Activity 在manifest 标签中的一些注意点**

    - taskAffinity

        在 singleTask 启动模式中，多次提到某个 Activity 所需的任务栈，什么是 Activity  所需要的任务栈呢？这就要从一个参数说起：taskAffinity，任务相关性。这个参数标识了一个 Activity  所需要的任务栈的名字，默认情况下，所有 Activity 所需的任务栈的名字为应用的包名。当然，我们可以为每个 Activity 都单独指定  taskAffinity 属性，这个属性值必须不能和包名相同，否则相当于没有设置。taskAffinity 属性主要和 singleTask  启动模式和 allowTaskReparentiong 属性配对使用，在其他情况下没有意义。

        **taskAffinity 与 singleTask 配对使用：**

        如果启动了设置了这两个属性的 Activity，这个 Activity 就会在 taskAffinity 设置的任务栈中。

        **taskAffinity 与 allowTaskReparenting 配对使用：**

        当一个应用 A 启动了应用 B 的某个 Activity 后，如果这个 Activity 的 allowTaskReparenting  属性为 true 的话，那么当应用 B 被启动后，此 Activity 会直接从应用 A 的任务栈转移到应用 B  的任务栈中。这个属性主要作用就是将这个 Activity  转移到它所属的任务栈中，例如一个短信应用收到一个带有网络链接的短信，点击链接会跳到浏览器，这时候如果 allowTaskReparenting  设置为 true 的话，打开浏览器应用就会直接显示刚才打开的网页页面，而打开短信应用后这个浏览器界面就会消失。

        启动模式了解之后，那是如何指定启动模式的方式呢？

        有两种：一种是在 AndroidMenifet 文件设置 launchMode 属性，一种是给 Intent 设置 Flag。

        如果两者都存在，后者优先级更高。

        

- **Scheme跳转协议**

    ##### 概念

    Android中的scheme是一种页面内跳转协议，是一种非常好的实现机制，通过定义自己的scheme协议，可以非常方便跳转app中的各个页面；通过scheme协议，服务器可以定制化告诉app跳转哪个页面，可以通过通知栏消息定制化跳转页面，可以通过H5页面跳转页面等。

    ##### 应用场景

    - 通过服务器下发跳转路径跳转相应页面
    - 通过在H5页面的锚点跳转相应的页面
    - 根据服务器下发通知栏消息，App跳转相应的页面（包括另外一个APP的页面，作为推广使用）

