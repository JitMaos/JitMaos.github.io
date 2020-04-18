---
title: 源码学习『EventBus』
tags: [源码学习,事件总线]
---

大概介绍一下EventBus是什么，它是一块流行基于事件总线模型实现的基于Java的消息通知类库。什么是事件总线模型？ 

事件总线模式主要是处理事件，包括4个主要组件：<font color="#dd0000">**事件源、事件监听器、通道和事件总线**</font>。消息源将消息发布到事件总线上的特定通道上。侦听器订阅特定的通道。侦听器会被通知消息，这些消息被发布到它们之前订阅的一个通道上。

#### 思考过程

有了事件总线模型的定义，现在我们先不看源码，思考一下如果自己去实现这么一个<font color="#dd0000">**跨界面、跨线程**</font>的数据对象传递库会怎么设计。

首先需要一个消息池，用于存放消息对象

其次每个消息需要关联一个通道

然后侦听器可以订阅他关注的通道，这意味着还需要一个容器用于存储侦听器，且这个容器需要和存储消息的容器关联。

最后，在Android中主线程和普通线程中的接收、发送消息怎么实现。我们知道大多数开源库中消息通知的机制最后都是基于Looper实现的，或者说Handler，因为这是Android中系统默认支持的基础消息传递机制。

还有，延迟消息怎么处理？怎么区分主线程和普通线程呢？

#### 准备知识

**CopyOnWriteArrayList**：如果对**遍历操作多于读写操作**，那么在多线程环境下使用CopyOnWriteArrayList的性能高于Vector。<font color="#dd0000">**底层使用由volatile修饰的数组+显式锁ReentrantLock实现读写分离，写时复制出一个新的数组，完成插入、修改或移除操作后将新数组赋值给array。**</font>

#### 

## 注册流程

![img](http://47.110.40.63:8080/img/blog/EventBus注册流程.png)

EventBus.register(Object)

 通过反射拿到传入的obj的Class对象，如果是在MainActivity里做的注册操作，

```java
public void register(Object subscriber) {

  Class<?> subscriberClass = subscriber.getClass();
  // 步骤1 
  List<SubscriberMethod> subscriberMethods = 		   subscriberMethodFinder.findSubscriberMethods(subscriberClass);
  //  步骤2
  synchronized (this) {
    for (SubscriberMethod subscriberMethod : subscriberMethods) {
      subscribe(subscriber, subscriberMethod);
    }
  }
}
```

SubscriberMethod

获取注册对象中所有被@Subscribe注解的方法集合。

```java
public class SubscriberMethod {
    final Method method;  //方法的句柄，后序用于反射调用
    final ThreadMode threadMode; //方法回调过来后，运行在哪种线程，线程模式
    final Class<?> eventType; //方法中的参数，即方法关注的数据类型
    final int priority;
    final boolean sticky;
    /** Used for efficient comparison */
    String methodString;

    public SubscriberMethod(Method method, Class<?> eventType, ThreadMode threadMode, int priority, boolean sticky) {
        this.method = method;
        this.threadMode = threadMode;
        this.eventType = eventType;
        this.priority = priority;
        this.sticky = sticky;
    }
}
...
```

---

findSubscriberMethods(subscriberClass)

```java
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
   //METHOD_CACHE是事先定义了一个缓存Map,以当前的注册对象的Class对象为key,注册的对象里所有的被@Subscribe注解的方法集合为value -> Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();
 
   List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);

   // 第一次进来的时候，缓存里面没有集合，subscriberMethods 为null
   if (subscriberMethods != null) {
     return subscriberMethods;
   }
   // ignoreGeneratedIndex是在SubscriberMethodFinder()的构造函数初始化的，默认值是 false
   if (ignoreGeneratedIndex) {
     //  通过反射去获取 
     subscriberMethods = findUsingReflection(subscriberClass);
   } else {
     //通过apt插件生成的代码。使用subscriber Index生成的SubscriberInfo来获取订阅者的事件处理函数，
     subscriberMethods = findUsingInfo(subscriberClass);
   }
   if (subscriberMethods.isEmpty()) {
     throw new EventBusException("Subscriber " + subscriberClass
                                 + " and its super classes have no public methods with the @Subscribe annotation");
   } else {
     METHOD_CACHE.put(subscriberClass, subscriberMethods);
     return subscriberMethods;
   }
}
```







## 反注册流程

![img](http://47.110.40.63:8080/img/blog/EventBus之unregister流程图.jpg)



#### 对象池

#### 设计模式

#### 线程切换

**判断是否是当前线程是否主线程**：

#### 对象回收



































