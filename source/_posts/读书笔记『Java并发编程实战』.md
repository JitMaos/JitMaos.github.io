---
title: 读书笔记『Java并发编程实战』
tags: [读书笔记,Java]
---

### 第一章 简介

**P1**：在不同的**进程**之间可以通过一些**粗粒度**的通讯机制来完成交换数据，包括：套接字、信号处理器、共享内存、信号量以及文件等。

**P7**：安全性的含义是"永远不发生糟糕的事情"，而**活跃性**则关注于另一个目标，即**某件正确的事情最终会发生**。当某个操作无法继续执行下去时，就会发生活跃性问题。

**P8**：在Servlet规范中，Servlet同样需要满足被多个线程同事调用，换句话说，**Servlet需要是线程安全的。**

### 第二章 线程安全性

**P11**：要编写线程安全的代码，其核心在于**要对状态访问操作进行管理**，特别是对**共享的**和**可变的**状态的访问。

**P13**：在线程安全性的定义中，最核心的概念就是**正确性**。正确性的含义是，<font color="#dd0000">**某个类的行为与其规范完全一致**</font>。

**P15**：当某个计算的正确性取决于多个线程**交替执行时序**时，那么久就会发生<font color="#dd0000">**竞态条件**</font>。

**P16**：竞态条件的本质——**基于**一种可能<font color="#dd0000">**失效的观察结果**</font>来做出判断或者执行某个计算。

**P18**：在java.util.concurrent.atomic包中包含了一些**原子变量类**，用于实现在**数值**和<font color="#dd0000">**对象引用**</font>上的原子状态转换。如：**AtomicLong**，**AtomicReference**。

<!-- more -->

**P19**：AtomicLong是一种替代long类型整形的线程安全类，类似地，AtomicReference是一种替代**对象引用**的线程安全类。<font color="#00dd6d">测试代码：</font>

```java
class TestAtomicClass {

    static int num = 0;
    static AtomicInteger safeNum = new AtomicInteger(0);

    public static void main(String[] args) {
        for(int i=0;i<100;i++) {
            if(i % 2 == 0) {
                new Thread(new RunnableMinus()).start();
            } else {
                new Thread(new RunnalbeAdd()).start();
            }
        }
    }

    static class RunnalbeAdd implements Runnable {

        int times = 100;
        @Override
        public void run() {
            while(times > 0) {
                System.out.println("n+:" + num++);
//                System.out.println("n+:" + safeNum.incrementAndGet()); //线程安全操作
                times -- ;
            }
        }
    }

    static class RunnableMinus implements Runnable {

        int times = 100;
        @Override
        public void run() {
            while(times > 0) {
                System.out.println("n-:" + num--);
//                System.out.println("n-:" + safeNum.decrementAndGet()); //线程安全操作
                times --;
            }
        }
    }
}
```

**P20**：Java 提供了一种**内置**的锁机制来支持原子性：<font color="#dd0000">**同步代码块**</font>。同步代码块包括两部分：一个作为锁的**对象引用**，一个作为由这个锁保护的**代码块**。每个Java对象都可以用做一个实现同步的锁，这些锁被称为<font color="#dd0000">**内置锁**</font>或<font color="#dd0000">**监视器锁**</font>。**线程在进入同步代码块之前会自动获得锁，并且在退出同步代码块时自动释放锁。**

**P21**：**重入**意味着获取锁的操作的粒度是**线程**，而不是**调用**。当线程请求一个未被持有的锁是，JVM将记下锁的持有者，并且将获取计数值置为1。如果同一个线程再次获取这个锁，计数值将递增，而当线程退出同步代码块时，计数器将会响应的递减。当计数器为0时，这个锁将被释放。<font color="#dd0000">**注意：这里的重入指的是一个同步代码块中可以调用另一个同步代码块。而不是反复进入一个方法。**</font><font color="#00dd6d">测试代码：</font>

```java
/**
 * 测试重入机制，如果无法重入，那么同一个锁下不同的同步代码之间的相互调用将发生死锁
 * Created by wanly on 2019-08-24
 */
class TestReEnter {
    static TestReEnter testReEnter;

    public static void main(String[] args) {
        testReEnter = new TestReEnter();
        Thread t1 = new Thread("t1") {
            @Override
            public void run() {
                super.run();
                testReEnter.doSomething1(2000);
            }
        };
        t1.start();
    }
    public  synchronized void doSomething1(long sleepTime) {
        System.out.println(Thread.currentThread() + "|" + "doSomething1.start");
        try {
            Thread.sleep(sleepTime);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        doSomething2();
        System.out.println(Thread.currentThread() + "|" + "doSomething1.start");
    }

    public synchronized void doSomething2() {
        System.out.println(Thread.currentThread() + "|" + "doSomething2.start");
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread() + "|" + "doSomething2.end");
    }
}
```

**扩展**：Java中每一个对象都可以作为锁，Synchronized主要有三种使用方法：

+ 普通同步方法（实例方法），锁是**当前实例对象** ，进入同步代码前要获得当前实例的锁
+ 静态同步方法，锁是当前**类的class对象** ，进入同步代码前要获得当前类对象的锁
+ 同步方法块，锁是**括号里面的对象**，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

### 第三章 对象的共享

