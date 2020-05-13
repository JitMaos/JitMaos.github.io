---
title: 读书笔记『Flutter实战』
tags:
---

##第一章 起步

### 1.2：初识Flutter

**静态编译与动态解释**：<font color="#dd0000">**静态编译的程序在执行前全部被翻译为机器码，通常将这种类型称为AOT （Ahead of time）即 “提前编译”；而解释执行的则是一句一句边翻译边运行，通常将这种类型称为JIT（Just-in-time）即“即时编译”**</font>。AOT程序的典型代表是用C/C++开发的应用，它们必须在执行前编译成机器码，而JIT的代表则非常多，如JavaScript、python等，事实上，所有脚本语言都支持JIT模式。有些语言既可以以JIT方式运行也可以以AOT方式运行，如Java、Python，它们可以在第一次执行时编译成**中间字节码**、然后在之后执行时可以直接执行字节码，也许有人会说，中间字节码并非机器码，在程序执行时仍然需要动态将字节码转为机器码，是的，这没有错，不过通常我们区分**是否为AOT的标准就是看代码在执行之前是否需要编译，只要需要编译，无论其编译产物是字节码还是机器码，都属于AOT**。

**为什么Flutter选择Dart作为开发语言？**

1. Dart运行时和编译器支持Flutter的两个关键特性的组合：

   **基于JIT的快速开发周期**：<font color="#dd0000">**Flutter在开发阶段采用，采用JIT模式，这样就避免了每次改动都要进行编译，极大的节省了开发时间**</font>；

   **基于AOT的发布包**:<font color="#dd0000">Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能</font>。而JavaScript则不具有这个能力。

2. **高性能**

   Flutter旨在提供流畅、高保真的的UI体验。为了实现这一点，Flutter中需要能够在每个动画帧中运行大量的代码。这意味着需要一种既能提供高性能的语言，而不会出现会丢帧的周期性暂停，而Dart支持AOT，在这一点上可以做的比JavaScript更好。

3. **快速内存分配**

   Flutter框架使用函数式流，这使得它在很大程度上依赖于底层的内存分配器。因此，拥有一个能够有效地处理琐碎任务的内存分配器将显得十分重要，在缺乏此功能的语言中，Flutter将无法有效地工作。当然Chrome V8的JavaScript引擎在内存分配上也已经做的很好，事实上Dart开发团队的很多成员都是来自Chrome团队的，所以在内存分配上Dart并不能作为超越JavaScript的优势，而对于Flutter来说，它需要这样的特性，而Dart也正好满足而已。

4. **类型安全**

   **由于Dart是类型安全的语言，支持静态类型检测，所以可以在编译前发现一些类型的错误，并排除潜在问题，这一点对于前端开发者来说可能会更具有吸引力**。与之不同的，JavaScript是一个弱类型语言，也因此前端社区出现了很多给JavaScript代码添加静态类型检测的扩展语言和工具，如：微软的TypeScript以及Facebook的Flow。相比之下，Dart本身就支持静态类型，这是它的一个重要优势。

5. **Dart团队就在你身边**

   看似不起眼，实则举足轻重。由于有Dart团队的积极投入，Flutter团队可以获得更多、更方便的支持，正如Flutter官网所述“我们正与Dart社区进行密切合作，以改进Dart在Flutter中的使用。例如，当我们最初采用Dart时，该语言并没有提供生成原生二进制文件的工具链（这对于实现可预测的高性能具有很大的帮助），但是现在它实现了，因为Dart团队专门为Flutter构建了它。同样，Dart VM之前已经针对吞吐量进行了优化，但团队现在正在优化VM的延迟时间，这对于Flutter的工作负载更为重要。”

### 1.4 Dart语言简介

#### 1.4.2 函数

1. Dart函数声明如果没有显式声明返回值类型时会默认当做`dynamic`处理，**注意，函数返回值没有类型推断**：

```dart
typedef bool CALLBACK();

//不指定返回类型，此时默认为dynamic，不是bool
isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

void test(CALLBACK cb){
   print(cb()); 
}
//报错，isNoble不是bool类型
test(isNoble);
```

2.对于只包含一个表达式的函数，可以使用简写语法

```dart
bool isNoble(int atomicNumber) => _nobleGases [ atomicNumber] != null;
```

3.函数作为变量

```dart
var say = (str) {
  print(str);
};
say('hi');
```

4.函数作为参数传递

```dart
void execute(var callback) {
  callback();
}
execute(() => print("xxx")); //参数为() => print("xxx")
```

5.可选的位置参数

包装一组函数参数，用[]标记为可选的位置参数，并放在参数列表的最后面：

```dart
String say(String from, String msg, [String device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```

下面是一个不带可选参数调用这个函数的例子：

```dart
say('Bob', 'Howdy'); //结果是： Bob says Howdy
```

下面是用第三个参数调用这个函数的例子：

```dart
say('Bob', 'Howdy', 'smoke signal'); //结果是：Bob says Howdy with a smoke signal
```

6.可选的命名参数

定义函数时，使用{param1, param2, …}，放在参数列表的最后面，用于指定命名参数。例如：

```dart
//设置[bold]和[hidden]标志
void enableFlags({bool bold, bool hidden}) {
    // ... 
}
```

调用函数时，可以使用指定命名参数。例如：`paramName: value`

```dart
enableFlags(bold: true, hidden: false);
```

可选命名参数在Flutter中使用非常多。

<font color="#dd0000">**注意，不能同时使用可选的位置参数和可选的命名参数**</font>

**Future.wait**

有些时候，我们需要等待多个异步任务都执行结束后才进行一些操作，比如我们有一个界面，需要先分别从两个网络接口获取数据，获取成功后，我们需要将两个接口数据进行特定的处理后再显示到UI界面上，应该怎么做？答案是`Future.wait`，它接受一个`Future`数组参数，只有数组中所有`Future`都执行成功后，才会触发`then`的成功回调，只要有一个`Future`执行失败，就会触发错误回调。下面，我们通过模拟`Future.delayed` 来模拟两个数据获取的异步任务，等两个异步任务都执行成功时，将两个异步任务的结果拼接打印出来，代码如下：

```dart
Future.wait([
  //2秒后返回结果
  Future.delayed(new Duration(seconds:2),(){
    return "hello";
  }),
  Futuren.delayed(new Duration(seconds:4),(){
    return "world";
  })
]).then((results){
  print(result[0] + result[1]);
}).catchError((e){
  print(e);
});
```

**Stream**

`Stream` 也是用于接收异步事件数据，和`Future` 不同的是，它可以接收多个异步操作的结果（成功或失败）。 也就是说，在执行异步任务时，可以通过多次触发成功或失败事件来传递结果数据或错误异常。 `Stream` 常用于会多次读取数据的异步任务场景，如网络内容下载、文件读写等。举个例子：

```dart
Stream.fromFutures([
  //1秒后返回
  Future.delayed(new Duration(seconds:1),(){
    return "hello 1";
  }),
  //抛出一个异常
  Future.delayed(new Duration(seconds:3),(){
    return "hello 3";
  })
]).listen((data){
  print(data);
}).onError:(e) {
  print(e.message);
}.onDone(){};
```

输出

```dart
I/flutter (17666): hello 1
I/flutter (17666): Error
I/flutter (17666): hello 3
```





