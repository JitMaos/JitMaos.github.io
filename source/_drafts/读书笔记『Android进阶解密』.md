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

