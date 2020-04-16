---
title: Android『事件通信』
tags:
---

集合文章，包含 基础知识(Intent、Bundle）、进程内通信(Handler)、进程间通信(AIDL、Bundle、ContentProvider）等内容。



## Binder机制

### 一次拷贝进程间数据传递

在Linux中规定一个进程分为**用户空间**和**内核空间**。区别是，用户空间的数据不可在进程间共享，为了保证数据安全性 、独立性，即进程隔离。而`内核空间可以被共享且所有用户空间共享一个内核空间`。用户空间与内核空间的通信是使用`系统调用`：

```java
copy_from_user（）;//将用户空间的数据拷贝到内核空间
copy_to_user（）;//将内核空间的数据拷贝到用户空间
```

![img](http://47.110.40.63:8080/img/blog/传统跨进程通信流程.png)

- “进程1”通过系统调用copy_form_user() 将需要发送的数据拷贝到Linux进程的内核空间中的缓冲区。（拷贝第一次）
- 内核服务程序唤醒“进程2”的接收线程，通过系统调用copy_to_user() 将数据发送到用户空间。（拷贝第二次）

由于传统的跨进程通信需拷贝数据2次，且接收方事先并不知道需要创建多大的缓存空间用来接收数据，造成了资源浪费。而采用Binder机制实现通信的时候，通过`内存映射调用Linux下的系统调用方法mmap()`，**腾讯的开源跨进程存储框架MMKV也是基于mmap系统调用实现的**，只需系统调用copy_from_user()拷贝1次数据就可以实现进程间传递数据，节省了内存，提高了效率。见下图：

![img](http://47.110.40.63:8080/img/blog/mmap实现一次拷贝完成进程通信.png)

### Binder

1. Binder是Android中的一个类，它实现了IBinder接口。

2. 从IPC角度来说，Binder是Android中的一种`跨进程通信方式`，Binder还可以理解为`一种虚拟的物理设备`，它的设备驱动是/dev/binder，该通信方式在Linux中没有。

3. 从Android Framework角度来说，`Binder是ServiceManager连接各种Manager(ActivityManager、WindowManager等等)和相应ManagerService的桥梁`。

4. 从Android应用层来说，`Binder是客户端和服务端进行通信的媒介`，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过Binder对象，客户端就可以获取服务端提供的服务或者数据。这里的服务包括普通服务和基于AIDL的服务。

| name                    | 价格                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Client 客户端进程       | 使用服务的进程                                               |
| **Server 服务端进程**   | 提供服务的进程                                               |
| **ServiceManager 进程** | 管理Service 的注册于查询，类似于路由器                       |
| **Binder驱动**          | 一种虚拟的设备驱动； <br>连接Server、Client和ServiceManager的桥梁；<br>能够通过内存映射在进程间传递消息；<br>采用Binder线程池进行控制线程。（默认线程数最大是16） |




## AIDL





## 参考

[Android IPC进程间通信，Binder机制原理及AIDL实例](https://blog.csdn.net/csdn_aiyang/article/details/81592394)