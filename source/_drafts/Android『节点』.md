title: Android『节点』
tags:

#虚拟机

## Dalvik虚拟机

Dalvik虚拟机主要具有以下特征：

1. 专有的dex文件格式。每个应用中会有很多类，编译完成后即会有很多的class文件，class文件中会有大量的冗余信息，而dex文件格式会把所有的class文件内容整合到一个文件中。这样除了减小尺寸，也提高了类的查找速度；增加了对新的操作码的支持；

2. dex的优化。文件结构尽量简洁，使用等长的指令，借以提高解析速度；尽量扩大只读结构的大小，借以提高跨进程的数据共享。

3. <font color="#dd0000">**基于寄存器，要知道一般的JVM是基于栈的**</font>。相对于基于堆栈实现的虚拟机，基于寄存器实现的虚拟机虽然在硬件、通用性上要差一些，但是它在代码的执行效率上更胜一筹。
4. 每一个Android应用都运行在一个Dalvik虚拟机实例中，而每一个虚拟机实例都是一个独立的进程空间。虚拟机的线程机制，内存分配和管理，Mutex等的实现都以来底层操作系统。所有Android应用的线程都对应一个Linux线程，虚拟机因而可以<font color="#dd0000">**更多地依赖操作系统的线程调度和管理机制**</font>。
   不同的应用在不同的进程空间里运行，对不同来源的应用都使用不同的Linux用户来运行，可以更大程度地保护应用的安全和独立运行。

#Android系统

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

摘自：https://www.jianshu.com/p/d1e8002df580

App启动从用户按下桌面图标开始。

- App都是由桌面启动器启动的，桌面启动器自身也是一个App，它也存在一个进程，称为Launcher进程，也叫调用者进程。
- 用户按下桌面上的App图标后，Launcher进程会将请求启动主活动（MainActivity）的请求以Binder的方式发送给AMS服务。
- AMS服务收到请求后，交付给ActivityStarter处理intent和flag信息，然后交给ActivityStackSuperVisior/ActivityStack处理Activity进程相关流程，同时通过Socket客户端向Zygote进程请求孵化新进程。
- Zygote进程收到请求后，创建一个新进程，这个新进程就是APP所在进程。
- 在新进程里创建ActivityThread线程，包含main方法，是Android程序的入口，ActivityThread所在线程即是主线程（UI线程）。同时创建ApplicationThread和W线程，他们都继承自Binder类。ApplicationThread线程在主活动创建之前创建，负责监听AMS发送来的创建Activity的请求。Activity创建后，W线程监听WMS发来的消息（比如点击和触摸事件），将消息发送给DectorView，如果DecoterView没有处理，则传递给PhoneWindow，如果PhoneWindow也没有处理，则传递给Activity通过Handler来处理消息。
- ActivityThread中会调用prepareMainLooper()方法，创建一个Looper对象，Looper对象会创建一个消息队列MessageQueue，调用Looper.loop()方法后UI线程会进入消息循环体，不断从消息队列中取出消息Message对象并处理消息。
- ApplicationThread类监听到了创建Activity的请求，ActivityThread通过ClassLoader类加载器加载Activity并创建Activity实例，然后回调onCreate()方法。

# 算法

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

## KMP字符串匹配算法

KMP算法是一种改进的字符串匹配算法，KMP算法的**核心是利用匹配失败后的信息，尽量减少模式串与主串的匹配次数以达到快速匹配的目的**。具体实现就是通过一个next()函数实现，函数本身包含了模式串的局部匹配信息。KMP算法的时间复杂度O(m+n)。

# 其他

##Hexo 博客实现文件同步与静态文件部署

核心方法是hexo -g d会自动提交静态文件到远程master分支，不管当前在哪个分支上。所以：

1. 在master分支上提交静态文件
2. 本地切换到hexo分支，并把本地文件提交到远程hexo分支，之后使用hexo clean；hexo -g d提交静态文件，静态文件会自动生成在master分支上。

#Android API

## ActivityThread

摘自：https://blog.csdn.net/xu_song/article/details/81983724

安卓应用程序的入口是什么呢？我想不少人可能回答说:application的onCreate方法，其实并不是的，即使是application，**也有一个方法比onCreate先执行，这个方法就是attachBaseContext(Context context)方法:一般情况下，可以在这个方法中进行多dex的分包注入**，比如下面的代码:

```
@Override
    protected void attachBaseContext(Context base) {
        MultiDex.install(base);
        super.attachBaseContext(base);
        try {
            HandlerInject.hookHandler(base);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

application并不是安卓程序的入口，跟Java程序类似，都是有一个入口的，而这个入口就是ActivityThread，<font color="#dd0000">**ActiviyThread也有一个main方法，这个main方法是安卓应用程序真正的入口**</font>。

**ActivityThread的作用很多，但最主要的作用是根据AMS(ActivityManagerService的要求，通过IApplicationThread的接口)负责调度和执行activities、broadcasts和其它操作**。

## RecyclerView和ListView缓存机制

摘自：https://mp.weixin.qq.com/s/ZT6v5CNz5yBEhHc9lpVICQ

**ListView**的缓存有两级，在ListView里面有一个内部类 RecycleBin，RecycleBin有两个对象Active View和Scrap View来管理缓存，Active View是第一级，Scrap View是第二级。

- **Active View**：是缓存在屏幕内的ItemView，当列表数据发生变化时，屏幕内的数据可以直接拿来复用，无须进行数据绑定。
- **Scrap view**：缓存屏幕外的ItemView，这里所有的缓存的数据都是"脏的"，<font color="#dd0000">**也就是数据需要重新绑定，也就是说屏幕外的所有数据在进入屏幕的时候都要走一遍getView（）方法**</font>。
- 图片1：![img](http://47.110.40.63:8080/img/blog/ListView缓存示意图.png)
- 图片2：![img](http://47.110.40.63:8080/img/blog/ListView缓存流程图.png)

**RecyclerView**的缓存分为四级

- Scrap

  对应ListView 的Active View，就是屏幕内的缓存数据，就是相当于换了个名字，可以直接拿来复用。

- Cache

  **刚刚移出屏幕的缓存数据，默认大小是2个**，当其容量被充满同时又有新的数据添加的时候，会根据FIFO原则，把优先进入的缓存数据移出并放到下一级缓存中，然后再把新的数据添加进来。<font color="#dd0000">**Cache里面的数据是干净的，也就是携带了原来的ViewHolder的所有数据信息，数据可以直接来拿来复用**</font>。需要注意的是，cache是根据position来寻找数据的，这个postion是根据第一个或者最后一个可见的item的position以及用户操作行为（上拉还是下拉）。

- ViewCacheExtension

  是google留给开发者自己来自定义缓存的，只有一个public abstract View getViewForPositionAndType(@NonNull Recycler recycler, int position, int type);

- RecycledViewPool

  刚才说了**Cache默认的缓存数量是2个**，当Cache缓存满了以后会根据FIFO（先进先出）的规则把Cache先缓存进去的ViewHolder移出并缓存到RecycledViewPool中，**RecycledViewPool默认的缓存数量是5个。RecycledViewPool与Cache相比不同的是，从Cache里面移出的ViewHolder再存入RecycledViewPool之前ViewHolder的数据会被全部重置**，相当于一个新的ViewHolder，而且Cache是根据position来获取ViewHolder，而RecycledViewPool是根据itemType获取的，如果没有重写getItemType（）方法，itemType就是默认的。因为RecycledViewPool缓存的ViewHolder是全新的，所以取出来的时候需要走onBindViewHolder（）方法。

## 关于Context

摘自：https://www.jianshu.com/p/f0fb461a2b2c

一个Context相当于一个事情发生的<font color="#dd0000">**场景**</font>，看以下类图，**Context类本身是一个纯abstract类，它有两个具体的实现子类：ContextImpl和ContextWrapper**。Context

![img](http://47.110.40.63:8080/img/blog/Context结构图.png)

<font color="#dd0000">**Context类本身是一个纯abstract类**</font>，它有两个具体的实现子类：ContextImpl和ContextWrapper。其中ContextWrapper类，如其名所言，这只是一个包装而已，<font color="#dd0000">**ContextWrapper构造函数中必须包含一个真正的Context引用，同时ContextWrapper中提供了attachBaseContext（）用于给ContextWrapper对象中指定真正的Context对象，调用ContextWrapper的方法都会被转向其所包含的真正的Context对象**</font>。ContextThemeWrapper类，如其名所言，其内部包含了与主题（Theme）相关的接口，这里所说的主题就是指在AndroidManifest.xml中通过android：theme为Application元素或者Activity元素指定的主题。当然，**只有Activity才需要主题，Service是不需要主题的**，因为Service是没有界面的后台场景，**所以Service直接继承于ContextWrapper**，Application同理。而ContextImpl类则真正实现了Context中的所以函数，应用程序中所调用的各种Context类的方法，其实现均来自于该类。一句话总结：**Context的两个子类分工明确，其中ContextImpl是Context的具体实现类，ContextWrapper是Context的包装类。Activity，Application，Service虽都继承自ContextWrapper（Activity继承自ContextWrapper的子类ContextThemeWrapper），但它们初始化的过程中都会创建ContextImpl对象，由ContextImpl实现Context中的方法**。

**一个应用程序有几个Context**

在应用程序中Context的具体实现子类就是：Activity，Service，Application。那么<font color="#dd0000">**Context数量=Activity数量+Service数量+1**</font>。**Broadcast Receiver，Content Provider并不是Context的子类，他们所持有的Context都是其他地方传过去的，所以并不计入Context总数**。

**Context能干什么**

Context到底可以实现哪些功能呢？这个就实在是太多了，弹出Toast、启动Activity、启动Service、发送广播、操作数据库等等都需要用到Context。

**Context使用场景表格**

![img](http://47.110.40.63:8080/img/blog/Context使用场景表格.png)

上图中Application和Service所不推荐的两种使用情况。

1：如果我们用ApplicationContext去启动一个LaunchMode为standard的Activity的时候会报错：

```java
android.util.AndroidRuntimeException: Calling startActivity from outside of an Activity context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?
```

<font color="#dd0000">**这是因为非Activity类型的Context并没有所谓的任务栈，所以待启动的Activity就找不到栈了**</font>。解决这个问题的方法就是**为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK**标记位，这样启动的时候就为它创建一个新的任务栈，而此时Activity是以singleTask模式启动的。所有这种用Application启动Activity的方式不推荐使用，Service同Application。

2：<font color="#dd0000">**在Application和Service中去layout inflate也是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。所以这种方式也不推荐使用**</font>。
 一句话总结：凡是跟UI相关的，都应该使用Activity做为Context来处理；其他的一些操作，Service,Activity,Application等实例都可以，当然了，注意Context引用的持有，防止内存泄漏。

**如何获取Context**

通常我们想要获取Context对象，主要有以下四种方法

1：**View.getContext()**,返回当前View对象的Context对象，通常是当前正在展示的Activity对象。
 2：**Activity.getApplicationContext()**,获取当前Activity所在的(应用)进程的Context对象，通常我们使用Context对象时，要优先考虑这个全局的进程Context。
 3：**ContextWrapper.getBaseContext()**:用来获取一个ContextWrapper进行装饰之前的Context，可以使用这个方法，这个方法在实际开发中使用并不多，也不建议使用。
 4：**Activity.this** 返回当前的Activity实例，如果是UI控件需要使用Activity作为Context对象，但是默认的Toast实际上使用ApplicationContext也可以。

**getApplication()和getApplicationContext()**

<font color="#dd0000">**这两者返回的是同一个内存地址的对象**</font>，区别更多是语义上的，**getApplication()只在Activity和Service中才能调用的到**。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application的，但是如果在一些其它的场景，**比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法了**。

**Context引起的内存泄露**

**1. 错误的单例模式**

```java
public class Singleton {
    private static Singleton instance;
    private Context mContext;

