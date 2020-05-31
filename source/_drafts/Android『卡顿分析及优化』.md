---
title: Android『卡顿分析及优化』
tags:
---

##概述

View的绘制流程有3个步骤，分别是measure、layout和draw，它们主要运行在系统的应用架构层，<font color="#dd0000">**而真正将数据渲染到屏幕上的则是系统Native层的SurfaceFlinger服务来完成的。**</font>

绘制过程主要由CPU来进行Measure、Layout、Record、Execute的数据计算工作，GPU负责栅格化、渲染。**CPU和GPU是通过图形驱动层来进行连接的，图形驱动层维护了一个队列，CPU将display list添加到改队列中，这样GPU就可以从这个队列中取出数据进行绘制**。

Android系统每16ms发出VSYNC(垂直同步型号)信号，触发UI进行渲染，VSYNC是一种<font color="#dd0000">**定时中断**</font>，一旦收到VSYNC信号，CPU就开始处理各帧数据。

产生卡顿的原因：

+ 布局Layout过于复杂，无法在16ms内完成渲染。
+ 同一时间动画执行的次数过多，导致CPU和GPU负载过重。
+ View过度绘制，导致某些像素在同一帧时间内被绘制多次。
+ 在UI线程中做了稍微耗时的操作。
+ GC回收时暂停时间过长或频繁的GC产生了大量暂停时间。

## 检测工作

可用的检测卡顿工具有：Profile GPU Rendering、Systrace、TraceView

### Profile GPU Rendering

 本节参考：[Android性能优化典范——GPU渲染（Profile GPU Rendering)](https://blog.csdn.net/o190847959/article/details/54411721)

Profile GPU Rendering是**Android4.1**系统提供的开发辅助功能。绿色的横先为警戒先，超过这条线意味着某一帧的绘制时长超过了16ms。

![img](http://47.110.40.63:8080/img/blog/非架构/ProfileGPURendering.jpeg)

每一条柱状线都包含三部分：
（1）蓝色代表<font color="#dd0000">**测量绘制**</font>的时间，也就是需要多长时间去创建和更新DisplayList。<font color="#dd0000">**如果蓝色部分很高，可能需要重新绘制，或者View的onDraw方法处理事情太多**</font>。
（2）红色代表<font color="#dd0000">**执行**</font>的时间，这部分是Android进行2D渲染Display List的时间。如果红色部分很高，**可能是由于重新提交了视图导致的。还有复杂的自定义View也会导致红色部分变高**。
（3）橙色部分代表<font color="#dd0000">**处理**</font>时间，是CUP高速GPU渲染一帧的地方，**这是一个阻塞调用**，因为CPU会一直等待GPU发送接到命令的回复，<font color="#dd0000">**如果橙色部分很高，说明GPU很繁忙**</font>。
**CPU 与 GPU**
现代的图形API不允许CPU直接与GPU通信，而是通过中间的一个图形驱动层（Graphics Driver）来连接这两部分。
![CPU——GPU任务传递1](http://47.110.40.63:8080/img/blog/CPU_GPU任务传递1.png)
图形驱动维护了一个队列，CPU把display list添加到队列里，GPU从这个队列取出数据进行绘制。
![CPU——GPU任务传递2](http://47.110.40.63:8080/img/blog/CPU_GPU任务传递2.png)

### Systrace

Systrace是**Android 4.1**中新增的性能数据采集和分析工具，它可以帮助开发者收集Android关键子系统（SurfaceFlinger、WMS等Framework部分关键模块、服务，View体系系统等）的运行信息。**Systrace的功能包括跟踪系统的I/O操作、内核工作队列、CPU负载以及Android各个子系统的运行状况等。对于UI显示性能，比如动画播放不流畅、渲染卡顿等问题提供了分析数据。**

## 优化建议

### 布局优化

