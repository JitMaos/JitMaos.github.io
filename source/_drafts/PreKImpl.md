---
title: PreKImpl
tags:
---

#### GCRoots类型

**所有Java线程当前活跃的栈帧里指向GC堆里的对象的引用；换句话说，当前所有正在被调用的方法的引用类型的参数/局部变量/临时值。**

 VM的一些静态数据结构里指向GC堆里的对象的引用

所有当前被加载的Java类

 Java类的引用类型静态变量

 Java类的运行时常量池里的引用类型常量（String或Class类型）

String常量池（StringTable）里的引用

- 虚拟机栈中引用的对象； 
- 方法区中类静态属性引用的对象； 
- 方法区中的常量引用的对象； 
- 本地方法栈中JNI（即一般说的Native方法）的引用的对象；

#### 多进程带来的问题

- 静态成员和单例模式完全失效(不是同一块内存，会产生不同的副本)
- 线程同步机制完全失效(不是同一块内存，所以对象也不是同一个，因此类锁、对象锁也不是同一个，不能保证线程同步)
- SharedPreferences 可靠性下降(SharedPreferences不支持多个进程同时写，会有一定的几率丢失数据)
- Application 多次创建(Android为每个进程分配独立的虚拟机，这个过程其实就是启动一个应用，所以Application会被创建多次)，所以我们不能直接将一些数据保存在Application中。

#### 一次完整的HTTP请求过程

​     **域名解析(DNS，万维网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。通过域名，最终得到该域名对应的IP地址的过程叫做域名解析（或主机名解析）)**

​     **发起TCP的3次握手** 

​     **建立TCP连接后发起http请求** 

​     **服务器响应http请求，浏览器得到html代码** 

​     **浏览器解析html代码，并请求html代码中的资源（如js、css、图片等）** 

​     **浏览器对页面进行渲染呈现给用户**

#### Parcelable与Serializable的区别

`Serializable` 使用 **I/O 读写存储在硬盘上**，而 `Parcelable` 是直接 **在内存中读写**。很明显，内存的读写速度通常大于 IO 读写，所以在 Android 中传递数据优先选择 `Parcelable`。

`Serializable` 会使用<font color="#dd0000">**反射**</font>，序列化和反序列化过程需要大量 I/O 操作， `Parcelable` 自已实现封送和解封（marshalled &unmarshalled）操作不需要用反射，数据也存放在 Native 内存中，效率要快很多。

#### JVM内存模型

+ 程序计数器

程序计数器是一块很小的内存空间，它是**线程私有的**，可以认作为当前线程的行号指示器。

+ Java栈

栈描述的是Java方法执行的内存模型**。**每个方法被执行的时候都会创建一个栈帧用于存储局部变量表，操作栈，动态链接，方法出口等信息。

+ 本地方法栈

本地方法栈是与虚拟机栈发挥的作用十分相似,区别是虚拟机栈执行的是Java方法(也就是字节码)服务，而本地方法栈则为虚拟机使用到的native方法服务，可能底层调用的c或者c++,我们打开jdk安装目录可以看到也有很多用c编写的文件，可能就是native方法所调用的c代码。

+ 堆

堆是java虚拟机管理内存最大的一块内存区域，因为堆存放的对象是线程共享的，所以多线程的时候也需要同步机制。

+ 方法区

方法区同堆一样，是所有线程共享的内存区域，为了区分堆，又被称为非堆。用于存储已被虚拟机加载的类信息、常量、静态变量，如static修饰的变量加载类的时候就被加载到方法区中。





Android怎么申请更大的内存

Java中的四种内部类<https://blog.csdn.net/sinat_37003267/article/details/80058544>

JetPack之WorkManager

哪些OOM的实例，及解决方案。

描述一个开源项目，EventBus如何切换线程

Toast在7.X上badk token异常：<https://blog.csdn.net/joye123/article/details/80738113>

Dalvik虚拟机和ART虚拟机，

Android launchMode中affinity

<https://blog.csdn.net/zhangjg_blog/article/details/10923643>

<https://www.cnblogs.com/yyz666/p/4674173.html>