---
title: 读书笔记『JavaScript高级程序设计』
tags: [读书笔记,JavaScript]
---

### 第一章 JavaScript 简介

**P2**：一个完整的JavaScript实现由三部分组成：核心(ECMAScript)、文档对象模型(DOM)、浏览器对象模型(BOM)

**P8**：HTML5致力于吧很多BOM功能写入正式规范。从根本上将，BOM只处理浏览器窗口和框架；但人们习惯上也把所有针对浏览器的JavaScript扩展算作BOM的一部分。

### 第二章 在HTML中使用JavaScript

**P10**：\<script>元素定义了6个属性。值得一提的有：async(可选，**表示应该立即下载脚本**，但不妨碍页面中的其他操作)；defer(可选，表示脚本可以延迟到文档完全解析和显示之后**再执行**)

**P12**：只要不存在defer和async属性，浏览器都会按照<script>元素在页面中出现的先后顺序对他们进行一次解析。因此最好把脚本的定义放到**\<body>元素内容的后面**。

**P18**：使用\<noscript>元素可以指定在不支持脚本的浏览器中显示替代内容。但是启用了脚本的情况下，浏览器不会显示\<noscript>元素中的任何内容。

### 第三章 基本概念

**P21**：ECMAScript的变量是松散类型的，所谓松散类型就是**可以用来保存任何类型的数据**。每个变量仅仅是一个用来存储值的<font color="#dd0000">**占位符**</font>而已。

**P23**：方法内变量，省略var，就变成了全局变量。ECMAScript包含6中数据类型：Undefined/Null/Boolean/Nu mber/String/Object。

**P25**：null值表示一个空对象指针，typeof null会返回object；**undefined和null的区别在于，无论适合情况下都没有必要显式地设置值为undefined，可同样的规则对null则不适用。**

**P26**：可以对**任何数据类型**的值调用Boolean()函数，而且总会返回一个Boolean值。一般用于判断值是否为空

```javascript
var msg = "Hi";
if(msg) { //自动调用Boolean()
  alert("msg is not empty")
}
```

**P31**：parseInt函数**会忽略字符串前面的空格，直到找到第一个非空字符**。如果第一个字符不是数字字符或符号，则会返回NaN。

**P33**：ECMAScript字符串是不可变的，字符串一旦创建爱你，



**P64**：ECMAScript函数**不介意传递进来多少个参数，也不在乎传进来的参数是什么数据类型**。因为ECMAScript中的参数在内部使用一个数组表示的。**函数接收到的始终都是这个数组**，而不关心数字中包含哪些参数。

### 第五章 引用类型

**P85**：对象**通常**使用点语法访问属性，但是也可以使用方括号访问属性。二者的区别是使用方括号可以使用变量访问属性：

```javascript
const propertyName = "name";
alert(person[propertyName])
```

**P86**：ECMAScript中数组中的每一项可以存储任意类型的对象，且可以动态扩展：

```javascript
var array = [1,"A",person]
alert(array[1])
array[3] = "B"
alert(array[3])
```

**P90**：数组还有栈方法和队列方法，分别用来实现栈的LIFO和队里的FIFO特性；具体是pop和push：

```javascript
var colors = ["red","blue"]
colors.push("yellow")
alert(colors.length) //3
var item = colors.pop()
alert(item) //yellow
```

**P103**：正则表达式的匹配模式支持3个标志：

​	g：表示全局(global)模式，即模式将被应用到所有字符窜，而非在发现第一个匹配项时立即停止。

​	i：表示不区分大小写(case-insensitive)模式。

​	m：表示多行(multiline)模式，即在到达一行文本末尾还会继续朝找下一行中是否存在与模式匹配的项。

**P110**：函数名仅仅是指向函数的指针，因此函数名与包含对象指针的其他变量没什么不同，一个函数可能有多个名字：

```javascript
function sum(num1,num2) {
    return num1 + num2;
}
var anotherSum = sum;
sum = null;
alert(anotherSum(10,10)); //20
```

### 第8章 BOM

**P193**：BOM的核心对象是window，它表示浏览器的一个实例。在浏览器中，window对象有双重角色，它既是通过JavaScript访问浏览器窗口的一个借口，优势ECMAScript规定的Global对象。



