---
title: 源码学习『kotlin_wanandroid』
tags: [源码学习,Kotlin,MVVM]
---

### 项目介绍

看一个别人的开源项目kotlin_wanandroid，项目规模较小，主要想学习一下MVVM模式的实际应用和Kotlin的实际应用。

项目介绍地址：[推荐一个Kotlin项目](<https://www.jianshu.com/p/a05b10976b08>)

项目地址：[haoshiy/kotlin_wanandroid](<https://github.com/haoshiy/kotlin_wanandroid>)

### 学习总结

#### Kotlin实践记录

1. let扩展函数的实际上是一个**作用域函数**，当你需要去定义一个变量在一个特定的作用域范围内，let函数的是一个不错的选择；let函数另一个作用就是可以避免写一些判断null的操作。

   ```kotlin
   null?.let { Log.e("TEST","Print:" + null) }
   //不会输出任何值
   ```

2. 扩展函数

   ```java
   package com.hao.easy.base.extensions
   
   import com.hao.easy.base.model.HttpResult
   import com.socks.library.KLog
   import io.reactivex.Observable
   import io.reactivex.android.schedulers.AndroidSchedulers
   import io.reactivex.schedulers.Schedulers
   
   fun <D, T : HttpResult<D>> Observable<T>.main() =
         	observeOn(AndroidSchedulers.mainThread())
   
   fun <D, T : HttpResult<D>> Observable<T>.io_main() =
           subscribeOn(Schedulers.io())
                   .observeOn(AndroidSchedulers.mainThread())
   
   fun <D, T : HttpResult<D>> Observable<T>.subscribeBy(onResponse: (D?) -> Unit) =
           this.subscribe({
               if (it.errorCode == 0) {
                   onResponse(it.data)
               }
           }, {
               KLog.d("onFailure", it)
           })!!
   
   fun <D, T : HttpResult<D>> Observable<T>.subscribeBy(onResponse: (D?) -> Unit, onFailure: (String) -> Unit) =
           subscribe({
               if (it.errorCode == 0) {
                   onResponse(it.data)
               } else {
                   onFailure(it.errorMsg)
               }
           }, {
               onFailure(it.message!!)
           })
   
   ```


3. 委托

   类的委托即一个类中定义的方法实际是调用另一个类的对象的方法来实现的。

   ```kotlin
   import kotlin.reflect.KProperty
   // 定义包含属性委托的类
   class Example {
       var p: String by Delegate()
   }
   
   // 委托的类
   class Delegate {
       operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
           return "$thisRef, 这里委托了 ${property.name} 属性"
       }
   
       operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
           println("$thisRef 的 ${property.name} 属性赋值为 $value")
       }
   }
   fun main(args: Array<String>) {
       val e = Example()
       println(e.p)     // 访问该属性，调用 getValue() 函数
   
       e.p = "Runoob"   // 调用 setValue() 函数
       println(e.p)
   }
   ```

4. 延迟属性Lazy

   lazy() 是一个函数, 接受一个 Lambda 表达式作为参数, 返回一个 Lazy <T> 实例的函数，返回的实例可以作为实现延迟属性的委托： **第一次调用 get() 会执行已传递给 lazy() 的 lamda 表达式并记录结果，**<font color="#dd0000">后续调用 get() 只是返回记录的结果。</font>

   ```kotlin
   val lazyValue: String by lazy {
       println("computed!")     // 第一次调用输出，第二次调用不执行
       "Hello"
   }
   
   fun main(args: Array<String>) {
       println(lazyValue)   // 第一次执行，执行两次输出表达式
       println(lazyValue)   // 第二次执行，只输出返回值
   }
   computed!
   Hello
   Hello
   ```

5. Kotlin中的四种修饰符

   public：默认修饰符，被其修饰的在任何位置都能访问
   private：表示只在这个类(以及它的所有成员)之内可以访问
   protected：在当前类及其子类内访问
   **internal：在同一模块内使用**

6. Kotlin中的Pair、Triple

   元组类型Pair和Triple，有二元、三元

   ```kotlin
   val course=kotlin.Triple(3,"学会","Kotlin")
   val pay=kotlin.Pair("学费",0)
   print("${course.first}天${course.second}${course.third},${pay.first}${pay.second}")
   ```

7. object关键字，有两种用法，一是对象声明，二是对象表达式。

   对象声明：

   ```kotlin
   
   ```

   这里有一个问题，对象声明的静态类，为什么可以静态访问父类中的非静态方法？？

   ```kotlin
   
   ```

   

8. List转vararg

   使用展操作符(spread operator)

   ```kotlin
   *(List<T>).toTypedArray())
   ```


9. kotlin的内部类与java的内部类有点不同**java的内部类可以直接访问外部类的成员，kotlin的内部类不能直接访问外部类的成员，必须用inner标记之后才能访问外部类的成员**

   ```java
   class AAA{
       var a = 0
       class BBB{
           //此时，BBB类的内部是不能直接用a变量的
           var b = a //编译无法通过
       }
   }
   
   class AAA{
       var a = 0
       inner class BBB{
           //此时，BBB类的内部是可以直接用a变量的
           var b = a //编译可以通过
       }
   }
   ```

10. 

#### MVVM

#### LiveData

```java
package android.arch.lifecycle;

import static android.arch.lifecycle.Lifecycle.State.DESTROYED;
import static android.arch.lifecycle.Lifecycle.State.STARTED;

import android.arch.core.executor.ArchTaskExecutor;
import android.arch.core.internal.SafeIterableMap;
import android.support.annotation.MainThread;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;

import java.util.Iterator;
import java.util.Map;

/**
 * LiveData是一个数据支持类，可以在一个生命周期中被观察(observe)。这意味着Observer和LiveData可以在LifecycleOwner中成对保存，并且只有当LifecycleOwner处于活跃(active)状态时，Observer可以收到数据改变的通知。当LifecycleOwner的状态是Lifecycle.State#STARTED和Lifecycle.State#RESUMED时，它被认为是活跃的。如果一个observer是通过observeForever()添加到LifecycleOwner的，那么他将被认为是一直活跃的，此时你需要手动调用removeObserver(Observer)来移除这类observer。
 一个Lifecycle管理下的observer会在LifecycleOwner的状态转为Lifecycle.State#DESTROYED时被自动移除，这在Activity或Fragment中防止内存泄漏十分有用。
 另外，LiveData有onActive()和onInactive()方法用来通知活跃的Observer的数量改变(0或1),这可以用来决定何时释放那些不再被观察的重量级的资源。
 这个类被设计成ViewModel的一个用来存放数据的成员变量。除此之外，它还可以用来在application的不同Module之间进行数据共享。
 * @param <T> The type of data held by this instance
 * @see ViewModel
 */
public abstract class LiveData<T> {

    private SafeIterableMap<Observer<T>, ObserverWrapper> mObservers =
            new SafeIterableMap<>();
    // how many observers are in active state
    private int mActiveCount = 0;
    
    /**
     * Adds the given observer to the observers list within the lifespan of the given
     * owner. The events are dispatched on the main thread. If LiveData already has data
     * set, it will be delivered to the observer.
     * <p>
     * The observer will only receive events if the owner is in {@link Lifecycle.State#STARTED}
     * or {@link Lifecycle.State#RESUMED} state (active).
     * <p>
     * If the owner moves to the {@link Lifecycle.State#DESTROYED} state, the observer will
     * automatically be removed.
     * <p>
     * When data changes while the {@code owner} is not active, it will not receive any updates.
     * If it becomes active again, it will receive the last available data automatically.
     * <p>
     * LiveData keeps a strong reference to the observer and the owner as long as the
     * given LifecycleOwner is not destroyed. When it is destroyed, LiveData removes references to
     * the observer &amp; the owner.
     * <p>
     * If the given owner is already in {@link Lifecycle.State#DESTROYED} state, LiveData
     * ignores the call.
     * <p>
     * If the given owner, observer tuple is already in the list, the call is ignored.
     * If the observer is already in the list with another owner, LiveData throws an
     * {@link IllegalArgumentException}.
     *
     * @param owner    The LifecycleOwner which controls the observer
     * @param observer The observer that will receive the events
     */
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<T> observer) {
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }

    /**
     * Adds the given observer to the observers list. This call is similar to
     * {@link LiveData#observe(LifecycleOwner, Observer)} with a LifecycleOwner, which
     * is always active. This means that the given observer will receive all events and will never
     * be automatically removed. You should manually call {@link #removeObserver(Observer)} to stop
     * observing this LiveData.
     * While LiveData has one of such observers, it will be considered
     * as active.
     * <p>
     * If the observer was already added with an owner to this LiveData, LiveData throws an
     * {@link IllegalArgumentException}.
     *
     * @param observer The observer that will receive the events
     */
    @MainThread
    public void observeForever(@NonNull Observer<T> observer) {
        AlwaysActiveObserver wrapper = new AlwaysActiveObserver(observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && existing instanceof LiveData.LifecycleBoundObserver) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        wrapper.activeStateChanged(true);
    }

    /**
     * Removes the given observer from the observers list.
     *
     * @param observer The Observer to receive events.
     */
    @MainThread
    public void removeObserver(@NonNull final Observer<T> observer) {
        assertMainThread("removeObserver");
        ObserverWrapper removed = mObservers.remove(observer);
        if (removed == null) {
            return;
        }
        removed.detachObserver();
        removed.activeStateChanged(false);
    }

    /**
     * Removes all observers that are tied to the given {@link LifecycleOwner}.
     *
     * @param owner The {@code LifecycleOwner} scope for the observers to be removed.
     */
    @SuppressWarnings("WeakerAccess")
    @MainThread
    public void removeObservers(@NonNull final LifecycleOwner owner) {
        assertMainThread("removeObservers");
        for (Map.Entry<Observer<T>, ObserverWrapper> entry : mObservers) {
            if (entry.getValue().isAttachedTo(owner)) {
                removeObserver(entry.getKey());
            }
        }
    }

    /**
     * Called when the number of active observers change to 1 from 0.
     * <p>
     * This callback can be used to know that this LiveData is being used thus should be kept
     * up to date.
     */
    protected void onActive() {

    }

    /**
     * Called when the number of active observers change from 1 to 0.
     * <p>
     * This does not mean that there are no observers left, there may still be observers but their
     * lifecycle states aren't {@link Lifecycle.State#STARTED} or {@link Lifecycle.State#RESUMED}
     * (like an Activity in the back stack).
     * <p>
     * You can check if there are observers via {@link #hasObservers()}.
     */
    protected void onInactive() {

    }

    /**
     * Returns true if this LiveData has observers.
     *
     * @return true if this LiveData has observers
     */
    @SuppressWarnings("WeakerAccess")
    public boolean hasObservers() {
        return mObservers.size() > 0;
    }

    /**
     * Returns true if this LiveData has active observers.
     *
     * @return true if this LiveData has active observers
     */
    @SuppressWarnings("WeakerAccess")
    public boolean hasActiveObservers() {
        return mActiveCount > 0;
    }

}

```



#### Paging分页库



























扩展学习

MVP、MVVM、AAC讨论

RxJava源码学习