    private Singleton(Context context) {
        this.mContext = context;
    }

    public static Singleton getInstance(Context context) {
        if (instance == null) {
            instance = new Singleton(context);
        }
        return instance;
    }
}
```

这是一个非线程安全的单例模式，instance作为静态对象，其生命周期要长于普通的对象，其中也包含Activity，假如Activity A去getInstance获得instance对象，传入this，常驻内存的Singleton保存了你传入的Activity A对象，并一直持有，即使Activity被销毁掉，但因为它的引用还存在于一个Singleton中，就不可能被GC掉，这样就导致了内存泄漏。

**2. View持有Activity引用**

```java
public class MainActivity extends Activity {
    private static Drawable mDrawable;

    @Override
    protected void onCreate(Bundle saveInstanceState) {
        super.onCreate(saveInstanceState);
        setContentView(R.layout.activity_main);
        ImageView iv = new ImageView(this);
        mDrawable = getResources().getDrawable(R.drawable.ic_launcher);
        iv.setImageDrawable(mDrawable);
    }
}
```

有一个静态的Drawable对象当ImageView设置这个Drawable时，ImageView保存了mDrawable的引用，而ImageView传入的this是MainActivity的mContext，因为被static修饰的mDrawable是常驻内存的，MainActivity是它的间接引用，MainActivity被销毁时，也不能被GC掉，所以造成内存泄漏。

**正确使用Context**

1. 当Application的Context能搞定的情况下，并且生命周期长的对象，优先使用Application的Context。
2. 不要让生命周期长于Activity的对象持有到Activity的引用。
3. <font color="#dd0000">**尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用，如果使用静态内部类，将外部实例引用作为弱引用持有**</font>。

## 屏幕适配

**今日头条适配方案**

项目地址：https://github.com/JessYanCoding/AndroidAutoSize

实现原理：根据当前屏幕密度，和设计稿基准密度进行对比。得到密度缩放scale，**调用Android api设置APP全局密度切换**。

优点：

使用成本非常低，操作非常简单
侵入性非常低
可适配三方库的控件和系统的控件

不足：

存在bug，屏幕横竖屏切换时设置的屏幕密度可能会被重置。

**使用自定义SizeUtil**

代码地址：http://47.110.40.63:8080/code/SizeUtil.java

实现原理：

传入需要适配的View，通常为Activity或Fragment的root view，遍历其中的所有子View，根据View的dp数结合屏幕密度比例进行缩放，更新LayoutParams即可。

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

## App刷新屏幕流程

摘自：https://blog.csdn.net/vicwudi/article/details/100191529

1. 界面上任何一个 View 的刷新请求最终都会走到 ViewRootImpl 中的 **scheduleTraversals()**里来安排一次遍历绘制 View 树的任务；

   ```java
   //ViewRootImpl.java
   void scheduleTraversals() {
     if (!mTraversalScheduled) { //过滤掉同一帧内的重复调用
       mTraversalScheduled = true;
       //插入同步内存屏障，拦截这个时刻之后所有同步消息的执行，但不会拦截异步消息
       mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
       mChoreographer.postCallback(
         Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
       if (!mUnbufferedInputDispatch) {
         scheduleConsumeBatchedInput();
       }
       notifyRendererOfFramePending();
       pokeDrawLockIfNeeded();
     }
   }
   ```

2. scheduleTraversals() 会先**过滤掉同一帧内的重复调用**，在同一帧内只需要安排一次遍历绘制 View 树的任务即可，这个任务会在下一个屏幕刷新信号到来时调用 performTraversals() 遍历 View 树，遍历过程中会将所有需要刷新的 View 进行重绘；

3. 接着 scheduleTraversals() 会往主线程的消息队列中发送一个<font color="#dd0000">**同步屏障**</font>，**拦截这个时刻之后所有的同步消息的执行，但不会拦截异步消息**，以此来尽可能的保证当接收到屏幕刷新信号时可以尽可能第一时间处理遍历绘制 View 树的工作；

4. 发完同步屏障后 scheduleTraversals() 才会开始安排一个遍历绘制 View 树的操作，作法是把 performTraversals() 封装到 Runnable 里面，然后调用 Choreographer 的 postCallback() 方法；

   ```java
   //TraversalRunnable.java
   final class TraversalRunnable implements Runnable {
     @Override
     public void run() {
       doTraversal();
     }
   }
   ```

   doTravelsal

   ```java
   void doTraversal() {
     if (mTraversalScheduled) {
       mTraversalScheduled = false;
       mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
   
       if (mProfile) {
         Debug.startMethodTracing("ViewAncestor");
       }
   
       performTraversals();
   
       if (mProfile) {
         Debug.stopMethodTracing();
         mProfile = false;
       }
     }
   }
   ```

   

5. postCallback() 方法会先将这个 Runnable 任务以当前时间戳放进一个待执行的队列里，然后如果当前是在主线程就会直接调用一个native 层方法，如果不是在主线程，会发一个最高优先级的 message 到主线程，让主线程第一时间调用这个 native 层的方法；

6. native 层的这个方法是用来向底层注册监听下一个屏幕刷新信号，当下一个屏幕刷新信号发出时，底层就会回调 Choreographer 的onVsync() 方法来通知上层 app；

7. onVsync() 方法被回调时，会往主线程的消息队列中发送一个执行 doFrame() 方法的消息，这个消息是异步消息，所以不会被同步屏障拦截住；

   ```java
   void doFrame(long frameTimeNanos, int frame) {
           final long startNanos;
           synchronized (mLock) {
               if (!mFrameScheduled) {
                   return; // no work to do
               }
   ...
               long intendedFrameTimeNanos = frameTimeNanos;
               startNanos = System.nanoTime();
               final long jitterNanos = startNanos - frameTimeNanos;
               if (jitterNanos >= mFrameIntervalNanos) {
                   final long skippedFrames = jitterNanos / mFrameIntervalNanos;
                   final long lastFrameOffset = jitterNanos % mFrameIntervalNanos;
                   frameTimeNanos = startNanos - lastFrameOffset;
               }
   
               if (frameTimeNanos < mLastFrameTimeNanos) {
                   scheduleVsyncLocked(); 
                   return;
               }
   
               if (mFPSDivisor > 1) {
                   long timeSinceVsync = frameTimeNanos - mLastFrameTimeNanos;
                   if (timeSinceVsync < (mFrameIntervalNanos * mFPSDivisor) && timeSinceVsync > 0) {
                       scheduleVsyncLocked();
                       return;
                   }
               }
   
               mFrameInfo.setVsync(intendedFrameTimeNanos, frameTimeNanos);
               mFrameScheduled = false;
               mLastFrameTimeNanos = frameTimeNanos;
           }
   
           try {
               Trace.traceBegin(Trace.TRACE_TAG_VIEW, "Choreographer#doFrame");
               AnimationUtils.lockAnimationClock(frameTimeNanos / TimeUtils.NANOS_PER_MS);
   
               mFrameInfo.markInputHandlingStart();
               doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);
   
               mFrameInfo.markAnimationsStart();
               doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
   
               mFrameInfo.markPerformTraversalsStart();
               doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
   
               doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
           } finally {
               AnimationUtils.unlockAnimationClock();
               Trace.traceEnd(Trace.TRACE_TAG_VIEW);
           }
       }
   ```

8. doFrame() 方法会去取出之前放进待执行队列里的任务来执行，取出来的这个任务实际上是 ViewRootImpl 的 doTraversal() 操作；

9. 上述第4步到第8步涉及到的消息都手动设置成了异步消息，所以不会受到同步屏障的拦截；

10. doTraversal() 方法会先移除主线程的同步屏障，然后调用 performTraversals() 开始根据当前状态判断是否需要执行performMeasure() 测量、perfromLayout() 布局、performDraw() 绘制流程，在这几个流程中都会去遍历 View 树来刷新需要更新的View；

<font color="#dd0000">**结来说，当有屏幕刷新操作时，系统会将View树的测量、布局和绘制等封装到一个Runnable，然后监听VSync信号，等Vsync信号来时，再触发此Runnable。**</font>

## LaunchMode & taskAffinity

摘自：https://www.cnblogs.com/yyz666/p/4674173.html

总结：

1. <font color="#dd0000">**SingleInstance的优先级很高，如果launchMode为SingleInstance，意味着所有的task中只能有一个目标Activity，且该Activity独占一个Task**</font>。
2. **SingleTop和Standard不受taskAffinity影响，即使两个Activity的taskAffinity不相同**，在这两个launchMode下，也会将两个Activity分在同一个Task中，这两者的区别在于SingleTop如果栈顶是目标Activity，则会复用Activity，走onNewIntent。
3. 当LaunchMode为SingleTask时，startActivity会先检测目标Activity的taskAffinity是否和当前Activity的taskAffinity相同，<font color="#dd0000">**如果相同，则依然将目标Activity分配到当前Task；反之新启一个Task**</font>。

问题：

1. A启动B，B启动SingleInstance的C，C返回，会返回到哪里？

2. A启动SingleTask且taskAffinity不同的B，B在启动standard的C，C在A的栈中，还是在B的栈中？

   

##利用Choreographer监测APP卡顿

摘自：https://www.jianshu.com/p/9e8f88eac490

Android系统每隔16ms发出VSYNC信号，触发对UI进行渲染。开发者可以使Choreographer#postFrameCallback**设置自己的callback**与Choreographer交互，你设置的FrameCallCack（doFrame方法）会在下一个frame被渲染时触发。理论上来说两次回调的时间周期应该在16ms，如果超过了16ms我们则认为发生了卡顿，我们主要就是利用两次回调间的时间周期来判断，

```java
Choreographer.getInstance()
  .postFrameCallback(new Choreographer.FrameCallback() {
    @Override
    public void doFrame(long l) {
      //移除消息
      Handler.removeMessage();
      //发送延时消息
      Hnadler.sendMessageAtTime(...)
      //we need to register for the next frame callback
      Choreographer.getInstance().postFrameCallback(this);
    }
  });
```

发送的延时消息在执行的时间没有被remove掉，说明发生了卡顿，这时候可以进行卡顿相关信息的采集，如果在渲染下一帧的时候该消息还没有被处理，这时候将该消息remove掉，此场景说明未发生卡顿；

<font color="#dd0000">**VSync信号由SurfaceFlinger实现并定时发送（每16ms发送）Choreographer.FrameDisplayEventReceiver收到信号后，调用onVsync方法组织消息发送到主线程处理。Choreographer主要功能是当收到VSync信号时，去调用使用通过postCallBack设置的回调函数，在postCallBack调用doFrame，在doFrame中渲染下一帧**</font>；FrameDisplayEventReceiver相关代码如下：

```java
//Choreographer.java
/**
     * FrameDisplayEventReceiver继承自DisplayEventReceiver接收底层的VSync信号开始处理UI过程。
     * VSync信号由SurfaceFlinger实现并定时发送。FrameDisplayEventReceiver收到信号后，
     * 调用onVsync方法组织消息发送到主线程处理。这个消息主要内容就是run方法里面的doFrame了，
     * 这里mTimestampNanos是信号到来的时间参数。
     */