**P27**：<font color="#dd0000">**内存可见性**</font>：我们不仅希望防止某个线程正在使用对象状态而另一个线程在同时修改该状态，**而且希望确保当前一个线程修改对象状态后，其他线程能够看到发生的状态变化。**

**扩展**：我们知道一般计算机CPU有会有几级缓存，而主存的存储空间往往是有限的，所以线程在工作时，会将他需要用到的数据对象放到自己的工作内存中进行读写。这样多线程情况下，每个线程都有自己的工作内存，从而导致**可见性问题**，即线程A在他的工作内存在修改了变量m的值，线程B在他的工作内存中无法**及时**知道m的改变。

**P28**：<font color="#dd0000">**重排序**</font>：在没有同步的情况下，编译器、处理器以及运行时等都有可能对操作的执行顺序进行一些意想不到的调整。

**P29**：Java内存模型要求，变量的读取操作和写入操作都必须是原子操作。但对于**非volatile类型的long和double**变量，JVM允许将64位的读操作和写操作分解为2个32位的操作。

**P31**：当把变量声明为volatile类型后，编译器与运行时都会注意到这个变量是共享的，因此<font color="#dd0000">**不会将该变量上的操作与其他内存操作一起重排序**</font>。volatile变量<font color="#dd0000">**不会被缓存在寄存器或者对其他处理器不可见的地方**</font>，因此读取volatile类型的变量时总会返回最新写入的值。

**扩展**：通常我们写单例模式时，有多种方式，比如：懒汉模式、饿汉模式、委托(静态私有内部类)、双重检测锁等。一般我们使用的最多的是双重检测锁来实现(**虽然这本书说JDK 1.6以后双重检测并不是一种好方法**)，anyway。这里讲一下双重检测和volatile的关系。<font color="#dd0000">**那就是如果忘了使用volatile修饰单例对象，由于存在指令重排序的可能，仍然有可能造成错误。**</font>

**P34**：当对象在其构造函数中创建一个线程时，无论是显式创建(通过将它传给构造函数)还是隐身创建(由于Thread或Runnable是该对象的一个内部类)，<font color="#dd0000">**this引用都会被新创建的线程共享**</font>。

**P35**：一种避免使用同步的方式就是不共享数据。如果仅在单线程内访问数据， 就不需要同步。这种技术被称为**线程封闭(Thread Confinement)**，它是实现线程安全性的最简单方式之一。**栈封闭**是一种特殊的线程封闭。

**P37**：维持线程封闭性的一种更规范的方法是使用ThreadLocal，这个类**能够使线程中的某个值于保存值的对象关联起来。**<font color="#dd0000">**get总是返回由当前执行线程在调用set时设置的最新值**</font>。

**扩展**：我们通常直接打交道的API是ThreadLocal，但实际上每个线程用于真正存储共享变量"副本"的是ThreadLocalMap，它是一个数组，其中的元素是Entry<ThreadLocal，Value>。

**P43**：要安全的发布一个对象，对象的引用以及对象的状态必须同时对其他线程可见。一个正确的构造的对象可以通过以下方式来安全发布：

+ 在静态初始化函数中初始化一个对象引用。
+ 将对象的引用保存到volatile类型的域对象AtomicReference对象中。
+ 将对象的引用保存到某个正确构造函数的final类型域中。
+ 将对象的引用保存到一个由锁保护的域中。

### 第四章 对象的组合

**P49**：实例封闭：如果某个对象不是线程安全的，那么可以通过多种技术使其在多线程程序中安全地使用，你可以确保该对象只能有单个线程方案(线程封闭)，或者通过一个锁来保护该对象的所有访问。

**P59**：在Java中有很多线程封闭的类库，其中有些类的唯一用途就是将非线程安全的类转换成线程安全的类，如：

Collections.synchronizedList(List<E> list)、Collections.synchronizedSet(Set<E> s)等。

**P61**：非线程安全的**putIfAbsent**方法，<font color="#00dd6d">代码演示：</font>

```java
class ListHelper<E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());
    public synchronized boolean putIfAbsent(E x) {
        boolean absent = !list.contains(x);
        if(absent) {
            list.add(x);
        }
        return absent;
    }
}
```

虽然容器list是线程安全的，putIfAbsent也是同步的，但是**putIfAbsent和list上的锁对象不是同一个锁对象**。这意味着<font color="#dd0000">**putIfAbsent相对于List的其他操作来说并不是原子的，因此就无法确保党putIfAbsent运行时另一个线程不会修改链表**</font>。

### 第五章 基础构建模块

**P68**：设计同步容器类的**迭代器**时并没有考虑到**并发修改**的问题，**并且他们表现出的行为是'及时失败(fail-fast)'的。**这意味着，当他们发现容器在迭代过程中被修改时，就会抛出一个ConcurrentModificationException异常。这是一种类似"善意地"捕获并发异常。<font color="#00dd6d">示例代码：</font>

```java
public void testForConcurrent() {
        Vector<String> vector = new Vector<>();
        vector.add("AA");
        //尽管遍历的是线程安全类Vector，但仍然不能保证在迭代size()时是安全的(可能抛出数组边界越界异常)
        for(int i=0;i<vector.size();i++) {
            System.out.println(vector.get(i));
        }
    }
```

