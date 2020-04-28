---
title: 源码学习『OKHTTP——连接管理』
tags:
---

网路请求任务分为同步请求和异步请求两种。在OKHTTP中，请求任务被封装为一个一个的RealCall对象，管理这些RealCall不是直接把任务扔到一个线程池里就不管了。而是使用了三个双端队列来实现的，他们分别是：

```java
/** Ready async calls in the order they'll be run. */
//M->OKHTTP，等待运行异步队列
private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

/** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
//M->OKHTTP，运行中异步队列
private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

/** Running synchronous calls. Includes canceled calls that haven't finished yet. */
//M->OKHTTP，运行中同步队列
private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
```

**线程池**

OKHTTP中使用到的线程池是一个基于同步队列(SynchronousQueue)的核心线程数为0，最大线程数无上限的线程池，线程存活时间60秒。

```java
executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
```

这里需要注意两点：

1. 虽然线程池没有线程上线，但是OKHTTP的任务管理类Dispatcher中对同时进行的任务数量做了限制

   ```java
   private int maxRequests = 64; //最大同时进行的”异步“请求数量
   private int maxRequestsPerHost = 5; //单个host上同时进行的最大请求数
   ```

2. 同步队列是一种没有实际存储空间的队列，一个runnable想要进队列，必须存在一个任务接收者，否则进队列的操作会阻塞。所以这里指定最大线程数为无上限。

**任务执行**

