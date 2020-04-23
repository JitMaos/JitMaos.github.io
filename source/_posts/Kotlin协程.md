1. 协程的开发人员 Roman Elizarov 是这样描述协程的：**协程就像非常轻量级的线程**。线程是由系统调度的，线程切换或线程阻塞的开销都比较大。而协程依赖于线程，但是协程挂起时不需要阻塞线程，几乎是无代价的，协程是由开发者控制的。所以**协程也像用户态的线程**，非常轻量级，一个线程中可以创建任意个协程。
2. **一个线程中可能包含多个协程，协程与协程之间是可以嵌套的。**<font color="#dd0000">**协程是可以直接运行在进程中的，不是一定要依赖于线程**</font>，只不过现在支持协程的 `Kotlin` `Python` `Go` 等语言都会以主线程的方式开启程序的运行。
3. 调用了 `runBlocking` 的**主线程**会一直 *阻塞* 直到 `runBlocking` 内部的协程执行完毕。

```kotlin
fun x() = runBlocking {
    GlobalScope.launch {
        println("x1:" + Thread.currentThread())
        delay(1000)
        println("world")
    }
    println("hello,")
    delay(2000)
    println("x2:" + Thread.currentThread())
}
>>>输出：
hello,
x1:Thread[DefaultDispatcher-worker-2,5,main] //Scope中子线程，不管是否在主线程下
world
x2:Thread[main,5,main] //runBlocking中直接操作为主线程
```

4. 使用await()可以阻塞主线程。

   <!--more-->

5. fun launch(): Job，返回的Job对象有三个方法，start、join、cancel分别对应协程的启动、切换、取消。
   fun async(): Deferred，返回的Deferred有方法await，该方法可以用来得到一个结果值。

6. **async方法返回一个Deferred对象，Deferred有一个await()方法，该方法可以返回一个结果,这样将可以用顺序的方式编写异步处理流程.**

   ```kotlin
   fun testAsync() {
       val deffered1 = GlobalScope.async {
           "Hello1"
       }
       val deffered2 = GlobalScope.async {
           println("Hello2")
           println(deffered1.await())
       }
   }
   ```

7. 在 [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html) 中启动的活动协程并不会使进程保活。它们就像守护线程。

```kotlin
fun main() = runBlocking {
    GlobalScope.launch {
        repeat(1000) { i ->
                println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // 在延迟后退出
}
//你可以运行这个程序并看到它输出了以下三行后终止：
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```







```java
/**
 * 协程的固定上下文，是一个带索引的Set。一个带索引的Set是Set和Map的合成。该Set中的每个元素都有一个唯一的Key。
 Persistent context for the coroutine. It is an indexed set of [Element] instances.
 * An indexed set is a mix between a set and a map.
 * Every element in this set has a unique [Key]. Keys are compared _by reference_.
 */
@SinceKotlin("1.3")
public interface CoroutineContext {
    /**
     * Returns the element with the given [key] from this context or `null`.
     * Keys are compared _by reference_, that is to get an element from the context the reference to its actual key
     * object must be presented to this function.
     */
    public operator fun <E : Element> get(key: Key<E>): E?

    /**
     * Accumulates entries of this context starting with [initial] value and applying [operation]
     * from left to right to current accumulator value and each element of this context.
     */
    public fun <R> fold(initial: R, operation: (R, Element) -> R): R

    /**
     * Returns a context containing elements from this context and elements from  other [context].
     * The elements from this context with the same key as in the other one are dropped.
     */
    public operator fun plus(context: CoroutineContext): CoroutineContext =
        if (context === EmptyCoroutineContext) this else // fast path -- avoid lambda creation
            context.fold(this) { acc, element ->
                val removed = acc.minusKey(element.key)
                if (removed === EmptyCoroutineContext) element else {
                    // make sure interceptor is always last in the context (and thus is fast to get when present)
                    val interceptor = removed[ContinuationInterceptor]
                    if (interceptor == null) CombinedContext(removed, element) else {
                        val left = removed.minusKey(ContinuationInterceptor)
                        if (left === EmptyCoroutineContext) CombinedContext(element, interceptor) else
                            CombinedContext(CombinedContext(left, element), interceptor)
                    }
                }
            }

    /**
     * Returns a context containing elements from this context, but without an element with
     * the specified [key]. Keys are compared _by reference_, that is to remove an element from the context
     * the reference to its actual key object must be presented to this function.
     */
    public fun minusKey(key: Key<*>): CoroutineContext

    /**
     * Key for the elements of [CoroutineContext]. [E] is a type of element with this key.
     * Keys in the context are compared _by reference_.
     */
    public interface Key<E : Element>

    /**
     * An element of the [CoroutineContext]. An element of the coroutine context is a singleton context by itself.
     */
    public interface Element : CoroutineContext {
        /**
         * A key of this coroutine context element.
         */
        public val key: Key<*>

        public override operator fun <E : Element> get(key: Key<E>): E? =
            @Suppress("UNCHECKED_CAST")
            if (this.key == key) this as E else null

        public override fun <R> fold(initial: R, operation: (R, Element) -> R): R =
            operation(initial, this)

        public override fun minusKey(key: Key<*>): CoroutineContext =
            if (this.key == key) EmptyCoroutineContext else this
    }
}

```

