---
title: 读书笔记『Flutter从0到1构建大前端应用』
tags: [Flutter]
---

### 第一章 Flutter简介

**P4**：Flutter Engine

+ Skia：2D渲染引擎(Android系统自带，**iOS系统不自带，因此iOS包所占用的存储空间更大**)
+ Dart：Dart运行时
+ Text：文本排版引擎

### 第二章 Dart 语言入门

**P18**：Dart是一门强类型语言，在第一次赋值时，如果已经确定了是字符串类型，则不能更改为别的类型。如果真的想要改变，可以使用dynamic关键字。

**P19**：num声明的变量可以加入的是int型，还可以被改成double型，但是反过来int声明的变量不能再赋值为double。

```dart
num a = 10;
a = 30.2;
```

**P23**：使用bool表示布尔值，布尔值只有true和false；可以在debug模式下通过assert断言判断布尔值

```dart
var a = '';
assert(a.isEmpty);
```

创建一个不可变数组，使用const[...]

```dart
var list = const[1,2,3];
```

**P25**：使用dynamic时会告诉编译器，我们不用做类型检测，并且知道自己在做什么。

**P26**：可以使用as，is关键字对类型进行检测

```dart
dynamic obj = <String,int>();
if(obj is Map<String,int>) {
  obj['age'] = 20;
}
```

<font color="#dd0000">**~/ 除法，返回一个整数结果**</font>

**级联操作符**：有点类似一些语言的链式调用。

```dart
String s = new StringBuffer()
  	..write('a')
  	..write('b').toString();
```

**P28-29**：函数Function

可选参数，可选的命名参数，即不传这些参数也可以。

```dart
void userSettings({int age,String name}) //可选参数使用{}包括
```

必传参数，使用@required修饰

```dart
void userSettings({@required int age},String name)
```

可选的位置参数

```dart
void userSettings({int age,String name,[String interests]}) {
  if(interests != null) {
    //...
  }
}
```

默认参数，默认值时编译时常量

```dart
void userSettings({int age = 21,String name='小米'})
```



