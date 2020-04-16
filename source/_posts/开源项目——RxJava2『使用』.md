---
title: RxJava2『使用』
date: 2019-04-14 07:39:43
tags: RxJava2
---

#### 基础概念

**RxJava是一个在基于Java VM扩展的使用可观测的序列来组成异步的、基于事件的程序的库。**它基于观察者模式支持数据和事件的流式流通处理，并增加了多种运算符以在多种场景下整合事件和数据。抽象了低级别的线程、同步、并发API。

+ **响应式编程**(Reactive Programming)是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型**会自动将变化的值通过数据流进行传播**。
+ **函数式编程**是也一种编程范式，它将函数视为一等公民，意味着函数和其他类型数据有同等的地位，可以作为函数的参数、返回值使用(高阶函数)。
+ 被观察者，Observable/Flowable
+ 观察者，Observer/Consumer/Subscriber
+ 上游：发射事件的地方，对应的是被订阅者(Observable/Flowable)
+ 下游：接收事件的地方，对应的是观察者(Observer/Subscriber)

#### 使用

##### 1. 创建被观察者(上游对象，Observable/Flowable)

```java
Observable<String> observable = Observable.fromArray(new String[]{"A","B","C"});
```

<!-- more -->

##### 2.创建观察者（下游对象，Observer/Consumer)

```java
//创建一个Observer
Observer<String> observer = new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(String s) {
        System.out.println("Thread:" + Thread.currentThread() + "|" + "accept:"+ s);
    }
    @Override
    public void onError(Throwable e) {
    }
    @Override
    public void onComplete() {
    }
};
//创建一个Consumer
Consumer<String> consumer = new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        System.out.println("Thread:" + Thread.currentThread() + "|" + "onNext:"+ s);
    }
};
```

**Consumer和Observer的不同之处在于Consumer不关心onNext()和onError()方法，它只管处理上游发送的数据。**

##### 3.订阅

订阅通过方法subscribe()实现，**当一个被观察者收到一个订阅请求时，他就会开始发送一遍事件**。所以下面2次subscribe动作，observable发送了2遍事件。

```java
observable.subscribe(observer);
observable.subscribe(consumer);
>>>输出：
Thread:Thread[main,5,main]|onNext:A
Thread:Thread[main,5,main]|onNext:B
Thread:Thread[main,5,main]|onNext:C
Thread:Thread[main,5,main]|accept:A
Thread:Thread[main,5,main]|accept:B
Thread:Thread[main,5,main]|accept:C
```

##### 4.使用调度器(Schedulers)指定上下游执行线程

在开发过程中，我们经常需要把一些耗时动作放在后台线程中进行，然后将结果放到主线程中处理。

在RxJava中可以使用调用器的<font color="#dd0000">**subscribeOn指定在哪个线程发生订阅(及上游)**</font>和<font color="#dd0000">**observeOn用于指定在哪个线程发生观察(即下游)**</font>从而实现这样的功能。我们先把上游指定到newThread中执行，任务为每隔一秒发送一个数字。

```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        for (int i = 0; i < 3; i++) {
            System.out.println("Thread:" + Thread.currentThread() + "|" + "emit:" + i);
            emitter.onNext(i + "");
            Thread.sleep(1000);
        }
    }
}).subscribeOn(Schedulers.newThread()).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        System.out.println("Thread:" + Thread.currentThread() + "|" + "accept:"+ s);
    }
});
>>>输出：

Process finished with exit code 0
```

奇怪的是根据console看到上游发送任何事件，下游更没有接收到任何事件。这是因为：

<font color="#dd0000">**在RxJava中默认的调度器运行在守护线程上**</font>，**这意味着只要Java主线程退出，所有这些默认的调度器会停止**。本段代码中上游运行在子线程中处理耗时操作，下游运行在主线程中，上游还没开始发送消息，下游的主线程已经结束任务了，所以调度器来不及调度也就停止了。现在我们让上游运行在默认的主线程中，下游运行在子线程中：

```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        for (int i = 0; i < 3; i++) {
            System.out.println("Thread:" + Thread.currentThread() + "|" + "emit:" + i);
            emitter.onNext(i + "");
            Thread.sleep(1000);
        }
    }
}).observeOn(Schedulers.newThread()).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        System.out.println("Thread:" + Thread.currentThread() + "|" + "accept:"+ s);
    }
});
>>>输出：
Thread:Thread[main,5,main]|emit:0
Thread:Thread[RxNewThreadScheduler-1,5,main]|accept:0
Thread:Thread[main,5,main]|emit:1
Thread:Thread[RxNewThreadScheduler-1,5,main]|accept:1
Thread:Thread[main,5,main]|emit:2
Thread:Thread[RxNewThreadScheduler-1,5,main]|accept:2
```

