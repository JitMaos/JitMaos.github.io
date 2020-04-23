---
title: Android『消息传递』
date: 2019-04-09 19:33:35
tags: [Android,Framework]
---

主流程图

### ThreadLocal

使用场景：
订单处理包含一系列操作：减少库存量、增加一条流水台账、修改总账，这几个操作要在**同一个事务**中完成，通常也即同一个线程中进行处理，如果累加公司应收款的操作失败了，则应该把前面的操作回滚，否则，提交所有操作，这要求这些操作使用**相同的数据库连接对象，而这些操作的代码分别位于不同的模块类中**。

该类提供线程局部变量。这些变量不同于它们的正常变量，即每一个线程访问自身的局部变量时，都有它自己的，独立初始化的副本。**和Thread直接有交互的是ThreadLocalMap，而不是ThreadLocal。我们平时在某个线程中使用ThreadLocal对当前线程的局部变量进行设置时，其内部实现只是ThreadLocal对当前线程的ThreadLocalMap进行操作而已。**

所以`一个线程有一个ThreadLocalMap，而一个ThreadLocalMap内部使用Entry(ThreadLocal作为索引的hash对象定位元素，Value是实际存储的内容)数组存储ThreadLocal对象的，一个ThreadLocal仅能存放一个数据对象的副本，使之成为线程局部私有变量`。

**虽然我们在ThreadLocal上调用set和get，但ThreadLocal本身并不存储数据对象，其背后都是从ThreadLocalMap中通过hash(ThreadLocal)来定位Entry元素，操作Entry中的value对象的。**

还有一个问题，看源码知道set最后还是通过引用赋值的方式实现的，那么引用是怎么在多线程环境下实现<font color="#dd0000">**独立备份**</font>的呢？

<!-- more -->

