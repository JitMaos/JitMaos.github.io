---
title: Java『Javap命令』
tags:
---

看JVM的运行时数据区，包括：虚拟机栈、程序计数器、本地方法栈、方法区和堆；其中虚拟机栈中存储的是栈帧，而栈帧是由<font color="#dd0000">**局部变量表、操作数栈**</font>、动态链接、方法出口等组成。今天学习一下Javap命令，通过对字节码进行反编译来查看局部变量表的细节。

javap -v cclassName，不仅会输出行号、本地变量表信息、反编译汇编代码，还会输出当前类用到的常量池等信息。
javap -l className，会输出行号和本地变量表信息。
javap -c className，会对当前class字节码进行反编译生成汇编代码。

1. 新建TreeNode.java

```java
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int x) { val = x; }
}
```

2. 使用javac编译生成class文件
3. 输入命令：javap -v TreeNode.class

```java
Classfile TreeNode.class
  Last modified 2020-2-3; size 370 bytes
  MD5 checksum 6db15fb197ff5952f8b106978416b74a
  Compiled from "TreeNode.java"
public class com.leon.LeetCode.TreeNode
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#19         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#20         // com/leon/LeetCode/TreeNode.val:I
   #3 = Class              #21            // com/leon/LeetCode/TreeNode
   #4 = Class              #22            // java/lang/Object
   #5 = Utf8               val
   #6 = Utf8               I
   #7 = Utf8               left
   #8 = Utf8               Lcom/leon/LeetCode/TreeNode;
   #9 = Utf8               right
  #10 = Utf8               <init>
  #11 = Utf8               (I)V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               LocalVariableTable
  #15 = Utf8               this
  #16 = Utf8               x
  #17 = Utf8               SourceFile
  #18 = Utf8               TreeNode.java
  #19 = NameAndType        #10:#23        // "<init>":()V
  #20 = NameAndType        #5:#6          // val:I
  #21 = Utf8               com/leon/LeetCode/TreeNode
  #22 = Utf8               java/lang/Object
  #23 = Utf8               ()V
{
  public int val;
    descriptor: I
    flags: ACC_PUBLIC

  public com.leon.LeetCode.TreeNode left;
    descriptor: Lcom/leon/LeetCode/TreeNode;
    flags: ACC_PUBLIC

  public com.leon.LeetCode.TreeNode right;
    descriptor: Lcom/leon/LeetCode/TreeNode;
    flags: ACC_PUBLIC

  public com.leon.LeetCode.TreeNode(int);
    descriptor: (I)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iload_1
         6: putfield      #2                  // Field val:I
         9: return
      LineNumberTable:
        line 10: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      10     0  this   Lcom/leon/LeetCode/TreeNode;
            0      10     1     x   I
}
```



常见的字节码命令

| Bytecode | Stackbefore->after      | Description                                           |
| -------- | ----------------------- | ----------------------------------------------------- |
| iconst_0 | ->0                     | Loads the int value 0 onto the stack //从栈中读取变量 |
| istore_1 | value->                 | Store int value into variable 1 //将变量压入栈        |
| istore_2 | value->                 | Store int value into variable 2 //将变量压入栈        |
| iinc     | No change               | Increment local variable #index by signed byte const  |
| iload_1  | ->value                 | Loads an int value from variable 1 //加载变量的值     |
| iadd     | value 1,value 2->result | Adds 2 ints together //将两个变量相加                 |

