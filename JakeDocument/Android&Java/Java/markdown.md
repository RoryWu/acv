# java 多线程

## h2

## JUC



### 线程排序

### countdownlatch

作减法 , 例如,开始启动了 6 个线程, 

countdownlatch.wait();

一次减为零时 , wait 结束



#### CyclicBarrier

作加法 , 

```java
CylicBarrier cylicBarrier = new CylicBarrier(7 , 
	()-> {System.out.print("--------结束-----")})
    
cylicBarrier.wait()
```



#### Semaphore

```java
final Semaphore semaphore = new Semaphore(3); // 模拟3个车位
        for (int i = 0; i < 6; i++) { // 6个车来枪战
            new Thread(){
                try{
                    semaphore.acquire();
                    // do something
                    sleep(3);
                }finally{
                    semaphore.release();
                }
            }
        }
```





### 阻塞队列

1. 阻塞队列概念:

    * 当队列里面是空的时候, 从队列里取的操作,将被阻塞
    * 当队列满的时候, 向队列里放的操作,将被阻塞

    

2. BlockQueue: 7 个实现类,常用的3个

    * ArrayBlockQueue
    * 

生产者, 消费者模式:

 两个线程, 一个生产, 一个消费 , 三个重点: synchronized , wait , notify

## 锁

1. 多个线程, 操作资源类, 高内聚, 低耦合
2. 判断,  干活, 通知
3. 预防虚假唤醒 , 线程的等待, 需要使用while

铁三角:

sync -------------- wait --------------- notify

lock --------------- await ------------- singal

y





Synchronized  和 Lock 的区别, LOCK的优点?

> 原始构成

* Synchronized 是JVM层面的, 是java 的关键字 / Lock 是api 层面的
* Synchronized 的底层是 monitorenter 和monitorexit (Monitor 这个类)
* Synchronized 是可重入锁, 一次monitorenter , 两次 monitorexit 保证在正常和异常的情况下都能退出
* Lock 是具体的类

> 使用方法

* Synchronized 不需要用户手动的去释放 , 执行完代码块中的代码,自动释放锁
* ReentrantLock 则需要用户手动的去释放, 如果不释放可能造成死锁的情况
* 需要Lock/unLock 配合try/finally 类占用和释放

> 等待是否可中断

* Synchronized 不可中断, 除非异常退出或正常执行完成
* ReentrantLock　可中断　(优点)
    * 设置超时　tryLock
    * lockinterruptibly  

> 加锁是否公平

* Synchronized 非公平锁

* Ｒeentrant　可以自己控制

> 锁绑定多个条件

* Synchronized 没有
* ReentrantLock 用来实现分组唤醒需要的线程 , 可以精确唤醒 , 而不是像Synchronized 要么随机一个,要么所有(优点)




​    



























