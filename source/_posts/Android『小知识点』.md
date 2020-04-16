---
title: Android『小知识点』
date: 2019-04-09 19:41:37
tags: Android
---

### API

##### HanderThread

一个包含一个looper的**线程**，这个looper可以用来创建hanlder，需要注意start()方法还是需要调用才能使用。

HandlerThread有**quit()和quitSafely()**方法用于退出HandlerThread对应的Looper，其实应该是直接调用了Looper.quit()和Looper.quitSafely()，**后者自API 18**添加。除此之外，还有其他方法：

**getLooper()：该方法是同步的，如果线程已经启动了，那么这个方法会在looper初始化完成之前保持阻塞。**

##### SharePreferences

提交修改可以使用apply()和commit()，两者是不同的，具体：

1. commit是同步的，会返回一个执行成功与否的boolean结果；apply是异步的，提交以后直接改变内容中的值，然后返回，将入数据写入磁盘的动作放到异步线程里进行，不返回成功与否的boolean结果。

2. 如果你不在乎数据写入的结果，而且又位于主线程中，那么apply比较合适，因为比较快。反之，如果存在则使用commit比较合适。

##### IntentService

IntentService是一个用来处理异步请求的的类，通过startService(Intent)启动，所有的Intent会在一个**工作线程按顺序处理**。工作队列处理器模式一般用来处理从application的主线程卸下的任务。IntentService的存在只是为了简化工作，IntentService是抽象类，集成它并实现onHandleIntent方法，IntentService接收到Intent，并启动一个线程。**所有的请求Intent都会被一个工作线程处理，会处理直到所有任务都完成，注意每次只能处理一个请求。**

##### FragmentTranscation

用于保证一些列Fragment操作的原子性。

1. transaction.add() //往Activity中添加一个Fragment
2. transaction.remove() //从Activity中移除一个Fragment
3. transaction.replace() //先remove在add的组合操作
4. transaction.hide() //隐藏当前Fragment，仅仅设为不可见，并不会销毁
5. transaction.show() //显示之前隐藏的Fragment

##### AsyncQueryHandler

AsyncQueryHandler是用于在ContentProvider上面执行<font color="#dd0000">**异步的CRUD**</font>操作的工具类，CRUD操作通常会被放到一个**单独的子线程**中执行，当操作结束获取结果后，将通过消息的方式传递给调用AsyncQueryHandler的线程，通常就是主线程。AsyncQueryHandler是一个抽象类，继承自Handler。通过封装ContentResolver、HandlerThread、Handler等实现对ContentProvider的异步操作。

##### JobService

在Android 5.0中引入**JobScheduler**来执行一些**需要满足特定条件但不紧急的后台任务**，当一系列预置的条件被满足时，JobScheduler API为你的应用执行一个操作，jobScheduler API允许同时执行多个任务。这允许你的应用执行某些指定的任务时不需要考虑时机控制引起的电池消耗。<font color="#dd0000">**JobScheduler是一个系统级的作业调度器，我们将某些任务扔给系统，当达到我们设定的条件以后，JobScheduler再吊起我们的JobService执行我们的业务逻辑。**</font>

##### ComponentCallbacks和ComponentCallbacks2

ComponentCallbacks定义了两个方法onConfigurationChanged()和onLowMemory()；**ComponentCallbacks2是API14新增的类，添加了onTrimMemory()**，<font color="#dd0000">**当系统决定对某个进程进行内存优化时会回调到。**</font>

```java
public interface ComponentCallbacks2 extends ComponentCallbacks {

    /**
     * 系统正处于低内存状态，app进程是首先被杀进程之一，如果系统现在没有恢复内存，应该立即释放对app不重要的		 * 所有内容
     */
    static final int TRIM_MEMORY_COMPLETE = 80;
    
    /**
     * 系统正处于低内存状态，app进程位于LRU List中部附近，如果系统进一步内存紧张，app进程可能会被杀掉
     */
    static final int TRIM_MEMORY_MODERATE = 60;
    
    /**
     * 系统正处于低内存状态，app进程位于LRU List开始附近，尽管app进程被杀死的概率低，系统可能已经开始杀在			 * LRU List中的后台进程,此时应该释放一些资源，防止被杀的几率
     */
    static final int TRIM_MEMORY_BACKGROUND = 40;
    
    /**
     * 你的用户界面不再可见，此时应该释放仅仅使用在UI上的大资源
     */
    static final int TRIM_MEMORY_UI_HIDDEN = 20;
    /**
     * 系统处于极低的内存状态，app仍然不会被杀死。但如果不是放资源，系统开始杀后台进程，此时app应该清理一些		 * 资源
     */
    static final int TRIM_MEMORY_RUNNING_CRITICAL = 15;

    /**
     * 系统处于更低内存状态，app正在运行且不会被杀死，可以释放不用的资源提升app性能
     */
    static final int TRIM_MEMORY_RUNNING_LOW = 10;

    /**
     * 系统处于低内存状态，app正在运行且不会被杀死
     */
    static final int TRIM_MEMORY_RUNNING_MODERATE = 5;

    /**
     * 当系统决定对一个进程的内存进行优化的时候调用会被调用。举个例子：在后台资源不够运行所需的进程时。你不应该将具体的数值和level做比较，因为后期可能会增加新的数值。你可以使用大于或等于某个你感兴趣的level。
     *
     * 你可以通过ActivityManager.getMyMemoryState(RunningAppProcessInfo)获取到当前进程的内存优			 * 化等级。
     * @param level 系统希望执行的内存清理等级
     */
    void onTrimMemory(int level);
}
```

