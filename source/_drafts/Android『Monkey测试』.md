---
title: Android『Monkey测试』
tags:
---

测试冷启动耗时

adb shell am start -W -n com.snda.wifilocating/com.lantern.launcher.ui.MainActivity



停止：

adb shell am force-stop com.snda.wifilocating



热启动

adb shell am start -W -n com.snda.wifilocating/com.lantern.launcher.ui.MainActivity