---
title: Java『数据容器』
date: 2019-04-09 13:53:02
tags: Java
---

#### HashMap

先从Java中Object的hashCode()方法说起，从方法注释的第一行可以看到该方法的存在主要是为了支持HashMap：

```java
//Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by HashMap.
public int hashCode() {
    return identityHashCode(this);
}
```

+ **结构**：HashMap的设计初衷是为了在有限的容器内放置若干范围很大的数据。**JDK1.8之前底层使用的是数组+链表，1.8之后当链表长度大于阈值(默认为8)时，会将链表转换成红黑树，以减少搜索时间**；HashMap中有一个hash方法用来得到元素Key的hashCode值，这个hash方法是对Object的HashCode再做了一次优化，为了减少发生哈希碰撞的次数。

+ **问题**

  + **为什么重写equals方法必须重写hashCode方法？**可以看hashCode方法注释中三点限制：

    1. 同一个对象上调用多次hashCode必须输出相同的整型数据。

    2. <font color="#dd0000">**两个对象如果equals，那么他们必须有相同的hashCode**</font>。

    3. 有相同的hashCode的2个对象，不一定equals。

    <font color="#dd0000">**由第2点可以得出结论，如果重写了equals，必须重写hashCode；因为euqals方法中设计的字段必须得参与到hashCode的计算中去，才能保证第2点能够正确**</font>。

  + 为什么HashMap的长度是2的幂次方？

    **为了更高效，因为如果除数是2的幂次方，那么取余操作等同于右移一位(hash % length) == (hash & (length -1))**
    
  + <font color="#dd0000">**如何解决散列表的哈希冲突问题？**</font>

    答：链表法(将相同的hash值的对象组织成一个链表放在hash值对应的槽位)和开发地址法(通过一个探测算法，当某个槽位已经被占据的情况下继续寻找下一个可以使用的槽位)。目前HashMap使用的是链表法。

  + 为什么HashMap不是线程安全的？

    1、HashMap 在插入的时候

    现在假如 A 线程和 B 线程同时进行插入操作，然后计算出了相同的哈希值对应了相同的数组位置，因为此时该位置还没数据，然后对同一个数组位置，两个线程会同时得到现在的头结点，然后 A 写入新的头结点之后，B 也写入新的头结点，那B的写入操作就会覆盖 A 的写入操作造成 A 的写入操作丢失。

    2、HashMap 在扩容的时候

    HashMap 有个扩容的操作，这个操作会**新生成一个新的容量的数组**，然后对原数组的所有键值对重新进行计算和写入新的数组，之后指向新生成的数组。当多个线程同时进来，检测到总数量超过门限值的时候就会<font color="#dd0000">**同时调用 resize 操作，各自生成新的数组并 rehash 后赋给该 map 底层的数组**</font>，结果最终只有最后一个线程生成的新数组被赋给该 map 底层，其他线程的均会丢失。

    

+ **注意**

  循环引用，当在多线程环境下，对HashMap进行put操作，可能会出发扩容；扩容意味着数组元素及元素下挂载的链表的复制，所以可能在扩容是会发生循环引用。

+ <font color="#dd0000">**equals和hashCode**</font>

  hashCode方法用于得到一个对象的整型类型的hashCode值。equals<font color="#dd0000">**通常**</font>用来比较两个**对象的值**是否相等。查看Object类中的equals方法的默认实现是这样的：

  ```java
   public boolean equals(Object obj) {
          return (this == obj);
      }
  ```
  
  从这里看出默认不重写equals方法的情况下，调用equals是等同于==的，然而很多类都重写了equals方法，**使之用于比较对象上的值是否相等**，如字符串类String中的equals：
  
  ```java
      public boolean equals(Object anObject) {
          if (this == anObject) {
              return true;
          }
          if (anObject instanceof String) {
              String anotherString = (String)anObject;
              int n = length();
              if (n == anotherString.length()) {
                  int i = 0;
                  while (n-- != 0) {
                      if (charAt(i) != anotherString.charAt(i))
                              return false;
                      i++;
                  }
                  return true;
              }
          }
          return false;
      }
  ```
  
  我们知道一个HashMap中不能存在两个相同的key。**这里的相同的key，指的是hash相等，且key的值相等(==或者equals)。**
  
  #### HashMap的putVal方法
  
  设有对象A要添加到HashMap中去，**putVal方法回返回插入对象的旧值(return previous value, or null if none)**，过程如下：
  
  1. 首先使用<font color="#dd0000">**(n-1) & hash**</font>得到key对应HashMap数组的索引位置，如果**该位置为空，则直接插入新Node。过程结束**，返回null
  
  2. 如果**1**中数组索引位置上**已存在对象**，那么
  
     1. 如果该位置上对象的hash和A的hash相等，且key的引用相等或值相等，直接替换（**即在数组上已经找到了hash和key相同的元素，不需要在从链表或红黑树中查找元素**)
  
     2. 如果桶上找到了hash相等的对象，但是他们的key不相等，那么需要判断节点是红黑树节点还是链表节点来选择不同的put方式。
  
     3. 是红黑树节点，调用putTreeVal()，**这个本段落暂不分析**。
  
     4. 是链表节点，那么久使用Node.next遍历链表，**直至遍历到末尾插入新Node**；或者找到一个key相等的，**替换之**。
  
        注意此时，记录了遍历的节点数，当<font color="#dd0000">**遍历次数**</font>大于等于阈值(TREEIFY_THRESHOLD，默认是8)，会进行树化操作（treeifyBin()）。
  
  ```java
    /**
       * Implements Map.put and related methods
       *
       * @param hash hash for key
       * @param key the key
       * @param value the value to put
       * @param onlyIfAbsent if true, don't change existing value
       * @param evict if false, the table is in creation mode.
       * @return previous value, or null if none
       */
      final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
          //1. 首先使用(n-1) & hash得到key对应HashMap桶上的位置(即数组的索引位置)，如果该位置为空，则直接插入新Node。
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else { //2. 数组索引位置上已存在对象
              Node<K,V> e; K k;
              //2.1 如果该位置上对象的hash和A的hash相等，且key的引用相等或值相等
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;
              //2.2 如果桶上找到了hash相等的对象，但是他们的key不相等，那么判断节点是红黑树节点还是链表节点
              else if (p instanceof TreeNode) 
                  //2.3 是红黑树节点，调用putTreeVal()，这个暂不分析
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              else {
                  //2.4 是链表节点，那么久使用Node.next遍历链表，直至遍历到末尾插入新Node；或者找到一个key相等的，替换之
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {
                          p.next = newNode(hash, key, value, null);
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);
                          break;
                      }
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
              if (e != null) { // existing mapping for key
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
          ++modCount;
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);
          return null;
      }
  ```
  
  <!-- more -->

