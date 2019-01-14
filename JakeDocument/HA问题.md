# HA项目总结

## 1. 项目简介

基于AISPEECH 公司的语音技术，进行开发的类似于SIRI的手机语音助手项目；

我们基于目前的助手产品，为多家手机厂商提供了手机的内置语音助手项目，如（Nokia ， 360 ， 凡卓（老人机）），以及最大的合作商OPPO也是与我们团队开发最新的语音助手以及对应的中控平台；

同时，我们也制作了自己的半2C产品，有深圳禾胜成耳机芯片厂商合作，为蓝牙耳机提供了耳机伴侣的手机端应用 ，相关产品可在各大应用市场下载，也可以通过耳机包装上的二维码下载；总下载量XXX ， 日活xxx



## 2. 开发内容





## 3. 使用GIT做项目管理





## 4. Android Points







## 5. Skill / 第三方

### 5.1. 高德地图问题汇总

> 高德地图在开发中的相关问题

#### 5.1.1 高德地图无反应不导航的问题

**描述：**在语音助手项目中出现了导航无反应的问题
**原因：** 因为语音助手设置了音乐播放器的属性，原因不知道为什么
**解决：**取消音乐播放器的属性



### 5.2. 音乐技能问题汇总



### 5.3 模拟点击





## 6. 关于优化

### 1. 内存

 - 角度
 - 办法
 - 方式



### 2. 电量

wakelock 相关

持有wakelock 的情况

https://www.cnblogs.com/leipDao/p/8241468.html

### 3. 泄露

使用LeaksCanary 来分析





## 7. 项目架构



## 8. 蓝牙

### 8.1 经典蓝牙部分

### 8.2 BLE蓝牙部分

### 8.3 自定义蓝牙部分(高级)

### 8.4 附加 蜜汁位运算





## 9. 管理工作总结



## 10. 研发中的问题

### 10. 1 界面开发

#### 10.1.1.  Popupwindow使用异常：unable to add window–token null is not valid

**描述：** Popupwindow必须依赖一个view进行弹窗，

```java
void android.widget.PopupWindow.showAtLocation(View parent, int gravity, int x, int y)
```

调用这个方法就能显示Popupwindow了，但是有时会碰到这样一个异常：

> unable to add window – token null is not valid;is your activity running?

**原因：**导致这个的原因一般是Activity的onCreate()函数里面调用了showAtLocation，由于你的popupwindow要依附于一个activity，而activity的onCreate()还没执行完就需要弹窗肯定会出问题的。

**解决：**在Handler中进行弹窗，在onCreate中通过延时调用就OK了，具体代码如下：

```java
private Handler popupHandler = new Handler(){
	@Override
	public void handleMessage(Message msg) {
		switch (msg.what) {
		case 0:
			popupWindow.showAtLocation(findViewById(R.id.rlShowImage), Gravity.CENTER|Gravity.CENTER, 0, 0);
			popupWindow.update();
			break;
		}
	}
};
```





#### 10.1.2. Android实现ListView或GridView首行/尾行距离屏幕边缘距离

**描述：** ListView或GridView首行/尾行距离失效。
**原因：** Android上ListView&GridView默认行都是置顶的。
**解决：**设置ListView或GridView的android:clipToPadding ＝ true，然后通过paddingTop和paddingBottom设置距离就好了。



#### 10.1.3. ListView点击条目无响应

**描述：**开发中很常见的一个问题，项目中的listview不仅仅是简单的文字，常常需要自己定义listview，自己的Adapter去继承BaseAdapter，在adapter中按照需求进行编写，问题就出现了，可能会发生点击每一个item的时候没有反应，无法获取的焦点。
**原因：**原因多半是由于在你自己定义的Item中存在诸如ImageButton，Button，CheckBox等子控件(也可以说是Button或者Checkable的子类控件)，此时这些子控件会将焦点获取到，所以常常当点击item时变化的是子控件，item本身的点击没有响应。
**解决：**使用descendantFocusability来解决，该属性是当一个为view获取焦点时，定义viewGroup和其子控件两者之间的关系。
属性的值有三种：
beforeDescendants：viewgroup会优先其子类控件而获取到焦点
afterDescendants：viewgroup只有当其子类控件不需要获取焦点时才获取焦点
blocksDescendants：viewgroup会覆盖子类控件而直接获得焦点
通常我们用到的是第三种，即在Item布局的根布局加上android:descendantFocusability=”blocksDescendants”的属性就好了。



#### 10.1.4. 解决ScrollView嵌套GridView/ListView 显示不全的问题

**描述：**在开发中用到了需要ScrollView嵌套GridView的情况，由于这两款控件都自带滚动条，当他们碰到一起的时候便会出问题，即GridView会显示不全。
**原因：**由于父控件是自动根据子控件的大小展示的，所以需要对子控件进行最大化显示处理。
**解决：**解决办法，自定义一个GridView控件，代码如下：

```java
public class MyGridView extends GridView { 
        public MyGridView(Context context, AttributeSet attrs) { 
            super(context, attrs); 
        } 
        public MyGridView(Context context) { 
            super(context); 
        } 
        public MyGridView(Context context, AttributeSet attrs, int defStyle) { 
            super(context, attrs, defStyle); 
        }     
        @Override 
        public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {      
            int expandSpec = MeasureSpec.makeMeasureSpec( 
                    Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST); 
            super.onMeasure(widthMeasureSpec, expandSpec); 
        } 
}
```

代码中主要是修改了onMeasure()方法，将大小设置为int类型的最大值，至于为啥需要右移两位，是因为前两位表示的是如AT_MOST类型的值。





### 10.2 IDE的问题  

#### 10.2.1. Failure [INSTALL_FAILED_OLDER_SDK]

**描述：**编译的时候，报Failure [INSTALL_FAILED_OLDER_SDK]错误。
**原因：**一般是系统自动帮你设置了compileSdkVersion，且版本过高导致的错误。
**解决：**修改build.gradle下的compileSdkVersion xxx为compileSdkVersion 19（或者你本机已有的SDK即可）.



#### 10.2.2. Installation error: INSTALL_FAILED_INSUFFICIENT_STORAGE

**描述：**运行时报错： Installation error: INSTALL_FAILED_INSUFFICIENT_STORAGE。
**原因：**一般应用默认安装都是手机存储空间，而该设备没有足够的存储空间来安装应用程序。
**解决：**一般手机都有SD卡，可以在AndroidManifest.xml文件中设置属性android:installLocation=”auto”就行了。



### 





## 11.  关于烧录手机

### 11.1 步骤

备忘：

> 1. source build/setupenv.sh
>
> 2. lunch 选择自己该烧录的版本
>
> 3. 执行make -j8
>
> 4. 执行APK编译 ， 可以在根目录下 执行 mmm framework/base/package/SystemUI
>
> 5. 修改 build 目录下的某个 .pk8  和 k509.pem  文件，则可以更换要编译的APK的证书
>
> 6. 然后把原来设备的对应APK pull出来
>
> 7. 最后把最新的APK push 到手机
>
>    （注：可能没有adb 的root 权限 也就是不能adb remount， 这是我们需要把APK push到 data/tmp的某个文件夹下， 在进入 adb shell ，把文件cp 到 system/priv_app 下 ）



### 11.2 关于如何让系统级别的APK 在Android Studio 下进行编译