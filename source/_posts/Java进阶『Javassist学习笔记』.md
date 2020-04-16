---
title: Java进阶『Javassist学习笔记』
tags: 
---

<http://www.javassist.org/>

### 文档翻译

1. 读取和写入字节码

   Javassist是一个用于处理Java字节码的类库。Java字节码被存储在class文件中，每个class文件包含一个Java类或者接口。

   Javassist.CtClass是一个class文件的抽象代表。一个CtClass对象可以(在编译期)用来处理一个class文件。以下面代码为例：

   ```java
   ClassPool pool = ClassPool.getDefault();
   CtClass cc = pool.get("test.Rectangle");
   cc.setSuperclass(pool.get("test.Point"));
   cc.writeFile();
   ```

   这段代码首先获得一个ClassPool对象，ClassPool是CtClass所代表的class文件的容器。ClassPool会在构造CtClass对象时去读取class文件，并且持有构造好的CtClass对象以便之后使用。想要修改一个class，首先需要通过ClassPool获取到一个CtClass对象。ClassPool对象的get()方法可以用来实现这个目的。

   从实现细节来说，ClassPool是一个以class的名字为key用来存储CtClass对象的哈希表，get()会从哈希表中根据传入的key查找CtClass对象。如果未能查找到缓存对象，get()方法会创建一个新的CtClass对象，并将这个新建的对象保存到哈希表中，然后返回给调用方。

   从ClassPool中获取到的CtClass对象可以被修改，在上面的代码中，cc的superclass被修改成了test.Point。

   writeFile()实现了从CtClass对象到class文件的转换，并将他写入到了本地磁盘。Javassist也提供了一种直接获取修改过的二进制代码的方法toBytecode()。除此之外，还可以直接载入CtClass：

   ```java
   Class clzzz = cc.toClass();
   ```

2. 定义一个新的class

   可以使用makeClass创建一个新的class，之后可以使用CtNewMethod类中的方法创建方法，然后通过addMethod添加到创建的class中：

   ```java
   ClassPool pool = ClassPool.getDefault();
   CtClass cc = pool.makeClass("Point");
   ```

   要创建接口，需要使用makeInterface()

3. 冻结的class

   如果一个CtClass对象已经通过writeFile()，toClass()，或toBytecode()转换成一个class文件，那么Javassist会冻结Ctclass对象，不允许对Ctclass对象进行修改。这是为了提示开发者在JVM已经加载class文件后不能修改class文件。**一个冰冻了的Ctclass可以通过defrost()方法解冻的方式来使能编辑。**

   ```java
   CtClasss cc = ...;
       :
   cc.writeFile();
   cc.defrost();
   cc.setSuperclass(...);    // OK since the class is not frozen.
   ```

   如果ClassPool.doPruning被设置为true，那么Javassist会在冻结对象是进行CtClass对象的剪裁，以减少内存开销。想要禁用剪裁，需要调用stopPruning()：

   ```java
   CtClasss cc = ...;
   cc.stopPruning(true);
       :
   cc.writeFile();                             // convert to a class file.
   // cc is not pruned.
   ```

4. 类搜索路径

   默认ClassPool.getDefault()从JVM的相同环境变量搜索类。但是如果程序运行在服务端，那么ClassPool对象就无法在用户的类搜索路径上找到目标类，此时可以通过insertClassPatch来增加类搜索路径：

   ```java
   ClassPool pool = ClassPool.getDefault();
   pool.insertClassPath("/usr/local/javalib");
   ```

   **搜索路径不仅可以是本地路径，还可以是线上的路径**，如：

   ```java
   ClassPool pool = ClassPool.getDefault();
   ClassPath cp = new URLClassPath("www.javassist.org", 80, "/java/", "org.javassist.");
   pool.insertClassPath(cp);
   ```

   除此之外，还可以直接传入byte数据来构造CtClass对象：

   ```java
   ClassPool cp = ClassPool.getDefault();
   byte[] b = a byte array;
   String name = class name;
   cp.insertClassPath(new ByteArrayClassPath(name, b));
   CtClass cc = cp.get(name);
   ```

5. 





### 使用总结

1. 如果想在运行期编辑类，给类添加方法，只能新增一个新的类。**因为一个ClassLoader不能加载同一个类两次**。