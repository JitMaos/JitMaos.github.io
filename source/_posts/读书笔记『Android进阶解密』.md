---
title: Android『进阶解密』
tags:
---

第二章 Android系统启动

### 2.1 init进程启动过程

**P13**：init进程是Android系统中**用户空间**的第一个进程，进程号为1，是Android系统启动流程中一个关键的步骤，作为第一个进程，它被赋予了很多极其重要的工作职责，比如<font color="#dd0000">**创建Zygote进程和属性服务等**</font>。init进程是由多个源文件共同组成的，这些文件位于源码目录system/core/init/中。

**P14**：Android系统启动的步骤

1. 启动电源以及系统启动

   当电源按下时引导芯片代码从预定义的地方（固化在ROM）开始执行。**加载引导程序BootLoader到RAM中，然后执行**。

2. 引导程序BootLoader

   引导程序BootLoader是在Android**操作系统开始运行前**的一个小程序，它的主要作用是把系统OS拉起来并运行。

3. Linux内核启动

   当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。**在内核完成系统设置后，它首先在系统文件中寻找init.rc文件，并启动init进程**。

4. init进程启动

   <font color="#dd0000">**init进程做的工作比较多，主要用来初始化和启动属性服务（后文P23提到类似Window的注册表，存储的是key-value形式的数据），也用来启动Zygote进程。**</font>

###2.1.6属性服务

P23：Window平台上有一个注册表管理器，其内容采用键值对的形式记录一些用户、软件的使用信息。即使系统重启，还能根据之前注册表中的记录来进行相应的初始化工作。**Android也提供了类似的机制，叫做属性服务**。

**P24**：start_property_service()

```java
void start_property_service() {
  property_set("ro.property_service.version","2");
  property_set_fd = create_socket(PROP_SERVICE_NAME,SOCK_STREAM | SOCK_CLOEXEC | SOCK_NONBLOCK,0666,0,0,NULL); //1
  if(property_set_fd == -1) {
    PLOG(ERROR) << "start_property_service socket creation failed";
    exit(1);
  }
  listen(property_set_fd,8); //2
  register_epoll_handler(property_set_fd,handle_property_set_fd); //3
}
```

在注释1处创建**非阻塞的Socket**。在注释2处调用listen函数对property_set_fd进行监听，这样创建的Socket就成为Server，也就是属性服务；listen函数的第二个参数设置为8，意味着属性服务最多可以同时为8个<font color="#dd0000">**视图设置属性的用户**</font>提供服务。

### 2.2.1 Zygote概述

