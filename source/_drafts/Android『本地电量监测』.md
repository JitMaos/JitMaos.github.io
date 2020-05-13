---
title: Android『本地电量监测』
tags:
---

##概述

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

#### 获取电量报告

（这里要注意手机版本，不然后期向Battery Historian导入bugreport.txt文件时会提示“bugreport.txt does not contain a valid bugreport file”，这里也算是一个坑吧）

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

















参考：https://www.jianshu.com/p/c86021fe958d

https://blog.csdn.net/wdx_1136346879/article/details/86493355