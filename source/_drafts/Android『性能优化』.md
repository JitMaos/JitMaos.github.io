---
title: Android『性能优化』
date: 2019-04-10 06:23:32
tags: Android
---

APP优化主要分为4个类别，分别是流畅、稳定、省电省流量、安装包大小



#### 流畅度

+ 常见问题

+ 检测工具

+ 优化

  + 局部刷新

    + RecyclerView的局部刷新

      一般就两种做法：第一种就是通过recyclerview的findViewHolderForAdapterPosition方法找到那个item所在的viewholder直接拿到viewholder进行操作，不过这种方法存在弊端，就是只能获取当前屏幕所显示的viewholder,如果不在当前屏幕那就会获取的是一个null。所以在使用的时候应该通过manager.findFirstVisibleItemPosition()和manager.findLastVisibleItemPosition()判断要改变的item的position是否在这区间内。第二种方式通过layoutManager.findViewByPosition()获取view 注意的地方也是和第一种一样。

      notifyItemChanged(logPosition,"test").第二个参数任意类型的参数，还需要在适配器中重写onBindViewHolder(VH holder, int position, List<Object> payloads)方法，第三个参数大小只有1.它的值就是我们之前传入的“test”。

  + APP启动方式

#### 内存优化

+ 问题描述
+ 性能分析
+ Java内存相关
+ 其他优化
  + 对象及时销毁，资源及时释放
  + 请求及时取消，LifeCycle
  + 



内存溢出

+ 定义
+ 
+ 游湖方式

内存泄漏

+ 定义
+ 常见场景
+ 优化方式

资源优化

+ 



用户体验优化

+ 启动优化
+ 加载对话框
+ 防止重复提交、抖动
+ 



参考：[Android 性能优化：使用 TraceView 找到卡顿的元](https://blog.csdn.net/u011240877/article/details/54347396)

<https://www.cnblogs.com/cr330326/p/8011523.html>