private final class FrameDisplayEventReceiver extends DisplayEventReceiver
  implements Runnable {
  private boolean mHavePendingVsync;
  private long mTimestampNanos;
  private int mFrame;

  public FrameDisplayEventReceiver(Looper looper, int vsyncSource) {
    super(looper, vsyncSource);
  }

  @Override
  public void onVsync(long timestampNanos, int builtInDisplayId, int frame) {
    mTimestampNanos = timestampNanos;
    mFrame = frame;
    // 发送Runnable(callback参数即当前对象FrameDisplayEventReceiver)到FrameHandler，请求执行doFrame
    Message msg = Message.obtain(mHandler, this);
    msg.setAsynchronous(true);
    // 此处mHandler为FrameHandler，该Handler对应的Looper是主线程的Looper
    mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
  }

  @Override
  public void run() {
    mHavePendingVsync = false;
    doFrame(mTimestampNanos, mFrame);
  }
}
```

## 属性动画驱动力来自哪里

摘自：https://www.jianshu.com/p/30cfa00fc7a4

https://www.jianshu.com/p/cdba60a1104f

**View Animation**

```java
//ViewRootImpl.java
void scheduleTraversals() {
  ...
    mChoreographer.postCallback(
    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
  ...
}
```

**Property Animation**

在AnimationHandler中的循环取帧的关键代码

```java
public class AnimationHandler {
    private final Choreographer.FrameCallback mFrameCallback = new Choreographer.FrameCallback() {
        @Override
        public void doFrame(long frameTimeNanos) {
            doAnimationFrame(getProvider().getFrameTime());
            if (mAnimationCallbacks.size() > 0) {
                getProvider().postFrameCallback(this);
            }
        }
    };
  
    interface AnimationFrameCallback {
        boolean doAnimationFrame(long frameTime);
        void commitAnimationFrame(long frameTime);
    }
}
```

```java
class ValueAnimator extends Animator implements AnimationHandler.AnimationFrameCallback{
      public final boolean doAnimationFrame(long frameTime) {
        ...
    }
}
```

1.动画是由许多帧组成的
2.PropertyValuesHolder封装了帧的数据，并被动画类加以利用
3.利用Choreographer.FrameCallback的回调方法，来不停的调用帧执行流程
3.在计算变量数据的时候，插值器会去影响到最后输出的数据，从而达到某些效果
4.通过反射的原理，去修改属性的值，连贯起来出现了动画

## BlockCanary原理

摘自：https://www.jianshu.com/p/768e21ce76b5

Android中Looper的loop()方法中有：

```java
if (logging != null) {
  logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
}
```

其中的logging可以通过外部设置进来：

```java
public void setMessageLogging(@Nullable Printer printer) {
  mLogging = printer;
}
```

BlockCanary中实现了自定义的Printer类，重写println方法，并在其中开始收集堆栈信息和CPU信息。

```java
//Looper中的loop()
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```





![img](http://47.110.40.63:8080/img/blog/BlockCanary流程图.png)

## ReferenceQueue & Reference

摘自：https://www.jianshu.com/p/f86d3a43eec5

**Reference**

java.lang.ref.Reference 为 软（soft）引用、弱（weak）引用、虚（phantom）引用的父类。

**因为Reference对象和垃圾回收密切配合实现，该类可能不能被直接子类化。**
可以理解为Reference的直接子类都是由jvm定制化处理的,因此在代码中直接继承于Reference类型没有任何作用。

Reference提供了2个构造函数，一个带queue，一个不带queue。<font color="#dd0000">**其中queue的意义在于，我们可以在外部对这个queue进行监控。即如果有对象即将被回收，那么相应的reference对象就会被放到这个queue里。**</font>我们拿到reference，就可以再作一些事务。

而如果不带的话，就只有不断地轮询reference对象，通过判断里面的get是否返回null( phantomReference对象不能这样作，其get始终返回null，因此它只有带queue的构造函数 )。这两种方法均有相应的使用场景，取决于实际的应用。**如weakHashMap中就选择去查询queue的数据，来判定是否有对象将被回收。而ThreadLocalMap，则采用判断get()是否为null来作处理。**

**ReferenceQueue**

<font color="#dd0000">**引用队列，在检测到适当的可到达性更改后，垃圾回收器将已注册的引用对象添加到该队列中.**</font>

实现了一个队列的入队(enqueue)和出队(poll还有remove)操作，内部元素就是泛型的Reference，并且Queue的实现，是由Reference自身的链表结构( 单向循环链表 )所实现的。**ReferenceQueue名义上是一个队列，但实际内部并非有实际的存储结构，它的存储是依赖于内部节点之间的关系来表达**。

## Java 注解

**元注解**

元注解指的是**用来修饰注解的注解**，包括：

+ @Retention：定义注解的保留策略
  + SOURCE，注解只会保留在.java源码中，<font color="#dd0000">**运行的时候注解会被编译器移除**</font>。
  + CLASS，注解可以编译器的保留在class文件中，**但是不能在虚拟机运行时保留**。这也是默认的注解保留行为。
  + RUNTIME，注解可以在编译器及虚拟机的运行都保留下来，所以这时<font color="#dd0000">**注解可以用反射读取**</font>。
+ @Target：定义注解的作用目标
  + TYPE 类，接口(含注解)
  + FIELD 成员变量(含枚举常量)
  + METHOD 方法声明
  + PARAMETER 参数声明
  + CONSTRUCTOR 构造方法声明
  + LOCAL_VARIABLE 局部变量声明
  + ANNOTATION_TYPE 注解声明
+ @Inherited：说明该注解将被包含在javadoc中
+ @Documented：说明子类可以继承父类中的该注解

## Android APT

摘自：https://www.jianshu.com/p/7af58e8e3e18

**APT**(Annotation Processing Tool)即**注解处理器**，是一种处理注解的工具，确切的说它是javac的一个工具，它用来在**编译时**扫描和处理注解。注解处理器以**Java代码**(或者编译过的字节码)作为输入，生成**.java文件**作为输出。
<font color="#dd0000">**简单来说就是在编译期，通过处理注解，根据自定义规则生成java文件。**</font>

需要继承**AbstractProcessor**，并实现其中的process方法，在process方法中，使用JavaPoet生成Java文件。

## Kotlin协程的底层实现原理

摘自：https://www.jianshu.com/p/d23c688feae7

线程是操作系统的内核资源，是 CPU 调度的最小单位，所有应用程序的代码都运行于线程之上。

无论是回调，还是 RxJava，又或者是 Future 与 Promise，**线程都是我们曾经实现并发与异步的最根本的支撑**。在 Java 的 API 中，Thread 类是实现线程最基本的类，每创建一个 Thread 对象，就代表着在操作系统内核启动了一个线程，如果我们阅读 Thread 类的源码，可以发现，**它的内部实现是大量的 JNI 调用，因为线程的实现必须由操作系统直接提供支持**，如果是在 Android 平台上，我们会发现 Thread 的创建过程中，都会调用 Linux API 中的 pthread_create 函数，这直接说明了 Java 层中的 Thread 和 Linux 系统级别的中的线程是一一对应的。

首先，<font color="#dd0000">**协程本质上可以认为是运行在线程上的代码块，协程提供的 *挂起* 操作会使协程暂停执行，而不会导致线程阻塞**</font>。其次，协程是一种轻量级资源，即使创建了上千个协程，对于系统来说也不是一种很大的负担，就**如同在 Java 创建上千个 Runable 对象也不会造成过大负担一样**。通过这样设计，开发者可以极大的提高线程的使用率，用尽量少的线程执行尽量多的任务，其次调用者无需在编程时思考过多的资源浪费问题，可以在每当有异步或并发需求的时候就不假思索的开启协程。

协程的挂起和 Java 的 NIO 机制是类似的，我们在一个线程中执行了一个原本会阻塞线程的任务，但是这个调用者线程没有发生阻塞，这是<font color="#dd0000">**因为它们有一个专门的线程来负责这些任务的流转**</font>，也就是说，当我们发起多个阻塞操作的时候，可能只会阻塞这一个专门的线程，它一直在等待，谁的阻塞结束了，它就把回调再分派过去，这样就完成了阻塞任务与阻塞线程的多对一，而不是以前的一对一，所以挂起也好，NIO 也好，本质上都没有彻底消灭阻塞，但是它们都使阻塞的线程大大减少，从而避免了大量的线程上下文状态切换以及避免了大量线程的产生，从而在 IO 密集型任务中大大提高了性能。

## SharedPreference优化

摘自：https://zhuanlan.zhihu.com/p/22913991

```java
public void apply() {
  final long startTime = System.currentTimeMillis();

  final MemoryCommitResult mcr = commitToMemory();
  final Runnable awaitCommit = new Runnable() {
    @Override
    public void run() {
      try {
        mcr.writtenToDiskLatch.await();
      } catch (InterruptedException ignored) {
      }

      if (DEBUG && mcr.wasWritten) {
        Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
              + " applied after " + (System.currentTimeMillis() - startTime)
              + " ms");
      }
    }
  };

  QueuedWork.addFinisher(awaitCommit);
  Runnable postWriteRunnable = new Runnable() {
    @Override
    public void run() {
      awaitCommit.run();
      QueuedWork.removeFinisher(awaitCommit);
    }
  };

  SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);

  // Okay to notify the listeners before it's hit disk
  // because the listeners should always get the same
  // SharedPreferences instance back, which has the
  // changes reflected in memory.
  notifyListeners(mcr);
}
```

## AIDL通信===

摘自：https://www.cnblogs.com/ganchuanpu/p/9263578.html



## 多进程带来的问题

- 静态成员和单例模式完全失效(不是同一块内存，会产生不同的副本)
- 线程同步机制完全失效(不是同一块内存，所以对象也不是同一个，因此类锁、对象锁也不是同一个，不能保证线程同步)
- SharedPreferences 可靠性下降(SharedPreferences不支持多个进程同时写，会有一定的几率丢失数据)
- Application 多次创建(Android为每个进程分配独立的虚拟机，这个过程其实就是启动一个应用，所以Application会被创建多次)，所以我们不能直接将一些数据保存在Application中。

## SparseArray

摘自：https://blog.csdn.net/u013493809/article/details/21699121

Sparse [spɑːrs] 稀疏的

**SparseArray是android里为<Interger,Object>这样的Hashmap而专门写的类**,<font color="#dd0000">**目的是提高内存效率，底层基于数组，其核心是二分查找函数（binarySearch）**</font>。注意内存二字很重要，因为它仅仅提高内存效率，而不是提高执行效率，所以也决定它只适用于android系统（内存对android项目有多重要，地球人都知道）。SparseArray有两个优点：

1. 避免了自动装箱（auto-boxing）

2. 不需要开辟内存空间来额外存储外部映射

   我们知道HashMap 采用一种所谓的“Hash 算法”来决定每个元素的存储位置，存放的都是数组元素的引用，通过每个对象的hash值来映射对象。而SparseArray则是用数组数据结构来保存映射，然后通过折半查找来找到对象。但其实一般来说，SparseArray执行效率比HashMap要慢一点，因为查找需要折半查找，而添加删除则需要在数组中执行，而HashMap都是通过外部映射。但相对来说影响不大，**最主要是SparseArray不需要开辟内存空间来额外存储外部映射，从而节省内存**。

## ArrayMap

和SparseArray类似，也是基于数组实现。区别在于ArrayMap的Key数组元素为任意对象的hash值。

## Android如何实现长连接

1. [WebSocket](https://www.jianshu.com/p/954f022671fc)

   初始化服务-----初始化websocket------设置心跳的间隔----启动服务----发送消息(心跳包)------接收消息即可

2. [自定义实现长连接，可以通过NIO或者第三方NIO框架，如MINA实现](https://www.jianshu.com/p/7b3e5edf1951)

   Mina与后来兴起的高性能IO新贵Netty一样，都是韩国人Trustin Lee的大作。<font color="#dd0000">**Mina的底层依赖的主要是Java NIO库，上层提供的是基于事件的异步接口。**</font>

   [Android基于Mina实现Socket长连接](https://www.jianshu.com/p/9c37612f227f)

3. 使用第三方长连接服务，如Push

#Java核心

## 线程状态===



## wait/sleep/notify

**wait和sleep的区别**

<font color="#dd0000">**wait 会释放锁，而 sleep 一直持有锁**</font>。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。如果能够帮助你记忆的话，**可以简单认为和锁相关的方法都定义在Object类中**，因此调用Thread.sleep是不会影响锁的相关行为。<font color="#dd0000">**调用wait后，需要别的线程执行notify/notifyAll才能够重新获得CPU执行时间。**</font>

## Minor/Major/Full GC

**Minor GC**指新生代GC，即发生在新生代（包括Eden区和Survivor区）的垃圾回收操作，当新生代无法为新生对象分配内存空间的时候，会触发Minor GC。因为新生代中大多数对象的生命周期都很短，所以发生Minor GC的频率很高，<font color="#dd0000">**虽然它会触发stop-the-world，但是它的回收速度很快**</font>。

**Major GC**清理Tenured区，用于回收老年代，出现Major GC通常会出现至少一次Minor GC。

**Full GC**是针对整个新生代、老生代、元空间（metaspace，java8以上版本取代perm gen）的全局范围的GC。<font color="#dd0000">**Full GC不等于Major GC，也不等于Minor GC+Major GC**</font>，发生Full GC需要看使用了什么垃圾收集器组合，才能解释是什么样的垃圾回收。

## Java的异常体系

Thorwable类所有异常和错误的超类，有两个子类Error和Exception，分别表示错误和异常。

Error是程序无法处理的错误，比如OutOfMemoryError、ThreadDeath等。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。

Exception是程序本身可以处理的异常，这种异常分两大类运行时异常和非运行时异常。程序中应当尽可能去处理这些异常。

## try-with-resources

try-with-resources 声明、try 块、catch 块。它要求在 try-with-resources 声明中<font color="#dd0000">**定义的变量实现了 AutoCloseable 接口，这样在系统可以自动调用它们的close方法，从而替代了finally中关闭资源的功能**</font>。

## 线程池种类，关键参数

当我们通过execute(Runnable)方法向线程池中添加一个待执行的任务时，会判断当前核心池(**corePoolSize**)是否已经满了，如果核心池满就将任务添加到工作队列(**workQueue**)(根据选择不同的BlockingQueue有不同的排队规则)，如果工作队列也满了，那么就再创建新的线程，直至线程数达到最大线程数(**maximumPoolSize**)；如果此时还有任务要添加进来，而此时已经处理不了任务了，就只能拒绝任务，此时可以根据指定的handler来指定拒绝策略。

<font color="#dd0000">**newSingleThreadExecutor**</font>：单线程化线程池的优点，串行执行所有任务。**如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。**底层使用**LinkedBlockingQueue**作为工作队列，和fixedThreadPool不同的是<font color="#dd0000">**它外面包了一层FinalizableDelegatedExecutorService**</font>，这导致newSingleThreadExecutor<font color="#dd0000">**无法向下转型成ThreadPoolExecutor**</font>，从而确保了对象不被修改。

<font color="#dd0000">**newFixedThreadPool**</font>：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。**线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。**底层使用**LinkedBlockingQueue**作为工作队列，LinkedBlockingQueue是一个无界的阻塞队列，<font color="#dd0000">**所以当任务的生成速度持续大于任务的处理速度。使用这种线程池，将造成大量阻塞，从而引发内存溢出等问题。**</font>

<font color="#dd0000">**newCachedThreadPool**</font>：创建一个可缓存的线程池，<font color="#dd0000">**核心线程数等于0**</font>。**如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程**，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。底层使用**SynchronousQueue**作为工作队列，**SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另外一个线程已经准备好参与到交付过程中。**

<font color="#dd0000">**newScheduledThreadPool**</font>：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。底层使用**DelayedWorkQueue**作为工作队列。

## readResolve()方法与序列化

摘自：https://blog.csdn.net/huangbiao86/article/details/6896565

 一般来说，一个类实现了Serializable接口, 我们就可以把它往内存地写再从内存里读出而"组装"成一个跟原来一模一样的对象。**不过当序列化遇到单例时，这里边就有了个问题: 从内存读出而组装的对象破坏了单例的规则。单例是要求一个JVM中只有一个类对象的，而现在通过反序列化，一个新的对象克隆了出来**。

```java
public final class MySingleton implements Serializable {
     private MySingleton() { }
     private static final MySingleton INSTANCE = new MySingleton();
     public static MySingleton getInstance() { return INSTANCE; }
}
```

改了readResolve()以后：

```java
public final class MySingleton implements Serializable{
    private MySingleton() { }
    private static final MySingleton INSTANCE = new MySingleton();
    public static MySingleton getInstance() { return INSTANCE; }
    private Object readResolve() throws ObjectStreamException {
       // instead of the object we're on,
       // return the class variable INSTANCE
      return INSTANCE;
   }
}
```

<font color="#dd0000">**这样当JVM从内存中反序列化地"组装"一个新对象时，就会自动调用这个 readResolve方法来返回我们指定好的对象了， 单例规则也就得到了保证**</font>。

## Synchronized与ReentrantLock区别

在Synchronized优化以前，synchronized的性能是比ReenTrantLock差很多的，但是自从Synchronized引入了偏向锁，轻量级锁（自旋锁）后，两者的性能就差不多了，在两种方法都可用的情况下，官方甚至建议使用synchronized，其实synchronized的优化我感觉就借鉴了ReenTrantLock中的CAS技术。

**Synchronized**

进过编译，会在同步块的前后分别形成monitorenter和monitorexit这个两个字节码指令。在执行monitorenter指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1，相应的，在执行monitorexit指令时会将锁计算器就减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止。

**ReentrantLock**

由于ReentrantLock是java.util.concurrent包下提供的一套互斥锁，相比Synchronized，ReentrantLock类提供了一些高级功能，主要有以下3项：

1. 等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况。通过lock.lockInterruptibly()来实现这个机制。

2. 公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，Synchronized锁是非公平锁，**ReentrantLock默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性能不是很好**。

3. 锁绑定多个条件，一个ReentrantLock对象可以同时绑定多个对象。ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

   Condition类能实现synchronized和wait、notify搭配的功能，另外比后者更灵活，Condition可以实现多路通知功能，也就是在一个Lock对象里可以创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中，从而**可以有选择的进行线程通知，在调度线程上更加灵活**。而synchronized就相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在这个对象上。线程开始notifyAll时，需要通知所有的WAITING线程，没有选择权，会有相当大的效率问题。

ReenTrantLock的实现是一种自旋锁，通过循环调用CAS+volatile操作来实现加锁。

ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。







## 一些比喻

摘自：https://www.jianshu.com/p/47eca41428d6

SurfaceFling是摄像机，它只负责客观的捕捉当前的画面，然后真实的呈现给观众；WMS就是导演，它要负责话剧的舞台效果、演员站位；ViewRoot就是各个演员的长相和表情，取决于它们各自的条件与努力。可见，WMS与SurfaceFling的一个重要区别就是——后者只做与“显示”相关的事情，而WMS要处理对输入事件的派发。

摘自：https://www.jianshu.com/p/5297e307a688

Activity就像工匠，Window就像是窗户，View就像是窗花，LayoutInflater像剪刀，Xml配置像窗花图纸。
Android根据他们不同的职能让他们各斯其活，同时也相互配合展示给我们灵活、精致的界面。

摘自：https://www.jianshu.com/p/18bf94da4a67

WindowManager.java抽象类，具体实现靠WindowManagerImpl

WindowManagerImpl.java委托windmanagerGlobal干活

WindowManagerGlobal.java真正完成windowmanager任务

ViewRootImpl.java

PhoneWindow.java这个才是我们看到的window

PhoneWindowManager.java

以上类全部在客户端中使用.

IWindowSession:

IWindowSession 在客户端中使用,他是window在客户端持有windowManagerService的类似于应用的东西,客户端通过IWindowSession向WMS发送请求.

IWindowSession 在viewRootImpl构造函数中初始化:

## Serializable和Parcelable区别===





## MeasureSpec详解

摘自：https://www.jianshu.com/p/cecd0de7ec27

**NSPECIFIED：不对View大小做限制，如：ListView，ScrollView**
**EXACTLY：确切的大小，如：100dp或者march_parent**
**AT_MOST：大小不可超过某数值，如：wrap_content**

总结一下：
不管父View是何模式，若子View有确切数值，则子View大小就是其本身大小，且mode是EXACTLY
若子View是match_parent，则模式与父View相同，且大小同父View（若父View是UNSPECIFIED，则子View大小为0）
若子View是wrap_content，则模式是AT_MOST，大小同父View，表示不可超过父View大小（若父View是UNSPECIFIED，则子View大小为0）

## setContentView流程

1. Activity有成员变量mWindow，是一个PhoneWindow，在Activity的attach()方法中初始化

   ```java
   //Activity.java
   private void attach(...) {
     mWindow = new PhoneWindow(this, window, activityConfigCallback);
   }
   ```

2. 调用setContentView，实际上调用的是PhoneWindow的setContentView方法，Window是一个抽象类，PhoneWindow实现了Window。WindownManager是一个接口继承自ViewManager接口，ViewManager接口中定义了三个方法用于添加，更新，删除View。

3. WindowManagerImpl是WindowManager的实现类，其有一个成员变量WindowManagerGlobal，**WindowManagerGlobal是实际干活的那个**，当调用PhoneWindow的setContentView时，最终会调到WindowManagerGlobal的addView，**addView又会调用ViewRootImpl的setView方法**，<font color="#dd0000">**而在ViewRootImpl的setView方法中，会调用IWindowSession类的addToDiaplay方法通过IPC将View和Window绑定**</font>。

4. 回到PhoneWindow的setContentView方法中

   首先判断decorview是否初始化过，如果没有，调用installDecor()。

   如果contentParent为空，则调用generateLayout()，generateLayout中根据设置的feature生成不同的contentParent。

   在前面讨论的ViewRootImpl的setView中会调用requestLayout()，requestLayout()又会调用scheduleTraversals()，scheduleTraversals()会post了一个mTraversalRunnable，mTraversalRunnable会调用doTraversal()，doTraversal最终调用performTraversals()，performTraversals()触发measure/layout/draw三个步骤。

   ```java
   public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
        synchronized (this) {
            if (mView == null) {
                mView = view;
                ……
                requestLayout();;
            }
        }
    }
   @Override
   public void requestLayout() {
     if (!mHandlingLayoutInLayoutRequest) {
       checkThread();
       mLayoutRequested = true;
       scheduleTraversals();
     }
   }
   void scheduleTraversals() {
     if (!mTraversalScheduled) {
       mTraversalScheduled = true;
       mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
       mChoreographer.postCallback(
         Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
       if (!mUnbufferedInputDispatch) {
         scheduleConsumeBatchedInput();
       }
       notifyRendererOfFramePending();
       pokeDrawLockIfNeeded();
     }
   }
   ```

   可以看到这里post了一个mTraversalRunnable，我们看看这个runnable做了啥事

   ```
    final class TraversalRunnable implements Runnable {
           @Override
           public void run() {
               doTraversal();
           }
       }
       final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
       void doTraversal() {
           if (mTraversalScheduled) {
               mTraversalScheduled = false;
               mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
   
               if (mProfile) {
                   Debug.startMethodTracing("ViewAncestor");
               }
   
               performTraversals();
   
               if (mProfile) {
                   Debug.stopMethodTracing();
                   mProfile = false;
               }
           }
       }
   
   ```

##measure，layout，draw流程

摘自：[https://jitmaos.github.io/2019/04/09/Android%E5%9F%BA%E7%A1%80%E4%B8%89%E9%83%A8%E6%9B%B2%E3%80%8EonDraw+onMessage+onLayout%E3%80%8F/](https://jitmaos.github.io/2019/04/09/Android基础三部曲『onDraw+onMessage+onLayout』/)

## 浏览器缓存机制

摘自：https://www.jianshu.com/p/39a9832847a6

浏览器缓存机制是指通过 HTTP Header 部分的 Cache-Control（或 Expires）和 Last-Modified（或 Etag）等字段来控制文件缓存的机制。

1. Last-Modified

   标识文件在服务器的最新更新修改时间。请求资源时，浏览器通过 If-Modified-Since 字段带上本地缓存的最新修改时间，服务器通过比较缓存文件的修改时间是否一致，来判断文件是否有修改。如果没有修改，则服务器返回 304 告知浏览器继续使用缓存；否则返回 200，同时返回最新的文件。

   

   ```cpp
   // 服务器返回
   Last-Modified: Tue, 12 Jan 2016 09:31:27 GMT
   // 客户端再次发送请求
   If-Modified-Since: Tue, 12 Jan 2016 09:31:27 GMT
   ```

2. Cache-Control

   相对值，单位是秒，指定某个文件从发出请求开始起的有效时长，在这个有效时长之内，浏览器直接使用缓存，而不发送请求。Cache-Control 不用要求服务器与客户端的时间同步，也不用服务器时刻同步修改配置 Expired 中的绝对时间，从而可以避免额外的网络请求。优先级比 Expires 更高。

   

   ```swift
   Cache-Control: max-age=600, public
   ```

   > Cache-Control 通常与 Last-Modified 一起使用。一个用于控制缓存有效时间，一个在缓存失效后，向服务查询是否有更新。

3. Expires

   表示到期时间，一般用在 response 报文中，当超过此时间后响应将被认为是无效的而需要网络连接，反之而是直接使用缓存

   

   ```css
   Expires: Thu, 12 Jan 2017 11:01:33 GMT
   ```

4. ETag

   是对文件进行标识的特征字符串。浏览器向服务器请求文件时，通过 If-None-Match 字段把特征字串发送给服务器，服务器通过 Etag 比对来判断文件是否更新。Etag 一致，则返回 304；否则返回 200 和最新的文件。

   

   ```dart
   // 服务器返回
   ETag: 248b11be4d6c7db6b2a699988a6603a5
   // 客户端再次发送请求
   If-None-Match: 248b11be4d6c7db6b2a699988a6603a5
   ```

   ETag 和 Last-Modified 一同使用，是要其中一个判断缓存有效，则浏览器使用缓存数据

5. Cache-Control:no-cache (Pragma:no-cache)

   浏览器忽略本地缓存，请求 HEADER 中代码带上改字段，请求服务器获取最新的数据

![img](http://47.110.40.63:8080/img/blog/网络资源缓存流程图.png)

## 解决启动黑白屏问题

摘自：https://www.jianshu.com/p/ac1fb0035aa3

1. 设置Application的样式，android:windowBackground设置为splash图片

   ```java
   <style name="AppTheme.lanuchTheme" >
         <item name="android:windowBackground">@drawable/launch_layout</item>
         <item name="android:windowFullscreen">true</item>
         <item name="android:windowNoTitle">true</item>
         <item name="android:windowContentOverlay">@null</item>
   ```

2. 将真的SplashActivity的背景设置为透明

   ```java
   <?xml version="1.0" encoding="utf-8"?>
   <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <ImageView
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:src="@mipmap/bottom_imag"//bottom_imag为底部的图标和文字
           android:layout_alignParentBottom="true"
           android:layout_centerHorizontal="true"
           android:layout_marginBottom="50dp"
           />
   </RelativeLayout>
   ```

## TCP/UDP/HTTP/SOCKET区别

摘自：https://blog.csdn.net/hai_chao/article/details/79626161

## 客户端的HTTPS原理

1. 客户端生成随机字符串A，作为数据**传输对称加密**的秘钥。
2. 使用<font color="#dd0000">**本地预置的RSA公钥**</font>对**随机字符串**进行加密A，得到加密字符串B。
3. 将加密字符串B传输到服务器，服务器使用<font color="#dd0000">**RSA私钥**</font>进行解密，得到对称加密字符串。
4. **之后只需要使用对称加密字符串进行文件加密传输**。

## Class文件解析

摘自：

1. https://blog.csdn.net/weelyy/article/details/78969412
2. 《深入理解Java虚拟机》



#Android架构设计

## 组件化通信

摘自：https://www.jianshu.com/p/82b994fe532c

一、页面跳转使用**统一路由管理**，如Arouter。

二、**使用共享数据，如把数据保存到SP、数据库等。然后发送一个广播**

三、使用EventBus这种进程内bus，**需要定义Base Bean**，通过key进行区分。



## 插件化实现原理===



# 源码分析



## LeakCanary

摘自：https://blog.csdn.net/braintt/article/details/99685243

LeakCanary实现内存泄漏的主要判断逻辑是这样的。<font color="#dd0000">**当我们观察的Activity或者Fragment销毁时，我们会使用一个弱引用去包装当前销毁的Activity或者Fragment,并且将它与本地的一个ReferenceQueue队列关联。我们知道如果GC触发了，系统会将当前的引用对象存入队列中**</font>。
如果没有被回收，队列中则没有当前的引用对象。所以LeakCanary会去判断，**ReferenceQueue是否有当前观察的Activity或者Fragment的引用对象，第一次判断如果不存在，就去手动触发一次GC，然后做第二次判断，如果还是不存在，则表明出现了内存泄漏。**

## 常见的内存泄漏

摘自：https://www.cnblogs.com/sjxbg/p/11643715.html

1. 对象被静态成员引用

   ```java
   private Random random = new Random();
   public static final ArrayList<Double> list = new ArrayList<Double>(1000000);
   for (int i = 0; i < 1000000; i++) { list.add(random.nextDouble()); }
   ```

   ArrayList是在堆上动态分配的对象，正常情况下使用完毕后，会被gc回收，但是在此示例中，由于被静态成员list引用，而静态成员是不会被回收的，所以会导致这个很大的ArrayList一直停留在堆内存中。

2. String的intern方法

   在大字符串上调用String.intern() 方法，intern()会将String放在jvm的内存池中（PermGen ），而jvm的内存池是不会被gc的。因此如果大字符串调用intern()方法后，会产生大量的无法gc的内存，导致内存泄漏。

   如果必须要使用大字符串的intern方法，应该通过-XX:MaxPermSize参数调整PermGen内存的大小。

3. 读取流后没有关闭

   开发中经常忘记关闭流，这样会导致内存泄漏。因为每个流在操作系统层面都对应了打开的文件句柄，流没有关闭，**会导致操作系统的文件句柄一直处于打开状态，jvm会消耗内存来跟踪操作系统打开的文件句柄**。

4. 将没有实现hashcode和equals方法的对象加入到HashSet中

   当一个对象被存储在Hashset中后，如果修改参与计算hashcode有关的字段，那么修改后的hashcode的值就与一开始存储进来的hashcode的值不同了，**这样contains无法通过hashcode找到该元素，所以无法删除**。

## AsyncTask原理及不足

摘自：https://blog.csdn.net/wjinhhua/article/details/60578133

<font color="#dd0000">**AsyncTask内部是由一个串行线程池(SerialExecutor) + Handler的方式实现的**</font>。

**涉及四个主要方法**

```java
onPreExecute() //在主线程运行，用于后台任务进行前做一些准备工作
doInBackground(Params... params) //在子线程运行，用来处理后台任务，像网络请求，耗时处理等操作
onProgressUpdate(Progress... values) //在主线程运行，在doInBackground通过publishProgress来更新进度
onPostExecute(Result result)//在主线程运行，后台任务处理完毕调用，并返回后台任务的结果
```

**存在的缺陷**

1. 线程池中已经有128个线程，缓冲队列已满，如果此时向线程提交任务，将会抛出RejectedExecutionException。过多的线程会引起大量消耗系统资源和导致应用FC的风险。

2. <font color="#dd0000">**AsyncTask不会随着Activity的销毁而销毁，直到doInBackground()方法执行完毕(因为它运行在子线程中吧)**</font>。如果我们的Activity销毁之前，没有取消 AsyncTask，这有可能让我们的AsyncTask崩溃(crash)。因为它想要处理的view已经不存在了。所以，我们总是必须确保在销毁活动之前取消任务。**如果在doInBackgroud里有一个不可中断的操作，比如BitmapFactory.decodeStream()，调用了cancle() 也未必能真正地取消任务**。关于这个问题，在4.4后的AsyncTask中，都有判断是取消的方法isCancelled()，可能参考的这些作者都分析较早的版本，当然，这是笔者落后的原因。

3. <font color="#dd0000">**如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对创建了AsyncTask的Activity的引用（不如说是其内部的Handler引用了Activity吧)**</font>。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄露。

4. <font color="#dd0000">**屏幕旋转或Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效**</font>。

## LinkedHashMap

摘自：https://www.jianshu.com/p/8f4f58b4b8ab

LinkedHashMap中存的Item是LinkedHashMapEntry

![img](http://47.110.40.63:8080/img/blog/LinkedHashMap结构图.png)

```java
class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V> {
	static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
	  LinkedHashMapEntry<K,V> before, after;
	  LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
 	   super(hash, key, value, next);
	  }
	}
	transient LinkedHashMapEntry<K,V> head;
	transient LinkedHashMapEntry<K,V> tail;
}
```

LinkedHashMap支持两种元素访问顺序，由构造函数参数accessOrder决定，accessOrder=false时，表示和存储顺序有关，否则按照访问的顺序。

LinkedHashMap是继承于HashMap，是基于HashMap和双向链表来实现的。

HashMap无序；LinkedHashMap有序，可分为插入顺序和访问顺序两种。如果是访问顺序，那put和get操作已存在的Entry时，都会把Entry移动到双向链表的表尾(其实是先删除再插入)。

LinkedHashMap存取数据，还是跟HashMap一样使用的Entry[]的方式，双向链表只是为了保证顺序。

LinkedHashMap是线程不安全的。

## LruCache实现原理

摘自：https://www.jianshu.com/p/6d41994f1fee

​			https://www.jianshu.com/p/1b25935acfa0

LruCache(Least Recently Used)是一个基于LinkedHashMap进行封装的**线程安全**的缓存容器。<font color="#dd0000">**LruCache是在内存中使用的**</font>，DiskLruCache是在磁盘中使用的。

自己实现一个LruCache的关键步骤

1.(必填)你需要提供一个缓存容量作为构造参数。

2.（必填） 覆写 sizeOf 方法 ，自定义设计一条数据放进来的容量计算，如果不覆写就无法预知数据的容量，不能保证缓存容量限定在最大容量以内。

3.（选填） 覆写 entryRemoved 方法 ，你可以知道最少使用的缓存被清除时的数据（ evicted, key, oldValue, newVaule ）。

4.（记住）LruCache是<font color="#dd0000">**线程安全**</font>的，在内部的 get、put、remove 包括 trimToSize 都是安全的（因为都上锁了）。

5.（选填） 还有就是覆写 create 方法 。create()方法的执行可能需要很长时间，此时如果有其他线程也操作相同key的对象，则将当前create的对象舍弃。

LruCache源码异常的精简，核心原理是通过<font color="#dd0000">**LinkedHashMap双向循环链表**</font>，每次访问过的数据会被移动到<font color="#dd0000">**队列末尾**</font>，在使用过程中我们需要重写sizeOf方法来帮助LruCache计算缓存大小，每当缓存数据发生覆盖或者清理时会回调entryRemoved方法，并且LruCache是<font color="#dd0000">**线程安全的，核心操作都上了同步锁**</font>。
 Ps:我们可以手动调用trimToSize清理一批数据，也可以调用resize方法，重新赋值缓存大小的上限并计算当前空间是否需要清理，snapshot来获取缓存map的切片，注意是浅拷贝。

## DiskLruCache使用及思考

摘自：https://blog.csdn.net/guolin_blog/article/details/28863651

**写入缓存**

1. 使用DiskLruCache.open(File directory, int appVersion, int valueCount, long maxSize)方法得到一个DiskLruCache对象，**注意处理存储权限问题**，存储路径可以是任意的。

2. 获取一个指定key的Editor对象

   ```java
   public Editor edit(String key) throws IOException
   ```

3. 调用editor.commit()提交缓存，或者调用editor.abort()放弃提交缓存。

4. 可以调用flush()方法，将内存中的操作记录一次性同步到journal文件，通常在onPause()方法中使用。

5. 和open方法相对应，close()用于关闭DiskLruCache对象的句柄（应该说是文件句柄吧)。可以在onDestory()中调用。

**读取缓存**

1. 调用Editor.get()返回一个目标key的SnapShot(存放key对应的values)。

**底层原理**

1. 整个DiskLruCache的运行依赖于journal文件，journal文件基本格式如下：

   ![img](http://47.110.40.63:8080/img/blog/DiskLruCache文件截图.png)

   **每次调用edit()会生成一条DIRTY记录，之后调用commit()会生成一条clean记录，调用abort会生成一条remove记录**。

2. 每次往journal文件中添加操作行的时候，redundantOpCount变量会计数，如果该文件的行数超过了2000行，就会重建一次journal文件，清除不需要的记录行。

3. DiskLruCache和LruCache的相同点是缓存文件都先写入，然后触发计算空间大小来判断是否删除近期时间最少使用的文件，直到缓存文件的大小大于阈值。

4. 调用open的时候会逐行读取journal文件中的内容，将需要

5. 每次先插入缓存，之后判断当前是否需要journalRebuildRequired(由2000条记录限制)，或者当前缓存总大小是否超过最大缓存文件限制，是的话，就调用trimToSize方法。

   ```java
   private void trimToSize() throws IOException {
     while (size > maxSize) {
       Map.Entry<String, Entry> toEvict = lruEntries.entrySet().iterator().next();
       remove(toEvict.getKey()); //remove里会更新size字段，size由read记录后的文件数字计算得到
     }
   }
   ```

6. 调用open()会遍历日志文件的每一行，解析每一行调用get(key)方法，<font color="#dd0000">**基于LinkedHashMap的特性，被访问过得entry会排在尾部。如果get(key)在内存中找不到对应的cache，则使用put()，依然会被安排在尾部**</font>。然后调用commit()的时候，**会判断缓存文件的总大小，如果超出限制。则调用remove()方法，移除内存中头部元素并删除文件并更新日志文件**。之后判断是否记录超过2000行，看是否需要rebuild日志文件。

## Glide三级图片缓存

摘自：https://www.jianshu.com/p/17644406396b

**Glide内存缓存源码分析**

内存存缓存的 读存都在Engine类中完成。缓存对象有两个

```java
private final MemoryCache cache; //LruCache，使用完的图片保存在这里
private final Map<Key, WeakReference<EngineResource<?>>> activeResources; //使用中的缓存
```

使用中的图片，保存在弱引用activeResources中；使用完以后保存在LruCache中；

loadFromCache方法读取缓存时，先判断isMemoryCacheable是不是false；之后从LruCache中读取缓存，读取到缓存以后将对象放置到使用中缓存activeResources，并且每次引用都会在图片对象Resources中使用计数+1。

<font color="#dd0000">**Glide内存缓存的特点**</font>

内存缓存使用<font color="#dd0000">**弱引用和LruCache**</font>结合完成的,<font color="#dd0000">**弱引用来缓存的是正在使用中的图片**</font>。**图片封装类Resources内部有个计数器判断是该图片否正在使用**。

<font color="#dd0000">**Glide内存缓存的流程**</font>

- 读：是先从lruCache取，取不到再从弱引用中取；
- 存：内存缓存取不到，从网络拉取回来先放在弱引用里，渲染图片，图片对象Resources使用计数加一；
- 渲染完图片，图片对象Resources使用计数减一，如果计数为0，图片缓存从弱引用中删除，放入lruCache缓存。

<font color="#dd0000">**Glide磁盘缓存流程**</font>

基于DiskLruCache算法实现

- 读：**先找处理后（result）的图片，没有的话再找原图**。
- 存：**先存原图，再存处理后的图**。



## Glide使用优化

内存缓存使用弱引用和LruCache结合完成的,弱引用来缓存的是正在使用中的图片。图片封装类Resources内部有个计数器判断是该图片否正在使用。

## ButterKnife源码解析

摘自：https://juejin.im/post/5acec2b46fb9a028c6761628

1. 使用注解标注bindview和onclick。

2. APT(Annotation Processing Tool)，准确的说是ButterKnifeProcessor会在编译器解析到这些注解，然后根据自定义规则，生成以"_ViewBinding"结尾的java文件。如：

   ```java
   public class MainActivity_ViewBinding implements Unbinder {
     private MainActivity target;
   
     private View view2131165217;
   
     @UiThread
     public MainActivity_ViewBinding(MainActivity target) {
       //使用target.getWindow().getDecorView()获取rootView
       this(target, target.getWindow().getDecorView());
     }
   
     @UiThread
     public MainActivity_ViewBinding(final MainActivity target, View source) {
       this.target = target;
   
       View view;
       //完成绑定1
       target.title = Utils.findRequiredViewAsType(source, R.id.tv_title, "field 'title'", TextView.class);
       view = Utils.findRequiredView(source, R.id.bt_submit, "method 'submit'");
       view2131165217 = view;
       //完成绑定2
       view.setOnClickListener(new DebouncingOnClickListener() {
         @Override
         public void doClick(View p0) {
           target.submit();
         }
       });
     }
   
     @Override
     @CallSuper
     public void unbind() {
       MainActivity target = this.target;
       if (target == null) throw new IllegalStateException("Bindings already cleared.");
       this.target = null;
   
       target.title = null;
   
       view2131165217.setOnClickListener(null);
       view2131165217 = null;
     }
   }
   ```

三、在Activity、Fragment中调用bind方法，改方法会传入Activity、Fragment为参数。<font color="#dd0000">**bind方法最终只是为了将当前的Activity、Fragment作为参数，来构造一个Xxx_ViewBinding对象，在该对象的构造方法中，完成了需要注入对象的左右绑定**</font>，值得一提的是，rootView是直接通过target.getWindow().getDecorView()获取的。

## OKHTTP任务管理

1. 在构造OkHttpClient对象的Builder时，会在Builder的构造方法中创建一个Dispatcher()，Dispatcher在OkHttp中扮演了一个管理调度器的角色。

2. **Dispatcher中包含三个任务队列，分别是：readyAsyncCalls(异步就绪等待队列)、runningAsyncCalls(运行中的异步任务队列)和runningSyncCalls(运行中的同步任务队列)。**同时，Dispatcher中还包含了一个线程池，<font color="#dd0000">**这是一个基于同步队列(SynchronousQueue)的线程池**</font>，Dispatcher在提交任务到线程池时会对当前线程执行情况做校验：

   ```java
   synchronized void enqueue(AsyncCall call) {
       if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
         runningAsyncCalls.add(call);
         executorService().execute(call);
       } else {
         readyAsyncCalls.add(call);
       }
     }
   ```

   **注意，maxRequests和maxRequestPerHost只对异步任务做限制。**

3. <font color="#dd0000">**不管是同步任务还是异步任务，最后都会执行execute()**</font>，这段代码被包含在一个catch块中，该catch块后面的finally中都执行了Dispatcher.finished(Call)方法

   ```java
   @Override public Response execute() throws IOException {
       synchronized (this) {
         if (executed) throw new IllegalStateException("Already Executed");
         executed = true;
       }
       captureCallStackTrace();
       eventListener.callStart(this);
       try {
         client.dispatcher().executed(this);
         Response result = getResponseWithInterceptorChain();
         if (result == null) throw new IOException("Canceled");
         return result;
       } catch (IOException e) {
         eventListener.callFailed(this, e);
         throw e;
       } finally {
         client.dispatcher().finished(this);
       }
     }
   ```

4. 在finished方法中会根据isPromoteCall来决定是否会触发promoteCalls方法(Dispatcher有多个重载的finished()方法，**同步任务不触发promoteCalls，异步方法触发promoteCalls方法**)：

   ```java
     private void promoteCalls() {
       if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
       if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.
   
       for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
         AsyncCall call = i.next();
   
         if (runningCallsForHost(call) < maxRequestsPerHost) {
           i.remove();
           runningAsyncCalls.add(call);
           executorService().execute(call);
         }
         if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
       }
     }
   ```

   从代码中可以看出promoteCalls方法是用于**从就绪等待队列中拿去任务到异步运行队列的**。

## OKHTTP的缓存机制

摘自：https://www.jianshu.com/p/fd37321f74cc

http缓存分为两种，一种强制缓存，一种对比缓存，强制缓存生效时直接使用以前的请求结果，无需发起网络请求。对比缓存生效时，无论怎样都会发起网络请求，如果请求结果未改变，服务端会返回304，但不会返回数据，数据从缓存中取，如果改变了会返回数据。

存储：

1. okhttp缓存存储是基于文件存储，从启用缓存机制传入的两个参数也可以看出：directory->缓存文件目录，maxSize->缓存支持存储的最大字节数。

清理：

1. 缓存清理是基于LRU机制，清理最老且使用最少的数据
2. 内部维护一个清理线程，当size大于或等于maxSize时就会执行清理操作

**CacheStrategy**

通过当前的request和一个<font color="#dd0000">**缓存中的response**</font>来决定当前request的响应是使用网络、缓存、或者二者都使用。

```java
class CacheStrategy {
  //M->OKHTTP，要发送到服务器的Request，如果不走网络的话，为Null
  public final @Nullable Request networkRequest;