##### 5.结合操作符实现数据操作

 我们知道RxJava有多种操作符可以操作流中的数据，下面是一个例子，假设学生的姓名和成绩来自两个接口，我们需要等这个接口都返回结果时，才刷新界面，看用下面这demo代码，需要注意将2个接口请求放到子线程中，否则2个模拟请求默认会在主线程中，进行串行调用。(可以看两个log的时间间隔)

```java
import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.widget.TextView;
import java.util.HashMap;
import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.functions.BiFunction;
import io.reactivex.functions.Consumer;
import io.reactivex.schedulers.Schedulers;

public class TestOperatorActivity extends Activity {

    private String TAG = "TestOP";

    class Stu {
        String name;
        int scoreMath, scoreEnglish;

        public Stu(String name, int scoreMath, int scoreEnglish) {
            this.name = name;
            this.scoreMath = scoreMath;
            this.scoreEnglish = scoreEnglish;
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.rx_act_test_operator);
        findViewById(R.id.btnTestZip).setOnClickListener(v -> testZip());
    }

    void testZip() {
        TextView tvName = findViewById(R.id.tvName);
        TextView tvScoreEnglish = findViewById(R.id.tvScoreEnglish);
        TextView tvScoreMath = findViewById(R.id.tvScoreMath);

        Observable nameTask = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                 //模拟请求耗时
                Thread.sleep(1000);
                emitter.onNext("小王");
                Log.d(TAG, "Thread:" + Thread.currentThread() + "发送姓名");
            }
        }).subscribeOn(Schedulers.newThread()); //注意将该模拟请求指定到子线程中，模拟并行请求
        Observable scoreTask = Observable.create(new ObservableOnSubscribe<HashMap<String, Integer>>() {
            @Override
            public void subscribe(ObservableEmitter<HashMap<String, Integer>> emitter) throws Exception {
                //模拟请求耗时
                Thread.sleep(5000);
                HashMap map = new HashMap();
                map.put("scoreEnglish", 80);
                map.put("scoreMath", 95);
                Log.d(TAG, "Thread:" + Thread.currentThread() + "发送成绩");
                emitter.onNext(map);

            }
        }).subscribeOn(Schedulers.newThread()); //注意将该模拟请求指定到子线程中，模拟并行请求
        Observable.zip(nameTask, scoreTask, new BiFunction<String, HashMap<String, Integer>, Stu>() {
            @Override
            public Stu apply(String s, HashMap<String, Integer> stringIntegerHashMap) throws Exception {
                Log.d(TAG, "Thread:" + Thread.currentThread() + "合并数据");
                return new Stu(s, stringIntegerHashMap.get("scoreMath"), stringIntegerHashMap.get("scoreEnglish"));
            }
        }).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<Stu>() {
            @Override
            public void accept(Stu stu) throws Exception {
                Log.d(TAG, "Thread:" + Thread.currentThread() + "收到最终数据");
                tvName.setText(stu.name);
                tvScoreEnglish.setText(stu.scoreEnglish + "");
                tvScoreMath.setText(stu.scoreMath + "");
            }
        });
    }
}
>>>输出，观察“发送姓名”和“发送成绩”的时间间隔为4秒，且不再同一线程中执行，说明他们是并行进行的。
14:12:35.455 20450-20516/com.ebm.rxjava D/TestOP: Thread:Thread[RxNewThreadScheduler-2,5,main]发送姓名
14:12:39.456 20450-20517/com.ebm.rxjava D/TestOP: Thread:Thread[RxNewThreadScheduler-3,5,main]发送成绩
14:12:39.456 20450-20517/com.ebm.rxjava D/TestOP: Thread:Thread[RxNewThreadScheduler-3,5,main]合并数据
14:12:39.461 20450-20450/com.ebm.rxjava D/TestOP: Thread:Thread[main,5,main]收到最终数据
```

#### 调度器

***RxJava操作符不直接和Threads或者ExecutorServices打交道，而是通过Schedulers(调度器)的通用API来实现对具体并发代码的抽象。RxJava通过工具类Schedulers提供了几个标准的调度方式***

RxJava operators don't work with `Thread`s or `ExecutorService`s directly but with so called `Scheduler`s that abstract away sources of concurrency behind a uniform API. RxJava 2 features several standard schedulers accessible via `Schedulers` utility class.

