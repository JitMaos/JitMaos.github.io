---
title: 项目架构——JetPack『Paging使用』
tags: [Android,JetPack,Paging]
---

本文结合Paging和Room实现一个简单的Demo，演示这两个库的用法。

### Paging

Paging是Google推出的的JetPack框架中的一员，用于处理列表的分页加载。

Paging主要包含三部分，分别是DataSource、PageList、PageListAdapter。

#### DataSource

表示数据源，通常是这样的DataSource<Key,Value>，其中Key表示**加载数据的条件信息**，Value表示**加载的数据实体**。<font color="#dd0000">**DataSource是一个抽象类**</font>，可以使用它的子类：

+ `PageKeyedDataSource<Key, Value>` ：适用于目标数据根据**页信息**请求数据的场景，即`Key` 字段是**页相关的信息**。比如请求的数据的参数中包含类似`next/previous`页数的信息。
+ `ItemKeyedDataSource<Key, Value>` ：适用于目标数据的**加载依赖特定item**的信息， 即Key字段包含的是Item中的信息，**比如需要根据第N项的信息加载第N+1项的数据，传参中需要传入第N项的ID时，该场景多出现于论坛类应用评论信息的请求**。
+ `PositionalDataSource<T>`：适用于目标数据总数固定，通过特定的位置加载数据，这里Key是Integer类型的位置信息，T即Value。 比如从数据库中的1200条开始加在20条数据。

<!--more-->

#### PageList

PageList是一个List的子类，支持所有List的操作，除此之外它主要有五个成员：

+ mMainThreadExecutor: 一个主线程的Excutor, 用于**将结果post到主线程**。

+ mBackgroundThreadExecutor: 后台线程的Excutor.

+ BoundaryCallback:加载DataSource中的数据加载到边界时的**回调**.
+ PagedStorage\<T\>: 用于存储加载到的数据，**它是真正的蓄水池所在**，它包含一个ArrayList<List> 对象mPages，按页存储数据。
+ Config: 配置PagedList从DataSource加载数据的方式， 其中包含以下属性：
  + pageSize：设置每页加载的数量
  + **prefetch**Distance：预加载的数量，默认为<font color="#dd0000">**pagesize**</font>
  + initialLoadSizeHint：初始化数据时加载的数量，默认为<font color="#dd0000">**pageSize*3**</font>
  + enablePlaceholders：当item为null是否使用PlaceHolder展示

PagedList会从DataSource中加载数据，更准确的说是通过DataSource加载数据， 通过Config的配置，可以设置一次加载的数量以及预加载的数量。 除此之外，PagedList还可以向RecyclerView.Adapter发送更新的信号，驱动UI的刷新。

#### PagedListAdapter

PagedListAdapte是RecyclerView.Adapter的实现，用于展示PagedList的数据。它本身实现的更多是Adapter的功能，但是它有一个小伙伴**PagedListAdapterHelper\<T\>**， PagedListAdapterHelper会负责监听PagedList的更新， Item数量的统计等功能。这样当PagedList中新一页的数据加载完成时， PagedAdapter就会发出加载完成的信号，通知RecyclerView刷新，这样就省略了每次loading后手动调一次`notifyDataChanged()`.

除此之外，当数据源变动产生新的PagedList,PagedAdapter会在后台线程中比较前后两个PagedList的差异，然后调用notifyItem…()方法更新RecyclerView.这一过程依赖它的另一个小伙伴`ListAdapterConfig`， ListAdapterConfig负责主线程和后台线程的调度以及`DiffCallback`的管理，<font color="#dd0000">**DiffCallback的接口实现中定义比较的规则，比较的工作则是由PagedStorageDiffHelper来完成。**</font>



#### Room

Room据说是一个工程师妹子写的Android数据库操作开源库。基于注解，结合LiveData可以实现数据源的统一管理。



#### 问题

1. Unfortunately, you can't have both **realtime updates and paging** at the same time. You have to choose one or the other. This is a limitation of the paging library, which needs to manage pages of results internally.

   不能同时使用Paging并及时获得数据库数据的及时更新。

2. 









参考：<https://www.loongwind.com/archives/367.html>



