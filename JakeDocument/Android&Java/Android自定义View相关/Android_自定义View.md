# ACV

## 说明

## 一、 控件组合

​	自定义控件组合，是一种最基础的自定义UI方式，把现有的一些控件组装成一个控件，在新的layout中整体使用。



**以下是自定义控件组合的步骤：**

 1. 将自定义的View 继承于一个ViewGroup (RelativeLayout , LinearLayout 等)

 2. a. 编写控件组合的layout

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout
        xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
        android:layout_height="188px">
        
        <com.aitek.headsetdemo.view.notify.ScrollGroup
            android:layout_width="match_parent"
            android:layout_height="188px">
    
            <include
                android:layout_width="match_parent"
                android:layout_height="188px"
                layout="@layout/notify_control_btn_page_top" />
    
            <include
                android:layout_width="match_parent"
                android:layout_height="188px"
                layout="@layout/notify_control_btn_page_bottom" />
    
        </com.aitek.headsetdemo.view.notify.ScrollGroup>
    
    </RelativeLayout>
    ```

    b. 编写细节layout(notify_control_btn_page_top.xml), 每个界面里面有两个ToggleButton，组成一个整体的layout

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:background="@color/colorBlackBg"                                     	 android:descendantFocusability="afterDescendants"
        android:layout_width="240px"
        android:layout_height="@dimen/notify_content_size">
    
        <ToggleButton
            android:id="@+id/button_wifi"
            android:layout_width="@dimen/notify_button_size"
            android:layout_height="@dimen/notify_button_size"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    
        <ToggleButton
            android:id="@+id/button_airplane"
            android:layout_width="@dimen/notify_button_size"
            android:layout_height="@dimen/notify_button_size"
            app:layout_constraintStart_toEndOf="@+id/button_wifi"
            app:layout_constraintTop_toTopOf="parent"
            tools:ignore="MissingConstraints" />
    </android.support.constraint.ConstraintLayout>
     
    
    ```

 3. 在构造函数中初始化layout 的相关信息：

    ```java
     public NotifyControlView(Context context, AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
            this.mContext = context;
            initViews();
        }
    
        public void initViews() {
            selfView = LayoutInflater.from(mContext).inflate(R.layout.notify_control_layout, this, true);
        }
    
    ```

	4. 在onFinishInflate 函数中，初始化layout 中的所有组件

    ```java
    protected void onFinishInflate() {
            super.onFinishInflate();
            // 初始化组件
            mWifiBtn = selfView.findViewById(R.id.button_wifi);
    }
    ```

	5. 接下来就可以把一个控件组合整体来使用

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <com.aitek.headsetdemo.view.notify.NotifyControlView xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="@dimen/notify_content_size">
    
    </com.aitek.headsetdemo.view.notify.NotifyControlView>
    ```


## 二、 继承现有的组件

​	有些时候，我们会对现有的Android组件感到不满意，但只需要进行微小的调整，这时就需要第二种自定义控件，继承想要修改的组件，然后进行部分修改。



















## 三、 继承 ViewGroup

​	通常情况下我们需要一个整体的容器来安放我们的组件，与此同时，我们也需要这个容器能够实现一些特定的效果，比如页面的整体滑动，特殊的滑动动画，等等，这是我们需要实现自己的ViewGroup



示例代码：

```java
public class ScrollGroup extends ViewGroup {
    
    private int topBorder;
    private int bottomBorder;
    
    private Scroller scroller;
    private int scrollY;
    private int downY;
    private int firstY;
    private int lastY;
    private int mTouchSlop;

    public ScrollGroup(Context context) {
        super(context);
    }

    public ScrollGroup(Context context, AttributeSet attrs) {
        super(context, attrs);
        scroller = new Scroller(context);
        ViewConfiguration configuration = ViewConfiguration.get(context);
        mTouchSlop = ViewConfigurationCompat.getScaledPagingTouchSlop(configuration);
    }

    public ScrollGroup(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int size = getChildCount();
        for (int i = 0; i < size; i++) {
            View childView = getChildAt(i);
            measureChild(childView, widthMeasureSpec, heightMeasureSpec);
        }
        setMeasuredDimension(MeasureSpec.getSize(widthMeasureSpec), MeasureSpec.getSize(heightMeasureSpec));
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {

        for (int i = 0; i < getChildCount(); i++) {
            View childView = getChildAt(i);
            childView.layout(0, i * childView.getMeasuredHeight(), childView.getMeasuredWidth(), (i + 1) * childView.getMeasuredHeight());
        }
        topBorder = getChildAt(0).getTop();
        bottomBorder = getChildAt(getChildCount() - 1).getBottom();
        Log.d("jake", getMeasuredWidth() + "父布局");
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downY = (int) ev.getRawY();
                firstY = downY;
                break;
            case MotionEvent.ACTION_MOVE:
                lastY = (int) ev.getRawY();
                int moveY = Math.abs(firstY - lastY);
                if (moveY > mTouchSlop) {
                    return true;
                }
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_MOVE:
                lastY = (int) event.getRawY();
                scrollY = firstY - lastY;
                if (getScrollY() + scrollY < topBorder) {
                    scrollTo(0, topBorder);
                } else if (getScrollY() + getHeight() + scrollY > bottomBorder) {
                    scrollTo(0, bottomBorder - getHeight());
                } else {
                    scrollBy(0, scrollY);
                }
                firstY = lastY;
                break;
            case MotionEvent.ACTION_UP:
                int treasureY = (getScrollY() + getHeight() / 2) / getHeight();
                int dy = treasureY * getHeight() - getScrollY();
                scroller.startScroll(0, getScrollY(), 0, dy);
                invalidate();
                break;
        }
        //        return super.onTouchEvent(event);
        return true;
    }

