---
title: Android『保活相关』
tags:
---



### 基础知识

#### Service的兼容

1. 5.0以后不再支持隐式Intent方式启动外部Service。
2. 8.0以后对后台服务有了更多的限制，对服务做了前台服务、后台服务的区分。**应用进入后台时，有一个短暂的时间窗口，可以在该时间窗口内创建和使用服务，在时间窗口结束后，应用将被视为空闲状态，此时，系统将停止应用的**<font color="#dd0000">**后台服务**</font>，就像应用已经调用服务的**Service.stopSelf()**一样。
3. 基于**2**，Android 8.0引入了一个新方法**Context.startForegroundService()**，在系统创建服务后，应用有五秒的时间调用startForeground()，来显示新服务的可见通知(Notification)。**如果应用在此期间限制内未调用startForeground()，则系统将停止并声明此应用为ANR**。
4. Android在8.0限制了后台服务这些，启动后台服务需要设置通知栏，使服务变成前台服务。但是在<font color="#dd0000">**9.0**</font>上，就会出现`Permission Denial: startForeground requires android.permission.FOREGROUND_SERVICE`。

<!--more-->

### 保活方案

1. 一像素包活

   据说这个是手Q的进程保活方案，基本思想，系统一般是不会杀死前台进程的。所以要使得进程常驻，我们只需要在锁屏的时候在本进程开启一个Activity，为了欺骗用户，让这个Activity的大小是1像素，并且透明无切换动画，在开屏幕的时候，把这个Activity关闭掉，所以这个就需要监听系统锁屏广播.
   ————————————————
   版权声明：本文为CSDN博主「_YoungMan」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
   原文链接：https://blog.csdn.net/qq_37199105/article/details/81224842

2. 通知栏显示前台服务

   据说这个微信也用过的进程保活方案，该方案实际利用了Android前台service的漏洞。
   原理如下
   对于 API level < 18 ：调用startForeground(ID， new Notification())，发送空的Notification ，图标则不会显示。
   对于 API level >= 18：在需要提优先级的service A启动一个InnerService，两个服务同时startForeground，且绑定同样的 ID。Stop 掉InnerService ，这样通知栏图标即被移除。
   ————————————————
   版权声明：本文为CSDN博主「_YoungMan」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
   原文链接：https://blog.csdn.net/qq_37199105/article/details/81224842

3. 双进程保活

   顾名思义,就是指的不同进程,不同app之间互相唤醒,如你手机里装了支付宝、淘宝、天猫、UC等阿里系的app，那么你打开任意一个阿里系的app后，有可能就顺便把其他阿里系的app给唤醒了。

4. JobScheduler

   JobSheduler是作为进程死后复活的一种手段，native进程方式最大缺点是费电， Native 进程费电的原因是感知主进程是否存活有两种实现方式，在 Native 进程中通过死循环或定时器，轮训判断主进程是否存活，当主进程不存活时进行拉活。其次5.0以上系统不支持。 但是JobSheduler可以替代在Android5.0以上native进程方式，这种方式即使用户强制关闭，也能被拉起来，亲测可行。
   ————————————————
   版权声明：本文为CSDN博主「_YoungMan」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
   原文链接：https://blog.csdn.net/qq_37199105/article/details/81224842

5. 粘性服务&与系统服务捆绑

   这个是系统自带的，onStartCommand方法必须具有一个整形的返回值，这个整形的返回值用来告诉系统在服务启动完毕后，如果被Kill，系统将如何操作，这种方案虽然可以，但是在某些情况or某些定制ROM上可能失效，**我认为可以多做一种保保守方案**。

   START_STICKY
   如果系统在onStartCommand返回后被销毁，系统将会重新创建服务并依次调用onCreate和onStartCommand（注意：根据测试Android2.3.3以下版本只会调用onCreate根本不会调用onStartCommand，Android4.0可以办到），这种相当于服务又重新启动恢复到之前的状态了）。

   START_NOT_STICKY
   如果系统在onStartCommand返回后被销毁，如果返回该值，则在执行完onStartCommand方法后如果Service被杀掉系统将不会重启该服务。

   START_REDELIVER_INTENT
   START_STICKY的兼容版本，不同的是其不保证服务被杀后一定能重启。
   ————————————————
   版权声明：本文为CSDN博主「cmyperson」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
   原文链接：https://blog.csdn.net/cmyperson/article/details/89524921

6. 显式Service互调，需要一定的流量，并和其他APP约定互调路径，参数

7. 显式Activity互调，需要一定的流量，并和其他APP约定互调路径，参数

8. 使用JetPack中的WorkManager进行保活

9. 播放无声背景音乐保活（仅限用户点击清理全部时，不被结束）

10. 集成厂商推送SDK进行进程保活，以华为为例：

    与个推、小米、极光推送类似，华为Push也是为开发者提供的一个消息推送平台，它建立了从云端到手机端的消息推送通道，支持上报标签、LBS信息、通知推送。换句话来说，就算我们的APP没有自己的服务器，也可以通过这些第三方推送平台，把最新消息及时地通知用户。

    **华为推送复活原理：**华为推送服务以后台Service方式运行在一个独立进程里，主程序不需要常驻内存。当该后台Service接收到push消息后会以广播的方式通知主进程，触发相关回调接口。通常，被强制停止的APP无法接收到广播，但是华为push通道(即推送后台Service，它会常驻内存)能够强行将APP拉起来，这是因为其在发广播时利用了Intent.FLAG_INCLUDE_STOPPED_PACKAGES标记实现的。

    <http://www.52im.net/thread-1140-1-1.html>

11. 自定义锁屏界面

12. 同派系APP广播互相唤醒

13. 仿TIM引导用户打开“后台自启动”和加入“手机白名单”。



### 已失效？？

盘点已经失效或者不适合的保活黑科技，这里的“不适合”是因为我们做政府项目，后台是布在内网上的。

1.双进程守护方案，华为6.0就失效

2.监听锁屏/亮屏/解锁广播，打开1像素Activity，华为6.0就失效，因广播被取消了

3.故意在后台播放无声的音乐，华为M10手机9.0失效

4.使用JobScheduler唤醒Service，7.0以上失效

5.集成华为/小米/oppo/vivo/魅族等push，因为项目本地化部署，不适合

6.推送互相唤醒复活：极光、友盟、以及各大厂商的推送，因为项目本地化部署，不适合

7.同派系APP广播互相唤醒：比如今日头条系、阿里系，因为项目本地化部署，不适合

8.使用自定义锁屏界面：覆盖了系统锁屏界面。网易云音乐就是如此，但是会生成一个常驻通知栏的通知

9.把APP设置为系统应用，不适合，因为需要权限等

10.native进程(已报废)

11.利用账号同步机制拉活,失效了

12. 提高Service优先级，比如onStartCommand返回START_STICKY，没什么效果
   ————————————————
   版权声明：本文为CSDN博主「嵩风抚」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
   原文链接：https://blog.csdn.net/u012482178/article/details/95218872
13. 

### API

#### ServiceConnection