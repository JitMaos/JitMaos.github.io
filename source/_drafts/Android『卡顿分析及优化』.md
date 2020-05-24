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





## 优化建议

### 布局优化

