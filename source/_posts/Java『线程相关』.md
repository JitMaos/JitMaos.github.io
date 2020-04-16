---
title: Java『线程相关』### Java线程
**线程是进程中实施调度和分派的基本单位**。一个进程可以包含多个线程，一个线程只能在一个进程的地址空间内活动。
##### 一、线程状态
​	线程有6种状态，new，runnable，blocked，waiting(等待其他线程执行结束)，timed_waiting(等待其他线程结束，设定超时)，terminated(Thread类中一个State枚举了线程的所有状态）。
#####  二、线程同步
1. 使用synchronized关键字修饰方法。由于java的<font color="#dd0000">**每个对象都有一个内置锁**</font>，当用此关键字修饰方法时， 内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。注： synchronized关键字也可以<font color="#dd0000">**修饰静态方法**</font>，此时如果调用该静态方法，将会<font color="#dd0000">**锁住整个类**</font>。
2. 即有synchronized关键字修饰的语句块。 被该关键字修饰的语句块会自动被加上内置锁，从而实现同步。
<!-- more -->
##### 三、如何正确的结束线程
参考:（https://www.jianshu.com/p/536b0df1fd55）
1. 最简单的方法使用设置一个用**volatile修饰的标志位**，**但这种方式有一个问题，那就是如果线程阻塞，该标志位就可能失效**；
2. Java提供了中断机制，包含是三个方法：
   ```
      public void interrupt()
      public boolean isInterrupted()
      public static boolean interrupted(); //清除中断标志，并返回原状态
   ```
3. 每个线程都有个boolean类型的中断状态。**当使用Thread的interrupt()方法时，线程的中断状态被设置为true。另外，当线程收到InterruptedExceptoin后，会重置线程的中断标志**。**Java中实现更好的工具类，ExecutorService扩展了Executor，提供了管理线程生命周期的关键能力，其中ExecutorService.submit返回了Future对象来描述一个线程任务，它有一个cancel()方法可以用来结束线程。
##### 四、生产者-消费者模型
  准确的说应该是**生产者-消费者-仓库**模型，该模型主要需要满足以下几点要求：
1. 生产者仅仅在仓储未满时候生产，仓库满了就停止生产。
 2. 消费者仅仅在仓储有产品时候才能消费，仓空则等待。
3. 当消费者发现仓储没产品可消费时候会通知生产者生产。
4. 生产者在生产出可消费产品时候，应该通知等待的消费者去消费。
参考：[Java生产者消费者的三种实现](<https://blog.csdn.net/xindoo/article/details/80004003>)
##### 五、线程池
参考：<https://www.cnblogs.com/dolphin0520/p/3932921.html>
JUC包中定义了线程池相关的类，可以线程池实现线程的复用，降低多并发时因频繁的创建、销毁线程带来的性能损耗。线程池的核心类是ThreadPoolExecutor，有四个构造方法分别对应四种不同的线程池策略。
```java
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue);
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
```
从上面几个构造方法中可以看到一些参数，了解这些参数的定义及规则有助于大致了解线程池的使用。
+ corePoolSize：**核心池的大小**，默认创建一个线程池后，该线程池不会创建任何线程。

+ maximumPoolSize：**线程池中最大的线程数。**

+ keepAliveTime：表示线程池中的线程最多空闲多久将被终止。默认情况下，只有当线程池中的线程数超过核心池的大小该参数才会生效，**而且默认情况下，只对线程池中除了核心池之外的线程起作用**。想要对核心池中的线程也起作用，可以使用allowCoreThreadTimeOut(true)方法设定。

+ unit：线程超时时间单位。

+ workQueue：任务的阻塞执行队列，一般有ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue。其中LinkedBlockingQueue和SynchronousQueue使用的多一些。

+ threadFactory：用于创建线程的工厂。

+ handler：表示当拒绝处理任务时的策略，有四个值：
  + AbortPolicy：丢弃任务并抛出RejectedExecutionException异常。
  + DiscardPolicy：也是丢弃任务，但是不报异常。
  + DiscardOldestPolicy：丢失队列最早的任务，然后重新尝试添加或执行任务。
  + CallerRunsPolicy：由调用线程处理该任务。
    总结：当我们通过execute(Runnable)方法向线程池中添加一个待执行的任务时，会判断当前核心池是否已经满了，**如果核心池满就将任务添加到工作队列**(根据选择不同的BlockingQueue有不同的排队规则)，**如果工作队列也满了，那么就再创建新的线程，直至线程数达到最大线程数**(maximumPoolSize)；如果此时还有任务要添加进来，而此时已经处理不了任务了，就只能拒绝任务，此时可以根据指定的handler来指定拒绝策略。

  Executors是一个JUC包中的Executor、ExecutorService、ThreadFactory的工具类。可以创建常用的四种ExecutorService：

  <font color="#dd0000">**newSingleThreadExecutor**</font>：单线程化线程池的优点，串行执行所有任务。**如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。**底层使用**LinkedBlockingQueue**作为工作队列，和fixedThreadPool不同的是<font color="#dd0000">**它外面包了一层FinalizableDelegatedExecutorService**</font>，这导致newSingleThreadExecutor<font color="#dd0000">**无法向下转型成ThreadPoolExecutor**</font>，从而确保了对象不被修改。

  <font color="#dd0000">**newFixedThreadPool**</font>：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。**线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。**底层使用**LinkedBlockingQueue**作为工作队列，LinkedBlockingQueue是一个无界的阻塞队列，<font color="#dd0000">**所以当任务的生成速度持续大于任务的处理速度。使用这种线程池，将造成大量阻塞，从而引发内存溢出等问题。**</font>

  <font color="#dd0000">**newCachedThreadPool**</font>：创建一个可缓存的线程池，<font color="#dd0000">**核心线程数等于0**</font>。**如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程**，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。底层使用**SynchronousQueue**作为工作队列，**SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另外一个线程已经准备好参与到交付过程中。**

  <font color="#dd0000">**newScheduledThreadPool**</font>：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。底层使用**DelayedWorkQueue**作为工作队列。
##### 六、同步经典问题
+ 读者、写者问题
  参考：https://blog.csdn.net/c275046758/article/details/50575407
  问题描述:设想一个飞机订票系统，其中有许多竞争的进程试图读写其中的数据。多个进程同时读取是可以接受的，但如果一个进程正在更新数据库，则所有的其他进程都不能访问数据库。即便是读操作也不行。
  Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。
+ 哲学家就餐问题
  参考：https://www.cnblogs.com/vettel/p/3438257.html
  问题描述：1965年由Dijkstra提出的一种线程同步的问题，假设一圆桌前坐着5位哲学家，两个人中间有一只筷子，桌子中央有面条。哲学家思考问题，当饿了的时候拿起左右两只筷子吃饭，必须拿到两只筷子才能吃饭。上述问题会产生死锁的情况，当5个哲学家都拿起自己右手边的筷子，准备拿左手边的筷子时产生死锁现象。
  解决办法：
  添加一个服务生，只有当经过服务生同意之后才能拿筷子，服务生负责避免死锁发生；每个哲学家必须确定自己左右手的筷子都可用的时候，才能同时拿起两只筷子进餐，吃完之后同时放下两只筷子；规定每个哲学家拿筷子时必须拿序号小的那只，这样最后一位未拿到筷子的哲学家只剩下序号大的那只筷子，不能拿起，剩下的这只筷子就可以被其他哲学家使用，避免了死锁。这种情况不能很好的利用资源。　
##### 七、其他
###### synchronized关键字
1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，**作用的对象是调用这个代码块的对象;**
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，**作用的对象是调用这个方法的对象；**
3. 修饰一个静态的方法，其作用的范围是整个静态方法，**作用的对象是这个类的所有对象；**
4. 修饰一个类，其作用的范围是synchronized后面括号括起来的部分，**作用主的对象是这个类的所有对象；**
5. synchronized同步代码块时，会有monitorenter指向同步代码块的开始位置，monitorexit指向同步代码块的结束位置；而当synchronized修饰同步方法时会有ACC_SYNCHRONIZED标识该方法。
###### volatile关键字
[ˈvɑ:lətl]易变的，不稳定的; （液体或油） 易挥发的
1. volatile是轻量级的synchronized，<font color="#dd0000">**它在多处理器开发中保证了共享变量的“可见性”**</font>。Java语言规范第三版中对volatile的定义如下： java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。那么volatile是如何来保证可见性的呢？被volatile继续的行，会向CPU发送一个lock前缀的指令，而该指令在多核处理器下会引发了两件事情。将当前处理器缓存行的数据会写回到系统内存。这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。
2. 在两种场景下不应该使用： 缓存行非64字节宽的处理器，如P6系列和奔腾处理器；共享变量不会被频繁的写。
###### synchronized和volatile的区别
1. <font color="#dd0000">**volatile本质是告诉JVM当前变量在寄存器(工作内存)中的值是不确定的，需要从主存中读取**</font>；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞。  
2. volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、类级别上。
3. **volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。 **
4. volatile仅能实现变量的修改可见性，**不能保证原子性**；而synchronized则可以保证变量的修改可见性和原子性。
参考：https://www.cnblogs.com/awkflf11/p/9218414.html
###### 显式锁ReentrantLock(可重入锁)
1. 与使用synchronized关键字由Java内部帮我们在调用方法之前和结束时加锁解锁来实现程序的原子性操作不同，显式锁是一种手动式的实现方式，程序员控制锁的具体实现，虽然现在越来越趋向于使用synchronized直接实现原子操作，但是了解了Lock接口的具体实现机制将有助于我们对synchronized的使用。

2. ```java
   public interface Lock {
       void lock() //调用该方法将获得一个锁的入口
       void lockInterruptibly() //该方法也是去获得一个锁，但是它是响应中断的，一旦在获取的过程中遭遇中断将抛出 InterruptedException。
       boolean tryLock(); //该方法尝试着去获得一个锁，如果获取失败将返回false，并不会阻塞当前线程
       boolean tryLock(long time, TimeUnit unit) //尝试着去获取一个锁，如果获取失败，将阻塞等待指定的时间，期间如果能够获得锁将返回true，否则返回false，响应中断请求。
       void unlock(); //释放一个锁
       Condition newCondition(); //条件变量
   }
   public ReentrantLock() {
       sync = new NonfairSync();
   }
   public ReentrantLock(boolean fair) { //参数 fair用于保证锁机制的公平策略，公平的策略会是的等待时间越长的线程优先获得锁。保证公平必然会降低性能，所以ReentrantLock默认并不保证公平
       sync = fair ? new FairSync() : new NonfairSync();
   }
   public static void park()  //调用park方法会使得当前线程丢失CPU使用权，从Runnable状态转变为Waiting状态。
   public static void parkNanos(long nanos) //parkNanos指定线程要等待的时间
   public static void parkUntil(long deadline) //指定线程要等待到什么时候，这个时间是一个绝对时间，相对于纪元的毫秒数。
   public static void unpark(Thread thread) //而unpark方法则反过来让Waiting状态的某个线程转变状态为Runnable，等待操作系统调度。
   ```
###### synchronized和ReentrantLock的比较
+ **两者都是可重入锁；**
+ synchronized依赖于JVM，而ReentrantLock依赖于API；相比于synchronized，ReentrantLock增加了一个功能：**等待可中断、可实现公平锁、可实现选择性通知(基于多Condition实现在特定时刻得到通知)。**
###### 守护线程
​	Java中有两类线程：User Thread(用户线程)、Daemon Thread(守护线程) 。用户线程即运行在前台的线程，而守护线程是运行在后台的线程。**守护线程作用是为其他前台线程的运行提供便利服务**，<font color="#dd0000">**而且仅在普通、非守护线程仍然运行时才需要**</font>，**比如垃圾回收线程就是一个守护线程**。当JVM检测仅剩一个守护线程，而用户线程都已经退出运行时，JVM就会退出，因为没有如果没有了被守护这，也就没有继续运行程序的必要了。如果有非守护线程仍然存活，JVM就不会退出。
​	守护线程并非只有虚拟机内部提供，用户在编写程序时也可以自己设置守护线程。用户可以用Thread的setDaemon（true）方法设置当前线程为守护线程。虽然守护线程可能非常有用，但必须小心确保其他所有非守护线程消亡时，不会由于它的终止而产生任何危害。因为你不可能知道在所有的用户线程退出运行前，守护线程是否已经完成了预期的服务任务。一旦所有的用户线程退出了，虚拟机也就退出运行了。 因此，<font color="#dd0000">**不要在守护线程中执行业务逻辑操作（比如对数据的读写等）**</font>。另外有几点需要注意：

1. **setDaemon(true)必须在调用线程的start()方法之前设置**，否则会抛出IllegalThreadStateException异常。
2. **在守护线程中产生的新线程也是守护线程。**
3. 不要认为所有的应用都可以分配给守护线程来进行服务，比如读写操作或者计算逻辑。

###### ThreadLocal 
**每个线程自带了成员变量ThreadLocalMap(ThreadLocalMap is a customized hash map suitable only for maintaining thread local values.)**

```java
//ThreadLocal.java
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
###### 乐观锁与悲观锁
+ 乐观锁**总是假设最好**的情况，每次去拿数据的时候都认为别人不会修改，**所以读操作不会上锁**，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用<font color="#dd0000">**版本号机制**</font>和<font color="#dd0000">**CAS算法**</font>实现。
+ 悲观锁**总是假设最坏**的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。
###### CAS概念
+ **介绍**：比较和交换(Compare And Swap)用于实现**无锁的多线程同步效果**，CAS需要有3个操作数：<font color="#dd0000">**内存地址V、旧的预期值A、即将要更新的目标值B**</font>。当且仅当内存地址V的值和预期值A相等时，将内存地址V的值修改为B。否则就拿到新的值，然后自旋重试，直至成功。Java中CAS代码多发在JUC的atomic包下，常见的类有：<font color="#dd0000">**AtomicBoolean、AtomicLong、AtomicLongArray**</font>等。
+ **不足**：循环时间长的时候开销大；**只能保证一个共享变量的原子操作**；**ABA问题**；
+ **ABA**问题：线程1读取了A，线程2读取了A，修改为B，再修改为A；线程1读取到A，认为没有别的线程改动过。解决办法：Java提供了AtomicStampedReference/AtomicMarkableReference，**在对象中额外再增加一个标记来标识对象是否有过变更。**
+ ##### 未分类~~
    1. 在线程安全性的定义中，最核心的概念就是正确性。正确性的含义是某个类的行为与其规范完全一致。
    2. 竞态条件，当某个计算的正确性取决于多个线程的交替执行时序是，就会发生竞态条件。
    3. 数据竞态，如果在访问共享的非final类型的域时没有采用同步来进行协同，那么就会出现数据竞态。如：当一个线程写入一个变量而另一个线程接下来读取这个变量，或者读取一个之前由另一个线程写入的变量时，并且再这两个线程之间没有使用同步，那么就可能出现数据竞争。
    4. 可以使用并发包下的AtomicLong.incrementAndGet()方法解决i ++ 线程不安全的问题。
    5. 内置锁，每个Java对象都可以用作一个实现同步的锁，这些锁被称为内置锁或者监视器锁。线程进入同步代码块之前会自动获得锁，并且在退出同步代码块时自动释放锁，获得内置锁的唯一途径就是进入这个由锁保护的同步代码块或方法。
    6. Java的并发包中有很多并发工具，ReentrantReadWriteLock，Semaphore，CountDownLatch，ReentrantLock等。这些工具有很多的共同特性，于是Java为我们抽象了一个类AbstractQueuedSynchronizer（AQS）来表示这些工具的共性。

##### 修改记录

1. 2019-09-03 ：添加几种系统自带线程池的工作队列的描述信息。