  //M->OKHTTP,缓存的response，可以用来作为结果返回，或者用来辅助验证，如果当前的call指定不使用缓存则返回null
  public final @Nullable Response cacheResponse;
  //M->OKHTTP,判断响应是否可缓存，根据http响应状态码判断是否可缓存
  public static boolean isCacheable(Response response, Request request) {}
  public static class Factory {
    //更新cacheResponse的header的字段Date/Expires/Last-Modified/ETag/Age
  }
  //M->OKHTTP，基于缓存Response返回一个满足request的CacheStrategy
  public CacheStrategy get() {
    //1.根据Request和Response的cacheControl来返回一个cacheStategy
    //2.根据Cache Response的header信息的etag/If-Modified-Since/If-Modified-Since返回一个cacheStrategy
  }
}
```

**CacheControl**

```java
- noCache();//不使用缓存，用网络请求
- noStore();//不使用缓存，也不存储缓存
- onlyIfCached();//只使用缓存
- noTransform();//禁止转码
- maxAge(10, TimeUnit.MILLISECONDS);//设置超时时间为10ms。
- maxStale(10, TimeUnit.SECONDS);//超时之外的超时时间为10s
- minFresh(10, TimeUnit.SECONDS);//超时时间为当前时间加上10秒钟。
```

**保存缓存**

```java
//CacheInterceptor.intercept
if (cache != null) {
  if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
    //M->OKHTTP,关键代码！！保存response到缓存中,只保存GET请求的结果
    CacheRequest cacheRequest = cache.put(response);
    //
    return cacheWritingResponse(cacheRequest, response);
  }

  //M->OKHTTP，post/patch/put/delete/move时，从缓存中移除request
  if (HttpMethod.invalidatesCache(networkRequest.method())) {
    try {
      cache.remove(networkRequest);
    } catch (IOException ignored) {
      // The cache cannot be written.
    }
  }
}
```

注意：cache.put(response)中只保存GET请求的结果

```java
@Nullable CacheRequest put(Response response) {
  String requestMethod = response.request().method();
  	...
    if (!requestMethod.equals("GET")) {
      // Don't cache non-GET responses. We're technically allowed to cache
      // HEAD requests and some POST requests, but the complexity of doing
      // so is high and the benefit is low.
      return null;
    }
 	  ...
    try {
      editor = cache.edit(key(response.request().url()));
      if (editor == null) {
        return null;
      }
      entry.writeTo(editor);
      return new CacheRequestImpl(editor);
    } catch (IOException e) {
      abortQuietly(editor);
      return null;
    }
}
```

**总结**

- OkHttp 的缓存机制是按照 Http 的缓存机制实现。
- OkHttp 具体的数据缓存逻辑封装在 Cache 类中，它利用 DiskLruCache 实现。
- 默认情况下，OkHttp 不进行缓存数据。
- 可以在构造 OkHttpClient 时设置 Cache 对象，在其构造函数中指定缓存目录和缓存大小。
- 如果对 OkHttp 内置的 Cache 类不满意，可以自行实现 InternalCache 接口，在构造 OkHttpClient 时进行设置，这样就可以使用自定义的缓存策略了。

## OKHTTP连接复用

摘自：https://www.jianshu.com/p/6166d28983a2

概述：<font color="#dd0000">**OKHTTP的连接复用有点类似于Http的keep-alive机制，在timeout的空闲时间(okhttp的默认时间为5分钟)内，连接不会关闭，相同重复的request将复用原先的connection，减少握手的次数，以提高效率**</font>。

**RealConnection是Connection的实现类，代表着链接socket的链路**，如果拥有了一个RealConnection就代表了我们已经跟服务器有了一条通信链路。

**allocations是关联StreamAllocation，它用来统计在一个连接上建立了哪些流**，通过StreamAllocation的acquire方法和release方法可以将一个allcation对方添加到链表或者移除链表。

**连接的复用**

```java
  //M->OKHTTP，从连接池中返回Address对应的连接，如果没有这样的连接，则返回null
  @Nullable RealConnection get(Address address, StreamAllocation streamAllocation, Route route) {
    //断言，判断线程是不是被自己锁住了
    assert (Thread.holdsLock(this));
    //M->OKHTTP,遍历已有连接集合
    for (RealConnection connection : connections) {
      //M->OKHTTP，关键代码！！如果connection和需求中的"地址"和"路由"匹配
      if (connection.isEligible(address, route)) {
        //M->OKHTTP，关键代码！！复用这个连接
        streamAllocation.acquire(connection, true);
        //返回这个连接
        return connection;
      }
    }
    return null;
  }
