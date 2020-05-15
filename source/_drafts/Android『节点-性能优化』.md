---
title: Android『节点-性能优化』
tags:
---

# 内存优化

## 基础知识

在Android中，5.0以下使用的是Dalvik虚拟机，5.0及以上使用的是ART虚拟机。

**ART与Dalvik虚拟机的区别**

摘自：https://www.zhihu.com/question/55652975/answer/154602317

Android 4.x(Interpreter + JIT)

- - 原理：平时代码走解释器，但热点trace会执行JIT进行即时编译
  - 优点：占用内存少
  - 缺点：耗电(退出App下次启动还会重复编译)，卡顿(JIT编译时)

Android 5.0/5.1/6.0(interpreter + AOT)

- - 原理: 在AOT模式下，<font color="#dd0000">**App在安装过程时， 就会完成所有编译**</font>。
  - 优点: 性能好
  - 缺点: App安装时间长，占用存储空间多。

Android 7.0/7.1的ART引入了全新的Hybrid模式(Interpreter + JIT + AOT)

- - 原理: 

  - - **App在安装时不编译， 所以安装速度快**。
    - <font color="#dd0000">**在运行App时， 先走解释器， 然后热点函数会被识别，并被JIT进行编译， 存储在jit code cache， 并产生profile文件(记录热点函数信息)**</font>。 
    - <font color="#dd0000">**等手机进入charging和idle状态下， 系统会每隔一段时间扫描App目录下profile文件，并执行AOT编译(Google官方称之为profile-guided compilation)**</font>。
    - 不论是jit编译的binary code, 还是AOT编译的binary code, 它们之间的性能差别不大， 因为它们使用同一个optimizing compiler进行编译。

  - 优点: App安装速度快，占用存储少(只编译热点函数)。

  - 缺点: 前几次运行会较慢， 只有用户操作得次数越多，jit 和AOT编译后， 性能才会跟上来。

综上，Android 7.0/7.1上的ART是将Android 4.x的JIT和Android 5.x/6.0上的AOT结合，取长补短。

## 内存检测

摘自：https://juejin.im/post/5b1b5e29f265da6e01174b84

### ADB 命令

1. 当前应用使用内存分析

```shell
adb shell dumpsys meminfo 包名
```

![img](http://47.110.40.63:8080/img/blog/非架构/adb-meminfo命令.jpg)

`Pss`: 该进程独占的内存+与其他进程共享的内存（按比例分配，比如与其他3个进程共享9K内存，则这部分为3K）

`Privete Dirty`:该进程独享内存

`Heap Size`:分配的内存

`Heap Alloc`:已使用的内存

`Heap Free`:空闲内存

### MAT工具



##内存优化实践



# 包体积优化



# 启动时间优化



# 稳定性优化



