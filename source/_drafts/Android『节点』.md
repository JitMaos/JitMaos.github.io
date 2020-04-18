---
title: Android『节点』
tags:
---

## Dalvik虚拟机

Dalvik虚拟机主要具有以下特征：

1. 专有的dex文件格式。每个应用中会有很多类，编译完成后即会有很多的class文件，class文件中会有大量的冗余信息，而dex文件格式会把所有的class文件内容整合到一个文件中。这样除了减小尺寸，也提高了类的查找速度；增加了对新的操作码的支持；

2. dex的优化。文件结构尽量简洁，使用等长的指令，借以提高解析速度；尽量扩大只读结构的大小，借以提高跨进程的数据共享。

3. <font color="#dd0000">**基于寄存器，要知道一般的JVM是基于栈的**</font>。相对于基于堆栈实现的虚拟机，基于寄存器实现的虚拟机虽然在硬件、通用性上要差一些，但是它在代码的执行效率上更胜一筹。
4. 每一个Android应用都运行在一个Dalvik虚拟机实例中，而每一个虚拟机实例都是一个独立的进程空间。虚拟机的线程机制，内存分配和管理，Mutex等的实现都以来底层操作系统。所有Android应用的线程都对应一个Linux线程，虚拟机因而可以<font color="#dd0000">**更多地依赖操作系统的线程调度和管理机制**</font>。
   不同的应用在不同的进程空间里运行，对不同来源的应用都使用不同的Linux用户来运行，可以更大程度地保护应用的安全和独立运行。



## Android的启动过程

摘自：https://www.jianshu.com/p/d1e8002df580

