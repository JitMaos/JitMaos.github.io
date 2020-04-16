---
title: MVVM『基础』
tags: [android,DataBinding,Room,MVVM,Dagger2]
---

### DataBinding

#### 基于XML的绑定



### Room

#### 常用注解

#### 初始化

#### 使用



### MVVM

#### LiveData

本节参考：[Android架构组件之LiveData](<https://www.jianshu.com/p/6e7e05a8b750>)

`LiveData`采用了观察者模式设计，其中`LiveData`是被观察者，当数据发生变化时会通知观察者进行数据更新。通过这点，可以确保数据和界面的实时性。这是因为`LiveData`能够感知到组件的生命周期，当组件状态处于`DESTROYED`状态时，观察者对象会被`remove`。这是因为组件处于非激活状态时，在界面不会收到来自`LiveData`的数据变化通知。这样规避了很多因为页面销毁之后，修改UI导致的`crash`。由于`LiveData`保存数据的时候，组件和数据是分离的，所以在配置更改（如横竖屏切换等）的时候，即便组件被重新创建，因为数据还保存在`LiveData`中，这样也能够做到实时的更新。

单例模式扩展`LiveData`对象并包装成系统服务，以便在应用程序中进行共享，需要该资源的只需要观察`LiveData`即可。

#### BoundaryCallBack

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



