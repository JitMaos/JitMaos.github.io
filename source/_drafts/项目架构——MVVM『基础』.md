---
title: MVVM『基础』
tags: [android,DataBinding,Room,MVVM,Dagger2]
---

### MVVM

摘自：https://www.jianshu.com/p/35d143e84d42

MVVM由M(Model)、V(View)、VM（ViewModel)三部分组成，其中V指的是Activity/Fragment；M指的是数据对象，比如各种Bean；VM 则是负责在V和M之间传递消息数据的对象，其中定义了操作M的方法，由V来调用。通常与ViewModel配合使用的是LiveData。



MVVM有什么特性

1. 数据持久化

   ViewModel的生命周期从Activity的onCreate到onDestory，这意味着在屏幕旋转时，执行activity的销毁与重建时，可以使用ViewModel保存数据，来避免数据的释放与重建。

2. 异步回调问题

   ViewModel配合LiveData使用，可以十分方便的实现**异步数据**的回调处理过程。

   

   

   

   

   

   BoundaryCallBack

```html
Note that a BoundaryCallback instance shared across multiple PagedLists (e.g. when passed to {@link LivePagedListBuilder#setBoundaryCallback}), the callbacks may be issued multiple times. If for example {@link #onItemAtEndLoaded(Object)} triggers a network load, it should avoid triggering it again while the load is ongoing.
```

#### DataSource

DataSource is queried to load pages of content into a {@link PagedList}. A PagedList can grow as it loads more data, <font color="#dd0000">**but the data loaded cannot be updated**</font>. If the underlying data set is modified, a new PagedList / DataSource pair must be created to represent the new data.

#### ViewModel



#### Dagger2

##### 注解

1. @Module(**includes** = [xxxModule::class])

   includes等同于modules={AModule::class,BModule::class}，**但是includes可以进行多层嵌套**。

2. 