- `Schedulers.computation()`: 在后台的固定数量的专用线程上运行计算密集型工作。大多数异步操作符使用这个作为默认的调度器  

  Run computation intensive work on a fixed number of dedicated threads in the background. Most asynchronous operator use this as their default `Scheduler`.

- `Schedulers.io()`: 在一个动态线程集合上运行类似I/O或阻塞的操作。

  Run I/O-like or blocking operations on a dynamically changing set of threads.

- `Schedulers.single()`: 以顺序和FIFO方式在单个线程上运行工作。  

  Run work on a single thread in a sequential and FIFO manner.

- `Schedulers.trampoline()`: ???在一个参与线程中以顺序和FIFO方式运行工作，通常用于测试目的。 

  Run work in a sequential and FIFO manner in one of the participating threads, usually for testing purposes.

#### 其他

##### Consumer、Observer、Publisher

Consumer和Observer都是接口，只是Consumer中只有一个accept方法，而Observer中有onNex、onComplete、onError、onSubsribe(**Disposable** d)四个个方法。其中onSubscribe中的Disposable可以用来作为上下游的开关，调用Disposable.dispose()可以停止接收上游发送的事件。

Publisher能够提供一个无边际数量的对象序列，当收到订阅请求时发布这些无数量限制的元素。Flowable继承自Publisher。

##### onNext、onComplete、onError

​	这三个方法是**Emitter(发射器)接口**中定义的三个方法，分别表示事件发送、完成、异常。

- 上游可以发送无限个onNext, 下游也可以接收无限个onNext.
- 当上游发送了一个onComplete后, 上游onComplete之后的事件将会`继续`发送, 而下游收到onComplete事件之后将**`不再继续`**接收事件.
- 当上游发送了一个onError后,  上游onError之后的事件将`继续`发送, 而下游收到onError事件之后将`不再继续`接收事件.
- 上游可以不发送onComplete或onError.
- 最为关键的是onComplete和onError必须唯一并且互斥, 即不能发多个onComplete, 也不能发多个onError,  也不能先发一个onComplete, 然后再发一个onError, 反之亦然
- 注: 关于onComplete和onError唯一并且互斥这一点,  是需要自行在代码中进行控制, 如果你的代码逻辑中违背了这个规则, **并不一定会导致程序崩溃. ** 比如发送多个onComplete是可以正常运行的, 依然是收到第一个onComplete就不再接收了, 但若是发送多个onError, 则收到第二个onError事件会导致程序会崩溃。

##### 并行处理

在RxJava中并行处理，体现在在不同的独立的flow中进行计算的结果，最终被合并到一个flow中去。以flowMap为例：

```java
Flowable.range(1, 10)
  .flatMap(v -> Flowable.just(v).subscribeOn(Schedulers.computation()).map(w -> w * w) )
  .blockingSubscribe(v -> Log.d(TAG, String.valueOf(v)));
>>>输出：
05:58:40.317 D/TestOP: 1
05:58:40.317 D/TestOP: 9
05:58:40.317 D/TestOP: 25
05:58:40.317 D/TestOP: 49
05:58:40.318 D/TestOP: 81
05:58:40.318 D/TestOP: 4
05:58:40.318 D/TestOP: 16
05:58:40.320 D/TestOP: 36
05:58:40.321 D/TestOP: 64
05:58:40.321 D/TestOP: 100
```

##### 背压

**当上下游在不同的线程中，发射和接收数据的速率可能有所不同，为了避免超出预期的内存开销，或者为了满足跳过、舍弃某些数据的需求，所谓的背压就产生了**。背压是一种不同操作能够表达他们能处理多少数据的流控制机制。背压用来在不知道上游将会发送多少数据量的时候，用来限制内存开销的。RxJava的Flowable是专门用来处理背压的。

```java
Flowable.create(new FlowableOnSubscribe<String>(){
  @Override
  public void subscribe(FlowableEmitter<String> emitter) throws Exception {
    for(int i=0;i<1000;i++) {
      emitter.onNext(i + "");
    }
  }
}, BackpressureStrategy.ERROR).subscribeOn(Schedulers.newThread())
  .observeOn(Schedulers.newThread())
  .subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
      Thread.sleep(1000);
      Log.e(TAG,"处理数据" + s);
    }
  });
```