### Hashtable

+ **结构**：Hashtable是基于数组+链表(和JDK 1.8之前的HashMap一样)，**继承自Dictionary(一个比较陈旧的类)**；<font color="#dd0000">**Hashtable是线程安全的**</font>，其内部方法通过使用synchronized修饰来实现线程安全，(相应的效率会低一些，如果需要使用线程安全的哈希表，可以使用ConcurrentHashMap)；
+ **注意**：影响Hashtable性能的有两个参数：**初始容量**和**加载因子**。Hashtable默认的初始大小为**11**，扩容方式为**2n+1**，而HashMap的扩容方式为扩充为2的冥次方。另外，<font color="#dd0000">**Hashtable中key和null都不能为null**</font>。

#### ConcurrentHashMap

+ **结构**：JDK 1.8之前采用**分段的数组+链表**实现，JDK1.8开始数据结构和JDK 1.8下的HashMap 一样，采用**数组+链表/红黑树**。
+ **注意**：在JDK 1.8之前，对整个桶进行分割分段(Segment)，每一把锁只锁定其中一部分数据，多线程访问容器中不同数据段的数据时就不存在锁竞争了。在JDK 1.8中摒弃了Segment的概念，而是使用JDK 1.8 HashMap的数据结构配合Synchronized和CAS来操作，**看起来就像是优化过且线程安全的HashMap**。

#### LinkedHashMap

+ **结构**：继承自HashMap，底层数据基本和HashMap一样，**只是增加了一条双向链表，使之可以保持键值对的插入顺序。**

#### TreeMap

+ **结构**：红黑树

### HashSet

+ **结构**：HashSet是基于HashMap实现的，可以看做一个所有Value都是null的HashMap。所以HashSet中不能存储相同的对象(即HashMap中的Key)
+ **注意**：当尝试把一个对象add到HashSet时，首先会在HashSet中寻找和该对象的hashCode一样的元素，如果存在，则会比较这两个元素是否equals，如果equals

#### LinkedHashSet

+ **结构**：继承自HashSet，不过其内部是通过LinkedHashMap来实现的。

#### TreeSet

+ **结构**：红黑树，有序，唯一。

#### ArrayList

+ **结构**：ArrayList是基于数组实现，<font color="#dd0000">**非线程安全**</font>，因为基于是基于数组实现的，所以检索时间复杂度为**O(1)**，增删时间复杂度为**O(n)**；
+ **扩容**：
+ 注意事项：
  + 删除元素时，索引移动

#### LinkedList

+ **结构**：LinkedList是基于**双向链表**实现的(JDK 1.7之前是循环列表)，<font color="#dd0000">**非线程安全**</font>，因为是基于链表实现的，所以检索时间复杂度为**O(n)**，增删时间复杂度为**O(1)**；

#### Vector

+ **结构**：Vector和ArrayList很相似，也是基于数组实现的，<font color="#dd0000">**不同的是它是线程安全的**</font>；当容量不足时，会进行扩容(基于扩容系数，如果系数为空，则将容量增加一倍)，扩容会将数组中的元素拷贝到新的数组中去；Vector实现了RandomAccess接口，支持随机访问。

### RandomAccess

RandomAccess是一个空的标记接口，用来标记对应的数据容器是否支持随机访问。ArrayList、Vector实现了RandomAccess接口，而LinkedList没有实现该接口；举例：在Arrays的binarySearch方法中，会判断数据容器是否被RandomAccess接口标记，进而选择相应的检索逻辑。

```java
public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
   		    return Collections.iteratorBinarySearch(list, key);
}
```





参考【<https://github.com/Snailclimb/JavaGuide>】