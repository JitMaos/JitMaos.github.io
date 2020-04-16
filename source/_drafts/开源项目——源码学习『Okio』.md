---
title: 源码学习『Okio』
tags:
---

### 介绍

Okio是一个对于java.io和java.nio进行完善的类库，使用它可以实现更简单的数据存储、访问和处理工作。

#### ByteStrings and Buffers

Okio围绕着这两个类型的对象提供了许多简单易用的API：

**ByteString**是一个**不可变**的二进制序列。对于字符数据来说String是基础类型。ByteString是String的远房表亲，使用它可以更简单的处理二进制数据。这个类可能更符合人类的思维模式：它自己知道怎么编码/解码hex，base64，UTF-8。

**Buffer**是一个**可变**的二进制序列。就像ArrayList，你不需要特意指定buffer的大小。读写buffer就像一个队列：数据将被写到队列尾，而读取则是从队列头读取的。<font color="#dd0000">**不需要管理具体的元素位置、容量限制等问题。**</font>

ByteString和Buffer为了节约CPU和内存做了一些优化。如果你一个UTF-8格式的字符串A编码成ByteString。它会缓存一个指向字符串A的引用。这样之后如果需要解码，则不需要什么额外的工作。

Butter的实现模式是一个由Segment构成的链表。当你将数据从一个buffer移动到另一个buffer时，<font color="#dd0000">**它会重新分配响应的Segments的拥有权，而不是仅仅来回移动数据而已**</font>。这种实现方式对于多线程程序特别有意义：**一个线程和网络打交道的线程可以和另外一个工作线程进行数据交换，而不需要任何拷贝或其他复杂的手续**。

#### Sources and Sinks

java.io的一个设计优雅之处在于传输流可以层层堆叠，就像加密或者压缩那样。Okio包含了它自定义的两个流类型：Source和Sink，这两者就像InputStream和OutputStream一样。不过还是有一些关键的不同之处的：

**Timeouts**：<font color="#dd0000">**流对象可以访问底层I/O超时状态，也就是说上层业务可以知道本次读写操作是否超时了**</font>。而不是像java.io的Socket对那样，调用read()或write()可能会无限阻塞在那里。

**Easy to implement**：Source接口只定义了三个方法：read()、close()、timeout()。去掉了鸡肋的available()和单字节读取方法，这些可能会错误或性能问题的方法。

**Easy to use**：尽管实现Source或Sink接口只需要实现三个方法，调用者可以在他们的实现接口BufferedSource和BufferedSink接口中找到更多丰富的API。

**不在有人为定义的字节流和字符流的区别**：所有的数据，读写都是基于二进制数据的，再没有InputStreamReader了！

**更容易测试**：<font color="#dd0000">**Buffer类同时实现了BufferedSource和BufferedSink接口**</font>，所以你可以写一段简单清除的测试代码。

Sources 和 Sinks跟InputStream和OutputStream基本上是等同的。你可以将任何Source视作InputStream，你也可以将InputStream视作Source。同样地，对于Sink和OutputStream也适用。

#### 使用示例

##### 一行一行的读取文本内容

使用Okio.source(File)打开一个Source流来读取一个文件。返回的Source接口十分的简单而且能力有限，所以我们可以在他上面使用buffer包装一层。这有两个好处：

+ 让API更丰富。BufferSource有许多方法来实现大部分通用的问题。
+ 可以让代码运行的更快。Buffer可以是Okio使用更少的I/O操作，因为有缓存嘛。

每个打开了的Source都需要被关闭。打开Stream的代码有义务在使用完流之后将它关闭。这里我们使用Java的try代码块来实现关闭操作。



### 几个问题

1. Okio中流是怎么知道本次读写操作是否超时了的？