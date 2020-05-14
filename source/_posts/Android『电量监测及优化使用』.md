---
title: Android『本地电量监测』
tags: [Android，性能优化，电量优化]
---

# 本地电量检测

做电量优化第一步，首先需要知道当前APP的耗电情况，大概步骤：

1. 使用Monkey、Appium、MonkeyRunner等自动化工具产生测试数据，本文基于Monkey进行讨论。
2. 采集耗电信息，本方案基于系统自带的命令，生成电池使用记录文本batterystats.txt文件。
3. 之后导出该文件
4. 最后使用第三方工具分析数据，本文使用。

## Monkey命令介绍

参考：https://blog.csdn.net/qq_30993595/article/details/80748559

![img](http://47.110.40.63:8080/img/blog/非架构/Monkey命令help.png)

1. <font color="#dd0000">**-p 用于约束限制，用此参数指定一个包**</font>，指定包后Monkey将被允许启动指定应用；**如果不指定包，  Monkey将被允许随机启动设备中的应用**（主Activity有android.intent.category.LAUNCHER 或android.intent.category.MONKEY类别 ）

   比如 adb shell monkey -p **xxx.xxx.xxx** 1  ; xxx.xxx.xxx 表示应用包名，1 表示monkey模拟用户随机事件参数，最低1，这样就能把应用启动起来

2. -c 指定Activity的category类别，如果不指定，默认是CATEGORY_LAUNCHER 或者 Intent.CATEGORY_MONKEY；不太常用的一个参数

3. **-v 用于指定反馈信息级别，也就是日志的详细程度**，分Level1、Level2、Level3；-v 默认值，仅提供启动提示，操作结果等少量信息 ，也就是Level1，比如adb shell monkey -p  xxx.xxx.xxx -v 1 ；-v -v 提供比较详细信息，比如启动的每个activity信息 ，也就是Level2，比如adb shell monkey -p xxx.xxx.xxx -v -v 1 ；-v -v -v 提供最详细的信息 ，比如adb shell monkey -p xxx.xxx.xxx -v -v -v 1 

4. <font color="#dd0000">**-s 伪随机数生成器的种子值，如果我们两次monkey测试事件使用相同的种子值，会产生相同的事件序列**</font>；如果不指定种子值，系统会产生一个随机值。种子值对我们复现bug很重要。使用如下adb shell monkey -p xxx.xxx.xxx -s 11111 10；这也是伪随机事件的原因，因为这些事件可以通过种子值进行复现

5. <font color="#dd0000">**--ignore-crashes 忽略异常崩溃，如果不指定，那么在monkey测试的时候，应用发生崩溃时就会停止运行**</font>；如果加上了这个参数，monkey就会运行到指定事件数才停止。比如adb shell monkey -p xxx.xxx.xxx -v -v -v  --ignore-crashes 10 

6. <font color="#dd0000">**--ignore-timeouts 忽略ANR**</font>，情况与4类似，当发送ANR时候，让monkey继续运行。比如adb shell monkey -p xxx.xxx.xxx -v -v -v  --ignore-timeouts 10

7. <font color="#dd0000">**--ignore-native-crashes 忽略native层代码的崩溃**</font>，情况与4类似，比如adb shell monkey -p xxx.xxx.xxx -v -v -v  --ignore-native-crashes 10

8. --ignore-security-exceptions 忽略一些许可错误，比如证书许可，网络许可，adb shell monkey -p xxx.xxx.xxx -v -v -v  --ignore-security-exceptions 10

9. --monitor-native-crashes 是否监视并报告native层发送的崩溃代码，adb shell monkey -p xxx.xxx.xxx -v -v -v  --monitor-native-crashes 10

10. --kill-procress-after-error 用于在发送错误后杀死进程

11. --hprof  设置后，在Monkey事件序列之前和之后立即生产分析报告，保存于data/mic目录，不过将会生成大量几兆文件，谨慎使用

12. <font color="#dd0000">**--throttle 设置每个事件结束后延迟多少时间再继续下一个事件，降低cpu压力**</font>；如果不设置，事件与事件之间将不会延迟，事件将会尽快生成；一般设置300ms，因为人最快300ms左右一个动作，比如 adb shell monkey -p xxx.xxx.xxx -v -v -v  --throttle 300 10

13. --pct-touch 设置触摸事件的百分比，即手指对屏幕进行点击抬起(down-up)的动作；不做设置情况下系统将随机分配各种事件的百分比。比如adb shell monkey -p xxx.xxxx.xxx --pct-touch 50 -v -v 100 ，这就表示100次事件里有50%事件是触摸事件

14. --pct-motion 设置移动事件百分比，这种事件类型是由屏幕上某处的一个down事件-一系列伪随机的移动事件-一个up事件，即点击屏幕，然后直线运动，最后抬起这种运动。

15. --pct-trackball 设置轨迹球事件百分比，这种事件类型是一个或者多个随机移动，包含点击事件，这里可以是曲线运动，不过现在手机很多不支持，这个参数不常用

16. --pct-syskeys 设置系统物理按键事件百分比，比如home键，音量键，返回键，拨打电话键，挂电话键等

17. --pct-nav 设置基本的导航按键事件百分比，比如输入设备上的上下左右四个方向键

18. --pct-appswitch 设置monkey使用startActivity进行activity跳转事件的百分比，保证界面的覆盖情况

19. --ptc-anyevent 设置其它事件百分比

20. --ptc-majornav 设置主导航事件的百分比

21. 保存dos窗口打印的monkey信息，在monkey命令后面补上输出地址，如adb shell monkey -p xxx.xxxx.xxx  -v -v 100 > D:\monkey.txt；这样monkey测试结束后，所有打印的信息都会输出到这个文件里

22. <font color="#dd0000">**通过adb bugreport 命令可以获取整个android系统在运行过程中所有app的内存使用情况，cpu使用情况，activity运行信息等，包括出现异常等信息。使用方法 adb bugreport > bugreport.txt ;这样在当前目录就会产生一个txt文件和一个压缩包，具体信息可在压缩包查看，txt文件只会记录压缩包的生成过程信息**</font>

23. -f 加载monkey脚本文件进行测试，比如 adb shell monkey -f sdcard/monkey.txt -v -v 500

##Monkey使用

1.进入adb目录

2.通过adb install apk名字

3.<font color="#dd0000">**输入adb shell monkey -p xxx.xxxx.xxx  -s 123123 --throttle 300 -v -v 20 > d:\monkey.txt，这里指定了seed值，每个事件之间休息300ms，执行了20个事件，然后将日志信息保存在了monkey.txt文件中**</font>

4.打开文件，查看信息如下:

>  Monkey: seed=123123 count=20 //<font color="#dd0000">**本次事件序列seed值是指定的123123，方便出现bug后再复现 执行事件次数是20**</font>
> :AllowPackage: com.android.mangodialog // 被测试的应用包名
> :IncludeCategory: android.intent.category.LAUNCHER //启动的主activity的两种类别
> :IncludeCategory: android.intent.category.MONKEY
> // Selecting main activities from category android.intent.category.LAUNCHER
> //   + Using main activity com.android.mangodialog.MainActivity (from package com.android.mangodialog) //该应用符合这种类别的activity
> // Selecting main activities from category android.intent.category.MONKEY
> // Seeded: 123123
> // Event percentages://各种事件的百分比，不同厂家设备可能有所不同
> //   0: 15.0%  //可通过--pct-touch 参数设置的事件的百分比 常用
> //   1: 10.0%  //可通过--pct-motion 参数设置的事件的百分比 常用
> //   2: 2.0%   //可通过--pct-pinchzoom 参数设置的事件的百分比
> //   3: 15.0%  //可通过--pct-trackball 参数设置的事件的百分比
> //   4: -0.0%  
> //   5: -0.0%  
> //   6: 25.0%  //可通过--pct-nav 参数设置的事件的百分比
> //   7: 15.0%  //可通过--pct-majornav 参数设置的事件的百分比
> //   8: 2.0%   //可通过--pct-syskeys 参数设置的事件的百分比 常用
> //   9: 2.0%   //可通过--pct-appswitch 参数设置的事件的百分比 常用
> //   10: 1.0%  //可通过--pct-flip 参数设置的事件的百分比
> //   11: 13.0% //可通过--pct-anyevent 参数设置的事件的百分比 
> //启动应用的activity
> :Switch: #Intent;action=android.intent.action.MAIN;category=android.intent.category.LAUNCHER;launchFlags=0x10200000;component=com.android.mangodialog/.MainActivity;end
>     // Allowing start of Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.android.mangodialog/.MainActivity } in package com.android.mangodialog
> Sleeping for 300 milliseconds //设置的事件之间间隔300ms 下面就是执行点击事件
> :Sending Key (ACTION_DOWN): 82    // KEYCODE_MENU
> :Sending Key (ACTION_UP): 82    // KEYCODE_MENU
> Sleeping for 300 milliseconds
> :Sending Key (ACTION_DOWN): 23    // KEYCODE_DPAD_CENTER
> :Sending Key (ACTION_UP): 23    // KEYCODE_DPAD_CENTER
> Sleeping for 300 milliseconds
>     // Allowing start of Intent { cmp=com.android.mangodialog/.MainActivity2 } in package com.android.mangodialog
> :Sending Key (ACTION_DOWN): 22    // KEYCODE_DPAD_RIGHT
> :Sending Key (ACTION_UP): 22    // KEYCODE_DPAD_RIGHT
> Sleeping for 300 milliseconds
> :Sending Key (ACTION_DOWN): 21    // KEYCODE_DPAD_LEFT
> :Sending Key (ACTION_UP): 21    // KEYCODE_DPAD_LEFT
> Sleeping for 300 milliseconds
> :Sending Touch (ACTION_DOWN): 0:(1017.0,280.0)
> :Sending Touch (ACTION_UP): 0:(1021.8751,281.12732)
> Sleeping for 300 milliseconds
> :Sending Touch (ACTION_DOWN): 0:(1005.0,1599.0)
> :Sending Touch (ACTION_UP): 0:(994.4962,1589.7715)
> Sleeping for 300 milliseconds
> :Sending Key (ACTION_DOWN): 2    // KEYCODE_SOFT_RIGHT
> :Sending Key (ACTION_UP): 2    // KEYCODE_SOFT_RIGHT
> Sleeping for 300 milliseconds
> :Sending Key (ACTION_DOWN): 20    // KEYCODE_DPAD_DOWN
> :Sending Key (ACTION_UP): 20    // KEYCODE_DPAD_DOWN
> Sleeping for 300 milliseconds
> :Sending Key (ACTION_DOWN): 22    // KEYCODE_DPAD_RIGHT
> :Sending Key (ACTION_UP): 22    // KEYCODE_DPAD_RIGHT
> Sleeping for 300 milliseconds //轨迹球运动
> :Sending Trackball (ACTION_MOVE): 0:(4.0,-5.0)//手机屏幕上的坐标
> Events injected: 20 //monkey共执行了20次事件
> :Sending rotation degree=0, persist=false
> :Dropped: keys=0 pointers=0 trackballs=0 flips=0 rotations=0
> //<font color="#dd0000">**测试过程中的网络状态，花费了3064ms连接，既没有连接上手机网络，也没有连接上wifi**</font>
>
> Network stats: elapsed time=3064ms (0ms mobile, 0ms wifi, 3064ms not connected) 
>
> // Monkey finished //monkey测试结束

5.平时会使用比较复杂的参数去测试，如下

adb shell monkey -v -v -v -s 123123 --throttle 300 --pct-touch 40 --pct-motion 25 --pct-appswitch 25 --pct-syskeys 10 --pct-majornav 0 --pct-nav 0 --pct-trackball 0 --ignore-crashes --ignore-timeouts --ignore-native-crashes -p xxx.xxx.xxx 100000 > d:\monkey.txt

具体什么意思就不再一一解释了。

6.其实我们比较关注的是app在使用过程中出现的错误信息，像上面我们选择忽略掉错误情况，这样当monkey执行结束后，相关的信息会被写入到monkey文件中，但是错误信息比如crash，anr等信息会打印在dos窗口，这些错误信息会明确的指出哪里发生的错误；<font color="#dd0000">**如果需要复现，我们可以把忽略参数去掉，然后通过相同的seed值再次进行monkey测试，直到发生错误跳出monkey测试，我们再查看**</font>，如下

```java
public class MainActivity2 extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.act_main2);
        float s = 1/0;
    }
}
```


我这第二个activity的oncreate方法中写这个会报错的代码，然后在第一个activity的一个按钮中进行跳转进入这个activity，接下来进行monkey测试来复现这个bug

```shell
adb shell monkey -p xxx.xxx.xxx -s 123456 -v -v 2000  > d:\monkey.txt
```

我们看这个文件

> :Sending Trackball (ACTION_MOVE): 0:(4.0,-3.0)
> :Sending Trackball (ACTION_MOVE): 0:(-2.0,1.0)
> :Sending Trackball (ACTION_DOWN): 0:(0.0,0.0)
> :Sending Trackball (ACTION_UP): 0:(0.0,0.0)
> Sleeping for 300 milliseconds
>     // Allowing start of Intent { cmp=com.android.mangodialog/.MainActivity2 } in package com.android.mangodialog
> ** Monkey aborted due to error.
>
> Events injected: 190


这里可以看到是当打开MainActivity2的时候monkey发生错误退出，只执行了190个事件。至于错误信息打印在了dos窗口。

<font color="#dd0000">**这里我们也可以通过adb bugreport命令将手机运行日志导出来查看，这里面的信息更详细，包括出错信息**</font>。

> //6.0及以下设备
> adb bugreport > bugreport.txt
> //7.0及以上设备
> adb bugreport bugreport.zip

###其他常用命令

测试冷启动耗时

adb shell am start -W -n PACKAGE/ACTIVITY

测试热启动耗时

adb shell am start -W -n PACKAGE/ACTIVITY

停止：

adb shell am force-stop PACKAGE

随机事件 

adb shell monkey -v -p PACKAGE --pct-touch 100 100

##数据准备

battery-historian工具需要使用bugreport中的Battery History

先断开adb服务，然后开启adb服务
adb kill-server 这一步很重要，因为当我们开发时做电量记录时会打开很多可能造成冲突的东西。为了保险起见我们重启adb。
adb devices就会自动连接查找手机。当然也可以

adb start-server
**重置电池数据收集**
数据，我们在开始的时候需要通过以下命令来打开电池数据的获取以及重置：

```shell
adb shell dumpsys batterystats --enable full-wake-history
adb shell dumpsys batterystats --reset
```

**获取电量报告**

这里要**注意手机版本**，不然后期向Battery Historian导入bugreport.txt文件时会提示“bugreport.txt does not contain a valid bugreport file”

1. 从Android 7.0和更高版本的开发设备中获得bug报告:

   `cd /Users/weixiangyang/Desktop/`

   `adb bugreport bugreport.zip`

2. 从设备6.0和更低版本的开发设备中获得bug报告:

   1. 获取bugreport信息（记录了从开机之后详细的dumpsys,dumpstate和logcat信息）：
      `adb bugreport > 存放的电脑地址/bugreport.txt`
   2. 获取dumpsys信息（获取系统信息：比如内存，CPU，accounts，activities，wifi等信息）
      `adb shell dumpsys batterystats > 存放的电脑地址/batterystats.txt`
      或者获取指定的应用程序的dumpsys信息：
      `adb shell dumpsys batterystats > 包名 > 存放的电脑地址/batterystats.txt`

##分析数据

### 准备分析工具环境

本文使用Google的-battery-historian，本文基于Docker搭建环境

**通过安装Docker环境来安装（这种简单方便）**

1. Docker官方下载地址： Mac：https://docs.docker.com/docker-for-mac/ Windows：https://docs.docker.com/docker-for-mac/
2. 查看是否安装成功： 执行：`docker version`

![img](http://47.110.40.63:8080/img/blog/非架构/docker版本.png)

3. 将docker添加到环境变量

```shell
//1.打开.bash_profile
vi .bash_profile
//2.插入Docker到环境变量
export DOCKER_PATH="/Applications/Docker.app/Contents/Resources/bin"
export PATH=".$PATH:$DOCKER_PATH"
```

4. 通过docker run启动分析页面（<font color="#dd0000">**这一步需要翻墙!!**</font>）

```shell
docker run -P gcr.io/android-battery-historian/stable:3.0 --port 9999
```

5. 上传分析文件进行数据分析

在浏览器打开：http://localhost:9999页面，上传得到的数据文件，进行分析

![img](http://47.110.40.63:8080/img/blog/非架构/电量分析截图.png)

#如何进行电量优化?

了解手机关键耗电的地方及分析耗电的工具后。接下来就是我们的核心，如何来进行电量的优 化?首先我们先简单总结汇总一下耗电的相关因素

- 屏幕亮暗相关
- 设备 awake,sleep 的切换,尤其是唤醒.
- CPU 运行相关
- 网络
- 传感器

我们都知道屏幕的渲染及 CPU 的运行是耗电的主要因素之一。所以当我们在做内存优化、渲染优化、计算优化的时候，就已然在做电量优化。所以在平时的开发中，我们要注意点滴性能 的优化积累，实际上当我们来做电量分析的时候，也是在找自己挖的坑。所以尽量有意识在项 目的开发过程中尽量少挖坑，这一点是我们在分析其他优化项首先要提到的一个点。

# 电量优化

## 监听手机充电状态

我们可以通过下面的代码来获取手机的当前充电状态:

这里我们就需要思考，根据具体的业务，<font color="#dd0000">**考虑将一些不需要及时地和用户交互的操作放到充电的时候去做**</font>。比如：360 手机助手，当充上电的时候，才会自动清理手机垃圾，自动备份上传图片、联系人 等到云端，从而避免当用户手机低电量时，任然继续进行耗电操作。

```java
/**
* 判断当前是否正在充电
*/
private boolean isCharging() {
  IntentFilter filter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
  Intent batteryStatus = this.registerReceiver(null,filter);
  int chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED,-1);
  boolean usbCharge = (chargePlug == BatteryManager.BATTERY_PLUGGED_USB);
  boolean acCharge = (chargePlug == BatteryManager.BATTERY_PLUGGED_AC);
  boolean wirelessCharge = false;
  if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
    wirelessCharge = (chargePlug == BatteryManager.BATTERY_PLUGGED_WIRELESS);
  }
  return (usbCharge || acCharge || wirelessCharge);
}
```

同样地，我们可以判断当前是否为充电器充电(acCharge)，从而选择性的进行一些耗时动作。

## 屏幕唤醒

当 Android 设备空闲时，屏幕会变暗，然后关闭屏幕，最后会停止 CPU 的运行，这样可以防 止电池电量掉的快。但有些时候我们需要改变 Android 系统默认的这种状态:比如玩游戏时我 们需要保持屏幕常亮，比如一些下载操作不需要屏幕常亮但需要 CPU 一直运行直到任务完成。

保持屏幕常亮比较好的方式是在 Activity 中使用 FLAG_KEEP_SCREEN_ON 的 Flag。

```java
getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
```

这个方法的好处是不像唤醒锁(wake locks)，需要一些特定的权限(permission)。并且能 正确管理不同 app 之间的切换，不用担心无用资源的释放问题。另一个方式是在布局文件中使用 android:keepScreenOn 属性：

```java
android:keepScreenOn="true"
```

所以这里我们需要根据自己的 APP 实际情况，根据业务来控制好是否保持屏幕常量。比如 APP 需要支持视频播放。那么在播放的界面需要控制好不熄屏，当退出播放时，当然就没有了 这个设置。

## WakeLock

<font color="#dd0000">**wakelock是一种锁的机制，只要有应用拿着这个锁，CPU就无法进入休眠状态，一直处于工作状态**</font>。比如，手机屏幕在屏幕关闭的时候，有些应用依然可以唤醒屏幕提示用户消息，这里就是用到了wakelock锁机制，虽然手机屏幕关闭了，但是这些应用依然在运行着。手机耗电的问题，大部分是开发人员没有正确使用这个锁，成为"待机杀手"。

Android手机有两个处理器，一个叫Application Processor（AP），一个叫Baseband Processor（BP）。**AP是ARM架构的处理器，用于运行Linux+Android系统；BP用于运行实时操作系统（RTOS），通讯协议栈运行于BP的RTOS之上。非通话时间，BP的能耗基本上在5mA左右，而AP只要处于非休眠状态，能耗至少在50mA以上，执行图形运算时会更高**。另外LCD工作时功耗在100mA左右，WIFI也在100mA左右。一般手机待机时，AP、LCD、WIFI均进入休眠状态，这时Android中应用程序的代码也会停止执行。

**Android为了确保应用程序中关键代码的正确执行，提供了Wake Lock的API，使得应用程序有权限通过代码阻止AP进入休眠状态**。但如果不领会Android设计者的意图而滥用Wake Lock API，为了自身程序在后台的正常工作而长时间阻止AP进入休眠状态，就会成为待机电池杀手。

**WakeLock的使用**

```java
PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE); 
WakeLock wakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, "MyWakelockTag");
```

| 标记值                  | CPU  | 屏幕 | 键盘 |
| :---------------------- | :--- | :--- | :--- |
| PARTIAL_WAKE_LOCK       | 开启 | 关闭 | 关闭 |
| SCREEN_DIM_WAKE_LOCK    | 开启 | 变暗 | 关闭 |
| SCREEN_BRIGHT_WAKE_LOCK | 开启 | 变亮 | 关闭 |
| FULL_WAKE_LOCK          | 开启 | 变亮 | 变亮 |

自API等级17开始，FULL_WAKE_LOCK将被弃用。应用应使用**FLAG_KEEP_SCREEN_ON**。

WakeLock类可以用来控制设备的工作状态。使用该类中的acquire可以使CPU一直处于工作的状态，如果不需要使CPU处于工作状态就调用release来关闭。

(1)、自动release

如果我们调用的是acquire(long timeout)那么就无需我们自己手动调用release()来释放锁，系统会帮助我们在timeout时间后释放。

(2)、手动release

如果我们调用的是acquire()那么就需要我们自己手动调用release()来释放锁。

最后使用WakeLock类记得加上如下权限：

```java
<uses-permission android:name="android.permission.WAKE_LOCK" />   
```

注意:自 API 等级 17 开始，FULL_WAKE_LOCK 将被弃用。 应用应使用 FLAG_KEEP_SCREEN_ON。

注意:<font color="#dd0000">**在使用该类的时候，必须保证 acquire 和 release 是成对出现的**</font>。不然当我们业务已经不需要时， 当 CPU 处于唤醒状态，这个时候就会损耗多余的电量。

## JobScheduler

自 Android 5.0 发布以来，JobScheduler已成为执行后台工作的很好的方式，其工作方式有利于用户在适当的时机执行正确的事情。应用可以在安排作业的同时允许系统基于内存、电源和连接情况进行优化。**JobSchedule 的宗旨就是把一些不是特别紧急的任务放到更合适的时机批量处理**。这样做有两个好处:

- **避免频繁的唤醒硬件模块，造成不必要的电量消耗**。
- **避免在不合适的时间(例如低电量情况下、弱网络或者移动网络情况下的)执行过多的任务消耗电量**。

## GPS

**选择合适的 Location Provider**

Android 系统支持多个 Location Provider:

- **GPS_PROVIDER:** GPS 定位，利用 GPS 芯片通过卫星获得自己的位置信息。定位精准度高，一般在 10 米左右， 耗电量大;但是在室内，GPS 定位基本没用。
- **NETWORK_PROVIDER:** 网络定位，利用手机基站和 WIFI 节点的地址来大致定位位置，这种定位方式取决于服务器，即取决于将基站或 WIF 节点信息翻译成位置信息的服务器的能力。
- **PASSIVE_PROVIDER:** <font color="#dd0000">**被动定位，就是用现成的，当其他应用使用定位更新了定位信息，系统会保存下来，该应用接 收到消息后直接读取就可以了**</font>。

**及时注销定位监听**

在获取到定位之后或者程序处于后台时，注销定位监听，此时监听 GPS 传感器相当于执行 no- op(无操作指令)，用户不会有感知但是却耗电。

**多模块使用定位尽量复**

多个模块使用定位，尽量复用上一次的结果，而不是都重新走定位的过程，节省电量损耗;例 如:在应用启动的时候获取一次定位，保存结果，之后再用到定位的地方都直接去取。

##传感器

**使用传感器，选择合适的采样率，越高的采样率类型则越费电**。

- SENSOR_DELAY_NOMAL (200000 微秒)
- SENSOR_DELAY_UI (60000 微秒)
- SENSOR_DELAY_GAME (20000 微秒)
- SENSOR_DELAY_FASTEST (0 微秒)







参考：https://www.jianshu.com/p/c86021fe958d

https://blog.csdn.net/wdx_1136346879/article/details/86493355