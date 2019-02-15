## Android Bitmap 详解

#### 1. Bitmap的理解

Bitmap是Android系统中的图像处理的最重要类之一。用它可以获取图像文件信息，进行图像剪切、旋转、缩放等操作，并可以指定格式保存图像文件。

#### 

#### 2. Bitmap的内存分配策略

在Androin3.0之前的版本，Bitmap像素数据存放在Native内存中，而且Nativie内存的释放是不确定的，容易内存溢出而Crash，不使用的图片要调用recycle()进行回收。

[![3.0之前](https://camo.githubusercontent.com/fb73b2141a1e0449877230724875f28a07fd1564/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d616432613161646432623866306463362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/fb73b2141a1e0449877230724875f28a07fd1564/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d616432613161646432623866306463362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

从Androin3.0开始，Bitmap像素数据和Bitmap对象一起存放在虚拟机的堆内存中（从源代码上看是多了一个byte[] buffer用来存放数据），也就是我们常说的Java Heap内存。

[![3.0之后](https://camo.githubusercontent.com/4679afa2759a30174365d29de47d29a8f2a294ac/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d363661333933623636303961383032382e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/4679afa2759a30174365d29de47d29a8f2a294ac/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d363661333933623636303961383032382e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

从Androin8.0开始，Bitmap像素数据存重新回到Native内存中

[![8.0之后](https://camo.githubusercontent.com/fb73b2141a1e0449877230724875f28a07fd1564/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d616432613161646432623866306463362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/fb73b2141a1e0449877230724875f28a07fd1564/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d616432613161646432623866306463362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

#### 

#### 3. Bitmap的内存占用计算

Bitmap的内存占用大小的计算：

- 一般情况下的计算

    Bitmap占用的内存 = width * height * 一个像素所占的内存

- 在Android中，考虑各种因素情况下的计算

    Bitmap占用的内存 = width * height * nTargetDensity/inDensity * nTargetDensity/inDensity * 一个像素所占的内存

Bitmap的内存占用大小与三个因素有关：

- 色彩格式，前面我们已经提到，如果是ARGB_8888那么就是一个像素4个字节，如果是RGB_565那就是2个字节
- 原始文件存放的资源目录（分辨率越小，内存占用越小）
- 目标屏幕的密度（屏幕的密度越小，内存占用越小）

有关Bitmap的色彩格式：

- ARGB_8888的内存消耗是RGB_565的2倍
- ARGB_8888格式比RGB_565多了一个透明通道
- 如果使用RGB_565格式解析ARGB_8888格式的图片(png)，可能会导致图片变绿

#### 

#### 4. Bitmap的回收

- 在Android3.0以前以及Android8.0之后Bitmap的像素数据是存放Native内存中，我们需要回收Native层和Java层的内存。
- 在Android3.0以后以及Android8.0之前Bitmap的像素数据是存放在Java层的内存中的，我们只要回收堆内存即可。
- 官方建议我们3.0以后使用recycle方法进行回收，该方法也可以不主动调用，因为垃圾回收器会自动收集不可用的Bitmap对象进行回收。
- recycle方法会判断Bitmap在不可用的情况下，将发送指令到垃圾回收器，让其回收native层和Java层的内存，则Bitmap进入dead状态。
- recycle方法是不可逆的，如果再次调用getPixels()等方法，则获取不到想要的结果。

#### 

#### 5. Bitmap的复用

Android在3.0之后BitmapFactory.Options引入了inBitmap属性，设置该属性之后解码图片时会尝试复用一张已经存在的Bitmap，避免了内存的回收以及重新申请的过程。

Bitmap复用的限制：

- 声明可被复用的Bitmap必须设置inMutable为true
- Android4.4(API 19)之前只有格式为jpg、png，同等宽高（要求苛刻），inSampleSize为1的Bitmap才可以复用
- Android4.4(API 19)之前被复用的Bitmap的inPreferredConfig会覆盖待分配内存的Bitmap设置的inPreferredConfig
- Android4.4(API 19)之前待加载Bitmap的Options.inSampleSize必须明确指定为1
- Android4.4(API 19)之后被复用的Bitmap的内存必须大于需要申请内存的Bitmap的内存

#### 

#### 6. Bitmap加载大图与防止OOM

加载大图的时候注意点：

- 在Android系统中，读取位图Bitmap时，分给虚拟机中的图片的堆栈大小只有8M，如果超出了，就会出现OutOfMemory异常
- 在加载大图、长图等操作当中，推荐对OutOfMemoryError进行捕获，并且返回一张默认图片
- 使用采样率（inSampleSize），如果需要显示缩列图，并不需要加载完整的图片数据，只需要按一定的比例加载即可
- 使用Matrix变形等，比如使用Matrix进行放大，虽然图像大了，但并没有占用更多的内存
- 推荐使用一些成熟的开源图片加载库，它们帮我们完成了很多工作。比如异步加载、Facebook的Fresco还自己开辟了Native内存用于存储图片，以得到更大的内存空间（兼容性问题）
- 使用分块解码（BitmapRegionDecoder）、硬解码等方案

获取图片缩略图的模板代码如下（主要分为3个步骤）：

```
public static Bitmap thumbnail(String path, int width, int height, boolean autoRotate) {

    //1. 获得Bitmap的宽高，但是不加载到内存
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(path, options);
    int srcWidth = options.outWidth;
    int srcHeight = options.outHeight;

    //2. 计算图片缩放倍数
    int inSampleSize = 1;
    if (srcHeight > height || srcWidth > width) {
        if (srcWidth > srcHeight) {
            inSampleSize = Math.round(srcHeight / height);
        } else {
            inSampleSize = Math.round(srcWidth / width);
        }
    }

    //3. 真正加载图片到内存当中
    options.inJustDecodeBounds = false;
    options.inSampleSize = inSampleSize;
    //ARGB_8888格式的图片，每像素占用 4 Byte，而 RGB565则是 2 Byte
    options.inPreferredConfig = Bitmap.Config.RGB_565;
    options.inPurgeable = true;
    options.inInputShareable = true;
    return BitmapFactory.decodeFile(path, options);
}
```

#### 

#### 7. LRU缓存机制

LRU缓存机制的核心原理：

- LruCache中维护了一个以访问顺序排序的集合LinkedHashMap（双向循环链表）
- 新数据插入到链表头部
- 每当缓存命中（即缓存数据被访问），则将数据移到链表头部
- 当链表满的时候，将链表尾部的数据丢弃

[![LruCache缓存机制](https://camo.githubusercontent.com/ae0c8eea02847d3f52b9ada32a2d32ffec998f82/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d346639323364643230323065643032662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/ae0c8eea02847d3f52b9ada32a2d32ffec998f82/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d346639323364643230323065643032662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

#### 

#### 8. 图片的三级缓存

关于Android图片的多级缓存，其中主要的就是内存缓存和硬盘缓存。它的核心思想是：

- 在获取一张图片时，首先到内存缓存(LruCache)中去加载
- 如果未加载到，则到硬盘缓存(DiskLruCache)中加载，如果加载到将其返回并添加进内存缓存
- 否则通过网络加载一张新的图片，并将新加载的图片添加进入内存缓存和硬盘缓存

[![图片三级缓存](https://camo.githubusercontent.com/cb55f2f23bd06b53a9b5a88c315833ed756997d2/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d613464343536663363356538353561622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/cb55f2f23bd06b53a9b5a88c315833ed756997d2/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323537303033302d613464343536663363356538353561622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

#### 

#### 9. 参考文章

[Bitmap详解与Bitmap的内存优化](https://www.jianshu.com/p/8206dd8b6d8b)

[Android性能调优(5)—Bitmap内存模型](https://www.jianshu.com/p/af1d451f9c4f)

[Android面试一天一题（Day 22: 图片到底是什么）](https://www.jianshu.com/p/3c597baa39e5)

[Android使用BitmapRegionDecoder加载超大图片方案](https://blog.csdn.net/jjmm2009/article/details/49360751)

[Android性能调优(6)—Bitmap优化](https://www.jianshu.com/p/7b8034284956)

[彻底解析Android缓存机制——LruCache](https://www.jianshu.com/p/b49a111147ee)

[浅谈图片加载的三级缓存](https://www.jianshu.com/p/eb6bc555b60b)