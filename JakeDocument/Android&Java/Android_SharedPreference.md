## Android_SharedPreference 详解

#### 1. SharedPreferences基本概念以及优缺点

- SharedPreferences用来保存基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息。数据存储在/data/data//shared_prefs目录下。
- 只能存储少量boolean、int、float、long、String五种简单的数据类型，操作简单，但是无法完全替代其他数据存储方式。
- 无法进行条件查询等复杂操作，对于数据的处理只能是简单的处理。

#### 

#### 2. SharedPreferences的实现原理

SharedPreferences是个单例，具体实现在SharedPrefencesImpl。任意Context拿到的都是同一个实例。 SharedPreferences在实例化的时候会把SharedPreferences对应的xml文件内容通过pull解析全部读取到内存当中（mMap）。 关于读操作：对于非多进程兼容的SharedPreferences的读操作是从内存读取的，不涉及IO操作。写入的时候由于内存已经保存了完整的xml数据，然后新写入的数据也会同步更新到内存，所以无论是用commit还是apply都不会影响立即读取。 关于写操作：除非需要关心xml是否写入文件成功，否则你应该在所有调用commit的地方改用apply。

#### 

#### 3. 同步与异步提交

##### 

##### commit方法的特点

- 存储的过程是原子操作
- commit方法有返回值，设置成功为ture，否则为false
- 同时对一个SharedPreferences设置值最后一次的设置会直接覆盖前次值
- 如果不关心设置成功与否，并且是在主线程设置值，建议用apply方法

##### 

##### apply方法的特点

- 存储的过程也是原子操作
- apply没有返回值，存储是否成功无从知道
- apply写入过程分两步，第一步先同步写入内存，第二部在异步写入物理磁盘
- apply写入的过程会阻塞同一个SharedPreferences对象的其他写入操作

##### 

##### 总结

apply比commit效率高，commit直接是向物理介质写入内容，而apply是先同步将内容提交到内存，然后在异步的向物理介质写入内容。这样做显然提高了效率。

#### 

#### 4. SharedPreferences不能实现跨进程同步问题

##### 

##### 现象

- 数据安全问题：数据读写不能实时更新而造成数据写入丢失等问题，每个进程都会维护一个SharedPreferences的内存副本，副本之间互不干扰
- getSharedPreferences时候的空指针问题

##### 

##### 解决方案

- 通过查看 API 文档发现，在API Level > 11即Android 3.0可以通过Context.MODE_MULTI_PROCESS属性来实现多进程间的数据共享
- 但是在API 23时该属性被废弃。官方文档中明确写明SharedPreferences不适用于多进程间共享数据，推荐使用ContentProvider等方式

#### 

#### 5. SharedPreferences存储的数据不能过大

- SharedPreferences存储的基本的配置型数据，不能存储大量数据
- SharedPreferences的读写操作可能会阻塞主线程，引起界面卡顿甚至ANR
- SharedPreferences的Key-Value的mMap是一直存放在内存当中的，这样会带来极大的内存消耗，甚至产生泄漏、OOM
- SharedPreferences对Key-Value频繁读写会产生大量的临时对象，会造成内存抖动，频繁GC会造成界面卡顿等问题

#### 

#### 6. 参考文章

[Android面试一天一题（14 Day：SharedPreferences）](https://www.jianshu.com/p/4dd53e1be5ba)

[SharedPreferences多进程共享数据爬坑之旅](https://www.jianshu.com/p/e8913d42181b)

[深入理解Android SharedPreferences的commit与apply](https://www.jianshu.com/p/3b2ac6201b33)

[SharedPreferences commit跟apply的区别](https://www.jianshu.com/p/790510b29efe)



**总结：**

1. sSharedPrefsCache 是一个  ArrayMap<String,ArrayMap<File,SharedPreferencesImpl>>，它会保存加载到内存中的  SharedPreferences 对象，ContextImpl 类中并没有定义将 SharedPreferences 对象移除  sSharedPrefsCache 的方法，所以一旦加载到内存中，就会存在直至进程销毁。相对的，也就是说，SP  对象一旦加载到内存，后面任何时间使用，都是从内存中获取，不会再出现读取磁盘的情况
2. SharedPreferences 和 Editor 都只是接口，真正的实现在 SharedPreferencesImpl 和  EditorImpl ，SharedPreferences 只能读数据，它是在内存中进行的，Editor  则负责存数据和修改数据，分为内存操作和磁盘操作
3. 获取 SP 只能通过 ContextImpl#getSharedPerferences 来获取，它里面首先通过  mSharedPrefsPaths 根据传入的 name 拿到 File ，然后根据 File 从 ArrayMap<File,  SharedPreferencesImpl> cache 里取出对应的 SharedPrederenceImpl 实例
4. SharedPreferencesImpl 实例化的时候会启动子线程来读取磁盘文件，但是在此之前如果通过  SharedPreferencesImpl#getXxx 或者 SharedPreferences.Editor 会阻塞 UI 线程，因为在从  SP 文件中读取数据或者往 SP 文件中写入数据的时候必须等待 SP 文件加载完
5. 在 EditorImpl 中 putXxx 的时候，是通过 HashMap 来存储数据，提交的时候分为 commit 和  apply，它们都会把修改先提交到内存中，然后在写入磁盘中。只不过 apply 是异步写磁盘，而 commit  可能是同步写磁盘也可能是异步写磁盘，在于前面是否还有写磁盘任务。对于 apply 和 commit 的同步，是通过 CountDownLatch  来实现的，它是一个同步工具类，它允许一个线程或多个线程一致等待，直到其他线程的操作执行完之后才执行
6. SP 的读写操作是线程安全的，它对 mMap 的读写操作用的是同一把锁，考虑到 SP 对象的生命周期与进程一致，一旦加载到内存中就不会再去读取磁盘文件，所以只要保证内存中的状态是一致的，就可以保证读写的一致性

##### 

##### 注意事项以及优化建议

1. 强烈建议不要在 SP 里面存储特别大的 key/value ，有助于减少卡顿 / ANR
2. 请不要高频的使用 apply，尽可能的批量提交；commit 直接在主线程操作，更要注意了
3. 不要使用 MODE_MULTI_PROCESS
4. 高频写操作的 key 与高频读操作的 key 可以适当的拆分文件，以减少同步锁竞争
5. 不要连续多次 edit，每次 edit 就是打开一次文件，应该获取一次 edit，然后多次执行 putXxx，减少内存波动，所以在封装方法的时候要注意了