## Android 布局优化

### 1. 绘制原理

#### 分工

* cpu 负责计算显示的内容
* gpu 负责栅格化（ui 元素绘制到屏幕上）

#### 刷新

* 16ms 发送出vsync 的信号来触发ui 的渲染

  需要在16ms内完成渲染代码的完成，才能在肉眼看起来UI 平滑

* 大多数的android 设备屏幕刷新的频率：60hz

  60帧/秒，是人眼的极限

  

### 2.工具

#### Systrace

* 关注Frames
* 正常： 绿色圆点，丢帧：黄色或者红色
* Alert栏

#### Layout Inspector

* AndroidStudio 自带工具

* 查看视图的层次机构

* 打开方式

  Tools-> LayoutInspector 





###3.布局加载的原理 

**setContentView**

```
Activity setcontentview -> 
AppCompatActivity   setcontentView(抽象) -> 
AppCompatDalegate (抽象) -> 
AppDalegateImpl setcontentView(实现) 
Resource loadXMLResourceParser
getLayout --> io过程， 需要把xml 文件整体load到内存中
createViewFromTag 
根据tag ，然后通过Factory 来创建不同的控件
layoutinflate  oncreateView  constractor
View view = constractor.newInstance() 使用反射来加载View

```

**性能瓶颈**

* 布局解析文件：IO过程
* 创建View对象： 反射

**LayoutInflater.Factory**

* LayoutInflater创建View 的一个HOOK
* 定制创建View 的一个过程：全局替换自定义的Textview 等
* 通过HOOK 可以优雅的获取控件的耗时

**异步inflate实战**

> 创建View 的过程比较慢的原因：

* 布局文件的读取： 通过IO过程 

* 创建View的过程比较慢，加载xml 文件创建View 是通过反射的方式（比new 慢3倍），嵌套的越多，反射的越多

* 解决思路

  1. 根本性解决，IO慢， 反射慢， 有没有方式来不通过这种形式来加载View

  2. 侧面缓解

     * 通过异步的inflate

       * WorkThread加载布局

       * 回调主线程

       * 节约主线程时间

       * 实战：

         ```java
         // 通过AsycLayoutInflate使用
         // com.android.support:asynclayoutinflater
         public void onCreate(){
             // setContentView()
             new AsyncLayoutInflate(R.layout.main_layout , getParent(), )
         }
         ```

         

         

     ``` 
     
     
     ```

     



