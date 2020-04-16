---
title: PreK
tags:
---

### Android基础

+ touch事件传递
+ Handler消息传递机制
+ Android组件绘制过程(measure、layout、draw)过程、invalidate()、postInvalidate()、requestLayout()（https://blog.csdn.net/yanbober/article/details/46128379）
+ 调用setContentView()发生了什么？
+ Service相关
  + Service启动方式
  + 生命周期
  + 8.0以后的兼容性问题
+ Launch Mode (https://blog.csdn.net/sinat_14849739/article/details/78072401)
+ 自定义组件
+ Android动画属性动画，动画集合
+ NDK、JNI使用
+ JSBridge交互(注意JS脚本的动态下发)
+ 界面适配
+ 屏幕翻转，保存状态
+ Fragment生命周期
  + 嵌套Fragment实现
  + ~~常用方法、类(FragmentTranscation)~~

### 性能优化

+ ANR定义，常见场景，如何避免
+ APP卡顿原因分析
+ APP内存优化
+ 布局优化
  + merge、include、viewstub的区别
  + 如何优化布局性能
  + RecylerView
    + 介绍一下缓存机制，及其与ListView的区别
    + 如何局部刷新指定数据？
    + 如何使用多种布局样式？
+ 性能检测方式
  + Profiler使用
  + LeakCanary原理
  + BlockCanary原理

+ APK体积优化
  + 图片优化及注意事项
  + ~~图片格式及计算方式~~
+ 常见内存泄漏场景及优化
+ 进程保活，推送到达率
  + 双进程包活
  + 一像素包活
  + Service互调包活
  + 空Activity互调，转发Intent包活
  + JobScheduler包活
  + 监听系统广播保活
  + AlarmManager定时唤醒
  + 将service设置为前台服务，startForeground()
  + AccountService定时同步保活

### Android系统源码

+ Binder机制
+ 一个APP的启动过程
+ 进程间通信ServiceManager、Binder驱动、Client、Server
+ AIDL
+ AMS
+ WMS
+ PMS

### 效率工具

+ JetKins自动构建，多渠道打包
+ 自动化测试
+ AB测试
+ 灰度发布

### 其他问题

+ 有哪些优化程序性能的操作
+ 工作中的难点及亮点
+ 手写一下项目架构设计
+ 觉得自己最擅长哪方面的技术
+ 项目中遇到的印象比较深的问题及解决方法

### Java线程相关

+ 列举Java的线程状态
+ 手写生产者-消费者模型
+ 同步、中断、线程池
+ Synchronized
+ Volatile
+ ReentrantLock
+ 乐观锁、悲观锁、自旋锁、重进锁
+ CAR

### 网络相关

+ 三次握手、四次挥手
+ 一次完整的GET请求过程分析
+ POST、GET区别
+ 数字证书、数字签名、非对称加密、对称加密

+ HTTPS原理
+ HTTPS和HTTP的区别，优缺点

### 开源框架

+ APT(Annotation Processing Tool))
+ EventBus
+ ButterKnife
+ RxJava
+ Retrofit
+ 热修复原理
+ Glide
+ 组件化架构
+ ~~插件化架构~~

### Java核心

+ Java 8新特性
+ HashMap实现原理、HashMap和HashTable区别
+ 类加载机制
+ JVM内存模型(堆、栈、方法区等）
+ Annotation(注解)
+ Full GC，Minor GC，Major GC
+ GCRoots类型
+ 反射机制
+ ThreadLocal

### 架构设计

- 面向对象
  - 多态、重载、重写等
- RxJava操作符、理念
- 设计模式
  - 观察者模式
  - 单例模式的多种写法
  - Builder模式

+ MVP设计模式
  + 使用过程中遇到什么问题，
+ MVVM设计模式
+ 插件化与组件化
+ Kotlin(多线程、协程)
+ 简述RxJava中用到的设计模式？
+ Retrofit中的设计模式

### 算法

+ 数据结构

  + 二叉树
  + 红黑树
  + 堆
  + 图

+ 排序

  + 快速排序
  + 插入排序
  + 归并排序

+ 动态规划

  + 

+ 经典问题

  

### 分享

[2018年秋招腾讯、饿了么、百度Android岗一面都问到哪些问题？](<https://blog.csdn.net/Coo123_/article/details/86682398>)

[饿了么-Android开发](<https://www.jianshu.com/p/99956c3c62e4>)

[饿了么java面试题(三轮面试亲身经历总结](<https://blog.csdn.net/lijian1975/article/details/51024904>)

[ANDROID工程师 面试经验( 上海普陀 ) - 饿了么网站](<https://www.job592.com/pay/ms315527.html>)

[饿了么面试回顾](<https://blog.51cto.com/12902480/1923918>)



### 链接

[九章算法](<https://www.jiuzhang.com/solution/>)