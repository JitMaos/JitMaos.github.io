---
title: 读书笔记『Android组件化框架』
date: 2019-03-26 05:50:12
tags: [读书笔记,组件化]
---

**P3**：组件：指的是单一的功能组件，如视频组件(VideoSDK)、支付组件(PaySDK)、路由组件(Router)等；模块：指的是独立的业务模块，如直播模块(LiveModule)、首页模块(HomeModule)、即时通讯模块(IMModule)等。

**P7**：每个module都有一份AndroidManifest文件用来记载其信息，最终生成一个App时只合成一份Manifest文件，生成地址:app/build/intermediates/manifests/full/debug/AndroidManifest.xml。

**P12**：module的Application合并过程中，**主module的优先级高于功能module**，如果功能module中定义了两个Application，则需要解决冲突(每个功能module的Applicationg节点需要添加**tools:replace**)。

**P15**：通过在manifest中声明android:shareUserId可以指定多个不同的App运行在<font color="#dd0000">**同一个进程中，且默认数据可以互相访问**</font>，只有在主module中声明sharedUserId才能最终打包到full AndroidManifest中。

**P17**：在Application中通过ActivityLifecycleCallbacks接口监听Activity的生命周期，可以获得当前的栈顶Activity，可以使用该Activity实现**全局弹框**。

<!-- more -->

**P23**：本地广播LocalBroadcastManager是Android Support包提供的一个工具，用来在<font color="#dd0000">**同一个应用内**</font>的不同组件间发送广播，基于观察者模式。因为设定为本地广播，所以无法像全局广播那样注册到AndroidManifest。<font color="#dd0000">**本地广播传播信息的过程，无法干预自定义**</font>。

**P29**：EventBus 2.x使用的是运行时注解，基于Java的反射机制，反射是耗费效率的。EventBus 3.0是基于编译时注解，使用AnnotationProcessor在编译器创建出对文件或类的索引关系，在使用时直接使用。

**P32**：使用xxxBus事件总线类机制实现组件间通信的问题：事件总线需要放置在BaseModule中，每个模块增删时，都需要删除或添加BaseModule中的事件信息，不删除又会造成项目臃肿。<font color="#dd0000">**关键在于如何动态的将信息模型添加到公共的地方，然后能被其他模块索引到**</font>。

**P37**：如果要确保只有自己的APP能启动组件，可以设置**exported=false**，这样别的APP将无法通过隐式intent启动APP中的Activity。

