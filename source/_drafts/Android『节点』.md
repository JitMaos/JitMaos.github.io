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

一、开机加电
BootLoader进行底层初始化，并加载内核代码，最终调转到内核的boot程序。
二、Linux内核引导
1）kernel核心初始化（内存初始化，打开中断，初始化进程表等等）
2）驱动初始化
3）启动内核后台线程
4）安装根文件系统
5）启动第一个用户级进程init
三、init进程启动
init进程的程序在system/core/init/init.c里，它是Android系统特有的初始化程序，最终它会以后台进程（daemon）的形式一直存在。该进程主要有以下功能：
1）创建/安装设备文件/进程文件/系统文件节点。
2）解析启动/init.rc和/init.<machine_name>.rc。
3）显示Logo画面。
4）打开Device Socket，Property Socket，child进程通信Socket。
5）执行脚本中指定的命令或动作，启动指定服务，监听特定事件。
6）进入死循环：检查是否有action需要执行，是否需要restart某服务
四、Native服务启动
1）Service Manager：Binder服务管理器，管理所有Android系统服务。
2）Zygote：启动Android Dalvik Runtime并负责孵化进程。
五、Android Runtime启动
zygote创建并启动Android Runtime（Dalvik属于Runtime的一部分），然后启动System Server进程进行系统初始化。
六、Android系统初始化
System Server作为Zygote的第一个子进程，是Android Framework的核心，它主要负责Android系统初始化并启动其它服务。其它的Android服务都由System Server启动并允许在该进程空间。
七、Home启动

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







