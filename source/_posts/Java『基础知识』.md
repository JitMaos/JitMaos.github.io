---
title: Java『基础知识』
date: 2019-04-01 10:29:31
tags:
---

#### JVM

JVM是运行Java字节码的虚拟机。**JVM有针对不同系统的特定实现**，目的在不同的系统平台上运行相同的字节码。**.java文件经过JDK的javac编译为.class文件，.class文件又被JVM编译成机器可执行的二进制机器码。**

我们需要格外注意的是 .class->机器码 这一步。在这一步 jvm 类加载器首先加载字节码文件，然后通过解释器<font color="#dd0000">**逐行**</font>解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的，也就是所谓的热点代码，所以后面引进了 JIT 编译器，JIT 属于运行时编译。**当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用**。而我们知道，机器码的运行效率肯定是高于 Java 解释器的。这也解释了我们 为什么经常会说 Java 是编译与解释共存的语言。	

HotSpot采用了惰性评估(Lazy Evaluation)的做法，根据二八定律，消耗大部分系统资源的只有那一小部分的代码（热点代码），而这也就是JIT所需要编译的部分。JVM会根据代码每次被执行的情况收集信息并相应地做出一些优化，因此执行的次数越多，它的速度就越快。**JDK 9引入了一种新的编译模式AOT(Ahead of Time Compilation)，它是直接将字节码编译成机器码**，这样就避免了JIT预热等各方面的开销。JDK支持分层编译和AOT协作使用。但是 ，AOT 编译器的编译质量是肯定比不上 JIT 编译器的。

<!--more-->

#### 重载与重写

**重载**：发生在同一类中，方法名相同，方法签名(参数类型、个数、顺序不同)可能不同；注意，**返回类型和访问修饰符不算在方法签名中，不能作为重载的判断依据。**

**重写**：发生在继承类中，方法签名需要相同，方访问权限子类中实现需要大于等于父类中实现。

#### 构造器Constructor是否可以被override？

父类的构造方法和私有属性不能被继承，所以Constructor也不能被重写(override)，但是可以被重载(overload)。

#### Java面向对象编程三大特性

**封装、继承、多态**，其中多态指定是程序的引用变量可以在运行时的向上转型动态确定。Java中可以通过**继承**和**接口**实现多态。

#### String、StringBuilder、StringBuffer

