## Android Fragment 详解

#### Fragment

##### 1. 什么是Fragment

Fragment，俗称碎片，自Android 3.0开始被引进并大量使用。作为Activity界面的一部分，Fragment的存在必须依附于Activity，并且与Activity一样，拥有自己的生命周期，同时处理用户的交互动作。同一个Activity可以有一个或多个Fragment作为界面内容，并且可以动态添加、删除Fragment，灵活控制UI内容，也可以用来解决部分屏幕适配问题。

##### 2. Fragment为什么被称为第五大组件

首先Fragment的使用次数是不输于其他四大组件的，而且Fragment有自己的生命周期，比Activity更加节省内存，切换模式也更加舒适，使用频率不低于四大组件。

##### 3. Fragment的生命周期

[![Fragment的生命周期](https://camo.githubusercontent.com/a81ee173f13d946f52256bd06dc7d4af5ff75e9a/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d626239363061356663653236336133662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/a81ee173f13d946f52256bd06dc7d4af5ff75e9a/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d626239363061356663653236336133662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

[![Fragment的生命周期](https://camo.githubusercontent.com/17ad5681745129f01c8c9a9723a40987c33089e5/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d396361363134666531643934313662302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/17ad5681745129f01c8c9a9723a40987c33089e5/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d396361363134666531643934313662302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

##### 4. Fragment创建/加载到Activity的两种方式

- 静态加载

    1. 创建Fragment的xml布局文件
    2. 在Fragment的onCreateView中inflate布局，返回
    3. 在Activity的布局文件中的适当位置添加fragment标签，指定name为Fragment的完整类名（这时候Activity中可以直接通过findViewById找到Fragment中的控件）

- 动态加载（需要用到事务操作，常用）

    1. 创建Fragment的xml布局文件
    2. 在Fragment的onCreateView中inflate布局，返回

    ```java
           @Nullable
           @Override
           public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
               return inflater.inflate(R.layout.activity_main, container, false);
           }
    ```

    1. 初始化Fragment
    2. 进行add()/remove()/replace()/attach()/detach()/hide()/addToBackStack()事务操作（都是对Fragment的栈进行操作，其中add()指定的tag参数可以方便以后通过findFragmentByTag()找到这个Fragment）
    3. 提交事务：commit()
        示例代码：

    ```java
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          getSupportFragmentManager().beginTransaction()
                  .add(R.id.fragment_container, new TestFragment(), "test")
                  .commit();
          TestFragment f = (TestFragment) getSupportFragmentManager().findFragmentByTag("test");
      }
    ```

##### 5. Fragment通信问题

1. 通过findFragmentByTag或者getActivity获得对方的引用（强转）之后，再相互调用对方的public方法。

    优点：简单粗暴 缺点：引入了“强转”的丑陋代码，另外两个类之间各自持有对方的强引用，耦合较大，容易造成内存泄漏

2. 通过Bundle的方法进行传值，在添加Fragment的时候进行通信

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Fragment fragment = new TestFragment();
        Bundle bundle = new Bundle();
        bundle.putString("key", "value");
        //Activity中对fragment设置一些参数
        fragment.setArguments(bundle);
        getSupportFragmentManager().beginTransaction()
                .add(R.id.fragment_container, fragment, "test")
                .commit();
    }
    ```

    优点：简单粗暴 缺点：只能在Fragment添加到Activity的时候才能使用，属于单向通信

3. 利用eventbus进行通信

    优点：实时性高，双向通信，Activity与Fragment之间可以完全解耦 缺点：反射影响性能，无法获取返回数据，EventBUS难以维护

4. 利用接口回调进行通信（Google官方推荐）

    ```java
       //MainActivity实现MainFragment开放的接口
       public class MainActivity extends FragmentActivity implements FragmentListener {
           @override
           public void toH5Page() {
               //...其他处理代码省略
           }
       }
       //Fragment的实现
       public class MainFragment extends Fragment {
           //接口的实例，在onAttach Activity的时候进行设置
           public FragmentListener mListener;
           //MainFragment开放的接口
           public static interface FragmentListener {
               //跳到h5页面
               void toH5Page();
           }
           @Override
           public void onAttach(Activity activity) {
               super.onAttach(activity);
               //对传递进来的Activity进行接口转换
               if (activity instance FragmentListener){
                   mListener = ((FragmentListener) activity);
               }
           }
           // ...其他处理代码省略
       }
    ```

    优点：既能达到复用，又能达到很好的可维护性，并且性能得到保证 

    缺点：假如项目很大了，Activity与Fragment的数量也会增加，这时候为每对Activity与Fragment交互定义交互接口就是一个很麻烦的问题（包括为接口的命名，新定义的接口相应的Activity还得实现，相应的Fragment还得进行强制转换）

5. 通过Handler进行通信（其实就是把接口的方式改为Handler）

    优点：既能达到复用，又能达到很好的可维护性，并且性能得到保证 缺点：Fragment对具体的Activity存在耦合，不利于Fragment复用和维护，没法获取Activity的返回数据

6. 通过广播/本地广播进行通信

    优点：简单粗暴 缺点：大材小用，存在性能损耗，传播数据必须实现序列化接口

7. 父子Fragment之间通信，可以使用getParentFragment()/getChildFragmentManager()的方式进行

##### 6. FragmentPageAdapter和FragmentPageStateAdapter的区别

- FragmentPageAdapter在每次切换页面的时候，是将Fragment进行分离，适合页面较少的Fragment使用以保存一些内存，对系统内存不会多大影响

    ```java
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if (DEBUG) Log.v(TAG, "Detaching item #" + getItemId(position) + ": f=" + object
                + " v=" + ((Fragment)object).getView());
        //FragmentPageAdapter在destroyItem的时候调用detach
        mCurTransaction.detach((Fragment)object);
    }
    ```

- FragmentPageStateAdapter在每次切换页面的时候，是将Fragment进行回收，适合页面较多的Fragment使用，这样就不会消耗更多的内存

    ```java
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        Fragment fragment = (Fragment) object;
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if (DEBUG) Log.v(TAG, "Removing item #" + position + ": f=" + object
                + " v=" + ((Fragment)object).getView());
        while (mSavedState.size() <= position) {
            mSavedState.add(null);
        }
        mSavedState.set(position, fragment.isAdded()
                ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
        mFragments.set(position, null);
        //FragmentPageStateAdapter在destroyItem的时候调用remove
        mCurTransaction.remove(fragment);
    }
    ```

##### 7. 参考文章

[Android：Activity与Fragment通信(99%)完美解决方案](https://www.jianshu.com/p/1b824e26105b)