![img](http://47.110.40.63:8080/img/blog/Android启动流程图.png)

用户按下电源键，引导芯片代码从预定义的地方开始执行，加载引导程序BootLoader到RAM，然后开始执行。

启动引导程序BootLoader，用来引导Android系统的启动工作。然后，Linux内核启动。

Linux内核启动后，设置缓存、被保护存储器、计划列表、加载驱动等操作。当内核完成系统设置后，会查找“init”文件，然后启动Root进程。

Linux内核创建用户级进程，init进程（上帝般存在）。

Init进程会创建Zygote孵化器进程。Zygote进程存在一个Socket服务端，以及一些Framework层共享的类和资源。

Zygote进程会先孵化出一个SystemServer进程。SystemServer进程用来加载一些系统服务，比如AMS、WMS、PMS等，保存有系统服务需要的类和资源，存在一个Socket客户端。

AMS服务用来管理Activity的创建，当需要启动Activity时，会通过SystemServer进程中的Socket客户端向Zygote进程发送消息，请求创建Activity。

## App启动过程

App启动从用户按下桌面图标开始。

- App都是由桌面启动器启动的，桌面启动器自身也是一个App，它也存在一个进程，称为Launcher进程，也叫调用者进程。
- 用户按下桌面上的App图标后，Launcher进程会将请求启动主活动（MainActivity）的请求以Binder的方式发送给AMS服务。
- AMS服务收到请求后，交付给ActivityStarter处理intent和flag信息，然后交给ActivityStackSuperVisior/ActivityStack处理Activity进程相关流程，同时通过Socket客户端向Zygote进程请求孵化新进程。
- Zygote进程收到请求后，创建一个新进程，这个新进程就是APP所在进程。
- 在新进程里创建ActivityThread线程，包含main方法，是Android程序的入口，ActivityThread所在线程即是主线程（UI线程）。同时创建ApplicationThread和W线程，他们都继承自Binder类。ApplicationThread线程在主活动创建之前创建，负责监听AMS发送来的创建Activity的请求。Activity创建后，W线程监听WMS发来的消息（比如点击和触摸事件），将消息发送给DectorView，如果DecoterView没有处理，则传递给PhoneWindow，如果PhoneWindow也没有处理，则传递给Activity通过Handler来处理消息。
- ActivityThread中会调用prepareMainLooper()方法，创建一个Looper对象，Looper对象会创建一个消息队列MessageQueue，调用Looper.loop()方法后UI线程会进入消息循环体，不断从消息队列中取出消息Message对象并处理消息。
- ApplicationThread类监听到了创建Activity的请求，ActivityThread通过ClassLoader类加载器加载Activity并创建Activity实例，然后回调onCreate()方法。

作者：sendtion
链接：https://www.jianshu.com/p/d1e8002df580
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## RecyclerView和ListView缓存机制

摘自：https://mp.weixin.qq.com/s/ZT6v5CNz5yBEhHc9lpVICQ

**ListView**的缓存有两级，在ListView里面有一个内部类 RecycleBin，RecycleBin有两个对象Active View和Scrap View来管理缓存，Active View是第一级，Scrap View是第二级。

- **Active View**：是缓存在`屏幕内的ItemView`，当列表数据发生变化时，屏幕内的数据可以直接拿来复用，`无须进行数据绑定`。
- **Scrap view**：缓存屏幕外的ItemView，这里所有的缓存的数据都是"脏的"，也就是数据需要重新绑定，也就是说`屏幕外的所有数据在进入屏幕的时候都要走一遍getView（）方法`。
- 图片1：http://47.110.40.63:8080/img/blog/ListView缓存示意图.png
- 图片2：http://47.110.40.63:8080/img/blog/ListView缓存流程图.png

**RecyclerView**的缓存分为四级

- Scrap

  对应ListView 的Active View，就是屏幕内的缓存数据，就是相当于换了个名字，可以直接拿来复用。

- Cache

  刚刚移出屏幕的缓存数据，默认大小是2个，当其容量被充满同时又有新的数据添加的时候，会根据FIFO原则，把优先进入的缓存数据移出并放到下一级缓存中，然后再把新的数据添加进来。Cache里面的数据是干净的，也就是携带了原来的ViewHolder的所有数据信息，数据可以直接来拿来复用。需要注意的是，cache是根据position来寻找数据的，这个postion是根据第一个或者最后一个可见的item的position以及用户操作行为（上拉还是下拉）。

- ViewCacheExtension

  是google留给开发者自己来自定义缓存的，只有一个public abstract View getViewForPositionAndType(@NonNull Recycler recycler, int position, int type);

- RecycledViewPool

  刚才说了**Cache默认的缓存数量是2个**，当Cache缓存满了以后会根据FIFO（先进先出）的规则把Cache先缓存进去的ViewHolder移出并缓存到RecycledViewPool中，**RecycledViewPool默认的缓存数量是5个。RecycledViewPool与Cache相比不同的是，从Cache里面移出的ViewHolder再存入RecycledViewPool之前ViewHolder的数据会被全部重置**，相当于一个新的ViewHolder，而且Cache是根据position来获取ViewHolder，而RecycledViewPool是根据itemType获取的，如果没有重写getItemType（）方法，itemType就是默认的。`因为RecycledViewPool缓存的ViewHolder是全新的，所以取出来的时候需要走onBindViewHolder（）方法`。

## 屏幕适配

**今日头条适配方案**

项目地址：https://github.com/JessYanCoding/AndroidAutoSize

实现原理：根据当前屏幕密度，和设计稿基准密度进行对比。得到密度缩放scale，**调用Android api设置APP全局密度切换**。

优点：

使用成本非常低，操作非常简单
侵入性非常低
可适配三方库的控件和系统的控件

不足：

`存在bug，屏幕横竖屏切换时设置的屏幕密度可能会被重置`。

**使用自定义SizeUtil**

代码地址：http://47.110.40.63:8080/code/SizeUtil.java

实现原理：

传入需要适配的View，通常为Activity或Fragment的root view，遍历其中的所有子View，根据View的dp数结合屏幕密度比例进行缩放，更新LayoutParams即可。

## APK压缩

一、转换图片格式，使用jpg，或者webp。WebP比PNG小45%。

二、去除多语言

三、删除不必要的so库，微信也只保留了armeabi-v7a

四、使用Lint进行无用资源检查（谨慎删除）

五、开启混淆

六、删除无用资源ShinkResource

七、使用微信资源压缩方案AndResGuard
**AndResGuard通过修改resources.arsc文件，从而可以混淆安卓的资源文件路径**（比如res/drawable/activity_advanced_setting_for_test=>r/d/a），达到减少apk包的体积的目的。`底层原理
andresGuard在原生的buildApk步骤之后，使用产生的apk作为输入文件，对其进行混淆压缩，产出一个新的apk。`

## Synchronized与ReentrantLock区别

在Synchronized优化以前，synchronized的性能是比ReenTrantLock差很多的，但是自从Synchronized引入了偏向锁，轻量级锁（自旋锁）后，两者的性能就差不多了，在两种方法都可用的情况下，官方甚至建议使用synchronized，其实synchronized的优化我感觉就借鉴了ReenTrantLock中的CAS技术。

**Synchronized**

进过编译，会在同步块的前后分别形成monitorenter和monitorexit这个两个字节码指令。在执行monitorenter指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1，相应的，在执行monitorexit指令时会将锁计算器就减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止。

**ReentrantLock**

由于ReentrantLock是java.util.concurrent包下提供的一套互斥锁，相比Synchronized，ReentrantLock类提供了一些高级功能，主要有以下3项：

1. 等待可中断，持有锁的线程长期不释放的时候，正在等待的线程`可以选择放弃等待`，这相当于Synchronized来说可以避免出现死锁的情况。通过lock.lockInterruptibly()来实现这个机制。

2. 公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，Synchronized锁是非公平锁，**ReentrantLock默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性能不是很好**。

3. 锁绑定多个条件，一个ReentrantLock对象可以同时绑定多个对象。ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

   Condition类能实现synchronized和wait、notify搭配的功能，另外比后者更灵活，`Condition可以实现多路通知功能，也就是在一个Lock对象里可以创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中`，从而**可以有选择的进行线程通知，在调度线程上更加灵活**。而synchronized就相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在这个对象上。线程开始notifyAll时，需要通知所有的WAITING线程，没有选择权，会有相当大的效率问题。

`ReenTrantLock的实现是一种自旋锁，通过循环调用CAS+volatile操作来实现加锁。`

`ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。`

##Hexo 博客实现文件同步与静态文件部署

核心方法是hexo -g d会自动提交静态文件到远程master分支，不管当前在哪个分支上。所以：

1. 在master分支上提交静态文件
2. 本地切换到hexo分支，并把本地文件提交到远程hexo分支，之后使用hexo clean；hexo -g d提交静态文件，静态文件会自动生成在master分支上。

## 跳跃表

摘自：https://www.jianshu.com/p/dc252b5efca6

![img](http://47.110.40.63:8080/img/blog/跳跃表.jpeg)

(1). 跳跃表的每一层都是一条有序的链表.
(2). 跳跃表的查找次数近似于层数，时间复杂度为O(logn)，插入、删除也为 O(logn)。
(3). 跳跃表空间复杂度为O(n)，实际上是O(2n)
(4). 跳跃表是一种随机化的数据结构(通过抛硬币来决定层数)。
(5). 插入元素时，对该元素进行投硬币，决定是否提取为关键节点。
(6). 删除元素时，需逐层同步删除。


跳跃表的优点是维持结构平衡的成本比较低，完全依赖随机。而二叉查找树在多次插入删除后，需要Rebalance来重新调整结构平衡

## 自定义权限

摘自：https://www.cnblogs.com/liuzhipenglove/p/7102889.html

A应用打开B应用可以通过显示Intent，或隐式Intent。但是如果有安全方面的考虑，可以自定义一个权限：

```java
<activity
    android:name="com.example.testandroid.BActivity"
    android:exported="true"
    android:label="B"
    android:permission="corn.permission.CORN_OWN" >
</activity>
```

那么A应用想打开BActivity，就需要在清单文件中添加权限

```java
<uses-permission android:name="corn.permission.CORN_OWN" ></uses-permission>
```

protectionLevel：normal, dangerous, signature, signatureOrSystem

## Binder一次拷贝实现进程间通信原理

摘自：



![img](http://47.110.40.63:8080/img/blog/mmap实现一次拷贝完成进程通信.png)



## SurfaceFlinger

摘自:https://blog.csdn.net/freekiteyu/article/details/79483406

![img](http://47.110.40.63:8080/img/blog/SurfaceFlinger启动流程.png)

  a 、首先每个surface 在屏幕上有它的位置，以及大小，然后每个surface 里面还有要显示的内容，内容，大小，位置 这些元素 在我们改变应用程序的时候都可能会改变，改变时应该如何处理 ？

b 、然后就各个surface 之间可能有重叠，比如说在上面的简略图中，绿色覆盖了蓝色，而红色又覆盖了绿色和蓝色以及下面的home ，而且还具有一定透明度。这种层之间的关系应该如何描述？  

我们首先来看第二个问题，我们可以想象在屏幕平面的垂直方向还有一个Z 轴，所有的surface 根据在Z 轴上的坐标来确定前后，这样就可以描述各个surface 之间的上下覆盖关系了，而这个在Z 轴上的顺序，图形上有个专业术语叫Z-order 。 

  对于第一个问题，我们需要一个结构来记录应用程序界面的位置，大小，以及一个buffer 来记录需要显示的内容，所以这就是我们surface 的概念，surface 实际我们可以把它理解成一个容器，这个容器记录着应用程序界面的控制信息，比如说大小啊，位置啊，而它还有buffer 来专门存储需要显示的内容。

  在这里还存在一个问题，那就是当存在图形重合的时候应该如何处理呢，而且可能有些surface 还带有透明信息，这里就是我们SurfaceFlinger 需要解决问题，它要把各个surface 组合(compose/merge) 成一个main Surface ，最后将Main Surface 的内容发送给FB/V4l2 Output ，这样屏幕上就能看到我们想要的效果。

  在实际中对这些Surface 进行merge 可以采用两种方式，一种就是采用软件的形式来merge ，还一种就是采用硬件的方式，软件的方式就是我们的SurfaceFlinger ，而硬件的方式就是Overlay 。

## 线程状态



## wait/sleep/notify

**wait和sleep的区别**

<font color="#dd0000">**wait 会释放锁，而 sleep 一直持有锁**</font>。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。如果能够帮助你记忆的话，**可以简单认为和锁相关的方法都定义在Object类中**，因此调用Thread.sleep是不会影响锁的相关行为。<font color="#dd0000">**调用wait后，需要别的线程执行notify/notifyAll才能够重新获得CPU执行时间。**</font>

## Minor/Major/Full GC

**Minor GC**指新生代GC，即发生在新生代（包括Eden区和Survivor区）的垃圾回收操作，当新生代无法为新生对象分配内存空间的时候，会触发Minor GC。因为新生代中大多数对象的生命周期都很短，所以发生Minor GC的频率很高，<font color="#dd0000">**虽然它会触发stop-the-world，但是它的回收速度很快**</font>。

**Major GC**清理Tenured区，用于回收老年代，出现Major GC通常会出现至少一次Minor GC。

**Full GC**是针对整个新生代、老生代、元空间（metaspace，java8以上版本取代perm gen）的全局范围的GC。<font color="#dd0000">**Full GC不等于Major GC，也不等于Minor GC+Major GC**</font>，发生Full GC需要看使用了什么垃圾收集器组合，才能解释是什么样的垃圾回收。