Flowable将数据对象发送到一个大小为**128**的缓冲区内，等待下游线程从该缓存区中取出数据对象进行处理后，等缓冲区有空闲位置时，再将数据对象放置到缓冲区内。如果此时，下游对象没有使用request(long n)从缓存区拉取数据，那么此时有几种不同的背压策略(BackpressureStrategy可供选择)：

```java
/**
 * Represents the options for applying backpressure to a source sequence.
 */
public enum BackpressureStrategy {
    /**
     * OnNext events are written without any buffering or dropping.
     * Downstream has to deal with any overflow.
     * <p>Useful when one applies one of the custom-parameter onBackpressureXXX operators.
     */
    MISSING,
    /**
     * Signals a MissingBackpressureException in case the downstream can't keep up.
     */
    ERROR,
    /**
     * Buffers <em>all</em> onNext values until the downstream consumes it.
     */
    BUFFER,
    /**
     * Drops the most recent onNext value if the downstream can't keep up.
     */
    DROP,
    /**
     * Keeps only the latest onNext value, overwriting any previous value if the
     * downstream can't keep up.
     */
    LATEST
}
```

##### RxJavaPlugins

API文档定义：Utility class to inject handlers to certain standard RxJava operations.

简单来说，<font color="#dd0000">**RxJavaPlugins提供了一种让我们通过hook的方式自定义RxJava行为的途径**</font>。首先查看该类的源码，可以看到所有的方法对象都是<font color="#dd0000">**static**</font>的，我们通过如下方式设置RxJavaPlugins的静态成员变量：

```java
RxJavaPlugins.setOnMaybeSubscribe(new BiFunction<Maybe, MaybeObserver, MaybeObserver>() {
  @Override
  public MaybeObserver apply(Maybe maybe, MaybeObserver maybeObserver) throws Exception {
    return new MyCustomMaybeObserver(maybeObserver); //这个maybeObserver就是我们真正的下游
  }
});
```

再来看一段Maybe的最终subscribe的源码：

```java
//Maybe.subscribe(MaybeObserver<? super T> observer)
public final void subscribe(MaybeObserver<? super T> observer) {
  ObjectHelper.requireNonNull(observer, "observer is null");
  observer = RxJavaPlugins.onSubscribe(this, observer);
  ObjectHelper.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null MaybeObserver. Please check the handler provided to RxJavaPlugins.setOnMaybeSubscribe for invalid null returns. Further reading: https://github.com/ReactiveX/RxJava/wiki/Plugins");

  try {
    subscribeActual(observer);
  } catch (NullPointerException ex) {
    throw ex;
  } catch (Throwable ex) {
    Exceptions.throwIfFatal(ex);
    NullPointerException npe = new NullPointerException("subscribeActual failed");
    npe.initCause(ex);
    throw npe;
  }
}
```

从上面可以看出，subscriberActural(observer)中的observer是经过RxJavaPlugins进行处理后得到的observer，再看RxJavaPlugins.onSubscribe(this,observer)代码：

```java
//RxJavaPlugins.onSubscribe(@NonNull Maybe<T> source, @NonNull MaybeObserver<? super T> observer)
public static <T> MaybeObserver<? super T> onSubscribe(@NonNull Maybe<T> source, @NonNull MaybeObserver<? super T> observer) {
  BiFunction<? super Maybe, ? super MaybeObserver, ? extends MaybeObserver> f = onMaybeSubscribe; //这个onMaybeSubscribe就是前面说的RxJavaPlugins的静态变量
  if (f != null) {
    return apply(f, source, observer);
  }
  return observer;
}
```

所以，可以把RxJavaPlugins理解为一个存储自定义observe行为的地方，在对Observerable、Maybe、Single等进行observe时，会判断RxJavaPlugins中是否定义过自定义的Observer，如果有就进行apply操作之后返回新的Observer，否则原样返回。