本段参考：[Android Handler机制之ThreadLocal](<https://www.jianshu.com/p/2a34d30806d4>)

### Hanlder构造方法

在创建Handler时，Handler在其构造方法中***尝试从当前线程(创建Hanlder实例对象的线程)的ThreadLocal中取到Looper对象***，并将该Looper对象赋值给当前Hanlder对象，同时还会将Looper对象的MessageQueue引用赋值给当前Handler。

```java
//Lopper.myLooper
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

```java
//new Handler()
public Handler(Callback callback, boolean async) {
		...
    mLooper = Looper.myLooper();
    if (mLooper == null) {
    	throw new RuntimeException(
      "Can't create handler inside thread " + Thread.currentThread()
      + " that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

### Looper.prepare()

在子线程中尝试创建Handler对象时，检测当前线程是否已经关联Looper对象，不存在就报错(需要调用**Looper.prepare**()进行关联)，存在的话直接返回当前线程关联的Looper对象，使Handler持有Looper和Looper.MessageQueue的引用，从而能够往MessageQueue中添加消息。**一个线程最多只能创建一个Looper，但是多个Handler可以对应同一个Looper，只要在一个线程中。**

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

### 主线程中的Looper

在主线程中创建Handler不需要手动调用Looper.prepare()，因为主线程的**ActivityThread**变量在**main**方法中调用了**prepareMainLooper**()来初始化了主线程的**Looper**对象。

```java
//ActivityThread.main()
public static void main(String[] args) {
    ...

    Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    if (false) {
        Looper.myLooper().setMessageLogging(new
            LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    Looper.loop();
}
```

### sendMessage()和sendMessageAtTime()

Handler的sendMessage调用的是MessageQueue.enqueueMessage(Message,uptimeMillis)，因为可能存在多个线程同时往同一个MessageQueue中插入Message，所以**MessageQueue.enqueueMessage是同步方法**。**sendMessage()最终调用sendMessageAtTime()**，只不过它的uptimeMillis参数为0，即不延时执行的意思。

```java
//Handler.sendMessage() 
public final boolean sendMessage(Message msg) {
    return sendMessageDelayed(msg, 0);
}
//Hanlder.sendMessageDelayed
public final boolean sendMessageDelayed(Message msg, long delayMillis){
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

这里的uptimeMillis随Msg存入到MQ，而**sendMessageDelayed()的延迟执行效果是有MQ在取出消息进行dispatch时完成的，取出消息时会比对当前时间和Msg的when属性**。

```java
//Handler.sendMessageAtTime()
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
//Handler.enqueueMessage()
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
//MessageQueue.enqueueMessage()
boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

### Looper.loop()

Looper调用loop()方法，其中不断从MessageQueue中取出Message(通过message.next()方法)，进行dispatchMessage，`其实在Handler调用sendMessage时已经将自己设置为Message.target了，所以dispatch只是将message发给自己的target处理而已`。

### 主线程如何通过Handler发送Message通信

定义一个全局的Handler对象mHandler；在子线程A中初始化这个mHandler,并重写handlerMessage()方法；在子线程B中调用mHandler.obtainMessage()方法从Message类自带的Message Pool中返回一个Message对象，然后使用message.sendToTarget()方法，发送消息。

## 常见问题

### 介绍一下消息传递的机制？



### Looper、Handler、Message、Thread之间的关系是怎样的？

一个线程对应一个Looper(你想啊，如果一个线程对应多个Looper，那么线程中创建的Handler发消息时是发到哪个Looper里呢？再说，创建Handler时，就要求初始化一个Looper，而在初始化Looper时，就将当前线程的ThreadLocal关联了)，一个Looper对应一个MessageQueue(这个很好理解，在调用Looper.prepare中创建了一个MessageQueue)，一个MessageQueue包含多个Message，一个Message对应一个Handler(Message对象有一个叫target的handler成员变量，在消息循环时，只是将message发送给他的target而已)。

### 主线程的Looper.loop()一直无限循环,为何没有ANR？

```java
//Looper.loop()
public static void loop() {
    ...
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
        ...
    }
    ...
}
```

首先需要知道造成ANR的原因：

1. **当前的事件没有机会得到处理**（即主线程正在处理前一个事件，没有及时的完成或者looper被某种原因阻塞住了）。
2. **当前的事件正在处理，但没有及时完成。**执行时间超过了ANR的阈值：按键事件 5s ，broadcast 10s、前台service无响应的超时时间为20秒，后台service为200秒。

这样就可以解释为什么主线程的Looper.loop()一直无限循环但是为什么没有造成ANR了，从上面的代码可以看出当没有Message需要处理时，**主线程没有进行任何和造成ANR有关的事件的处理**，虽然是在无限循环，但是比起频繁的开启、关闭消息循环动作，这样的开销反而更小吧。

主线程的死循环一直运行是不是特别消耗CPU资源呢？ 其实不然，这里就涉及到Linux pipe/epoll机制，简单说就是在主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

### Q：没有主线程Handler引用，如何通过Handler往主线程发消息?

查看Looper源码，可以找到一个Looper的变量定义，可以看到该sMainLooper是个`内部静态变量`。

```java
private static Looper sMainLooper;  // guarded by Looper.class 
```

我们知道Activity在ActivityThread中调用prepareMainLooper()自动为主线程关联了一个Looper，这个sMainLooper就保存了主线程Looper的引用。

```java
/**
* Initialize the current thread as a looper, marking it as an
* application's main looper. The main looper for your application
* is created by the Android environment, so you should never need
* to call this function yourself.  See also: {@link #prepare()}
*/
public static void prepareMainLooper() {
  prepare(false);
  synchronized (Looper.class) {
    if (sMainLooper != null) {
      throw new IllegalStateException("The main Looper has already been prepared.");
    }
    sMainLooper = myLooper();
  }
}
```

同时Looper还提供了一个外部访问sMainLooper的方法：

```java
/**
* Returns the application's main looper, which lives in the main thread of the application.
*/
public static Looper getMainLooper() {
  synchronized (Looper.class) {
    return sMainLooper;
  }
}
```

基于以上，我们可以在没有调用prepare()的情况下，在子线程中创建一个Handler，并使用它与主线程进行通信

```java
void testMainLooper() {
  new Thread(){
    @Override
    public void run() {
      new Handler(Looper.getMainLooper()){
        @Override
        public void handleMessage(Message msg) {
          super.handleMessage(msg);
          if(msg.what == 2019) {
            ((Button)findViewById(R.id.btnTestMainLooper))
            .setText("收到来自子线程的消息" + msg.what);
          }
        }
      }.sendEmptyMessageDelayed(2019,5000);
    }
  }.start();
}
```

### Handler如何在`子线程之间`通过Message传递数据？

定义一个全局的Handler对象mHandler；在子线程A中初始化这个mHandler,并重写handlerMessage()方法；在子线程B中调用mHandler.obtainMessage()方法从Message类自带的Message Pool中返回一个Message对象，然后使用message.sendToTarget()方法，发送消息。

### Handler是如何关联线程?



### 子线程是如何发Message给主线程的，底层是怎么实现的？

首先线程不同于进程，两个线程之间是不存在内存隔离。因此他们可以访问所在进程的公共内存空间，这里具体说的就是java运行时数据区的堆中的对象。所以这里可以直接访问主线程中定义的Handler对象，然后，在子线程中进行的操作**仅仅是将Message放到Handler对应的MessageQueue中去而已**，至于后面消息是怎么在MessageQueue中流转的，得