<font color="#dd0000">**在Android系统中，Dalvik和ART、应用程序进程以及运行系统的关键服务的SystemServer进程都是有Zygote进程创建的**</font>，Zygote进程通过fork(复制进程）的形式来创建**应用程序进程**和**SystemServer进程**，由于Zygote进程在启动时会创建Dalvik和ART，因此通过fork创建的应用程序进程和SystemServer进程可以在内部获取一个Dalvik或ART的虚拟机实例副本。

<!--more-->

### 2.2.3 Zygote进程启动过程介绍

**P34**：ZygoteInit的main方法主要做了4件事：

1. 创建一个Server端的Socket(个人理解：因为此时连SystemServer进程都还没创建，没有Binder，所以进程间通信先通过Socket)
2. 预加载类和资源
3. 启动SystemServer进程
4. 等待AMS请求创建新的应用程序进程（P35->还是通过Socket来实现的）

**P35**：在Zygote进程将SystemServer进程启动后，就会在这个服务器端的Socket上等待AMS请求Zygote进程来创建新的应用程序进程。







### 4.1 根Activity的启动过程

根Activity的启动过程主要分为3步，分别是Launcher请求AMS、AMS到ApplicationThread的调用过程、ActivityThread启动Activity。

### 4.1.1 Launcher 请求AMS过程

![img](http://47.110.40.63:8080/img/blog/启动流程_Launcher请求AMS.png)

**扩展**

1. 为什么从Application/Service的Context启动Activity，需要指定Intent.FLAG_ACTIVITY_NEW_TASK？

   **因为非Activity的Context没有所谓的任务栈(这里可以在扩展出Context的继承关系）**，而Activity的存在需要任务栈来承载。所以指定NEW_TASK的Flag用于为Activity指定一个新的任务栈。

2. ApplicationThread虽然看上去是个Thread，但其实它是ActivityThread的内部类，<font color="#dd0000">**继承自IApplicationThread.Stub**</font>，其中包含了很多scheduleXxx方法用于操作Service和BroadcastReceiver，具体可以参考这篇博文：[Android主线程(ActivityThread)源代码分析](https://blog.csdn.net/xu_song/article/details/81983724)

3. Instrumentation可以看做是ActivityThread的管家

**P85**：Android8.0之前并没有采用AIDL，而是采用了类似AIDL的形式，用AMS的代理对象ActivityManagerProxy来与AMS进行进程间通信，Android8.0使用IActivityManager替代了ActivityManagerProxy，**它是AMS在本地的代理。**execStartActivity方法最终调用的是AMS的startActivity方法。

```java
private static final Singleton<IActivityManager> IActivityManagerSingleton = new 
  Singleton<IActivityManager>() {
  @override 
  protected IActivityManager create() {
    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE); //1
    final IActivityManager am = IActivityManager.Stub.asInterface(b); //2
    return am;
  }
}
```

###4.12 AMS到ApplicationThread的调用过程

![img](http://47.110.40.63:8080/img/blog/AMS到ApplicationThread过程时序图.jpg)

**P93**

```java
final boolean realStartActivityLocked(ActivityRecord r,ProcessRecord app,boolean andResume,boolean checkConfi) throws RemoteException {
  ...
    app.thread.scheduleLaunchActivity(new Intent(r.intent),r.appToken,System.identityHashCode(r) .....)
    ...
    return true;
}
```

这里的app指的是Process（要启动Activity的应用程序进程），**app.thread指的是IApplicationThread，它的实现是ActivityThread的内部类ApplicationThread，其中ApplicaitonThread继承了IApplicaitonThread.Stub**。通过ApplicationThreadL来与应用程序进行Binder通信，换句话说，<font color="#dd0000">**ApplicationThread是AMS所在进程（SystemServer进程）和应用程序进程的通信桥梁**</font>。

###4.1.3 ActivityThread启动Activity过程

![img](http://47.110.40.63:8080/img/blog/ActivityThread启动Activity时序图.jpg)

**P96**：因为ApplicationThread是一个Binder，它的调用逻辑运行在Binder线程池中，所以需要用H将代码的逻辑切换到主线程中。

```java
frameworks/base/core/java/android/app/AndroidThread.java
private Activity performLaunchActivity(ActivityClientRecord r,Intent customIntent) {
	...
  //创建要启动Activity的上下文环境
  ContextImpl appContext = createBaseContextForActivity(r); 
  java.lang.ClassLoader cl = appContext.getClassLoader(); 
  activity = mInstrumentation.newActivity(cl,component.getClassName(),r.intent); //1
  ...
  activity.attach(...); //2
  ...
  mInstrumentation.callActivityOnCreate(activity,r.state,r.persistentState); //3
}
```

1-> 调用Instrumentation的newActivity创建Activity，<font color="#dd0000">**有些需求可以通过hook Instrumentation来实现一些功能，就是基于newActivity创建Activity的机制。**</font>

2-> 在attach中初始化Activity，<font color="#dd0000">**在attach方法中会创建Window对象，并与Activity自身进行关联。**</font>

扩展：

摘自：https://blog.csdn.net/huaxun66/article/details/78151361

<font color="#dd0000">**总体流程**</font>
1.Launcher通过Binder机制通知AMS启动一个Activity.
2.AMS使Launcher栈最顶端Activity进入onPause状态.
3.AMS通知Process使用Socket和Zygote进程通信，请求创建一个新进程.
4.Zygote收到Socket请求，fork出一个进程，并调用ActivityThread#main().
5.ActivityThread通过Binder通知SystemServer进程中的AMS启动应用程序.
6.AMS通知ActivityStackSupervisor真正的启动Activity.
7.ActivityStackSupervisor通知ApplicationThread启动Activity.
8.ApplicationThread发消息给ActivityThread，需要启动一个Activity.
9.ActivityThread收到消息之后，通知LoadedApk创建Applicaition，并且调用其onCteate()方法.
10.ActivityThread装载目标Activity类，并调用Activity#attach().
11.ActivityThread通知Instrumentation调用Activity#onCreate().
12.Instrumentation调用Activity#performCreate()，在Activity#performCreate()中调用自身onCreate()方法.

### 4.2Service的启动过程



图片来源：

https://blog.csdn.net/huaxun66/article/details/78151361

## 第七章 理解WindowManager

### 7.1Window、WindowManager和WMS

**P187**：Window是一个抽象类，实现类为PhoneWindow，它对View进行管理。WindowManager是一个接口类，继承自接口ViewManager，从名字可以知道它是用来管理Window的，它的实现类为WindowManagerImpl。WindowManager和AMS通过Binder来进行跨进程通信。

###7.2 WindowManager的关联类

**P189**：PhoneWindow的创建时机在

```java
//frameworks/base/core/java/android/app/Activity.java
final void attach(Context context,ActivityThread aThread,...) {
  ...
  mWindow = new PhoneWindow(this,window,activityConfigCallback);
}
```

**P191**：WindowManagerImpl虽然是WindowManager的实现类，但是它把功能的实现委托给了WindowManagerGlobal。

```java
@Override
public void addView(View view,ViewGroup.LayoutParams params) {
  mGlobal.addView(view,params,mContext.getDisplay(),mParentWindow);
}
```

**P192**：WindowManagerGlobal是一个<font color="#dd0000">**单例**</font>，一个进程中只有一个WIndowManagerGlobal实例。

PhoneWindow继承自Window，Window通过setWindowManager与WindowManager进行关联。WindowManager继承ViewManager接口，WindowManagerImpl是WindowManager的实现类，但是具体的功能由WindowManagerGlobal来实现。

![img](http://47.110.40.63:8080/img/blog/WindowManager类图.png)

图片来源：https://www.jianshu.com/p/1c4059d3865b?utm_campaign=maleskine

### 7.3.1 Window的类型和显示次序

**P194**：应用程序窗口的Type值范围为1~99；子窗口的Type值范围为1000~1999；系统窗口为2000~2999，数值越大离用户越近(屏幕Z轴)。 

### 7.4.1 系统窗口的添加过程

**P196**：

1.调用WindowManagerImpl的addView

```java
//frameworks/base/core/java/android/WindowManagerImpl.java
@Override
public void addView(View view,ViewGroup.LayoutParams params) {
  mGlobal.addView(view,params,mContext.getDisplay(),mParentWindow);
}
```

2.WindowManagerGlobal的addView

在窗体添加、更新、删除的过程中涉及3个列表，**View列表(ArrayList<View> mViews)、布局参数列表(ArrayList<WindowManager.LayoutParams> mParams)和ViewRootImpl列表（ArrayList<ViewRootImpl> mRoots)**。

```java
//frameworks/base/core/java/android/view/WindowManagerGlobal.java
public void addView(View view,ViewGroup.LayoutParams params,Display display,Window parentWindow) {
  ...
  root = new ViewRootImpl(view.getContext,display);
  mViews.add(view); //添加到view列表
  mRoots.add(root); //添加到ViewRootImpl列表
  mParams.add(wparams); //添加到布局参数列表
  root.setView(View,wparams,panelParentView);
}
```

ViewRootImpl有很多功能，主要有以下几点

1. View树的根并管理View树
2. **触发View的测量、布局和绘制**
3. 输入事件的中转站
4. 管理Surface
5. <font color="#dd0000">**负责与WMS进行进程间通信**</font>

```java
//frameworks/base/core/java/android/view/ViewRootImpl.java
public void setView(View view,WindowManager.LayoutParams attrs,View panelParentView) {
  ...
  res = mWindowSession.addToDisplay(mWindow,mSeq,...);
  ...
}
```

**mWindowSession是IWindowSession类型的，它是一个Binder对象，用于进程间通信，IWindowSession是Client端的代理，它的Server端为Session，此前的代码逻辑都是运行在本地进程的，而Session的addToDisplay方法则是运行在WMS所在的进程（SystemServer进程）中。**

```java
//frameworks/base/services/core/java/com/android/server/wm/Session.java
@Override
public int addToDisplay(IWindow window,int seq,WindowManager.LayoutParams attrs,...) {
  return mService.addWindow(this,window,seq,attrs...);
}
```

<font color="#dd0000">**在addToDisplay方法中调用了WMS的addWindow方法，并将自身也就是Session作为参数传进去，每个应用程序进程都会对应一个Session，WMS会用ArrayList来保存这些Session。**</font>剩下的工作由WMS完成，在WMS中会为这个添加的窗口分配surface，并确定窗口的显示次序，可见负责显示界面的画布Surface，而不是窗口本身。<font color="#dd0000">**WMS会将它管理的Surface交由SurfaceFlinger处理，SurfaceFlinger会将这些Surface混合并绘制在屏幕上**</font>。

### 7.4.3 Window的更新过程

**P203**：

1.Window的更新过程调用ViewManager接口中的updateViewLayout，在WindowManagerGlobal的实现

```java
//frameworks/base/core/java/android/view/WindowManagerGlobal.java
public void updateViewLayout(View view,ViewGroup.LayoutParams params) {
  ...
  root.setLayoutParams(wparams,false);
  ...
}
```

2.跟进去，ViewRootImpl的setLayoutParams最后会滴啊用ViewRootImpl的scheduleTraversals

```java
void setLayoutParams(WindowManager.LayoutParams attrs, boolean newView) {
  ...
    scheduleTraversals();
}
```

3.<font color="#dd0000">**scheduleTraversals通过Choreographer设置了下一帧的回调mTraversalRunnable**</font>

```java
void scheduleTraversals() {
  ...
  mChoreographer.postCallback(
    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
  ...
}
```

4.TraversalRunnable

```java
final class TraversalRunnable implements Runnable {
  @Override
  public void run() {
    doTraversal();
  }
}
```

5.doTraversal

```java
void doTraversal() {
  if (mTraversalScheduled) {
    ...
    performTraversals();
		...
  }
}
```

6.最后是我们熟悉的performTraversals，在里面执行measure/layout/darw过程

```java
private void performTraversals() {
  ...
  relayoutResult = relayoutWindow(params, viewVisibility, insetsPending);
  ...
  if (!mStopped || mReportNextDraw) {
    ...
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
  } 
  if (didLayout) {
    performLayout(lp, mWidth, mHeight);
  }
  if (!cancelDraw && !newSurface) {
    ...
    performDraw();
  } 
}
```

## 第八章 理解WindowManagerService

**P215**：Watchdog用来监控系统的一些关键服务的运行状况（比如传入的WMS的运行状况），这些被监控的服务都会实现Watchdog.Monitor接口。<font color="#dd0000">**Watchdog每分钟都会对被监控的系统服务进行检查，如果被监控的系统服务出现死锁，则会杀死Watchdog所在的进程，也就是SystemServer进程**</font>。

**P224**：addWindow方法总结

+ 对所要添加到窗口进行检查，如果窗口不满足一些条件，就不会执行下面段代码逻辑。
+ WindowToken相关的处理，比如有的窗口类型需要提供WindowToken，没有提供的话就不会执行下面的代码逻辑，有的窗口类型则需要由WMS隐式创建WindowToken。
+ WindowState的创建和相关处理，将WindowToken和WindowState相关联。
+ 创建和配置DisplayContent，完成窗口添加到系统前的准备工作。

**P230**：删除Window过程总结

+ 检查删除线程的正确性，如果不正确就抛出异常。
+ **从ViewRootImple列表、布局参数列表和View列表中删除与V对应的元素。**
+ 判断是否可以直接执行删除操作，如果不能就推迟删除操作。
+ 执行删除操作，清理和释放与V相关的一切资源。

## 第九章 JNI原理



**P235**：Native方法注册分为静态注册和动态注册，其中静态注册多用于NDK开发，而动态注册多用于Framework开发。

**扩展**：[Android JNI 函数注册的两种方式](https://www.jianshu.com/p/1d6ec5068d05)

**P242**：JNI的方法签名格式为

​	(参数签名格式...) 返回值签名格式

```java
private native final void native_setup(Object meidarecorder_this,String fileName,String opPackageName) =>它在JNI中方法签名为：
(Ljava/lang/Object;Ljava/lang/String;Ljava/lang/String;)V
```



**P244**：JNIEnv是Native世界中Java环境的代表，通过JNIEnv*指针就可以在Native世界中访问Java世界的代码进行操作，<font color="#dd0000">**它只在创建它的线程中有效，不能跨线程传递**</font>，因此不同线程的JNIEnv是彼此独立的。

**P249**：JNI也有引用类型，它们分别是本地引用(Local References)、全局引用(Global References)和弱全局引用(Weak Global References)

本地引用

+ 当Nativa函数返回时，这个本地引用就会被自动释放。

+ 只在创建它的线程中有效，不能跨线程使用。
+ 局部引用是JVM负责的引用类型，受JVM管理。

全局引用

+ 在native函数返回时不会被自动释放，因此全局引用需要手动来进行释放。并且不会被GC回收。
+ 全局引用是可以跨线程使用的。
+ 全局引用不受到JVM管理。

弱全局引用

+ 是一种特殊的全局引用，不同点在于弱全局引用是可以被GC回收的，弱全局引用被GC回收之后会指向NULL

## 第十章 Java虚拟机

**P256**：classfile格式

```java
//u1/u2/u4/u8分别表示1，2，4，8个字节的无符号类型。
ClassFile {
  u4 magic: //魔数，OxCAFEBABE
  u2 minor_version; //副版本
  u2 major_version; //主版本
  u2 constant_pool_count; //常量池计数器
  cp_info constant_pool[constant_pool_count-1]; //常量池
  u2 access_flags; //类和接口层次的访问标志
  u2 this_class; //类索引
  u2 super_class; //父类索引
  u2 interfaces_count; //接口计数器
  u2 interfaces[interfaces_count]; //接口表
  u2 fields_count; //字段计数器
  field_info_fields[fields_count]; //字段表
  u2 methods_count; //方法计数器
  method_info_methods[methods_count]; //方法表
  u2 attributes_count; //属性计数器
  attribute_info attributes[attributes_count]; //属性表
}
```

**P257**：类加载子系统

1. BootStrap ClassLoader(引导类加载器)

   用C/C++代码实现的加载器，用于加载指定的JDK的核心库，比如java.lang，java.util等。

2. Extensions ClassLoader(扩展类加载器)

   用于加载Java的扩展类，提供除了系统类之外的额外功能。

3. Application ClassLoader(应用程序加载器)

   这个类加载器可以通过ClassLoader的getSystemLoader获取到，也成为系统类加载器。

**P258-P260**：运行时数据区域

在[读书笔记『深入理解Java虚拟机』]([https://jitmaos.github.io/2020/04/19/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%E3%80%8E%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8F/](https://jitmaos.github.io/2020/04/19/读书笔记『深入理解Java虚拟机』/))有记录过，不再重复记录。

**P261**：对象的创建

1. 判断对象对应的类是否加载、链接和初始化

2. 为对象分配内存

3. 处理并发安全问题

4. 初始化分配到的内存空间

   出对象头外，都初始化为零。

5. 设置对象的对象头

   将对象的所属类、对象的HashCode和对象的GC分代年龄等数据存储在对象头中。

6. 执行init方法进行初始化

**P262**：对象的堆内布局主要分为三部分：对象头、实例数据、对齐填充(不一定存在，没有特殊意义)

对象头：

​	hash：对象的哈希码

​	age：<font color="#dd0000">**对象的分代年龄**</font>。

​	biased_lock：<font color="#dd0000">**偏向锁标志位**</font>。

​	lock：锁状态标志位。

​	JavaThread*：<font color="#dd0000">**持有偏向锁的线程ID**</font>。

​	epoch：**偏向时间戳**

**P266-P275**：垃圾回收算法

在[读书笔记『深入理解Java虚拟机』]([https://jitmaos.github.io/2020/04/19/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%E3%80%8E%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8F/](https://jitmaos.github.io/2020/04/19/读书笔记『深入理解Java虚拟机』/))有记录过，不再重复记录。

## 第十一章 Dalvik和ART

**P276**：<font color="#dd0000">**Dalvik并不是一个Java虚拟机**</font>，因为Dalvik没有遵循Java虚拟机规范，Dalvik是基于寄存器的，而Java虚拟机是基于栈的。**基于寄存器的Dalvik，减少了出入栈的指令，更紧凑、简洁**。

**P277**：Dex文件将所包含的class文件的信息整合在一起，去除冗余信息，更紧凑。

**P282**：Dalvik和ART的区别

1. 在ART中，系统在安装时会进行一次AOT（Ahead of time compilation，预编译)，将字节码预先编译成机器码并存储在本地，这样应用程序每次运行时就不需要进行编译了，程序的运行效率得以提高，耗电量也会减少。ART在7.0前是全编译的，但是这样导致安装时间过长，且占据更大的存储空间。<font color="#dd0000">**在7.0以后，ART加入了即时编译器JIT，在安装时不会将字节码全部编译成机器码，而是在运行中将热点代码编译成机器码（据说是在屏幕息屏，或者设备空闲的时候执行的)，存储在本地，从而取得一个平衡**</font>。
2. Dalvik是基于32位CPU设计的，而ART支持64位并兼容32位CPU，这是Dalvik被淘汰的主要原因之一。
3. ART对垃圾回收机制进行了改进，比如更频繁地执行并行垃圾回收，将GC暂停有2次减少为1次。
4. ART的运行时堆空间划分和Dalvik不同。

**P288**：Zygote进程创建了Dalvik或ART虚拟机的实例，之后fork出来的进程都同样带有Dalvik或ART虚拟机的实例。

## 第十二章 理解ClassLoader

**P295**：ClassLoader双亲委托加载机制的好处

+ 防止重复加载，已经加载过得Class，直接使用就可以了。
+ 更加安全，防止被自定义类已经加载的类。

自定义ClassLoader

1. 定义一个自定义ClassLoader类
2. 复写findClass方法，并在findClass方法中调用defineClass方法

**P299**：Android中的ClassLoader

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

DexClassLoader构造函数包含4个参数

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

**ClassLoader的加载流程**

![img](http://47.110.40.63:8080/img/blog/Android 类加载流程示例.png)

## 第十三章 热修复原理

**P313**：热修复的核心技术包括三类，分别是**代码修复**，**资源修复**，**动态链接库修复**。

### 13.3.1 Instant Run概述

![img](http://47.110.40.63:8080/img/blog/InstantRun编译部署.png)

**Hot Swap**：不需要重启程序，不需要重启Activity，如：修改一个现有方法。

**Warm Swap**：需要Activity重启，如：修改或删除一个现有的资源文件。

**Code Swap**：App需要重启，但是不需要重新安装，如：添加、删除一个字段和方法、添加一个类等。









图片来源：https://www.jianshu.com/p/fe2f739928ec

**P318**：InstantRun中资源热修复的原理

1. 创建新的AssetManager，通过**反射**调用addAssetPath方法加载外部的资源，这样新创建的AssetManager就含有了外部资源。
2. 将AssetManager类型的mAssets字段的引用全部替换为新创建的AssetManager。

###13.4 代码修复

代码修复主要有3个方案，分别是<font color="#dd0000">**底层替换方案、类加载方案和Instant Run方案**</font>。

### 13.4.1 类加载方案

**P319**：

**65536限制**：**类加载方案基于Dex分包方案**，<font color="#dd0000">**DVM指令集的方法调用指令invoke-kind索引为16bits，最多能引用65536个方法**</font>。

**LinearAlloc限制**：DVM中LinearAlloc是一个固定的缓存区，当方法数超过了缓存区的大小时会报错。

**Dex分包方案**：将程序启动必须要用到的方法的类放到**主Dex**中，其余代码放置到**次Dex**中。

```java
//libcore/dalvik/src/main/java/dalvik/system/DexPathList.java
public Class<?> findClass(String name,List<Throwable> suppressed) {
  for(Element element:dexElements) {
    Class<?> clazz = element.findClass(name,definingContext,suppressed);
    if(clazz != null) {
      return clazz;
    }
  }
  if(dexElementsSuppressedExceptions != null) {
    suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
  }
  return null;
}
```

Element内部封装了DexFile，DexFile用于加载dex文件，因此每个dex文件对应衣蛾Element。多个Element组成了有序的Element数组dexElements。当要查找类时，会遍历Element数组dexElements(相当于遍历dex文件数组），调用Element的findClass方法，其内部调用DexFile的loadClassBinaryName方法，**如果在当前Element中找到了目标Class，就返回。否则接着找下一个Element**。

根据加载顺序，我们可以将有问题的代码修复好以后，打成一个dex，<font color="#dd0000">**将其放置在dexElements数组的首位**</font>，这样可以是虚拟机加载到修复后的Class。

为什么需要重启呢？<font color="#dd0000">**因为已经加载过的类是无法卸载的，只能重新走一遍类加载流程加载修复过的Class**</font>。

QQ空间的超级补丁和Nuwa按上面说的将补丁包放在Element数组的第一个元素以优先加载；微信Tinker将新旧APK做diff，得到patch.dex，再将patch.dex与手机中APK的classes.dex合并，生成新的classes.dex，<font color="#dd0000">**然后在运行时通过反射将classes.dex放到Elements数组的第一位**</font>；饿了么的Amigo则是将补丁包中每个dex对应的Element取出来，之后组成新的Element数组，**在运行时通过反射用新的Element数组替换掉现在的Element数组**。

### 13.4.2 底层替换方案

**P322**：

底层替换方案不需要再次加载类，而是直接在Native层修改原有类，**由于在原有类进行修改限制会比较多，且不能增减原有类的方法和字段，如果我们增加了方法数，那么方法索引数也会增加，这样访问方法时无法通过索引找到正确的方法，同样的字段也有类似的情况。**

<font color="#dd0000">**替换ArtMethod结构体中的字段或者替换整个ArtMethod结构体，这就是底层替换方案。**</font>AndFix采用替换ArtMethod结构体中的字段，这样会有兼容问题，因为厂商可能修改ArtMethod结构体，导致方法替换失败。**Sophix采用的是替换整个ArtMethod结构体，这样不会存在兼容问题**。

### 13.4.3 Instant Run方案

**P323**：

<font color="#dd0000">**Instant Run在第一次构建APK时，使用ASM在每一个方法中注入了类似如下的代码**</font>

```java
IncrementalChange localIncrementalChange = $change;//1
if(localIncrementalChange != null) { //2
  localIncrementalChange.access$dispatch(
  	"onCreate.(Landroid/os/bundle;)V",new Object[]{this,paramBundle});
  return ;
}
```

当我们点击InstantRun时，如果方法没有变化则$change为null；就调用return如果方法有变化，就生成替代类，这里我们假设MainActiivity的onCreate做了修改，就会生成MainActivity#override，这个类实现了IncrementalChange接口。

### 13.5 动态链接库的修复

**p323**：热修复框架的so修复主要更新so，因此so的修复的基础原理是加载so。

**P324**：<font color="#dd0000">**System的load方法传入的参数是so在磁盘的完整路径，用于加载指定路径的so。System的loadLibrary传入的参数是so的名称，用于加载App安装后自动从apk包中复制到/data/data/packangename/lib下的so.**</font>

**P332**：修复SO文件的两个方案

1. 将SO补丁插入到NativeLibraryElement数组的前部，让SO补丁的路径先被返回和加载（类似代码dex修复)。
2. 调用System的load方法来接管So的加载入口。

## 第十四章 Hook技术

**P335**：被劫持的对象，称作Hook点，为了保证Hook的稳定性，Hook点一般选择**容易找到且不易变化** 的对象，**静态变量和单例**就符合这一条件。

**P336**：根据Hook的实现方式可以分为两种

+ 通过**反射和代理**实现，只能Hook当前的应用程序进程。
+ 通过**Hook框架**来实现，比如Xposed，可以实现全局Hook(Zygote进程)，但是<font color="#dd0000">**需要root**</font>。

**P343**：一个进程中只有一个ActivityThread，ActivityThread是主线程的管理类。

<font color="#dd0000">**扩展：ActivityThread**</font>

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

**ActivityThread的作用很多，但最主要的作用是根据AMS(ActivityManagerService的要求，通过IApplicationTHread的接口)负责调度和执行activities、broadcasts和其它操作**。

## 第十五章 插件化原理

**P352**：如果加载的插件不需要和宿主有任何耦合，也无须和宿主进行通信，比如加载第三方APP，那么推荐使用RePlugin，其他情况推荐使用VirtualApk。

### 15.4 Activiyt插件化

三种实现方式：反射、接口、Hook。主流的采用Hook的方式，Hook的实现有两种方案：

+ 通过Hook  IActivityManager实现
+ 通过Hook Instrumentation实现

## 第十六章 绘制优化

### 16.1.1绘制原理

**P395**：View的绘制流程有3个步骤，分别是measure、layout和draw，它们主要运行在系统的应用架构层，<font color="#dd0000">**而真正将数据渲染到屏幕上的则是系统Native层的SurfaceFlinger服务来完成的。**</font>

绘制过程主要由CPU来进行Measure、Layout、Record、Execute的数据计算工作，GPU负责栅格化、渲染。**CPU和GPU是通过图形驱动层来进行连接的，图形驱动层维护了一个队列，CPU将display list添加到改队列中，这样GPU就可以从这个队列中取出数据进行绘制**。

Android系统每16ms发出VSYNC(垂直同步型号)信号，触发UI进行渲染，VSYNC是一种<font color="#dd0000">**定时中断**</font>，一旦收到VSYNC信号，CPU就开始处理各帧数据。

产生卡顿的原因：

+ 布局Layout过于复杂，无法在16ms内完成渲染。
+ 同一时间动画执行的次数过多，导致CPU和GPU负载过重。
+ View过度绘制，导致某些像素在同一帧时间内被绘制多次。
+ 在UI线程中做了稍微耗时的操作。
+ GC回收时暂停时间过长或频繁的GC产生了大量暂停时间。



## 第十七章 内存优化

### 17.1.2 内存泄漏的场景

1. 非静态内部类的静态实例

   **非静态内部类会持有外部类实例的引用**，如果非静态内部类的实例是静态的，就会间接地长时间持有外部类的引用，阻止外部类实例被系统回收。

   ```java
   public class SecondActivity extends AppCompatActivity {
     private static Object inner;
     private Button button;
     
   }
   ```

   

2. 