**可变性**：首先String是用final修饰的保存字符的数组，因为字符串是不可变的，所以当直接定义字符串(如："abc")时，会去**常量池**中寻找该变量值，存在直接返回使用，不存在再行创建；StringBuilder和StringBuffer都是继承自AbstractStringBuilder类，该类也是使用数组保存字符串，和String不同的是没有使用final修饰，所以是可变的。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    ...
}
```

**线程安全性**：String是不可变的，所以线程安全；StringBuffer是线程安全的；StringBuilder是非线程安全的，所以性能比StringBuffer高一点。

#### 为什么不能再静态方法内调用外部类的非静态成员？？

<font color="#dd0000">**待完成**</font>

#### ==和equals()

默认情况下==用来比较两个对象的引用地址是否相同，或者比较两个常量(**基础类型分布在常量池中**)是否相等；equals用来比较对象的**具体属性的值**是否相等；有时我们**只需要比较两个对象包含的属性值是否相等**，这时可以重写equals方法来实现。

#### Object的hashCode()和HashMap中的hashCode()

在使用==判断两个变量是否相等，比较的是两个变量的内存地址，和Object的hashCode()并没有直接的关系；先看**Object中的hashCode()**：

```java
/**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     * <p>
     * As much as is reasonably practical, the hashCode method defined by
     * class {@code Object} does return distinct integers for distinct
     * objects. (This is typically implemented by converting the internal
     * address of the object into an integer, but this implementation
     * technique is not required by the
     * Java&trade; programming language.)
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
    public native int hashCode();
```

首先看到这是一个native方法，方法注释

+ **This method is supported for the benefit of hash tables such as those provided by {@link java.util.HashMap}，所以这是为什么比较两个对象是否相等(值相等使用equals，内存地址相等使用==)没有用到hashCode方法，却非要在Object类中添加对hashCode的支持的原因；**
+ 在同一对象上调用多次hashCode()应该始终返回相同的int值；
+ 如果ObjA.equals(ObjB)，那么ObjA的hashCode一定等于ObjB；
+ 两个对象相互equals，不代表他们的hashCode一定不相等(即所谓的哈希碰撞)。

其次HashMap只是一种数据结构，**其存在的理由是为了集成类似数组的快速查找，又想避免为了存储少数大范围值的数据而创建一个大数组的问题**。通过hash函数可以把这些大范围值的数据**映射**到HashMap的数组中去，如果发生哈希碰撞，则会是使用链表，而在Java 8.0以后，使用红黑树替代了链表；

具体HashMap可参考：[HashMap实现原理及源码分析](https://www.cnblogs.com/chengxiao/p/6059914.html)

#### equals()和hashCode()

​	以HashSet为例，当需要插入、查询一个对象时，**先计算该对象的hashCode,看是否存在相同的hashCode，如果存在，再对比他们的equals方法是否相等**，如果equals方法也相等，则表明是同一个对象，拒绝插入；如果equals方法不相等，则对对象的重新进行hash，插入到其他位置。

#### Java中的序列化和反序列化(Serializable)

​	**Java序列化就是将一个对象转化为一个**<font color="#dd0000">**二进制表示的字节数组**</font>，如果不想对某些字段进行序列化，可以使用**transient**关键字修饰；在反序列化时如果serialVersionUID被修改的话，反序列化会失败；当父类实现了Serializable接口的时候，所有的子类都能被序列化，当子类实现Serializable接口时，父类没有，则父类中的属性不能被序列化。

#### Collections类和Arrays工具类的常用方法

##### Collections类

+ 排序：sort(List)、sort(List, Comparator)、swap(list, i , j)
+ 查找：binarySearch(list , key)、max(Collection)、indexOfSubList(List list, List target)
+ 替换：replaceAll(List list, Object oldVal, Object newVal)

##### Arrays类

+ 排序：sort(List)

```java
    val iArray = intArrayOf(1, 3, 4, 5, 6, 7, 8, 8, 6, 5, 4)
        Arrays.sort(iArray)
        for (v in iArray) {
            print("$v ")
        }
    >>>输出：1 3 4 4 5 5 6 6 7 8 8
```

+ 比较：equals()

```java
    val arrayA = intArrayOf(1,3,5)
    val arrayB = intArrayOf(1,3,5)
    println(Arrays.equals(arrayA,arrayB))
    >>>输出：true
```

+ 转列表：asList()

```java
    val names = Arrays.asList("Larry", "Moe", "Curly")
    println(names)
    >>>输出：[Larry, Moe, Curly]
```

#### Java基础关键字

#####  final

+ 当final用来修饰变量时，如果变量是基本数据类型，则其数值不能改变；如果变量时引用类型，则改变量不能在再指向另一个对象。
+ 当一个方法被final修饰时，表示该方法不能被继承类重写。
+ 当一个类使用final修饰时，该类不能被继承，**该类中所有方法都会隐式的被定义为final方法**。

##### static

- 被static修饰的成员属于类，被类中所有对象共享，静态变量被分配在方法区，被所有线程共享。
- **静态代码块**，静态代码在非静态代码执行之前执行，且<font color="#dd0000">**不管改类创建多少对象，静态代码只执行一次**</font>。
- **静态内部类**，和非静态内部类最大的区别在于，<font color="#dd0000">**非静态内部类在编译后会隐含地保存着一个指向创建他的外部类的引用，而静态内部类没有这个引用**</font>。经常这里会问到内存泄露的问题。

##### 为什么static方法中不能使用this和super？

static修饰静态方法是属于类的；而this指向当前对象，super代表父类对象引用；**静态方法属于类，而this、super针对对象。**同样的问题还有：**为什么不能在静态方法内调用外部类的非静态成员？**一样的答案。

#### 内部类、System.GC()、弱引用及内存泄漏



##### 内部类

大题上Java中包含四种内部类，它们分别是：

+ 静态内部类
+ 匿名内部类
+ 实例内部类
+ 本地内部类(方法内的类)

##### System.GC()

```java
System.gc() //调用System.gc()并不会马上出发GC操作
//Runtime.getRuntime().runFinalization()
System.runFinalization()
System.gc()
```

#####弱引用

##### 内存泄漏

**当某些对象不再被应用程序所使用,但是由于仍然被引用而导致垃圾收集器不能释放(Remove,移除)他们。**Android中常见的内存泄漏场景：

+ 资源使用未关闭，如：数据库连接、Bitmap、File、BroadcastReceiver等。

+ Java中匿名内部类，默认持有外部对象引用，可能导致外部对象无法及时回收，如Handler和Activity。

+ 如果单例中持有外部对象的引用，那么这个外部对象将不能被JVM正常回收。**类似的还有静态集合对象**

  