---
title: Android『节点-性能优化』
tags:
---

# 内存优化

## 内存检测

### ADB 命令

1. 当前应用使用内存分析

```shell
adb shell dumpsys meminfo 包名
```

![img](http://47.110.40.63:8080/img/blog/非架构/adb-meminfo命令.jpg)

`Pss`: 该进程独占的内存+与其他进程共享的内存（按比例分配，比如与其他3个进程共享9K内存，则这部分为3K）

`Privete Dirty`:该进程独享内存

`Heap Size`:分配的内存

`Heap Alloc`:已使用的内存

`Heap Free`:空闲内存

# 包体积优化



# 启动时间优化



# 稳定性优化



