---
title: Flutter网摘及笔记
tags:
---

## Flutter如何解决版本冲突

摘自：https://www.jianshu.com/p/3af57fbb7efe

1. 先使用any使编译通过，any会自动调用pub的版本分析器，寻找合适的能够避免冲突的依赖版本并下载。Flutter会生成pubspec.lock文件。
2. 打开pubspec.lock文件，查看生成的版本，然后将版本替换掉any即可。