**P69**：有些情况下，迭代器会隐藏起来。编译器将字符串的连接操作转换为调用StringBuilder.append(Object)，而这个方法<font color="#dd0000">**又会调用容器的toString方法，标准容器的toString方法将迭代容器，并在每个元素上调用toString来生成容器内容的格式化表示**</font>，<font color="#00dd6d">示例代码：</font>

```java
class HideenIterator{
        final Set<Integer> set = new HashSet<>();
        public synchronized void add(Integer i) {set.add(i);}
        public synchronized void remove(Integer i) {set.remove(i);}
        public void addTenThings() {
            Random r = new Random();
            for (int i=0;i<10;i++) {
                add(r.nextInt());
            }
            //这里会迭代遍历set中的每一个元素，用来生成格式化内容
            System.out.println(set);
        }
    }
```

**P70**：<font color="#dd0000">**容器**</font>的**hashCode**和**equals**等方法也会间接的执行迭代操作，**当容器作为另一个容器的元素或健值时，就会出现这种情况**。 同样，contiainsAll、removeAll和retainAll等方法，以及把同步容器作为参数的构造函数，都会对容器进行迭代。在Java 5.0中新增了ConcurrentHashMap用以替代同步且基于散列的Map，以及CopyOnWriteArrayList，用于在遍历操作为主要操作的情况下替代同步的List。

**P71**：Java 5.0新增了Queue和BlockingQueue。Queue用来**临时**保存一组**等待处理**的元素。Queue上的操作不会阻塞，如果列表为空，那么获取元素的 操作将返回空值。<font color="#dd0000">**事实上，Queue是通过LinkedList实现的，之所以还需要一个Queue，是因为它能去除List的随机访问需求从而实现更高效的并发。**</font>BlockingQueue扩展了Queue，增加了<font color="#dd0000">**可阻塞**</font>的插入和获取等操作。

**P71**：**ConcurrentHashMap**并不是将每个方法都在同一个锁上同步并使得每次只有一个线程访问容器，而是使用一种粒度更细的加锁机制来实现更大程度的共享，这种机制称为分段锁(Lock Striping)。ConcurrentHashMap与其他并发容器提供的迭代器不会抛出ConcurentModificationException，因此不需要再迭代过程中对容器加锁。尽管有这些改进，但仍然有一些需要权衡的因素。<font color="#dd0000">**对于一些需要在整个Map上进行的计算的方法，例如size和isEmpty，这些方法的语义被忽略减弱了以反映容器的并发特性。**</font>实际上它们只是一个<font color="#dd0000">**估计值**</font>。

**CopyOnWriteArrayList**：用于替代同步List，子某些情况下它提供更好的并发性能，并且在**迭代期间**不需要对容器进行加锁或复制。

**P73**：<font color="#dd0000">**写入时复制容器的线程安全性在于，只要正确地发布一个事实不可变的对象，那么在访问该对象时就不再进一步的同步，在每次修改时，都会创建并更新发布一个新的容器副本，从而实现可变性。**</font>仅当**迭代**操作远远多于**修改**操作时，才应该使用**写入时复制**容器。这个准则很好地描述了许多事件通知系统：在分发通知时需要迭代已注册监听器链表，并调用每一个监听器，在大多数情况下，注册和注销事件监听器的操作远小于接受事件通知的操作。

阻塞队列提供了<font color="#dd0000">**可阻塞**</font>的put和take方法，以及支持<font color="#dd0000">**定时**</font>的offer和poll方法。

**P74**：在类库中包含了BlockingQueue的多种实现，其中LinkedBlockingQueue和ArrayBlockingQueue是FIFO队列，二者分别与LinkedList和ArrayList相似，但比同步List拥有更好的并发性能。

PriorityBlockingQueue是一个按优先级排序的队列，当你希望按照某种顺序而不是FIFO来处理元素时，这个队列将非常有用。

最后一个BlockingQueue实现是SynchronousQueue，<font color="#dd0000">**实际上它并不是一个真正的队列，因为它不会为队列中元素维护存储空间。与其他队列不同的是，它维护一组线程，这些线程在等待着把元素加入或移除队列。**</font>这种实现队列的方式看似很奇怪，但由于可以直接交付工作，从而降低了将数据从生产者移动到消费者的延迟。直接交付方式还会将更多关于任务状态的信息反馈给生产者(任务交付成功，或失败)。**因为SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另外一个线程已经准备好参与到交付过程中。仅当有足够多的消费者，并且总是有一个消费者准备好获取交付的工作时，才适合使用同步队列**

**P77**：Java6新增了两种容器类型，Deque(发音为"deck")和BlockingDeque，他们分别对Queue和BlockingQueue进行了扩展。Deque是一个<font color="#dd0000">**双端**</font>队列，**实现了在队列头和队列尾的高效插入和移除。具体实现包括ArrayDeque和LinkedBlockingDeque。**双端队列适用于<font color="#dd0000">**工作密取(Work Stealing)**</font>，工作密取非常适用于**既是生产者又是消费者问题——当执行某个工作时可能导致更多的工作。**<font color="#dd0000">**例如：在网页爬虫程序中处理一个页面时，通常会发现有更多的页面需要处理。**</font>
**扩展**：工作窃取(work-stealing)算法是<font color="#dd0000">**指某个线程从其他队列里窃取任务来执行**</font>。一个大任务分割为若干个互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并未每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。比如线程1负责处理1队列里的任务，2线程负责2队列的。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务待处理。干完活的线程与其等着，不如帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们可能会访问同一个队列，所以为了**减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务线程永远从双端队列的尾部拿任务执行。**