```

遍历connections缓存列表，<font color="#dd0000">**当某个连接计数的次数小于限制的大小并且request的地址和缓存列表中此连接的地址完全匹配**</font>。则直接复用缓存列表中的connection作为request的连接。

判断给定的Address和Route是否可以复用

```java
  //M->OKHTTP,判断面对给出的addres和route，这个realConnetion是否可以重用。判断依据：
  //  如果连接达到共享上限，则不能重用
  //  非host域必须完全一样，如果不一样不能重用
  //  如果此时host域也相同，则符合条件，可以被复用
  //  如果host不相同，在HTTP/2的域名切片场景下一样可以复用
  public boolean isEligible(Address address, @Nullable Route route) {
    // If this connection is not accepting new streams, we're done.
    //M->OKHTTP 判断某个连接的计数是否小于限制
    if (allocations.size() >= allocationLimit || noNewStreams) return false;

    // If the non-host fields of the address don't overlap, we're done.
    if (!Internal.instance.equalsNonHost(this.route.address(), address)) return false;

    // If the host exactly matches, we're done: this connection can carry the address.
    if (address.url().host().equals(this.route().address().url().host())) {
      return true; // This connection is a perfect match.
    }

    // At this point we don't have a hostname match. But we still be able to carry the request if
    // our connection coalescing requirements are met. See also:
    // https://hpbn.co/optimizing-application-delivery/#eliminate-domain-sharding
    // https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/

    // 1. This connection must be HTTP/2.
    if (http2Connection == null) return false;

    // 2. The routes must share an IP address. This requires us to have a DNS address for both
    // hosts, which only happens after route planning. We can't coalesce connections that use a
    // proxy, since proxies don't tell us the origin server's IP address.
    if (route == null) return false;
    if (route.proxy().type() != Proxy.Type.DIRECT) return false;
    if (this.route.proxy().type() != Proxy.Type.DIRECT) return false;
    if (!this.route.socketAddress().equals(route.socketAddress())) return false;

    // 3. This connection's server certificate's must cover the new host.
    if (route.address().hostnameVerifier() != OkHostnameVerifier.INSTANCE) return false;
    if (!supportsUrl(address.url())) return false;

    // 4. Certificate pinning must match the host.
    try {
      address.certificatePinner().check(address.url().host(), handshake().peerCertificates());
    } catch (SSLPeerUnverifiedException e) {
      return false;
    }

    return true; // The caller's address can be carried by this connection.
  }
