---
title: JetPack『LiveData』
tags:
---

#### LiveData

https://blog.csdn.net/gdutxiaoxu/article/details/86660760

LiveData是一个可以被观察的数据容器类，它可以感知Activity、Fragment或Service等组件的生命周期。

1. 它可以做到在组件处于激活状态的时候才会回调相应的方法，从而刷新相应的 UI。

2. 不用担心发生内存泄漏。

3. 当 config 导致 activity 重新创建的时候，不需要手动取处理数据的储存和恢复。它已经帮我们封装好了。

4. 当 Actiivty 不是处于激活状态的时候，如果你想 livedata setValue 之后立即回调 obsever 的 onChange 方法，而不是等到 Activity 处于激活状态的时候才回调 obsever 的 onChange 方法，你可以使用 **observeForever**方法，但是你必须在 onDestroy 的时候 removeObserver。

LiveData的使用

1. 自定义ViewModel

   ```java
   public class TestViewModel extends ViewModel {
       private MutableLiveData<String> mNameEvent = new MutableLiveData<>();
       public MutableLiveData<String> getNameEvent() {
           return mNameEvent;
       }
   }
   ```

2. 在Activity中监听mNameEvent的改变

   ```java
   mTestViewModel = ViewModelProviders.of(this).get(TestViewModel.class);
   MutableLiveData<String> nameEvent = mTestViewModel.getNameEvent();
   nameEvent.observe(this, new Observer<String>() {
       @Override
       public void onChanged(@Nullable String s) {
           Log.i(TAG, "onChanged: s = " + s);
           mTvName.setText(s);
       }
   });
   ```

3. 最后当我们数据源改变的时候，我们需要调用LiveData的 setValue或者postValue方法。他们之间的区别是，<font color="#dd0000">**调用 setValue 方法，Observer 的 onChanged 方法会在调用 serValue 方法的线程回调。而postvalue 方法，Observer 的 onChanged 方法将会在主线程回调**</font>。

   ```java
   mTestViewModel.getNameEvent().setValue(name);
   ```

4. LiveData中常用的方法

   + void observe (LifecycleOwner owner, Observer observer)

     Adds the given observer to the observers list within the lifespan of the given owner. The events are dispatched on the main thread. If LiveData already has data set, it will be delivered to the observer.

   + void onActive ()

     当回调该方法的时候，表示该 liveData 正在背使用，因此应该保持最新

   + void onInactive ()

     当该方法回调时，表示他所有的 obervers 没有一个状态处理 STARTED 或者 RESUMED，注意，这不代表没有 observers。

   + Void observeForever

     跟 observe 方法不太一样的是，它在 Activity 处于 onPause ，onStop， onDestroy 的时候，都可以回调 obsever 的 onChange 方法，但是有一点需要注意的是，我们必须手动 remove obsever，否则会发生内存泄漏。

本节参考：[Android架构组件之LiveData](<https://www.jianshu.com/p/6e7e05a8b750>)

`LiveData`采用了观察者模式设计，其中`LiveData`是被观察者，当数据发生变化时会通知观察者进行数据更新。通过这点，可以确保数据和界面的实时性。这是因为`LiveData`能够感知到组件的生命周期，当组件状态处于`DESTROYED`状态时，观察者对象会被`remove`。这是因为组件处于非激活状态时，在界面不会收到来自`LiveData`的数据变化通知。这样规避了很多因为页面销毁之后，修改UI导致的`crash`。由于`LiveData`保存数据的时候，组件和数据是分离的，所以在配置更改（如横竖屏切换等）的时候，即便组件被重新创建，因为数据还保存在`LiveData`中，这样也能够做到实时的更新。

单例模式扩展`LiveData`对象并包装成系统服务，以便在应用程序中进行共享，需要该资源的只需要观察`LiveData`即可。

#### 