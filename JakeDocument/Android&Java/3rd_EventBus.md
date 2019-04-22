# EventBus 代码分析

### 基础使用

事件总线



### 原理

1. register

    ```java
    public void register(Object subscriber){
        // 首先获得class 对象
        Class<?> subscriberClass = subscriber.getClass();
        // 遍历这个class 找到所有带 subscribe 注解的方法
        // 通过 subscriberMethodFinder 来找到订阅者订阅了哪些事件.返回一个 SubscriberMethod 对象的 List, SubscriberMethod
            // 里包含了这个方法的 Method 对象,以及将来响应订阅是在哪个线程的 ThreadMode ,以及订阅的事件类型 eventType ,以及订阅的优
            // 先级 priority ,以及是否接收粘性 sticky 事件的 boolean 值，其实就是解析这个类上的所有 Subscriber 注解方法属性。
        List<SubscriberMethod> subscriberMethods = subscriberMethoderFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods){
                // 将这些方法 和 这个Class 绑定订阅起来
                subscribe(subscriber , subscriberMethod);
            }
        }
            
    }
    ```

    ```java
     // Must be called in synchronized block
        private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
            // 获取方法参数的 class
            Class<?> eventType = subscriberMethod.eventType;
            // 创建一个 Subscription
            Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
            // 获取订阅了此事件类的所有订阅者信息列表
            CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
            if (subscriptions == null) {
                // 线程安全的 ArrayList
                subscriptions = new CopyOnWriteArrayList<>();
                // 添加
                subscriptionsByEventType.put(eventType, subscriptions);
            } else {
                // 是否包含，如果包含再次添加抛异常
                if (subscriptions.contains(newSubscription)) {
                    throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                            + eventType);
                }
            }
            // 处理优先级
            int size = subscriptions.size();
            for (int i = 0; i <= size; i++) {
                if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                    subscriptions.add(i, newSubscription);
                    break;
                }
            }
            // 通过 subscriber 获取  List<Class<?>>
            List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
            if (subscribedEvents == null) {
                subscribedEvents = new ArrayList<>();
                typesBySubscriber.put(subscriber, subscribedEvents);
            }
            // 将此事件类加入 订阅者事件类列表中
            subscribedEvents.add(eventType);
    
            // 处理粘性事件
            if (subscriberMethod.sticky) {
                if (eventInheritance) {
                    // Existing sticky events of all subclasses of eventType have to be considered.
                    // Note: Iterating over all events may be inefficient with lots of sticky events,
                    // thus data structure should be changed to allow a more efficient lookup
                    // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                    Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                    for (Map.Entry<Class<?>, Object> entry : entries) {
                        Class<?> candidateEventType = entry.getKey();
                        if (eventType.isAssignableFrom(candidateEventType)) {
                            Object stickyEvent = entry.getValue();
                            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                        }
                    }
                } else {
                    Object stickyEvent = stickyEvents.get(eventType);
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        }
    ```

2. 重点类:SubscriberMethodFinder.java

    作用, 遍历clazz 中的所有方法, 判断方法 参数, 线程 等信息

    然后封装造成一个 SubscribeMethod 的对象

3. post

    







### 理解

* 首先 getDefault   ,在两个对象间发送消息, 说明使用了单例
* unregister() 说明防止内存泄露
* 使用注解 说明使用了 反射 获取了 注解的方法 和 对象