本段参考：[给初学者的RxJava2.0教程(十)](<https://www.jianshu.com/p/d6552d020307>)

##### RxJava与RxAndroid

+  RxAndroid是RxJava的一个针对Android平台的扩展，主要用于 Android 开发。
+  RxAndroid只是在RxJava的基础上添加了少数的几个类，目的是能够在Android中更简单的使用RxJava，**特别提供了可以指定到Android主线程的Scheduler**，除此之外还增加了基于RxJavaPlugins的扩展RxAndroidPlugins。

#### 操作符

操作符分为创建、转换、延时、条件、过滤这几大类。

##### 转换操作符

###### map

将输入对象经过Function<SourceType,TargetType>**从SourceType转换为TargetType**：

```java
Observable.fromArray(new String[]{"A","B","C"})
  	.map(new Function<String, String>() {
  	  @Override
  	  public String apply(String s) throws Exception {
  	    return s + "_" + s;
  	  }
	  }).subscribe(new Consumer<String>() {
 	 @Override
 	 public void accept(String s) throws Exception {
 	   System.out.println(s);
	 }
});
>>>输出：
A_A
B_B
C_C
```

###### flatMap

和map类似对流过的数据做转换，只是flatMap会将数据转换成ObservableSource类型的数据对象，**被转换的流被分散到多个并行执行的ObservableSource中，所以不能保证执行的顺序**。

```java
Flowable.range(1, 10)
    .flatMap(v -> Flowable.just(v).subscribeOn(Schedulers.computation()).map(w -> w * w) )
    .subscribe(new Consumer<Integer>() {
        @Override
        public void accept(Integer integer) throws Exception {
            Log.d(TAG, String.valueOf(integer));
        }
    });
>>>输出：
D/TestOP: 1
D/TestOP: 9
D/TestOP: 4
D/TestOP: 16
D/TestOP: 49
D/TestOP: 64
D/TestOP: 81
D/TestOP: 100
D/TestOP: 25
D/TestOP: 36
```

###### concatMap

```java
Flowable.range(1, 10)
    .concatMap(v -> Flowable.just(v).subscribeOn(Schedulers.computation()).map(w -> w * w) )
    .subscribe(new Consumer<Integer>() {
        @Override
        public void accept(Integer integer) throws Exception {
            Log.d(TAG, String.valueOf(integer));
        }
    });
D/TestOP: 1
D/TestOP: 4
D/TestOP: 9
D/TestOP: 16
D/TestOP: 25
D/TestOP: 36
D/TestOP: 49
D/TestOP: 64
D/TestOP: 81
D/TestOP: 100
```

###### zip

合并多个事件流(Observable)，生成一个新的事件流(Observable)。**最终合并的事件流的事件数有前面被合并的事件流中事件个数最少的决定**。举一个请求两个耗时数据的例子：

```java
void testZip() {
        TextView tvName = findViewById(R.id.tvName);
        TextView tvScoreEnglish = findViewById(R.id.tvScoreEnglish);
        TextView tvScoreMath = findViewById(R.id.tvScoreMath);

        Observable nameTask = Observable.create((ObservableOnSubscribe<String>)
                emitter -> {
            Thread.sleep(1000); //模拟耗时
            emitter.onNext("小王");
            Log.d(TAG, "Thread:" + Thread.currentThread() + "发送姓名");
        }).subscribeOn(Schedulers.newThread());
        Observable scoreTask = Observable.create((ObservableOnSubscribe<HashMap<String, Integer>>)
                emitter -> {
            Thread.sleep(5000); //模拟耗时
            HashMap map = new HashMap();
            map.put("scoreEnglish", 80);
            map.put("scoreMath", 95);
            Log.d(TAG, "Thread:" + Thread.currentThread() + "发送成绩");
            emitter.onNext(map);
        }).subscribeOn(Schedulers.newThread());
        Observable.zip(nameTask, scoreTask, (BiFunction<String, HashMap<String, Integer>, Stu>)
                (s, stringIntegerHashMap) -> {
            Log.d(TAG, "Thread:" + Thread.currentThread() + "合并数据");
            return new Stu(s, stringIntegerHashMap.get("scoreMath"), stringIntegerHashMap.get("scoreEnglish"));
        }).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread())
                .subscribe((Consumer<Stu>) stu -> {
            Log.d(TAG, "Thread:" + Thread.currentThread() + "收到最终数据");
            tvName.setText(stu.name);
            tvScoreEnglish.setText(stu.scoreEnglish + "");
            tvScoreMath.setText(stu.scoreMath + "");
        });
    }
```

###### merge

将2个并行进行的事件流合并到一个事件流中，按时间顺序执行。

```java
void testMerge() {
    Observable observable1 = Observable.interval(1000l,TimeUnit.MILLISECONDS).take(3);
    Observable observable2 = Observable.interval(2000l,TimeUnit.MILLISECONDS).take(3);
    Observable.merge(observable1,observable2).subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long i) throws Exception {
            Log.d(TAG,i.toString());
        }
    });
}
//输出：
17:42:42.555 D/TestOP: 0
17:42:43.554 D/TestOP: 1
17:42:43.555 D/TestOP: 0
17:42:44.555 D/TestOP: 2
17:42:45.555 D/TestOP: 1
17:42:47.555 D/TestOP: 2  
```

###### concat

```java
Observable observable1 = Observable.interval(1000l,TimeUnit.MILLISECONDS).take(3);
Observable observable2 = Observable.interval(2000l,TimeUnit.MILLISECONDS).take(3);
Observable.concat(observable1,observable2).subscribe(new Consumer<Long>() {
    @Override
    public void accept(Long i) throws Exception {
        Log.d(TAG,i.toString());
    }
});

>>>输出：
17:46:18.587 D/TestOP: 0
17:46:19.587 D/TestOP: 1
17:46:20.587 D/TestOP: 2
17:46:22.588 D/TestOP: 0
17:46:24.588 D/TestOP: 1
17:46:26.588 D/TestOP: 2
```

##### 延时操作符

###### delay

使事件流经过一段时间延迟后再开始执行

```java
void testDelay() {
  Log.d(TAG,"TestDelay");
  Observable.fromArray(new String[]{"A","B","C"})
    .delay(2000, TimeUnit.MILLISECONDS)
    .subscribe(s -> Log.d(TAG,s));
}
>>>输出：
05:53:34.615 D/TestOP: TestDelay
05:53:36.643 D/TestOP: A
05:53:36.643 D/TestOP: B
05:53:36.643 D/TestOP: C
```

###### interval

在经过最初的延时(initialDelay)开始从0L发送数据，每次经过一个时间间隔(period)，发送一个增大的数字。

Returns an Observable that emits a {@code 0L} after the {@code initialDelay} and ever increasing numbers after each {@code period} of time thereafter, on a specified {@link Scheduler}.

```java
Observable.interval(1000l,TimeUnit.MILLISECONDS).subscribe(new Consumer<Long>() {
  @Override
  public void accept(Long aLong) throws Exception {
    Log.d(TAG, String.valueOf(aLong));
  }
});
>>>输出：
0
1
2
4
5
6
...
```

###### skip

跳过事件序列的前n项，以interval的代码为基础，跳过前面10项

```java
Observable.interval(1000l,TimeUnit.MILLISECONDS).skip(10).subscribe(new Consumer<Long>() {
  @Override
  public void accept(Long aLong) throws Exception {
    Log.d(TAG, String.valueOf(aLong));
  }
});
>>>输出：
10
11
12
13
14
15
...
```

###### take

只取事件序列的前N项，以skip的代码为基础，跳过前10项后，取5项

```java
Observable.interval(1000l,TimeUnit.MILLISECONDS).skip(10).take(5)subscribe(new Consumer<Long>() {
  @Override
  public void accept(Long aLong) throws Exception {
    Log.d(TAG, String.valueOf(aLong));
  }
});
>>>输出：
10
11
12
13
14
```

###### defer

**直到有观察者(Observer)订阅时，才动态创建观察者对象(Observable) **

```java
int i = 1;
void testDefer() {
  Observable<Integer> observable = Observable
    .defer((Callable<ObservableSource<Integer>>) () -> Observable.just(i));
  i = 2;
  observable.subscribe(it -> Log.d(TAG,it.toString()));
}
>>>输出：2
```

##### 过滤操作符

###### filter

```java
Flowable.range(0,10).filter(new Predicate<Integer>() {
    @Override
    public boolean test(Integer integer) throws Exception {
        if(integer.intValue() % 2 == 0) {
            return true;
        }
        return false;
    }
}).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG,integer.toString());
    }
});
>>>输出：
0
2
4
6
8
```

##### 组合使用

###### 定时发送自定义事件

```java
//可以使用from+interval+zip实现
/**
     * 测试固定时间间隔发送自定义消息
     */
void testIntervalCustomMsg() {
    Observable.zip(Observable.fromArray(new String[]{"A","B","C"})
                   ,Observable.interval(3000l,TimeUnit.MILLISECONDS)
                   ,new BiFunction<String,Long,String>(){
                       @Override
                       public String apply(String s, Long aLong) throws Exception {
                           return s;
                       }
                   }).subscribe(new Consumer<String>() {
        @Override
        public void accept(String o) throws Exception {
            Log.d(TAG,o.toString());
        }
    });
}
```

最后，来一张别人整理的操作符大全图片(原地址：<https://www.jianshu.com/p/12883e1b59f9>)

![avatar](https://upload-images.jianshu.io/upload_images/10339856-d7820326a140706c.jpg)





参考：

[Rxjava操作符大全](https://www.jianshu.com/p/12883e1b59f9)

