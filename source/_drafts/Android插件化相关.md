---
title: Android插件化相关
tags:
---

## 概述

[Android 插件化：从入门到放弃](https://www.infoq.cn/article/android-plug-ins-from-entry-to-give-up/)

[《Android插件化技术——原理篇》](https://mp.weixin.qq.com/s/Uwr6Rimc7Gpnq4wMFZSAag?utm_source=androidweekly&utm_medium=website)



## 开源项目

[Small](https://github.com/wequick/Small)



## 实现细则



###发展历程

![img](http://47.110.40.63:8080/img/blog/Android插件化发展进程.png)

**第一代：**dynamic-load-apk最早使用ProxyActivity这种静态代理技术，由ProxyActivity去控制插件中PluginActivity的生命周期。该种方式缺点明显，插件中的activity必须继承PluginActivity，开发时要小心处理context。而DroidPlugin通过Hook系统服务的方式启动插件中的Activity，使得开发插件的过程和开发普通的app没有什么区别，但是由于hook过多系统服务，异常复杂且不够稳定。

**第二代：**为了同时达到插件开发的低侵入性（像开发普通app一样开发插件）和框架的稳定性，在实现原理上都是趋近于选择尽量少的hook，并通过在manifest中预埋一些组件实现对四大组件的插件化。另外各个框架根据其设计思想都做了不同程度的扩展，其中Small更是做成了一个跨平台，组件化的开发框架。

**第三代：**VirtualApp比较厉害，能够完全模拟app的运行环境，能够实现app的免安装运行和双开技术。Atlas是阿里今年开源出来的一个结合组件化和热修复技术的一个app基础框架，其广泛的应用与阿里系的各个app，其号称是一个容器化框架。

**实现插件化需要解决的问题**

- 插件中**代码**的加载和与主工程的**互相调用**
- 插件中**资源的加载**和与主工程的**互相访问**
- 四大组件生命周期的管理

### 类加载器

Android中常用的有两种类加载器，DexClassLoader和PathClassLoader，它们都继承于BaseDexClassLoader。

```java
// DexClassLoader
public class DexClassLoader extends BaseDexClassLoader {
    //第一个参数为apk的文件目录；第二个参数为内部存储目录；第三个为库文件的存储目录；第四个参数为父加载器
		public DexClassLoader(String dexPath, String optimizedDirectory,
        String libraryPath, ClassLoader parent) {
 		    super(dexPath, new File(optimizedDirectory), libraryPath, parent);
    }
}

// PathClassLoader
public class PathClassLoader extends BaseDexClassLoader {
    public PathClassLoader(String dexPath, ClassLoader parent) {
        super(dexPath, null, null, parent);
    }
    public PathClassLoader(String dexPath, String libraryPath,ClassLoader parent) {   
        super(dexPath, null, libraryPath, parent);
    }
}
```

Android中常用的有两种类加载器，DexClassLoader和PathClassLoader，它们都继承BaseDexClassLoader。

区别在于调用父类构造器时，DexClassLoader多传了一个optimizedDirectory参数，这个目录必须是内部存储路径，**用来缓存系统创建的Dex文件**。而PathClassLoader该参数为null，只能加载内部存储目录的Dex文件。

#### 双亲委托机制

```java
protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException { 
    //首先从已经加载的类中查找
    Class<?> clazz = findLoadedClass(className);    
    if (clazz == null) {
        ClassNotFoundException suppressed = null;     
        try {   
            //如果没有加载过，先调用父加载器的loadClass
            clazz = parent.loadClass(className, false);
        } catch (ClassNotFoundException e) {
            suppressed = e;
        }      
        if (clazz == null) {        
            try {           
                //父加载器都没有加载，则尝试加载
                clazz = findClass(className);
            } catch (ClassNotFoundException e) {
                e.addSuppressed(suppressed);       
                throw e;
            }
        }
    }    
    return clazz;
}
```

可以看出ClassLoader加载类时，先查看自身是否已经加载过该类，如果没有加载过会首先让父加载器去加载，如果父加载器无法加载该类时才会调用自身的<font color="#dd0000">**findClass**</font>方法加载，该机制很大程度上避免了类的重复加载。





参考：

[《Android插件化技术——原理篇》](https://mp.weixin.qq.com/s/Uwr6Rimc7Gpnq4wMFZSAag?utm_source=androidweekly&utm_medium=website)

