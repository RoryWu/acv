# Title

## 使用

* #### 创建

    * 继承Thread
    * 实现Runnable

* #### 区别

    

* #### 问题

    

## 线程间通信

* #### Synchronized 关键字

    1. synchronized 对象锁

        ```java
        ThreadA a = new ThreadA(object);
        ThreadB b = new Threadb(object);
        
        不同的线程持有同一个对象的引用, 来进行锁的管理
        ```

        

    2. synchronized 线程间通信

    3. Synchronized 和 volatile

        ```java
        线程中的变量是: 线程本地内存的一个值拷贝, 和其他线程中的同一个变量的值不一定会一致的
        
        java 中的主内存机制 , 每个线程都有自己的独立内存 (一种缓存机制)
        
        而使用volatile 则获取的变量的值 是主内存中的值
        
        volatile 和 synchronized 关键字的区别
        synchronized 的获取和释放 会有一个monitor 来监听控制,如果两个线程使用同一个监听器, 则监听器可以强制两个线程在同一时间只有一个线程在执行代码块
        (锁住当前的变量, 只有当前线程能够访问变量, 其他线程则被锁住(等待变量更新))
        
        volatile 同步线程内存和主内存之间来同步一个变量的值 (只能使用在变量)
        synchronized 是同步所有变量的值 (可以使用在变量 方法 和代码块 )
        ```

        

    4. Synchronized 和 lock

        ```java
        lock 用法
        需要指定起始位置和终止位置
        在finally 需要unlock
        
        性能上
        synchronized 是依赖于java 虚拟机 (悲观锁)
        lock 则是用户自己控制 (乐观锁)
        
        ```

        

    

* sleep / wait 区别

    ```
    
    ```

* wait/notify 机制

    ```java
    等待锁
    释放锁
    ```

    

## 









system.exit(0) 后不会执行finally

return 会执行 , 而且在return 之前

但是在finally 修改 return的值 并不会生效

建议不要在finally 中return































































































