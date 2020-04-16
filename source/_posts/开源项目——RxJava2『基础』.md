---
title: RxJava2『基础』
date: 2019-04-08 14:27:58
tags: [编程语言,RxJava2]
---

### RxJava介绍

RxJava – Reactive Extensions for the JVM – a library for composing asynchronous and event-based programs using observable sequences for the Java VM.

RxJava – 基于JVM的响应式编程扩展 – 一个为Java虚拟机编写的基于可观察序列来实现异步及事件驱动编程的类库(**英语水平有限，翻译的不好，见谅**)。

It extends the [observer pattern](http://en.wikipedia.org/wiki/Observer_pattern) to support sequences of data/events and adds operators that allow you to compose sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization, thread-safety and concurrent data structures.

RxJava基于了观察者设计模式实现对一个数据或事件队列的观察，同时允许你在抽离一些抽象概念(如：低层次的线程，同步，线程安全，并发数据结构等)时指定操作(adds operators)，来显式(declaratively)的将多个序列整合在一起？？(翻译的是个啥，先不管了，先往下看，回过头再来整理)

### 一些专有名词

#### Upstream, downstream 上游，下游

The dataflows in RxJava consist of a source, zero or more intermediate steps followed by a data consumer or combinator step (where the step is responsible to consume the dataflow by some means):

RxJava中的数据流由一个源数据、零个或者多个中间步骤

```java
source.operator1().operator2().operator3().subscribe(consumer);
source.flatMap(value -> source.operator1().operator2().operator3());
```

Here, if we imagine ourselves on `operator2`, looking to the left towards the source, is called the **upstream**. Looking to the right towards the subscriber/consumer, is called the **downstream**. This is often more apparent when each element is written on a separate line:

现在，假设我们位于Operator 2，那么往左是上游，往右通向订阅者/消费者的是下游。

<!-- more -->

#### Objects in motion 流动中的对象

在RxJava文档中, **发射物**, **发射**, **项目**, **事件**, **信号**, **数据** 和**消息** 都是用来标识在数据流中流动的对象。

In RxJava's documentation, **emission**, **emits**, **item**, **event**, **signal**, **data** and **message** are considered synonyms and represent the object traveling along the dataflow.

#### Backpressure 背压

当数据流在多个异步行为中流动时，不同的操作有不同的速率。为了避免超出预期的内存开销，或者为了满足跳过、舍弃某些数据的需求，所谓的背压就产生了。背压是一种不同操作能够表达他们能处理多少数据的流控制机制。背压用来在不知道上游将会发送多少数据量的时候，用来限制内存开销的。

When the dataflow runs through asynchronous steps, each step may perform different things with different speed. To avoid overwhelming such steps, which usually would manifest itself as increased memory usage due to temporary buffering or the need for skipping/dropping data, a so-called backpressure is applied, which is a form of flow control where the steps can express how many items are they ready to process. This allows constraining the memory usage of the dataflows in situations where there is generally no way for a step to know how many items the upstream will send to it.

在RxJava中，<font color="#dd0000">**Flowable类是设计成专门用于处理背压的**</font>，而Observable是用来专门处理非背压操作的(短序列，UI交互等)。<font color="#dd0000">**???.其他的类型，单一的，可能的和可完成的类型不支持也不应该支持背压；总有地方能放下一个对象。**</font>

In RxJava, the dedicated `Flowable` class is designated to support backpressure and `Observable` is dedicated for the non-backpressured operations (short sequences, GUI interactions, etc.). The other types, `Single`, `Maybe` and `Completable` don't support backpressure nor should they; there is always room to store one item temporarily.

#### Assembly time 装配时

准备数据流时指定的各种中间操作发生在所谓的：**装配时**

The preparation of dataflows by applying various intermediate operators happens in the so-called **assembly time**:

```java
Flowable<Integer> flow = Flowable.range(1, 5)
.map(v -> v * v)
.filter(v -> v % 3 == 0)
;
```

此时，数据还未开始流转，并且没有

At this point, the data is not flowing yet and no side-effects are happening. 

#### Subscription time 订阅时

这是调用在流上调用subscribe()后的临时状态，它建立了处理步骤之间的连接

This is a temporary state when `subscribe()` is called on a flow that establishes the chain of processing steps internally:

```java
flow.subscribe(System.out::println)
```

这时订阅作用生效了，某些数据源阻塞或者马上开始发射数据。

This is when the **subscription side-effects** are triggered (see `doOnSubscribe`). Some sources block or start emitting items right away in this state.

#### Runtime 运行时

当处于运行时，数据流可以正常的发送数据、异常信号及完成信号

This is the state when the flows are actively emitting items, errors or completion signals: 

```java
Observable.create(emitter -> {
     while (!emitter.isDisposed()) {
         long time = System.currentTimeMillis();
         emitter.onNext(time);
         if (time % 2 != 0) {
             emitter.onError(new IllegalStateException("Odd millisecond!"));
             break;
         }
     }
}).subscribe(System.out::println, Throwable::printStackTrace);
```

事实上，上面这个方法的方法体执行的时候就是运行时。

Practically, this is when the body of the given example above executes.

### Simple background computation 简单后台计算

**一个使用RxJava最常见的场景是在后台线程进行一些计算、网络请求，然后在UI线程中显示结果**

One of the common use cases for RxJava is to run some computation, network request on a background thread and show the results (or error) on the UI thread:

```java
import io.reactivex.schedulers.Schedulers;

Flowable.fromCallable(() -> {
    Thread.sleep(1000); //  imitate expensive computation
    return "Done";
})
  .subscribeOn(Schedulers.io())
  .observeOn(Schedulers.single())
  .subscribe(System.out::println, Throwable::printStackTrace);

Thread.sleep(2000); // <--- wait for the flow to finish
```

**这种链式的方法调用方式成为流式API，和建造者模式很相似。不过RxJava的响应类型(reactive types)是不可变的；每个方法返回一个增加了更多行为的新的Flowable对象。**

This style of chaining methods is called a **fluent API** which resembles the **builder pattern**. However, RxJava's reactive types are immutable; each of the method calls returns a new `Flowable` with added behavior. To illustrate, the example can be rewritten as follows:

***

```java
Flowable<String> source = Flowable.fromCallable(() -> {
    Thread.sleep(1000); //  imitate expensive computation
    return "Done";
});

Flowable<String> runBackground = source.subscribeOn(Schedulers.io());

Flowable<String> showForeground = runBackground.observeOn(Schedulers.single());

showForeground.subscribe(System.out::println, Throwable::printStackTrace);

Thread.sleep(2000);
```

***通常情况下，你可以将一些计算操作合作密集的IO操作通过subscribeOn制定到其他线程，只要数据准备好了，你可以通过observeOn来指定前台线程或UI线程来处理。***

Typically, you can move computations or blocking IO to some other thread via `subscribeOn`. Once the data is ready, you can make sure they get processed on the foreground or GUI thread via `observeOn`.

### Schedulers 调度

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

这几个调度模型在所有JVM平台上都是可用的，但是一些特定的平台，比如Android，有他们自己的典型调度器。

These are available on all JVM platforms but some specific platforms, such as Android, have their own typical `Scheduler`s defined: `AndroidSchedulers.mainThread()`, `SwingScheduler.instance()` or `JavaFXSchedulers.gui()`.

此外，可以通过schedulers.from(Executor)来包装一个现有的Executor。比如可以实现数量巨大且数量固定的线程池(不同于computation()或io()的实现方式)

In addition, there is option to wrap an existing `Executor` (and its subtypes such as `ExecutorService`) into a `Scheduler` via `Schedulers.from(Executor)`. This can be used, for example, to have a larger but still fixed pool of threads (unlike `computation()` and `io()` respectively).

最后一行的Thread.sleep(2000)并不是不小心添加的，在RxJava中默认的调度器运行在守护线程上，这意味着只要Java主线程退出，所有这些默认的调度器会停止，同时后台计算也不会发生。这个例子中让线程sleep一点时间是**为了主线程存活时间久一点从而保证守护线程能够持续调度任务，让你能够看到console信息**。

The `Thread.sleep(2000);` at the end is no accident. In RxJava the default `Scheduler`s run on daemon threads, which means once the Java main thread exits, they all get stopped and background computations may never happen. Sleeping for some time in this example situations lets you see the output of the flow on the console with time to spare.

### Concurrency within a flow 流中的并发

RxJava中的流

Flows in RxJava are sequential in nature split into processing stages that may run **concurrently** with each other:

```
Flowable.range(1, 10)
  .observeOn(Schedulers.computation())
  .map(v -> v * v)
  .blockingSubscribe(System.out::println);
```

这个例子中数字1到10的平方运算被指定在计算线程中进行，而其结果被指定在主线程中使用。这个流中的lambda表达式v -> v * v不是并行进行的，而是从1到10一个一个在同一个计算线程中进行的。

This example flow squares the numbers from 1 to 10 on the **computation** `Scheduler` and consumes the results on the "main" thread (more precisely, the caller thread of `blockingSubscribe`). However, the lambda `v -> v * v` doesn't run in parallel for this flow; it receives the values 1 to 10 on the same computation thread one after the other.

### Parallel processing 并行处理

并行的处理数字1到10会稍微复杂一点：

Processing the numbers 1 to 10 in parallel is a bit more involved:

```
Flowable.range(1, 10)
  .flatMap(v ->
      Flowable.just(v)
        .subscribeOn(Schedulers.computation())
        .map(w -> w * w)
  )
  .blockingSubscribe(System.out::println);
```

事实上，并行在RxJava中意味着运行在独立的流中，然后再将他们的结果合并到同一个流中去。操作符flatMap会将首次map到的每一个数放置止到独立的Flowable中计算，然后再合并他们的结果。

Practically, parallelism in RxJava means running independent flows and merging their results back into a single flow. The operator `flatMap` does this by first mapping each number from 1 to 10 into its own individual `Flowable`, runs them and merges the computed squares.

注意，无论如何，flatMap不保证执行顺序，并且从内层flow得到最后的结果可能是交叉出现的。有其他能够保证执行顺序的操作符：

Note, however, that `flatMap` doesn't guarantee any order and the end result from the inner flows may end up interleaved. There are alternative operators:

+ concatMap每次只映射并结算一个inner flow

- `concatMap` that maps and runs one inner flow at a time and
- concatMatEager一次性运行所有的inner flows，但是输出flow会按照inner flows的创建顺序输出。
- `concatMapEager` which runs all inner flows "at once" but the output flow will be in the order those inner flows were created.

另外，Flowable.parllel()操作符和ParallelFlowable类型可以实现同样的并行处理方式：

Alternatively, the `Flowable.parallel()` operator and the `ParallelFlowable` type help achieve the same parallel processing pattern:

```
Flowable.range(1, 10)
  .parallel()
  .runOn(Schedulers.computation())
  .map(v -> v * v)
  .sequential()
  .blockingSubscribe(System.out::println);
```

### Dependent sub-flows 流的依赖处理

flatMap是一个强大的操作符，在很多场景下十分有效。比如，已有一个返回Flowable的service A，我们想要调用另一个使用service A的返回值的service B：

flatMap` is a powerful operator and helps in a lot of situations. For example, given a service that returns a `Flowable`, we'd like to call another service with values emitted by the first service:

```
Flowable<Inventory> inventorySource = warehouse.getInventoryAsync();

inventorySource.flatMap(inventoryItem ->
    erp.getDemandAsync(inventoryItem.getId())
    .map(demand 
        -> System.out.println("Item " + inventoryItem.getName() + " has demand " + demand));
  )
  .subscribe();
```

### Continuations 后续操作

有时当一个对象变得可用时，我们希望在它上面进行一些依赖操作。这就是所谓的后续操作，根据不同的情况以及不同的类型，可以使用多种操作符来完成后续操作。

Sometimes, when an item has become available, one would like to perform some dependent computations on it. This is sometimes called **continuations** and, depending on what should happen and what types are involved, may involve various operators to accomplish.

#### Dependent 依赖

最典型场景是给定一个值，去调用另外一个服务，等待并使用结果。

The most typical scenario is to given a value, invoke another service, await and continue with its result:

```
service.apiCall()
.flatMap(value -> service.anotherApiCall(value))
.flatMap(next -> service.finalCall(next))
```

通常情况下后续的操作序列依赖于早些时候的map结果。这可以通过将外层flatMap移动到前一个flatMap的内部来实现，比如：

It is often the case also that later sequences would require values from earlier mappings. This can be achieved by moving the outer `flatMap` into the inner parts of the previous `flatMap` for example:

```
service.apiCall()
.flatMap(value ->
    service.anotherApiCall(value)
    .flatMap(next -> service.finalCallBoth(value, next))
)
```

此时，原始的value在内部的flatMap也是可用的，这得益于lambda的变量捕获能力。 

Here, the original `value` will be available inside the inner `flatMap`, courtesy of lambda variable capture.

#### Non-dependent 非依赖

In other scenarios, the result(s) of the first source/dataflow is irrelevant and one would like to continue with a quasi independent another source. Here, `flatMap` works as well:

```
Observable continued = sourceObservable.flatMapSingle(ignored -> someSingleSource)
continued.map(v -> v.toString())
  .subscribe(System.out::println, Throwable::printStackTrace);
```

however, the continuation in this case stays `Observable` instead of the likely more appropriate `Single`. (This is understandable because from the perspective of `flatMapSingle`, `sourceObservable` is a multi-valued source and thus the mapping may result in multiple values as well).

Often though there is a way that is somewhat more expressive (and also lower overhead) by using `Completable` as the mediator and its operator `andThen` to resume with something else:

```
sourceObservable
  .ignoreElements()           // returns Completable
  .andThen(someSingleSource)
  .map(v -> v.toString())
```

The only dependency between the `sourceObservable` and the `someSingleSource` is that the former should complete normally in order for the latter to be consumed.

#### Deferred-dependent  延期依赖

Sometimes, there is an implicit data dependency between the previous sequence and the new sequence that, for some reason, was not flowing through the "regular channels". One would be inclined to write such continuations as follows:

```
AtomicInteger count = new AtomicInteger();

Observable.range(1, 10)
  .doOnNext(ignored -> count.incrementAndGet())
  .ignoreElements()
  .andThen(Single.just(count.get()))
  .subscribe(System.out::println);
```

Unfortunately, this prints `0` because `Single.just(count.get())` is evaluated at **assembly time** when the dataflow hasn't even run yet. We need something that defers the evaluation of this `Single` source until **runtime** when the main source completes:

```
AtomicInteger count = new AtomicInteger();

Observable.range(1, 10)
  .doOnNext(ignored -> count.incrementAndGet())
  .ignoreElements()
  .andThen(Single.defer(() -> Single.just(count.get())))
  .subscribe(System.out::println);
```

or

```
AtomicInteger count = new AtomicInteger();

Observable.range(1, 10)
  .doOnNext(ignored -> count.incrementAndGet())
  .ignoreElements()
  .andThen(Single.fromCallable(() -> count.get()))
  .subscribe(System.out::println);
```

### Type conversions 类型转换

Sometimes, a source or service returns a different type than the flow that is supposed to work with it. For example, in the inventory example above, `getDemandAsync` could return a `Single<DemandRecord>`. If the code example is left unchanged, this will result in a compile time error (however, often with misleading error message about lack of overload).

In such situations, there are usually two options to fix the transformation: 1) convert to the desired type or 2) find and use an overload of the specific operator supporting the different type.

#### Converting to the desired type 转换成目标数据类型

Each reactive base class features operators that can perform such conversions, including the protocol conversions, to match some other type. The following matrix shows the available conversion options:

|                 | Flowable      | Observable     | Single                                                       | Maybe                                          | Completable      |
| --------------- | ------------- | -------------- | ------------------------------------------------------------ | ---------------------------------------------- | ---------------- |
| **Flowable**    |               | `toObservable` | `first`, `firstOrError`, `single`, `singleOrError`, `last`, `lastOrError`1 | `firstElement`, `singleElement`, `lastElement` | `ignoreElements` |
| **Observable**  | `toFlowable`2 |                | `first`, `firstOrError`, `single`, `singleOrError`, `last`, `lastOrError`1 | `firstElement`, `singleElement`, `lastElement` | `ignoreElements` |
| **Single**      | `toFlowable`3 | `toObservable` |                                                              | `toMaybe`                                      | `ignoreElement`  |
| **Maybe**       | `toFlowable`3 | `toObservable` | `toSingle`                                                   |                                                | `ignoreElement`  |
| **Completable** | `toFlowable`  | `toObservable` | `toSingle`                                                   | `toMaybe`                                      |                  |

1: When turning a multi-valued source into a single valued source, one should decide which of the many source values should be considered as the result.

2: Turning an `Observable` into `Flowable` requires an additional decision: what to do with the potential unconstrained flow of the source `Observable`? There are several strategies available (such as buffering, dropping, keeping the latest) via the `BackpressureStrategy` parameter or via standard `Flowable` operators such as `onBackpressureBuffer`, `onBackpressureDrop`, `onBackpressureLatest` which also allow further customization of the backpressure behavior.

3: When there is only (at most) one source item, there is no problem with backpressure as it can be always stored until the downstream is ready to consume.

#### Using an overload with the desired type 

Many frequently used operator has overloads that can deal with the other types. These are usually named with the suffix of the target type:

| Operator    | Overloads                                                    |
| ----------- | ------------------------------------------------------------ |
| `flatMap`   | `flatMapSingle`, `flatMapMaybe`, `flatMapCompletable`, `flatMapIterable` |
| `concatMap` | `concatMapSingle`, `concatMapMaybe`, `concatMapCompletable`, `concatMapIterable` |
| `switchMap` | `switchMapSingle`, `switchMapMaybe`, `switchMapCompletable`  |

The reason these operators have a suffix instead of simply having the same name with different signature is type erasure. Java doesn't consider signatures such as `operator(Function<T, Single<R>>)` and `operator(Function<T, Maybe<R>>)` different (unlike C#) and due to erasure, the two `operator`s would end up as duplicate methods with the same signature.

### Operator naming conventions

Naming in programming is one of the hardest things as names are expected to be not long, expressive, capturing and easily memorable. Unfortunately, the target language (and pre-existing conventions) may not give too much help in this regard (unusable keywords, type erasure, type ambiguities, etc.).

#### Unusable keywords

In the original Rx.NET, the operator that emits a single item and then completes is called `Return(T)`. Since the Java convention is to have a lowercase letter start a method name, this would have been `return(T)` which is a keyword in Java and thus not available. Therefore, RxJava chose to name this operator `just(T)`. The same limitation exists for the operator `Switch`, which had to be named `switchOnNext`. Yet another example is `Catch` which was named `onErrorResumeNext`.

#### Type erasure

Many operators that expect the user to provide some function returning a reactive type can't be overloaded because the type erasure around a `Function<T, X>` turns such method signatures into duplicates. RxJava chose to name such operators by appending the type as suffix as well:

```
Flowable<R> flatMap(Function<? super T, ? extends Publisher<? extends R>> mapper)

Flowable<R> flatMapMaybe(Function<? super T, ? extends MaybeSource<? extends R>> mapper)
```

#### Type ambiguities

Even though certain operators have no problems from type erasure, their signature may turn up being ambiguous, especially if one uses Java 8 and lambdas. For example, there are several overloads of `concatWith` taking the various other reactive base types as arguments (for providing convenience and performance benefits in the underlying implementation):

```
Flowable<T> concatWith(Publisher<? extends T> other);

Flowable<T> concatWith(SingleSource<? extends T> other);
```

Both `Publisher` and `SingleSource` appear as functional interfaces (types with one abstract method) and may encourage users to try to provide a lambda expression:

```
someSource.concatWith(s -> Single.just(2))
.subscribe(System.out::println, Throwable::printStackTrace);
```

Unfortunately, this approach doesn't work and the example does not print `2` at all. In fact, since version 2.1.10, it doesn't even compile because at least 4 `concatWith` overloads exist and the compiler finds the code above ambiguous.

The user in such situations probably wanted to defer some computation until the `someSource` has completed, thus the correct unambiguous operator should have been `defer`:

```
someSource.concatWith(Single.defer(() -> Single.just(2)))
.subscribe(System.out::println, Throwable::printStackTrace);
```

Sometimes, a suffix is added to avoid logical ambiguities that may compile but produce the wrong type in a flow:

```
Flowable<T> merge(Publisher<? extends Publisher<? extends T>> sources);

Flowable<T> mergeArray(Publisher<? extends T>... sources);
```

This can get also ambiguous when functional interface types get involved as the type argument `T`.

#### Error handling

Dataflows can fail, at which point the error is emitted to the consumer(s). Sometimes though, multiple sources may fail at which point there is a choice whether or not wait for all of them to complete or fail. To indicate this opportunity, many operator names are suffixed with the `DelayError` words (while others feature a `delayError` or `delayErrors` boolean flag in one of their overloads):

```
Flowable<T> concat(Publisher<? extends Publisher<? extends T>> sources);

Flowable<T> concatDelayError(Publisher<? extends Publisher<? extends T>> sources);
```

Of course, suffixes of various kinds may appear together:

```
Flowable<T> concatArrayEagerDelayError(Publisher<? extends T>... sources);
```

#### Base class vs base type

The base classes can be considered heavy due to the sheer number of static and instance methods on them. RxJava 2's design was heavily influenced by the [Reactive Streams](https://github.com/reactive-streams/reactive-streams-jvm#reactive-streams) specification, therefore, the library features a class and an interface per each reactive type:

| Type                  | Class         | Interface           | Consumer              |
| --------------------- | ------------- | ------------------- | --------------------- |
| 0..N backpressured    | `Flowable`    | `Publisher`1        | `Subscriber`          |
| 0..N unbounded        | `Observable`  | `ObservableSource`2 | `Observer`            |
| 1 element or error    | `Single`      | `SingleSource`      | `SingleObserver`      |
| 0..1 element or error | `Maybe`       | `MaybeSource`       | `MaybeObserver`       |
| 0 element or error    | `Completable` | `CompletableSource` | `CompletableObserver` |

1The `org.reactivestreams.Publisher` is part of the external Reactive Streams library. It is the main type to interact with other reactive libraries through a standardized mechanism governed by the [Reactive Streams specification](https://github.com/reactive-streams/reactive-streams-jvm#specification).

2The naming convention of the interface was to append `Source` to the semi-traditional class name. There is no `FlowableSource` since `Publisher` is provided by the Reactive Streams library (and subtyping it wouldn't have helped with interoperation either). These interfaces are, however, not standard in the sense of the Reactive Streams specification and are currently RxJava specific only.







参考：

<https://github.com/ReactiveX/RxJava>

<https://www.jianshu.com/p/464fa025229e>