    @Override
    public void computeScroll() {
        if (scroller.computeScrollOffset()) {
            scrollTo(scroller.getCurrX(), scroller.getCurrY());
            invalidate();
        }
    }

}
```



## 四、 继承 View

继承View 主要要注意两方面

onMeasure

onDraw



## 五、 具体细节

自定义View 的

​	

## 附录一 速查表

| 操作分类     | 相关API                                                      | 备注                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 绘制颜色     | drawColor, drawRGB, drawARGB                                 | 使用单一颜色填充整个画布                                     |
| 绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧                |
| 绘制图片     | drawBitmap, drawPicture                                      | 绘制位图和图片                                               |
| 绘制文本     | drawText,    drawPosText, drawTextOnPath                     | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字 |
| 绘制路径     | drawPath                                                     | 绘制路径，绘制贝塞尔曲线时也需要用到该函数                   |
| 顶点操作     | drawVertices, drawBitmapMesh                                 | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |
| 画布剪裁     | clipPath,    clipRect                                        | 设置画布的显示区域                                           |
| 画布快照     | save, restore, saveLayerXxx, restoreToCount, getSaveCount    | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 会滚到指定状态、 获取保存次数 |
| 画布变换     | translate, scale, rotate, skew                               | 依次为 位移、缩放、 旋转、错切                               |
| Matrix(矩阵) | getMatrix, setMatrix, concat                                 | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。 |

​      



## 附录二 View 的生命周期

要理解View 的生命周期：

**首先在View 的各个生命周期打印Log**

代码如下:

```java

public class LifeCycleView extends View {
	private static final String TAG = "AigeStudio:LifeCycleView";
 
	public LifeCycleView(Context context) {
		super(context);
		Log.d(TAG, "Construction with single parameter");
	}
 
	public LifeCycleView(Context context, AttributeSet attrs) {
		super(context, attrs);
		Log.d(TAG, "Construction with two parameters");
	}
 
	@Override
	protected void onFinishInflate() {
		super.onFinishInflate();
		Log.d(TAG, "onFinishInflate");
	}
 
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		super.onMeasure(widthMeasureSpec, heightMeasureSpec);
		Log.d(TAG, "onMeasure");
	}
 
	@Override
	protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
		super.onLayout(changed, left, top, right, bottom);
		Log.d(TAG, "onLayout");
	}
 
	@Override
	protected void onSizeChanged(int w, int h, int oldw, int oldh) {
		super.onSizeChanged(w, h, oldw, oldh);
		Log.d(TAG, "onSizeChanged");
	}
 
	@Override
	protected void onDraw(Canvas canvas) {
		super.onDraw(canvas);
		Log.d(TAG, "onDraw");
	}
 
	@Override
	protected void onAttachedToWindow() {
		super.onAttachedToWindow();
		Log.d(TAG, "onAttachedToWindow");
	}
 
	@Override
	protected void onDetachedFromWindow() {
		super.onDetachedFromWindow();
		Log.d(TAG, "onDetachedFromWindow");
	}
 
	@Override
	protected void onWindowVisibilityChanged(int visibility) {
		super.onWindowVisibilityChanged(visibility);
		Log.d(TAG, "onWindowVisibilityChanged");
	}
}

```

打印结果：

![first](./acv/acv1)

### 理解各个生命周期函数

构造函数后，调用了onFinishInflate方法，这个方法当xml布局中我们的View被解析完成后则会调用，具体的实现在LayoutInflater的rInflate方法中：

```java

public abstract class LayoutInflater {
	void rInflate(XmlPullParser parser, View parent, final AttributeSet attrs,
            boolean finishInflate) throws XmlPullParserException, IOException {
        // 省去无数代码…………
 
        if (finishInflate) parent.onFinishInflate();
    }
}
```

也就是说如果我们不从xml布局文件中解析的话，该方法就不会被调用，我们在Activity直接加载View的实例：

```java

public class MainActivity extends Activity {
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(new LifeCycleView(this));
	}
}
```

这时候的log 没有执行onFinishflate 方法。



继续来看，紧接着调用的是onAttachedToWindow方法，此时表示我们的View已被创建并添加到了窗口Window中，

该方法后紧接着一般会调用onWindowVisibilityChanged方法，只要我们当前的Window窗口中View的可见状态发生改变都会被触发，这时View是被显示了，

随后就会开始调用onMeasure方法对View进行测量，

如果测量结果被确定则会先调用onSizeChanged方法通知View尺寸大小发生了改变，

紧跟着便会调用onLayout方法对子元素进行定位布局，

然后再次调用onMeasure方法对View进行二次测量，

如果测量值与上一次相同则不再调用onSizeChanged方法，

接着再次调用onLayout方法，如果测量过程结束，则会调用onDraw方法绘制View。

我们看到，onMeasure和onLayout方法被调用了两次，很多童鞋会很纠结为何onMeasure方法回被多次调用，其实没必要过于纠结这个问题，onMeasure的调用取决于控件的父容器以及View Tree的结构，不同的父容器有不同的测量逻辑，比如上一节自定义控件其实很简单2/3中，我们在SquareLayout测量子元素时就采取了二次测量，在API 19的时候Android对测量逻辑做了进一步的优化，比如在19之前只会对最后一次的测量结果进行Cache而在19开始则会对每一次测量结果都进行Cache，如果相同的代码相同布局相同的逻辑在19和19之前你有可能会看到不一样的测量次数结果，所以没必要去纠结这个问题，一般情况下只要你逻辑正确onMeasure都会得到正确的调用。

---------------------


onFinishInflate -> onAttchedToWindow -> onWindowVisibilityChanged -> onMeasure -> onSizeChanged -> onLayout -> onMeasure -> onLayout -> onDraw



参考文档：

https://www.jianshu.com/p/d507e3514b65