```

**连接池的清理**

**一个连接池最多对应一个清理线程，清理线程不断的调用cleanup(long now)方法，清理连接上失效的Stream，记录空闲最久的连接，然后将该连接从连接池中移除**。

## EventBus源码

一句话概括：运行时动态解析register类中被@Subscribe注册的方法，并收集到集合，每次postEvent会触发一次遍历。根据不同的线程模式，分别反射调用方法。

**如果没有在Activity的onDestory中反注册unregister，可能会发生内存泄漏**，虽然EventBus的invokeSubscriber方法做了捕获：

```java
void invokeSubscriber(Subscription subscription, Object event) {
        try {
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
        } catch (InvocationTargetException e) {
            handleSubscriberException(subscription, event, e.getCause());
        } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
        }
    }
```

详情查看流程图

## EventBus优点与缺点

**优点**

- 简单统一数据传递
- 清晰明了的主次线程
- 在activity与activity，或者Service与activity传递大数据时的唯一选择。因为序列化大数据进行传递时，是十分耗时缓慢的。用EventBus是最优解法。

**缺点**

- 容易滥用，你需要定义大量的常量或者新的实体类来区分接收者。管理EventBus的消息类别将会你的痛苦
- EventBus，并不是真正的解耦。请不要在你独立的模块里使用EventBus来分发。你这个模块如果那天要直接放入另外一个项目里，你怎么解耦EventBus？





## TCP/IP五层模型

![img](http://47.110.40.63:8080/img/blog/TCPIP五层模型.png)

## Android类加载机制

Android中的ClassLoader和Java中的ClassLoader有些类似，也分为系统类加载器和自定义类加载器，其中系统类加载器分为：BootClassLoader、PathClassLoader、DexClassLoader。

**BootClassLoader**：是ClassLoader的**内部类**，并继承自ClassLoader。<font color="#dd0000">**BootClassLoader是一个单例类，需要注意的是BootClassLoader的访问修饰符是默认的，只有在同一个包中才能访问，因此我们在应用程序中是无法直接调用的。**</font>

**DexClassLoader**：可以加载dex文件以及包含dex的压缩文件(apk和jar文件)，<font color="#dd0000">**DexClassLoader继承自BaseDexClassLoader，方法都在BaseDexClassLoader中实现。**</font>

```java
public class DexClassLoader extends BaseDexClassLoader {
  public DexClassLoader(String dexPath,String optimizedDirectory,String 			  		    librarySearchPath,ClassLoader parent) {
    super(dexPath,new File(optimizedDirectory),librarySearchPath,parent);
  }
}
```

<font color="#dd0000">**DexClassLoader构造函数包含4个参数**</font>

+ dexPath：dex相关文件路径集合，多个路径用文件分隔符分隔默认文件分隔符为":"；
+ optimizedDirectory：解压的dex文件的存储路径，这个路径必须是一个内部存储路径，在一般情况下，使用当前应用程序的私有路径：/data/data/<Package Name>/...
+ librarySearchPath：包含C/C++库的路径集合，多个路径用文件分隔符分隔，可以为null
+ Parent：父加载器

**PathClassLoader**：Android系统使用PathClassLoader来加载**系统类和应用程序的类**。和DexClassLoader一样继承自BaseDexClassLoader，也在BaseDexClassLoader中实现。

在PathClassLoader的构造方法中没有optimizedDirectory，**这是因为PathClasLoader的optimizedDirectory已经被指定为/data/data/dalvik-cache**，该目录用于存放已经加载过得dex，<font color="#dd0000">**因此PathClassLoader通常用来加载已经安装的apk的dex文件**</font>。

![img](http://47.110.40.63:8080/img/blog/Android 8.0ClassLoader继承关系.png)

+ ClassLoader是一个抽象类，其中定义了ClassLoader的主要功能。**BootClassLoader是它的内部类**。
+ SecureClassLoader类和JDK 8中的SecureClassLoader的代码是一样的，它继承了抽象类ClassLoader。SecureClassLoader并不是ClassLoader的实现类，而是<font color="#dd0000">**拓展了ClassLoader类加入了权限方面的功能，加强了ClassLoader的安全性**</font>。
+ URLClassLoader类和JDK 8中的URLClassLoader是一样的，<font color="#dd0000">**它继承自SecureClassLoader，用来通过URL路径从jar文件和文件夹中加载类和资源**</font>。
+ InMemoryDexClassLoader是Android 8.0新增的类加载器，继承自BaseDexClassLoader，<font color="#dd0000">**用来加载内存中的dex文件**</font>。
+ BaseDexClassLoader继承自ClassLoader，是抽象类ClassLoader的具体实现类，PathClassLoader、DexClassLoader、InMemoryDexClassLoader都继承自它。

# 组件化

![img](http://47.110.40.63:8080/img/blog/Android应用分层实例一.jpg)

![img](http://47.110.40.63:8080/img/blog/Android应用分层实例二.png)

## ARouter原理剖析及手动实现

摘自：https://www.jianshu.com/p/857aea5b54a8



# 集合类



## HashMap===



# RxJava===



## RxJava2基础使用===



## RxJava2常用操作符===



## RxJava2实现原理===

## 内存泄漏问题

摘自：https://blog.csdn.net/alex01550/article/details/86496045

方案一

1. 将disposal添加到CompositeDisposable中
2. 在Activity的onDestory中clear调

方案二

1. 引入RxLifecycler，缺点是Activity、Fragment需要继承RxActivity、RxFragment。

2. 调用compose()

   ```kotlin
   Observable.interval(1, TimeUnit.SECONDS)
   .subscribeOn(Schedulers.io())
   .observeOn(AndroidSchedulers.mainThread())
   .compose(this.bindUntilEvent(ActivityEvent.DESTROY))
   .subscribe {
     tv_count.text = it.toString()
     loge(TAG, "count:$it")
   }
   ```

方案三

1. 引入autoDisposable库

   ```java
   implementation "com.uber.autodispose:autodispose-android-archcomponents-ktx:$autodispose_version"
   implementation "com.uber.autodispose:autodispose-lifecycle-ktx:$autodispose_version"
   ```

2. 调用autoDisposable方法

   ```java
    Observable.interval(1, TimeUnit.SECONDS)
      .subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      .autoDisposable(AndroidLifecycleScopeProvider
                      .from(this,Lifecycle.Event.ON_DESTROY))
      .subscribe {
      	tv_count.text = it.toString()
        loge(TAG, "count:$it")
     }
   ```

# 性能优化

## 包体积优化

摘自：https://juejin.im/post/5e7ad1c0e51d450edc0cf053#heading-83

![img](http://47.110.40.63:8080/img/blog/包体积优化思维导图.png)

## APK压缩

1. 转换图片格式，使用jpg，或者webp。WebP比PNG小45%。

2. 去除多语言

3. 删除不必要的so库，微信也只保留了armeabi-v7a

4. 使用Lint进行无用资源检查（谨慎删除）

5. 开启混淆

6. 删除无用资源ShinkResource

7. 使用**微信资源压缩**方案AndResGuard
   **AndResGuard通过修改resources.arsc文件，从而可以混淆安卓的资源文件路径**（比如res/drawable/activity_advanced_setting_for_test=>r/d/a），达到减少apk包的体积的目的。**底层原理
   andresGuard在原生的buildApk步骤之后，使用产生的apk作为输入文件，对其进行混淆压缩，产出一个新的apk**。

8. SO文件动态下发

   需要注意几个地方

   1. 安全性问题，md5校验，或者结合APK安装包签名校验的方式进行校验
   2. SO文件支持升降级配置，强制升级，或者回退到旧版本。本地需要保留旧版本文件。

9. Dex分包优化

   把相互依赖的代码分到同一个dex文件中，也可以优化一点空间。

10. 插件化方案

    插件化的前提是组件化

11. D8和R8工具的使用

    D8是优化的dex打包工具，R8是优化后的proguard工具

## 动态下发SO库

摘自：https://mp.weixin.qq.com/s/X58fK02imnNkvUMFt23OAg

Android 加载 so 文件本身就是一种运行时动态加载可执行代码的行为，所以把 so 做成动态下发的没有什么技术风险。

动态下发 so 库，看上只是把原本就算运行时动态加载的 so 文件，从 APK 安装包里面抽离出来，工作流程上变化不大，但实际上这也是一种完备的插件化技术，也就是说所有插件需要面临问题的问题我们统统需要考虑。

**安全性问题**

动态化本质上就是运行时加载可执行代码，而所有可执行代码在拷贝安装到安全路径（比如 Android 的 data/data 内部路径）之前，都有被劫持或者破坏的风险。**最好的做法是每次加载 so 库之前都对其做一次安全性校验**。考虑到检查带来的时间成本，可以假设内部路径是无条件可信的；而且除了劫持风险外，内部路径文件有可能被应用自身一些不当文件操作给破坏导致插件不完整，因此如果要考虑绝对安全，内部路径插件被加载也必须做安全检查），在 so 文件拷贝到内部路径后单独做一次检查，检查失败就丢弃文件走 fail 逻辑，检查通过就生成一个 flag 文件作为标志，以后通过判断 flag 标志是否存在来决定是否需要执行安全检查。

**怎么校验安全性呢？**

**最简单的方式是记录 so 文件的 MD5 或者 CRC 等 Hash 信息，不过 Hash 信息一般都会随之 so 文件的变动而改变，每次都需要调整这些数据比较麻烦**。优化方案是“通过类似 APK 安装包签名校验的方式来确保安全性”：<font color="#dd0000">**将 so 文件打包成 APK 格式的插件包并使用 Android Keystore 进行签名，将 Keystore 的指纹信息保存在宿主包内部，安全检验环节只需要校验插件包的签名信息是否和内置的指纹信息一致即可**</font>。（一种优化的方案是，使用和宿主包一样的 Keystore 给插件包签名，检验环节只需要检查插件和宿主的签名信息是否一致。）

**版本控制问题**

从 APK 里把 so 剥离出来后，我们除了要保证 so 文件的安全性，还要保证 so 文件和依赖它的宿主代码是 API 兼容的。<font color="#dd0000">**如果需要做到动态升降级，还需要保留最近一两个版本的存量 so 文件，用于 fallback 逻辑需要**</font>。

通过版本控制流程，我们可以在服务端禁用这个版本的 so 插件，从而使客户端进入“so 插件不可用”的逻辑，而不至于执行有问题的代码。如果 so 插件支持动态升降级，还可以配置让客户端强制更新到 fix 插件版本，或者 fallback 回没有问题的存量旧版。

## Monkey测试

**环境安装**

摘自：https://www.jianshu.com/p/c86021fe958d

**Full power：**  能量最高的状态，移动网络连接被激活，允许设备以最大的传输速率进行操作。

**Low power：**  一种中间状态，对电量的消耗差不多是 Full power 状态下的 50%。

**Standby：** 最低的状态，没有数据连接需要传输，电量消耗最少。

![img](http://47.110.40.63:8080/img/blog/Android电量优化.png)

## 启动提速之初始化任务分级

摘自：http://www.imooc.com/article/40398

**原因**

应用启动过程从用户点击launcher图标到看到第一帧这个过程中，主要会经过以下这些过程：

> main()->Application:attachBaseContext()->onCreate()->Activity:onCreate()->onStart()->onPostCreate()->onResume()->onPostResume()

而一般我们的初始化任务主要都会集中化在Application:onCreate()方法中，这就使得初始化任务在第一帧绘制之前得完成，这就造成了卡图标、应用启动慢。

Activity的创建都会辗转到ActivityThread:performLaunchActivity()这个方法中，在这个方法中可以知道这么几件事：

1、先通过Instrumentation:newActivity()来创建一个Activity实例
2、再判断Application实例是否已创建，已创建则直接返回，否则调用
Instrumentation:newApplication()来创建Application实例，在这个过程中会依次执行attachBaseContext()和onCreate()方法
3、之后Activity:attach()方法会创建一个PhoneWindow对象，它就是界面，它有一个DecorView，调用setContentView()时会给配置DecorView，其中就会设置一个背景：

![img](https://img2.sycdn.imooc.com/5b3899ef00018dba13160278.jpg)

我们的View也是add进DecorView中显示，它作为RootView肯定是最先显示，所以可以给它设置个默认背景

4、最后依次调用Activity的onCreate、onStart等方法

**措施**

1、任务分级
2、任务并行
3、界面预显示

![img](http://47.110.40.63:8080/img/blog/启动提速_初始化任务分级.jpg)

**分级带来的问题**

正常启动过程那肯定是没问题的，不过有这么几种场景：

> 1、App切回后台，内存不足导致Application被回收，从最近任务列表中恢复界面时Application需重新创建
> 2、应用没挂起时，Push推送需从Notification跳入应用内某界面
> 3、应用没挂起时，浏览器外链需跳入应用内某界面

**这些Case可能导致的问题是被跳入的界面使用到了未初始化的SDK，可能导致Crash或者数据异常，所以目标页面启动前必须确保SDK已经初始化，这个过程的原因是没有唤起启动页来初始化SDK，可以通过hook newActivity解决**。

```
public Activity newActivity(ClassLoader cl, String className, Intent intent) throws InstantiationException, IllegalAccessException, ClassNotFoundException {    if (InitializeOptimizer.isApplicationCreated()
            && (InitializeUtil.isOuterChainIntent(intent) || 
InitializeUtil.isNotificationIntent(intent)) && (!InitializeOptimizer.isHighSDKInitialized()
            || !InitializeOptimizer.isLowSDKInitialized()
            || !InitializeOptimizer.isAsyncSDKInitialized())) {
        InitializeOptimizer.setApplicationCreated(false);
        intent.addCategory(InitializeUtil.INITIALIZE_CATEGORY);        return (Activity) cl.loadClass(InitializeOptimizer.getLaunchClassName()).newInstance();
    }
    InitializeOptimizer.setApplicationCreated(false);    return super.newActivity(cl, className, intent);
}
```

# Flutter

## 异步编程

**isolate**

摘自：https://blog.csdn.net/email_jade/article/details/88941434

在dart中，这里不是称呼线程，是Isolate，直译叫做隔离，这么古怪的名字，是因为隔离不共享数据，每个隔离中的变量都是不同的，不能相互共享。

但是由于dart中的Isolate比较重量级，UI线程和Isolate中的数据的传输比较复杂，因此flutter为了简化用户代码，在foundation库中封装了一个轻量级compute操作，我们先看看compute，然后再来看Isolate。

要使用compute，必须注意的有两点，一是我们的compute中运行的函数，必须是顶级函数或者是static函数，二是compute传参，只能传递一个参数，返回值也只有一个。

但是，compute的使用还是有些限制，它没有办法多次返回结果，也没有办法持续性的传值计算，每次调用，相当于新建一个隔离，如果调用过多的话反而会适得其反。在某些业务下，我们可以使用compute，但是在另外一些业务下，我们只能使用dart提供的Isolate了

**event loop**

摘自：https://zhuanlan.zhihu.com/p/103398523

Dart VM启动后，那么一个新的Thread就会被创建，并且只会有一个线程，它运行在自己的Isolate中。

当这个Thread被创建后，DartVM会自动做以下3件事情：

- 初始化2个队列，一个叫“MicroTask”，一个叫“Event”，都是FIFO队列
- 执行 main() 方法，一旦执行完毕就做下一步
- 启动 Event Loop

Event Loop就像一个 infinite loop，被内部时钟来调谐，每一个tick，如果没有其他Dart Code在执行，就会做如下的事情（伪代码）：

## cc

1. 熟练Android基础，如四大组件、Context相关、Touch事件分发、Handler消息分发、View的绘制、界面的刷新机制流程等原理。
2. 掌握常见算法，如排序，动态规划，递归，二分查找等。掌握常用数据容器，如：HashMap、ArrayList、ConcurrentHashMap、HashSet等。
3. 掌握OkHttp、Retrofit、ButterKnife、EentBus、Dagger、BlockCanary、LeakCanary等主流框架的使用，并阅读过其中多数的源码。
4. 能够熟练使用MVC、MVP、MVVM等架构模式进行项目开发。
5. 掌握常用设计模式，如单例模式、适配器模式、责任链、观察者模式等设计模式。
6. 熟练掌握动态代理模式及Java反射机制；了解Hook技术；了解注解、注解处理器等原理。
7. 有热修复实战经验，了解热修复、插件化实现原理。
8. 熟练掌握组件化架构实现。
9. 了解Android系统源码，如系统启动流程、Activity启动流程、Binder机制、AIDL、AMS、PMS等原理。
10. 对Java类加载、运行时、GC、内存模型有一定了解；





## Platfrom Channel

摘自：https://www.jianshu.com/p/cb96d62f5042

Flutter提供了三种platform和dart端通信的方式：BasicMessageChannel，MethodChannle，EventChannel，通信过程都是全双工的。

# NDK相关

## CMake必知必会

摘自：https://mp.weixin.qq.com/s/1NJZ17rmgFeQ6c3TPLIN_g

https://blog.csdn.net/sjt19910311/article/details/51660209

CMake 是一个跨平台构建系统，在 Android Studio 引入 CMake 之前，它就已经被广泛运用了。

总结官网对 CMake 的使用，其实也就如下的步骤：

1. add_library 指定要编译的库，并将**所有**的 .c 或 .cpp 文件包含指定。
2. include_directories 将**头文件**添加到搜索路径中
3. set_target_properties 设置库的一些属性
4. target_link_libraries **将库与其他库相关联**

在 cpp 的同一目录下创建 CMakeLists.txt 文件，内容如下：

```
# 指定 CMake 使用版本
cmake_minimum_required(VERSION 3.9)
# 工程名
project(HelloCMake)
# 编译可执行文件
add_executable(HelloCMake main.cpp )
```

其中，通过 cmake_minimum_required 方法指定 CMake 使用版本，通过 project 指定工程名。

而 add_executable 就是指定**最后编译的可执行文件名称和需要编译的 cpp 文件**，如果工程很大，有多个 cpp 文件，那么都要把它们添加进来(**使用空格分隔**)。

**add_library**

````shell
add_library(<name> [STATIC | SHARED | MODULE]
[EXCLUDE_FROM_ALL]
source1 source2 … sourceN)
````

添加一个名为< name >的库文件，该库文件将会根据调用的命令里列出的源文件来创建。< name >对应于逻辑目标名称，而且在一个工程的全局域内必须是唯一的。待构建的库文件的实际文件名根据对应平台的命名约定来构造（比如lib< name >.a或者< name >.lib）。STATIC库是目标文件的归档文件，在链接其它目标的时候使用。SHARED库会被动态链接，在运行时被加载。MODULE库是不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数动态链接。如果没有类型被显式指定，这个选项将会根据变量BUILD_SHARED_LIBS的当前值是否为真决定是STATIC还是SHARED。

###Android JNI 函数注册的两种方式(静态注册/动态注册)

摘自：https://www.jianshu.com/p/1d6ec5068d05

静态注册
 优点: 理解和使用方式简单, 属于傻瓜式操作, 使用相关工具按流程操作就行, 出错率低
 缺点: 当需要更改类名,包名或者方法时, 需要按照之前方法重新生成头文件, 灵活性不高

动态注册
 优点: 灵活性高, 更改类名,包名或方法时, 只需对更改模块进行少量修改, 效率高
 缺点: 对新手来说稍微有点难理解, 同时会由于搞错签名, 方法, 导致注册失败


