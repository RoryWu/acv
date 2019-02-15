## Android_Intent 详解

**IntentFilter知识点**

​	* IntentFilter 的三个属性:

​		Action
​		URL
 		Category

​	IntentFilter 的匹配规则

- 加载所有的Intent Filter列表 　　
- 去掉action匹配失败的Intent Filter 　　
- 去掉url匹配失败的Intent Filter 　　
- 去掉Category匹配失败的Intent Filter 　　
    判断剩下的Intent Filter数目是否为0。如果为0查找失败返回异常；如果大于0，就按优先级排序，返回最高优先级的Intent Filter	

