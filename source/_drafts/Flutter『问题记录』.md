---
title: Flutter『问题记录』
date: 2019-11-14 11:14:08
tags:
---



### Tips

1. AS中clean Flutter项目

   在命令行中输入：

   ```java
   flutter clean
   ```

2. 



### 问题记录

1. Gradle Build error: versionCode not found. Define flutter.versionCode in the local.properties file.

   解决：在Android目录的local.properties文件中添加flutter的版本相关信息

   ```dart
   flutter.buildMode=debug
   flutter.versionName=1.0.0
   flutter.versionCode=1
   ```

2. AAPT: error: resource android:attr/fontVariationSettings not found after Flutter Upgrade.

   解决：设置compileSdkVersion到对应的版本

