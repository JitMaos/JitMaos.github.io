---
title: Android实例『仿抖音直播侧滑广场实现』
tags:
---

最近在做直播带货的项目，要模仿抖音来做。于是有了本文，本文的目的是为了通过一个具体的按钮，演示自定义touch传递流程在实际场景中的使用。

首先先看目标效果



这里有几点要求

1. 首先三层嵌套，主题，弹幕，广场。可以通过左右滑动隐藏或打开弹幕、广场层。
2. 其次，有广场的时候，纵向滑动焦点先交给广场里的recyclerview，如果水平超过一定距离，或速度，再把焦点交给广场，同时要处理下拉刷新和分页，不能切换主播。
3. 还有，还有如果直接点击广场外面，或者一上来就是水平滑动，焦点直接交给广场，RecyclerView不能中途获得焦点。
4. 最后，需要处理广场中recyclerview的下拉刷新和分页加载；还需要处理广场中直播房间的分页加载。

基于以上几点要求，进行布局框架选型：ViewDragHelper + DrawerLayout + VerticalViewPager + RecyclerView来实现





内存卡顿记录



新方案：

RecyclerView + 收拾 + PagerSnapHelper + DrawerLayouer + 播放器动态添加；目前没发现问题，且解决了使用FragmentPagerAdapter带来的内存管理的问题。

值得一提的是，RecyclerView的Item在滚入滚出的回调时机放到PagerSnapHelper的findSnapView回调方法中进行，此时item已经归位，就算连续快速滑动，也只会触发最后一次归位动作。试过在findTargetSnapPosition和RecyclerView.OnChildAttachStateChangeListener中处理，皆因触发时机太早而不符合要求。