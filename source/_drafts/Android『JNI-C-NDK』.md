---
title: Android『JNI/C++/NDK』
tags: [Android,jni,ndk]
---

### JNI

#### 加载方式

so文件是不能随便放到一个指定的目录然后再通过参数pathName直接引用的。因为System.load只能加载两个目录路径下的so文件： /system/lib ; 安装包的路径，即：/data/data//… **而且这两个路劲又是有权限保护的不能直接访问;**

System.load(filename)：filename必须是绝对路径

1. 静态加载

   

2. 动态加载



参考文章：

[Android——JNI加载so两种方式](<https://blog.csdn.net/iliupp/article/details/70765891?utm_source=blogxgwz8>)

[System.load 与 System.loadLibrary 的区别](https://www.cnblogs.com/butterfly-clover/p/5684301.html)

#### Android中生成SO文件



参考文章：

[学习JNI编程第1篇--写一个FactorialDemo APP](<https://www.jianshu.com/p/7b57a19448c6>)

