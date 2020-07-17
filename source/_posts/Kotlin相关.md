---
title: Kotlin相关
date: 2019-04-08 16:58:45
tags:
---

### 一、Kotlin基础

#### 1.构造方法

#### 1.1 主构造方法

参考：https://blog.csdn.net/xlh1191860939/article/details/79412319

在类的顶部声明，初始化语句块包含在类被创建时执行的代码，并与主构造方法一起使用。因为**主构造方法不能包含初始化代码**，所以需要使用初始化语句块。

```kotl
class User constructor(_nickname: String) { // 带一个参数的主构造方法
    val nickname: String
    init { // 初始化代码块
        nickname = _nickname
    }
}
```

可以简化为：

```ko
class User(_nickname: String) {
    val nickname = _nickname // 不使用初始化代码块，用参数来初始化属性
}
```

进一步简化：

```kot
class User(val nickname: String) // "val" 表示相应的属性会用构造方法的参数来初始化
```

还有一种写法：

```kotlin
class User(nickname:String) {
  private val nickname:String = nickname;
}
```

#### 1.2 

#### 2 with apply let run区别

摘自：https://www.jianshu.com/p/e7abbfb109ac



+ @LayoutRes

用来标注当前方法返回一个Int类型的资源。

+ 重写Setting方法

```java
var title: CharSequence? = ""
        set(value) {
            field = value
            toolbarTitle?.text = title
        }
```

+ apply函数，变换对象，返回自身 && 动态设置按钮点击效果

```kotlin
private fun tintIcon(imageView: ImageView, colors: Int) {
  imageView.apply {
    val wrappedDrawable = DrawableCompat.wrap(drawable)
      DrawableCompat.setTintList(wrappedDrawable, ColorStateList.valueOf(colors))
      setImageDrawable(wrappedDrawable)
  }
}
```

+ Kotlin实现单例

```java
class AppManager private constructor(){
    companion object {
        private var instance:AppManager? = null
        @Synchronized
        fun instance():AppManager {
            if(instance == null) {
                instance = AppManager()
            }
            return instance!!
        }
    }
}
```

+ 高阶函数使用

```java
 private fun process (b:(Activity) -> Boolean) {
   if(list.isEmpty()) {
     return
   }
   val iterator = list.iterator()
     while(iterator.hasNext()) {
       val next = iterator.next()
         if(b(next)) {
           if(!next.isFinishing) {
             next.finish()
           }
         }
     }
 }
//测试
fun pop(activity:Activity) {
  process { it == activity }
}
```

+ Extensions使用

  定义好Extensions后，需要导入才能使用

```kotlin
fun View.visible() {
    if (visibility != View.VISIBLE) {
        visibility = View.VISIBLE
    }
}
```

+ By lazy 使用

```kotlin
private val compositeDisposable:CompositeDisposable by lazy { CompositeDisposable() }
```

+ 使用类的simpleName

```kotlin
private var TAG = "BaseView_${javaClass.simpleName}"
```

+ 在类中定义别的类得扩展函数，<font color="#dd0000">**这怎么想得到？**</font>

```kotlin
//BaseViewModel.kt
//TODO WTF.ZMXDD
open fun Disposable.add() {
  compositeDisposable.add(this) //this指代Disposable
}
```

#### typealias

可用于提供一个更语义精简的类型别名取代具体泛型类型、匿名函数等含糊定义。※ typealias 不会生成新的类型，编译器只做简单内联替换

```kotlin
// 泛型别名
typealias NodeSet = Set<Network.Node>
typealias FileTable<K> = MutableMap<K, MutableList<File>>

// 函数别名
typealias MyHandler = (Int, String, Any) -> Unit
typealias Predicate<T> = (T) -> Boolean

// 同名类型别名
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```





### 二、Kotlin函数

#### 1.高阶函数

```kotlin
//定义
var itemLongClickListener: ((View, T, Int) -> Boolean)? = null
//使用
adapter.itemClickListener = { view, item, position -> Boolean
    view.background = ColorDrawable(resources.getColor(R.color.red))
    true
}
```









### Kotlin协程

**介绍**：一个线程中可以包含多个协程，协程中可以嵌套协程；当线程需要执行阻塞任务时，可以把阻塞任务放到线程的协程中完成，相对于线程，**协程更轻量，开销更少**。总得来说使用协程可以提高CPU的利用率，减少多线程切换带来的开销。

**使用**：



### 问题记录

1. Kotlin中创建的类的属性默认是private的，所以使用Dagger进行注入时会报**Dagger does not support injection into private fields**，解决办法使用@JvmField注释要注入的字段即可。

