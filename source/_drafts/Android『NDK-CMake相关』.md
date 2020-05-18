---
title: Android『NDK_CMake相关』
tags:
---

###概述

摘自：https://mp.weixin.qq.com/s/1NJZ17rmgFeQ6c3TPLIN_g

CMake 是一个跨平台构建系统，在 Android Studio 引入 CMake 之前，它就已经被广泛运用了。

总结官网对 CMake 的使用，其实也就如下的步骤：

1. add_library 指定要编译的库，并将**所有**的 `.c` 或 `.cpp` 文件包含指定。
2. include_directories 将**头文件**添加到搜索路径中
3. set_target_properties 设置库的一些属性
4. target_link_libraries **将库与其他库相关联**

在 cpp 的同一目录下创建 `CMakeLists.txt` 文件，内容如下：

```
# 指定 CMake 使用版本
cmake_minimum_required(VERSION 3.9)
# 工程名
project(HelloCMake)
# 编译可执行文件
add_executable(HelloCMake main.cpp )
```

其中，通过 `cmake_minimum_required` 方法指定 CMake 使用版本，通过 `project` 指定工程名。

而 `add_executable` 就是指定**最后编译的可执行文件名称和需要编译的 cpp 文件**，如果工程很大，有多个 cpp 文件，那么都要把它们添加进来(**使用空格分隔**)。



###命令解释

**add_library**

摘自：https://blog.csdn.net/sjt19910311/article/details/51660209

````shell
add_library(<name> [STATIC | SHARED | MODULE]
[EXCLUDE_FROM_ALL]
source1 source2 … sourceN)
````

添加一个名为< name >的库文件，该库文件将会根据调用的命令里列出的源文件来创建。< name >对应于**逻辑目标名称**，`而且在一个工程的全局域内必须是唯一的`。待构建的库文件的实际文件名根据对应平台的命名约定来构造（比如lib< name >.a或者< name >.lib）。STATIC库是目标文件的`归档文件`，**在链接其它目标的时候使用**。SHARED库会被`动态链接`，在运行时被加载。`MODULE库是不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数动态链接`。如果没有类型被显式指定，这个选项将会根据变量BUILD_SHARED_LIBS的当前值是否为真决定是STATIC还是SHARED。





### 实践

摘自：https://blog.csdn.net/quwei3930921/article/details/78820991

Demo地址：