本段参考：<https://www.jianshu.com/p/1bf70b535eb9>

SP标记multi-process用于标记SP可以在多进程中访问，但是多进程访问SP会带来新问题：

[Use SharedPreferences on multi-process mode](https://stackoverflow.com/questions/27827678/use-sharedpreferences-on-multi-process-mode)

##### hasMessage(int what)

Check if there are any pending posts of messages with code 'what' in the message queue.

### Method

##### 获取当前进程名的方法一

在Linux系统中，根据进程号得到进程的命令行参数，常规的做法是读取**/proc/{PID}/cmdline**，如下：

```java
public static String getProcessName() {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader("/proc/" + Process.myPid() + "/cmdline"));
            String processName = reader.readLine();
            if (!TextUtils.isEmpty(processName)) {
                processName = processName.trim();
            }
            String var2 = processName;
            return var2;
        } catch (Exception var12) {
            var12.printStackTrace();
        } finally {
            try {
                if (reader != null) {
                    reader.close();
                }
            } catch (IOException var11) {
                var11.printStackTrace();
            }
        }
        return null;
    }
```

### Other

**查看Activity堆栈情况**，会到的很长的输出内容，检索**Running activities**可以查看最近执行的Activity情况。

##### AAR相关

###### AAR 与 Jar

jar：**只包含了class文件与清单文件 ，不包含资源文件，如图片等所有res中的文件。**

aar： **Android库项目的二进制归档文件，包含所有资源，class以及res资源文件全部包含。**

**aar问题：**

<font color="#dd0000">**其实AAR只是把编译好的class文件和资源文件打包成一个二进制的包**</font>，打包后会<font color="#dd0000">**丢失依赖**</font>，所以引用AAR时，不存在重复依赖的问题，需要同时引入AAR包中用到的类文件才能正确编译。

1. 资源文件不能同名,大量改动 
2. **Application需要主APP传递给AAR，AAR再做单例模式 **
3. 扫描模块需要AAR自己初始化 
4. jar尽量用maven，如果是本地的libs，需要exlude so和provided jar 
5. AAR可以通过主APP的application初始化自己的数据库 
6. 轻量数据，可以用intent启动AAR的时候传递 
7. 如果有基础数据需要提供服务，可以用contentProvider 
8. butterknife需要统一用8.4.0以上的版本，不然报错 
9. AAR模块用butterknife需要把资源R改为R2(批量替换)

##### AndroidManifest.xml

+ android:exported = true

  在Activity中该属性用来标示：**当前Activity是否可以被另一个Application的组件启动：true允许被启动；false不允许被启动。**android:exported 是Android中的四大组件 Activity，Service，Provider，Receiver 四大组件中都会有的一个属性。总体来说它的主要作用是：是否支持其它应用调用当前组件

+ android:enabled="true"

  这个属性用于指示该服务是否能够被实例化。如果设置为true，则能够被实例化，否则不能被实例化。默认值是true。	

+ android:permission

  这个属性定义了要启动或绑定服务的实体必须要有的权限。**如果调用者的startService()、bindService()和stopService()方法没有被授予这个权限，那么这些方法就不会工作，并且Intent对象也不会发送给改服务。**如果这个属性没被设置，**那么通过\<appliction\>元素的permission属性所设定的权限就会适用于该服务。**如果\<application\>元素也没有设置权限，则该服务不受权限保护。

##### 数据压缩

###### GZIP

GZIP是网站压缩加速的一种技术，对于开启后可以加快我们网站的打开速度，原理是经过服务器压缩，客户端浏览器快速解压的原理，可以大大减少了网站的流量。**一般对纯文本内容可压缩到原大小的40％**。



### 调试

1. 模拟器访问本地Tomcat地址为：<font color="#dd0000">**10.0.2.2**</font>

##### AndroidJUnit4单元测试

###### 获取Context

```java
InstrumentationRegistry.getInstrumentation().getTargetContext()
```

##### ADB相关

###### adb install-multiple 命令

```java
adb install [-lrtsdg] (file)
-l 锁定应用
-r 替换已存在的应用
-t 允许安装测试包
-s 安装到sd卡中
-d 可以安装低版本安装包
-p 安装部分应用
-g 授权所有运行时权限
```

###### adb uninstall [-k] (package)

卸载安装包

###### adb shell dumpsys activity activities



### 兼容性API

##### UI

+ 动态设置按钮点击效果

  ```kotlin
  val wrappedDrawable = DrawableCompat.wrap(drawable)
  DrawableCompat.setTintList(wrappedDrawable, ColorStateList.valueOf(colors))
  setImageDrawable(wrappedDrawable)
  ```

  

+ getColor

  ```java
  ContextCompat.getColor(context, R.color.text_black)
  ```

  

### 性能优化

##### Google Protocol Buffer

一般项目中多使用JSON作为数据传递和解析的格式，但实际上有其他的的传输格式比JSON有更优秀的效率、体积上的表现，比如Google出品的ProtocolBuffer，比起JSON来说具有更小、更快、使用更简单等特点，具体介绍可以查看这篇博客：[快来看看Google出品的Protocol Buffer，别只会用Json和XML了](https://www.cnblogs.com/xinmengwuheng/p/7070393.html)