本节转自：[工作窃取算法 work-stealing](<https://blog.csdn.net/pange1991/article/details/80944797>）

**P78**：当某方法抛出InterruptedException时，表示该方法是一个**阻塞方法**，如果这个方法被中断，那么它将<font color="#dd0000">**努力提前结束阻塞状态**</font>。Thread提供了interrupt方法，用于**中断线程**或者**查询**线程是否已经被中断。每个线程都有一个**布尔类型的属性**，表示线程的**中断状态**，当中断线程时将设置这个状态。

中断是一种<font color="#dd0000">**协作机制**</font>。一个线程**不能强制**其他线程停止正在执行的操作而去执行其他操作。 当线程A中断B时，A<font color="#dd0000">**仅仅是要求**</font>B在执行到某个可以暂停的地方停止正在执行的操作——前提是如果线程B愿意停下来。虽然在API或者语言规范中并没有为中断定义任何特定应用级别的语义，**但最常使用中断的情况就是取消某个操作**。<font color="#00dd6d">示例代码：</font>

```java
public static void main(String[] args) {
        Thread t1 = new Thread("t1"){
            @Override
            public void run() {
                super.run();
                try {
                    System.out.println("线程1开始休眠---");
                    Thread.sleep(5000);
                    System.out.println("线程1中断状态检测1:" +
                            Thread.currentThread().isInterrupted());
                } catch (InterruptedException e) {
                    System.out.println("线程1休眠被中断---");
                    System.out.println("线程1中断状态检测2:" +
                            Thread.currentThread().isInterrupted());
                    Thread.currentThread().interrupt();
                    System.out.println("线程1中断状态检测3:" +
                            Thread.currentThread().isInterrupted());
                }
                System.out.println("线程1休眠结束---");
            }
        };
        t1.start();

        new Thread() {
            @Override
            public void run() {
                super.run();
                try {
                    System.out.println("线程2开始执行");
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                t1.interrupt();
                System.out.println("线程2执行结束");

            }
        }.start();
    }
//输出：
线程1开始休眠---
线程2开始执行
线程2执行结束
线程1休眠被中断---
线程1中断状态检测2:false
线程1中断状态检测3:true
线程1休眠结束---
```

**P79**：所有的同步工具类都包含一些特定的结构化属性：它们**封装**了一些状态，这些状态将决定执行同步工具类的线程时**继续执行**还是**等待**，此外还提供了一些**方法**对状态进行操作，以及另一些方法用于高效地等待同步工具类进入到预期状态。同步工具类有堵塞队列(BlockingQueue)、信号量(Semaphore)、栅栏(Barrier)、闭锁(Latch)等。

<font color="#dd0000">**闭锁(Latch)**</font>是一种同步工具类，可以**延迟线程的进度**直到到达终止状态。闭锁相当于一扇门：在闭锁到达**结束状态**之前，这扇门一直是<font color="#dd0000">**关闭**</font>的，并且没有任何线程能通过，当到达**结束**状态时，这扇门会**打开并允许所有的线程通过**。<font color="#dd0000">**当闭锁达到结束状态后，将不会再改变状态**</font>，因此这扇门将**永远保持打开**状态。<font color="#dd0000">**闭锁可以用来确保某些活动直到其它活动都完成后才继续执行。**</font>例如：

+ 确保某个计算在其需要的所有资源都被初始化之后才继续执行。
+ 确保某个服务在其依赖的所有其它服务都已经启动之后才启动。
+ **等待直到某个操作的所有参与者(例如，在多玩家游戏中所有的玩家)都就绪在继续执行。这种情况下，当所有玩家都准备就绪时，闭锁就达到结束状态。**

<font color="#dd0000">**CountDownLatch**</font>是一种灵活的闭锁实现，可以在上述各种情况中使用，它可以使**一个或多个线程等待一组时间发生。**闭锁状态包括一个计数器，该计数器被**初始化为一个正数，表示需要等待的事件数量**。countDown方法递减计数器，表示一个事件已经发生了，**而await方法等待计数器达到零，这表示所有需要等待的事件都已经发生**。如果计数器的值非零，那么await会一直阻塞知道计数器为零，或者等待中的线程中断，或者<font color="#dd0000">**等待超时**</font>。<font color="#00dd6d">示例代码：</font>

```java
 public long timeTasks(int nThreads,final Runnable task)
            throws InterruptedException {
        final CountDownLatch startGate = new CountDownLatch(1);
        final CountDownLatch endGate = new CountDownLatch(nThreads);
        for(int i=0;i<nThreads;i++) {
            Thread t = new Thread() {
                @Override
                public void run() {
                    super.run();
                    try {
                        //在线程启动后，阻塞在这里，等待闭锁开关打开
                        startGate.await();
                        try {
                            //这里直接调用Runnalbe.run方法，没有新起线程
                            task.run();
                        } finally {
                           endGate.countDown();
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            t.start();
        }
        long start = System.nanoTime();
        //打开闭锁开关，使等待在上面的线程能够继续执行
        startGate.countDown();
        endGate.await();
        long end = System.nanoTime();
        return end - start;
    }
```

**P80**：FutureTask也可以用做闭锁。(FutureTask实现了Future语义，表示一种**抽象**的<font color="#dd0000">**可生成结果**</font>的计算)。FutureTask表示的计算是通过Callable来实现的，相当于一种可生成结果的Runnable。Future.get的行为取决于任务的状态，**如果任务已经完成，那么get会立即返回结果，否则get将阻塞知道任务进入完成状态，然后返回结果或者抛出异常**。

**扩展**：FutureTask继承RunnableFuture接口，而RunnableFuture接口又继承Runnable和Future接口。Future接口中包含get()、cancel()、isCancelled()、isDonw()等方法。**使用FutureTask可以在线程A中计算结果，在线程B中获取计算值。**

**P82**：计数信号量(Counting Semapore)用来<font color="#dd0000">**控制同时访问某个特定资源的操作数量，或者同时执行某个指定操作的数量**</font>。计数信号量还可以用来实现某种资源池，或者对容器施加边界。Semaphore中管理着一组**虚拟的许可**，许可的初始数量可通过构造函数来指定。在执行操作时可以**首先获得许可(只要还有剩余的许可)**，并在使用以后**释放**许可。<font color="#dd0000">**如果没有许可，那么acquire将阻塞直到有许可(或者直到被中断或者操作超时）**</font>，release方法将返回一个许可给信号量。计算信号量的一种**简化形式**是二值信号量，即初始值为1的Semaphore。二值信号量可以用作互斥体(mutex)，并具备<font color="#dd0000">**不可重入**</font>的加锁语义：谁拥有这个唯一的许可，谁就拥有了互斥锁。<font color="#00dd6d">示例代码：</font>

```java
/**
 * 使用信号量为容器设置边界
 */
public class TestBoundedHashSetWithSemaphore {
    public static void main(String[] args) {
        BoundedHashSet<String> boundedHashSet = new BoundedHashSet<>(1);
        boundedHashSet.add("A");
        System.out.println("1.Set.size=" + boundedHashSet.size());

        new Thread(){
            @Override
            public void run() {
                super.run();
                boundedHashSet.add("B");
                System.out.println("2.Set.size=" + boundedHashSet.size());
            }
        }.start();
    }
}
class BoundedHashSet<T> {
    private final Set<T> set;
    private final Semaphore sem;
    public BoundedHashSet(int bound) {
        this.set = Collections.synchronizedSet(new HashSet<T>());
        sem = new Semaphore(bound);
    }

    public int size() {
        if(set == null) {
            return 0;
        } else {
            return set.size();
        }
    }
    public boolean add(T o) {
        try {
            sem.acquire(); //请求许可
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        boolean wasAdded = false;
        try {
            wasAdded = set.add(o);
            System.out.println("插入元素：" + o.toString());
            return wasAdded;
        } finally {
            if(!wasAdded) {
                sem.release();
            }
        }
    }
    public boolean remove(Object o) {
        boolean wasRemoved = set.remove(o);
        System.out.println("移除元素：" + o.toString());
        if(wasRemoved) {
            sem.release();
        }
        return wasRemoved;
    }
}
```

**P83**：<font color="#dd0000">**栅栏（zha lan)**</font>类似于闭锁，它能阻塞一组线程直到某个事件发生。栅栏与闭锁的**关键区别**在于，<font color="#dd0000">**所有线程必须同时到达栅栏位置，才能继续执行。闭锁用于等待事件，而栅栏用于等待其他线程**</font>。栅栏用于实现一些协议，例如几个家庭决定在某个地方集合：“所有人6:00在麦当劳碰头，到了以后要等其他人，之后再讨论下一步要做的事情。”

CyclicBarrier可以使一定数量的参与方**反复**地在栅栏位置汇集，它在**并行迭代算法**中非常有用：这种算法通常将一个问题拆分成一系列相互独立的子问题。当线程到达栅栏位置时将调用await方法，这个方法将阻塞直到所有线程到达栅栏位置。如果所有线程都到达栅栏位置，那么栅栏将打开，此时所有线程都被释放，而栅栏将被<font color="#dd0000">**重置**</font>以便下次使用。如果对await的调用超时，或者await阻塞的线程被中断，那么栅栏就被认为是<font color="#dd0000">**打破**</font>了，所有阻塞的await调用都将终止并抛出**BrokenBarrierException**。如果成功通过栅栏，那么await将为每个线程返回一个**唯一**的**到达索引号**，我们可以利用这些索引来**选举**产生一个**领导线程**，并在下次迭代中由该领导线程执行一些**特殊**的工作。CyclicBarrier还可以使你将一个栅栏操作传递给构造函数，这是一个Runnable，当成功通过栅栏时会(在一个子任务线程中)执行它，但在阻塞现场被释放之前是不能执行的。

在模拟程序中通常需要使用栅栏，例如某个步骤中的计算可以并行执行，但必须等到该步骤中的所有计算都执行完毕才能进入下一个步骤。

**P86-P90**：构建高效且可伸缩的结果缓存

V1：使用HashMap<A,V>，然后对其上的读写操作方法使用synchronized加锁，**缺点是每次只有一个线程能够执行读写操作**。

V2：使用ConcurrentHashMap<A,V>替换HashMap<A,V>，优点是这样不需要对读写方法再进行一次同步，ConcurrentHashMap本身处理了同步问题，但是这里还是有一个不足的地方：**就是可能导致重复的计算，如果某个线程启动了一个开销很大的计算，而其他线程并不知道这个计算正在进行，那么很可能会重复这个计算。**

V3：使用ConcurrentHashMap<A,Future<V>>替换ConcurrentHashMap<A,V>，**当前版本首先检查某个相应的计算是否已经启动，而V2中是首先判断某个计算是否已经完成。V3可以避免重复计算，配合putIfAbsent原子方法实现。**

```java
public class ConcurrentHashMapCache<A,V> implements Computable<A,V>{

    private final ConcurrentHashMap<A, Future<V>> cache =
            new ConcurrentHashMap<>();

    private final Computable<A,V> c;

    public ConcurrentHashMapCache(Computable<A,V> c) {
        this.c = c;
    }

    public V compute(final A arg)  {
        while(true) {
            Future<V> f = cache.get(arg);
            if(f == null) {
                Callable<V> eval = new Callable<V>() {
                    @Override
                    public V call() throws Exception {
                        return c.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<>(eval);
                //使用ConcurrentHashMap中的原子操作putIfAbsent
                f = cache.putIfAbsent(arg,ft);
                if(f == null) {
                    System.out.println(Thread.currentThread().getName() + "|缓存为空，开始计算");
                    f = ft;
                    ft.run();
                } else {
                    System.out.println(Thread.currentThread().getName() + "|缓存不为空，阻塞等待");
                }
                try {
                    return f.get();
                } catch (CancellationException e) {
                    cache.remove(arg,f);
                } catch (ExecutionException | InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



```java
public interface Computable<A,V> {
    V compute(A a);
}
```



```java
public class TestConcurrentMapCache {

    public static void main(String[] args) {
        Computable<String,String> c = new Computable<String, String>() {
            @Override
            public String compute(String s) {
                return s + "++++++++";
            }
        };

        ConcurrentHashMapCache<String,String> concurrentHashMapCache = new ConcurrentHashMapCache<String,String>(c);
        for(int i=0;i<100;i++) {
            new Thread(){
                @Override
                public void run() {
                    super.run();
                    System.out.println(concurrentHashMapCache.compute("AA"));
                }
            }.start();
        }
    }
}
```

### 第六章 任务执行

**P94**：在某些情况中，串行处理方式能带来简单性或安全性。**大多数GUI框架都是通过单一的线程来串行地处理任务**。

**P95**：如果可运行的线程数量多于可用处理器的数量，那么有线程将闲置。大量闲置的线程会占用许多内存，给垃圾回收器带来压力，而且大量线程在竞争CPU资源时还将产生其他的性能开销。

**P96**：虽然Executor是个简单的接口，但它却为灵活且强大的异步任务执行框架提供了基础，该框架支持多种不同类型的任务执行策略。**它提供了一种标准的方法将任务的提交过程和执行过程解耦开来，并用Runnable来表示任务**。<font color="#dd0000">**Executor的实现还提供了对生命周期的支持，以及统计信息收集、应用程序管理机制和性能监视等机制**</font>。

```java
public interface Executor {
  void execute(Runnable command)
}
```

**P99**：可以通过调用**Executors中的静态工厂方法之**一来创建一个线程池：

**newFixedThreadPool**。newFixedThreadPool将创建一个**固定长度**的线程池，每当提交一个任务时就创建一个县城，直至达到线程池的最大数量，这时线程池的规模不在变化(<font color="#dd0000">**如果某个线程由于发生了未预期的Exception而结束，那么线程池会补充一个新的线程**</font>)

**newCachedThreadPool**。newCachedThreadPool将创建一个可缓存的线程池，如果线程池的当前规模超过处理需要时，那么将回收空闲的线程，而当需求增加时，则可以添加新的线程，<font color="#dd0000">**线程池的规模不存在任何限制。**</font>

**newSingleThreadPool**。是一个**单线程**的Executor，它创建单个工作线程来执行任务，<font color="#dd0000">**如果这个线程异常结束，会创建另一个线程来替代**</font>。newSingleThreadExecutor能确保依照任务在队列中的顺序来串行执行(例如FIFO、LIFO、优先级)。

备注：单线程的Executor还提供了了大量的内部同步机制，从而确保了任务执行的任何内存写入操作对于后续任务来说都是可见的。**这意味着，及时这个线程会不时地被另一个线程替代，但对象总是可以安全地封闭在"任务线程"中。**

**newScheduledThreadPool**。newScheduledThreadPool将创建一个<font color="#dd0000">**固定长度**</font>的线程池，而且以<font color="#dd0000">**延迟或定时**</font>的方式来执行任务，类似于Timer。

**P100**：<font color="#dd0000">**为了解决执行服务的生命周期问题**</font>，Executor扩展了ExecutorService接口，**添加了一些用于生命周期管理的方法**(同时还有一些用于任务提交的便利方法)。

```java
public interface ExecutorService extends Executor {
  void shutdown();
  List<Runnable> shutdownNow();
  boolean isShutdown();
  boolwan isTerminated();
  boolwan awaitTimination(long timeout,TimeUnit unit) throws InterruptedException;
  // ...... 其他用于任务提交的便利方法
}
```

ExecutorService的生命周期有3中状态：运行、关闭和已终止。ExecutorService在初始创建时处于**运行状态**。shutdown方法将执行**平缓**的关闭过程：**不再接收新的任务，同时等待已经提交的任务执行完成**——<font color="#dd0000">**包括哪些还未开始执行的任务**</font>。shutdownNow方法将执行**粗暴**的关闭过程：它将<font color="#dd0000">**尝试（备注：并非一定成功）**</font>**取消所有运行中的任务，并且不再启动队列中尚未开始执行的任务。**

在ExecutorService关闭后提交的任务将由**拒绝执行处理器(Rejected Execution Handler)**来处理，它将抛弃任务，或者使得execute方法抛出一个未检查的RejectedExecutionException。等所有任务都完成后，ExecutorService将转入终止状态。**可以调用awaitTermination来等待ExecutorService达到终止状态，或者通过调用isTerminated来轮询ExecutorService是否已经终止。通常在调用awaitTermination之后会立即调用shutdown，从而产生同步地关闭ExecutorService的效果。**

**P101**：Timer存在一些缺陷，<font color="#dd0000">**因此应该考虑使用SheduledThreadPool来替代它**</font>，Timer在执行所有定时任务时<font color="#dd0000">**只会创建一个线程**</font>。**如果某个任务的执行时间过长，那么将破坏其他TimerTask的定时精确性**。例如某个周期TimerTask需要每10ms执行一次，而另一个TimerTask需要执行40ms，**那么这个周期任务或者在40ms任务执行完成后快速连续调用4次，或者彻底丢失4次调用(取决于它是基于固定速率来调度还是基于固定延时来调度）**。

Timer的另一个问题是，如果TimerTask抛出一个未检查的异常。那么Timer将表现出糟糕的行为：Timer线程并不捕获异常，因此当TimerTask抛出未检查的异常时将终止定时线程。这种情况下，Timer也不会恢复线程的执行，而是会错误地认为整个Timer都被取消了。这被称为**线程泄漏**。

备注：Timer支持基于**绝对时间**而不是**相对时间**的调度机制，因此任务的执行对系统时钟变化很敏感，而ScheduledThreadPoolExecutor值支持基于相对时间的调度。

```java
public class OutOfTime {
    public static void main(String[] args) throws Exception {
        Timer timer = new Timer();
        timer.schedule(new ThrowTask(),1);
        SECONDS.sleep(1);
        timer.schedule(new ThrowTask(),1);
        SECONDS.sleep(1);
    }

    static class ThrowTask extends TimerTask {
        public void run() { throw new RuntimeException(); }
    }
}
```

**P103**：Runable是一种由很大局限的抽象，虽然run能写入到日志文件或者将结果放入某个共享的数据结构，但它不能返回一个值或抛出一个受检查的异常。**Callable是一种更好的抽象：它认为主入口点(即call)将返回一个值，并且可能抛出一个异常**。在Executor中包含了一些辅助方法能够将其他类型的任务封装为一个Callable，例如Runnable和java.security.PrivilegedAction。

在Executor框架中，已提交单尚未开始的任务可以取消，**但对于那些已经开始执行的任务，只有当他们能够响应中断时，才能取消。**

Future表示一个任务的<font color="#dd0000">**生命周期**</font>，并提供相应的方法来**判断**是否已经完成或取消，以及**获取**任务的结果和取消任务等。

```java
//Callable与Future接口
public interface Callable<V> {
  V call() throws Excepiton;
}
public interface Future<V> {
  boolean cancel(boolean mayInterruptIfRunning);
  boolean isCancelled();
  boolean isDown();
  V get() throws InterruptedException,ExcutionException,CancellationException;
  V get(long timeout,TimeUnit unti)
    throws InterruptedException,ExcutionException
    ,CancellationException,TimeoutException;
}
```

get方法的行为取决于任务的状态(尚未开始、正在运行、已完成)。如果任务已经完成，那么get会立即返回或者抛出一个Exception，**如果任务没有完成，那么get将阻塞并直到任务完成**。如果任务抛出了异常，那么get将该异常封装为ExecutionException并**重新抛出**，如果任务被取消，那么get将抛出CancellationException。如果get抛出了ExecutionException，那么可以通过getCause()来获得被封装的初始异常。

**P104**：可以通过许多种方法创建一个Future来标书任务。ExecutorService中所有submit方法都将返回一个Future，从而将衣蛾Runnable或Callable提交给Executor，并得到一个Futrue用来获得任务的执行结果或者取消任务。还可以显示地为某个指定的Runnable或Callable实例化一个FutureTask。(由于FutureTask实现Runnable,因此可以将它提交给Executor来执行，或者直接调用它的run方法。)

**P106**：<font color="#dd0000">**CompletionService将Executor和BlockingQueue的功能融合在一起**</font>。你可以将Callable任务提交给它来执行，人后使用类似于**队列操作额take和poll**等方法来获得已完成的结果，而这些结果会在完成时将被封装为Future。ExecutorCompletionService实现了CompletionService，并将计算结果委托给了一个Executor。

ExecutorCompletionService的实现非常简单。在构造函数中创建一个BlockingQueue来保存计算完成的结果。当计算完成时，**调用Future-Task的down方法**。当提交某个任务时，该任务将首先包装为一个QueueingFuture，这是FutureTask的一个子类，然后在改写该子类的done方法，并将结果放入BlockingQueue中：

```java
private class QueueingFuture<V> extends FutureTask<V> {
    public QueueingFuture(Callable<V> callable) {
        super(callable);
    }

    public QueueingFuture(Runnable runnable, V result) {
        super(runnable, result);
    }

    @Override
    protected void done() {
        super.done();
        completionQueue.add(this);
    }
}
```

```java
public class Renderer {
    private final ExecutorService executor;
    Renderer(ExecutorService executor) {this.executor = executor;}
    void renderPage(CharSequence source) {
        List<ImageInfo> info = scanForImagaeInfo(source);
        CompletionService<ImageData> completionService =
                new ExecutorCompletionService<ImageData>(executor);
        for(final ImageInfo imageInfo:info) {
            completionService.submit(new Callable<ImageData>(){
                @Override
                public ImageData call() throws Exception {
                    return imageInfo.downloadImage();
                }
            });
        }
        try {
            for(int t=0,n=info.size;t<n;t++) {
                Future<ImageData> f = completionService.take();
                ImageData imageData = f.get();
                renderImage(imageData);
            }
        }catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }
}
```

**P108**:为任务设置超时——在使用限时任务时需要注意，当这些任务超时后应该立即停止，从而避免为继续计算一个不再使用的结果而浪费计算资源。要实现这个功能，可以由任务本身来管理它的限定时间，并且在超时后中止执行或取消任务。

```java
public class ExecTimeOut {
    Page renderPageWithAd() throws InterruptedException {
        long endNanos = System.nanoTime() + 10000;
        Future<Ad> f = exec.submit(new FetchAdTask());
        //在等待广告的同时显示页面
        Page page = renderPageBody();
        Ad ad;
        try {
            //只等待指定的时间长度
            long timeLeft = endNanos - System.nanoTime();
            ad = f.get(timeLeft,NANOSECONDS);
        } catch(ExecutionException e) {
            ad = DEFAULT_AD;
        } catch(TimeoutException e){
            ad = DEFAULT_AD;
            f.cancel(true);
        }
        page.setAd(ad);
        return page;
    }
}
```

**P109**：创建n个任务，将其提交到一个线程池，保留n个Future，并使用限时的get方法通过Future串行地获取每一个结果，这一些都很简单，但还有一个更简单的方法——invokeAll。

```java
public class TestInvokeAll {
    ExecutorService exec;

    
    class QuoteTask implements Callable<String> {
        private final TravelCompany company;
        private final TravelInfo travelInfo;
        ...

        @Override
        public TravelQuote call() throws Exception {
            return company.solicitQuote(travelInfo);
        }
    }

    public List<TravelQuote> getRankedTravelQuotes(TravelInfo travelInfo
        ,Set<TravelCompany> companies,Comparator<TravelQuote> ranking
            , long time,TimeUnit unit) throws InterruptedException {
        List<QuoteTask> tasks = new ArrayList<>();
        for(TravelCompany company:companies) {
            tasks.add(new QuoteTask(company,travelInfo));
        }
        List<Future<TravelQuote>> futures =
                exec.invokeAll(tasks,time,unit);
        List<TravelQuote> quotes =
                new ArrayList<TravelQuote>(tasks.size());
        Iterator<QuoteTask> taskIter = tasks.iterator();
        for(Future<TravelQuote> f:futures) {
            QuoteTask task = taskIter.next();
            try {
                quotes.add(f.get());
            }catch (ExecutionException e) {
                quotes.add(task.getFailureQuote(e.getCause()));
            } catch(CancellationException e) {
                quotes.add(task.getTimeoutQuote(e));
            }
        }
        Collections.sort(quotes,ranking);
        return quotes;
    }
}
```

### 第七章 取消与关闭

**P111**：钥匙任务和线程能安全、快速、可靠地停下来，并不是一件容易的事。Java没有提供任何机制来安全地终止线程。**但它提供了中断(Interruption)，这是一种协作机制，能够使一个线程终止另一个线程的当前工作。**

**P112-P113**：可以使用volatile修饰的标志位来取消线程的执行；但是当在BlockingQueue中使用这种volatile标志位的方式可能会因为阻塞队列的操作方法被阻塞住而永远无法得到检查机会。

**P114**：阻塞库方法，例如：Thread.sleep和Object.wait等，都会检查线程何时中断，并且在发现中断时提前返回。他们在响应中断时执行的操作包括：清除中断状态，抛出InterruptedException表示阻塞操作由于中断而提前结束。

**P115**：对中断操作正确的理解是：它不会真正地中断一个正在运行的线程，而只是发出中断请求，然后由线程在**下一个合适的时机**<font color="#dd0000">**中断自己**</font>。在使用静态的interrupted时应该小心，因为它会清除当前线程的中断状态。如果在调用interrupted时返回了true，那么除非你想屏蔽这个中断，否则必须对它进行处理——**可以抛出InterruptedException，或者通过再次调用interrupt来恢复中断状态**。