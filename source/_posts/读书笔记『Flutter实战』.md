---
title: 读书笔记『Flutter实战』
tags:
---

##第一章 起步

### 1.2：初识Flutter

**静态编译与动态解释**：<font color="#dd0000">**静态编译的程序在执行前全部被翻译为机器码，通常将这种类型称为AOT （Ahead of time）即 “提前编译”；而解释执行的则是一句一句边翻译边运行，通常将这种类型称为JIT（Just-in-time）即“即时编译”**</font>。**是否为AOT的标准就是看代码在执行之前是否需要编译，只要需要编译，无论其编译产物是字节码还是机器码，都属于AOT**。

**为什么Flutter选择Dart作为开发语言？**

1. Dart运行时和编译器支持Flutter的两个关键特性的组合：

   **基于JIT的快速开发周期**：<font color="#dd0000">**Flutter在开发阶段采用，采用JIT模式，这样就避免了每次改动都要进行编译，极大的节省了开发时间**</font>；

   **基于AOT的发布包**:<font color="#dd0000">Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能</font>。而JavaScript则不具有这个能力。

2. **高性能**

   Flutter旨在提供流畅、高保真的的UI体验。为了实现这一点，Flutter中需要能够在每个动画帧中运行大量的代码。这意味着需要一种既能提供高性能的语言，而不会出现会丢帧的周期性暂停，而Dart支持AOT，在这一点上可以做的比JavaScript更好。

3. **快速内存分配**

   Flutter框架**使用函数式流**，这使得它在很大程度上依赖于底层的内存分配器。因此，拥有一个能够有效地处理琐碎任务的内存分配器将显得十分重要，在缺乏此功能的语言中，Flutter将无法有效地工作。当然Chrome V8的JavaScript引擎在内存分配上也已经做的很好，事实上Dart开发团队的很多成员都是来自Chrome团队的，所以在内存分配上Dart并不能作为超越JavaScript的优势，而对于Flutter来说，它需要这样的特性，而Dart也正好满足而已。

4. **类型安全**

   **由于Dart是类型安全的语言，支持静态类型检测，所以可以在编译前发现一些类型的错误，并排除潜在问题，这一点对于前端开发者来说可能会更具有吸引力**。与之不同的，JavaScript是一个弱类型语言，也因此前端社区出现了很多给JavaScript代码添加静态类型检测的扩展语言和工具，如：微软的TypeScript以及Facebook的Flow。相比之下，Dart本身就支持静态类型，这是它的一个重要优势。

5. **Dart团队就在你身边**

   看似不起眼，实则举足轻重。由于有Dart团队的积极投入，Flutter团队可以获得更多、更方便的支持，正如Flutter官网所述“我们正与Dart社区进行密切合作，以改进Dart在Flutter中的使用。例如，当我们最初采用Dart时，该语言并没有提供生成原生二进制文件的工具链（这对于实现可预测的高性能具有很大的帮助），但是现在它实现了，因为Dart团队专门为Flutter构建了它。同样，Dart VM之前已经针对吞吐量进行了优化，但团队现在正在优化VM的延迟时间，这对于Flutter的工作负载更为重要。”

### 1.4 Dart语言简介

#### 1.4.2 函数

1. Dart函数声明如果没有显式声明返回值类型时会默认当做dynamic处理，**注意，函数返回值没有类型推断**：

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

调用函数时，可以使用指定命名参数。例如：paramName: value

```dart
enableFlags(bold: true, hidden: false);
```

可选命名参数在Flutter中使用非常多。

<font color="#dd0000">**注意，不能同时使用可选的位置参数和可选的命名参数**</font>

<!--more-->

**Future.wait**

它接受一个Future数组参数，只有数组中所有Future都执行成功后，才会触发then的成功回调，只要有一个Future执行失败，就会触发错误回调。下面，我们通过模拟Future.delayed 来模拟两个数据获取的异步任务，等两个异步任务都执行成功时，将两个异步任务的结果拼接打印出来，代码如下：

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

Stream 也是用于接收异步事件数据，和Future不同的是，它可以接收多个异步操作的结果（成功或失败）。 也就是说，在执行异步任务时，可以通过多次触发成功或失败事件来传递结果数据或错误异常。 Stream 常用于会多次读取数据的异步任务场景，如网络内容下载、文件读写等。举个例子：

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

## 第二章 第一个Flutter应用

### 2.4 资源管理

**加载 assets**

您的应用可以通过[AssetBundle](https://docs.flutter.io/flutter/services/AssetBundle-class.html)对象访问其asset 。有两种主要方法允许从Asset bundle中加载字符串或图片（二进制）文件。

**加载文本assets**

- 通过[rootBundle](https://docs.flutter.io/flutter/services/rootBundle.html) 对象加载：每个Flutter应用程序都有一个[rootBundle](https://docs.flutter.io/flutter/services/rootBundle.html)对象， 通过它可以轻松访问**主资源包**，直接使用package:flutter/services.dart中全局静态的rootBundle对象来加载asset即可。
- 通过 [DefaultAssetBundle](https://docs.flutter.io/flutter/widgets/DefaultAssetBundle-class.html) 加载：建议使用 [DefaultAssetBundle](https://docs.flutter.io/flutter/widgets/DefaultAssetBundle-class.html) 来获取当前BuildContext的AssetBundle。 这种方法不是使用应用程序构建的默认asset bundle，而是使父级widget在运行时动态替换的不同的AssetBundle，这对于本地化或测试场景很有用。

通常，可以使用DefaultAssetBundle.of()在应用运行时来间接加载asset（例如JSON文件），而在widget上下文之外，或其它AssetBundle句柄不可用时，可以使用rootBundle直接加载这些asset，例如：

```dart
import 'dart:async' show Future;
import 'package:flutter/services.dart' show rootBundle;

Future<String> loadAsset() async {
  return await rootBundle.loadString('assets/config.json');
}
```

在设备像素比率为1.8的设备上，.../2.0x/my_icon.png 将被选择。对于2.7的设备像素比率，.../3.0x/my_icon.png将被选择。

当主资源缺少某个资源时，会按分辨率从低到高的顺序去选择 ，也就是说1x中没有的话会在2x中找，2x中还没有的话就在3x中找。

**加载图片**

要加载图片，可以使用 [AssetImage](https://docs.flutter.io/flutter/painting/AssetImage-class.html)类。例如，我们可以从上面的asset声明中加载背景图片：

```dart
Widget build(BuildContext context) {
  return new DecoratedBox(
    decoration: new BoxDecoration(
      image: new DecorationImage(
        image: new AssetImage('graphics/background.png'),
      ),
    ),
  );
}
```

注意，AssetImage 并非是一个widget， 它实际上是一个ImageProvider，有些时候你可能期望直接得到一个显示图片的widget，那么你可以使用Image.asset()方法，如：

```dart
Widget build(BuildContext context) {
  return Image.asset('graphics/background.png');
}
```

使用默认的 asset bundle 加载资源时，内部会自动处理分辨率等，这些处理对开发者来说是无感知的。 (如果使用一些更低级别的类，如 [ImageStream](https://docs.flutter.io/flutter/painting/ImageStream-class.html)或 [ImageCache](https://docs.flutter.io/flutter/painting/ImageCache-class.html) 时你会注意到有与缩放相关的参数)

## 第三章 基础组件

### 3.3.3 TextSpan

通过TextSpan实现了一个基础文本片段和一个链接片段，然后通过Text.rich 方法将TextSpan 添加到Text中，之所以可以这样做，是因为Text其实就是RichText的一个包装，而RichText是可以显示多种样式(富文本)的widget。

```dart
Text.rich(TextSpan(
    children: [
     TextSpan(
       text: "Home: "
     ),
     TextSpan(
       text: "https://flutterchina.club",
       style: TextStyle(
         color: Colors.blue
       ),  
       recognizer: _tapRecognizer
     ),
    ]
))
```

### 3.4.1 Material组件库中的按钮

Material 组件库中提供了多种按钮组件如RaisedButton、FlatButton、OutlineButton等，它们都是直接或间接对RawMaterialButton组件的包装定制，所以他们大多数属性都和RawMaterialButton一样。

所有Material 库中的按钮都有如下相同点：

1. 按下时都会有“水波动画”（又称“涟漪动画”，就是点击时按钮上会出现水波荡漾的动画）。
2. 有一个onPressed属性来设置点击回调，当按钮按下时会执行该回调，如果不提供该回调则按钮会处于禁用状态，禁用状态不响应用户点击。

**RaisedButton**

RaisedButton 即"漂浮"按钮，它默认带有阴影和灰色背景。按下后，阴影会变大。

**FlatButton**

FlatButton即扁平按钮，默认背景透明并不带阴影。按下后，会有背景色。

**OutlineButton**

OutlineButton默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和阴影(较弱)。

**IconButton**

IconButton是一个可点击的Icon，不包括文字，默认没有背景，点击后会出现背景

### 3.5.1图片

**ImageProvider**

ImageProvider 是一个抽象类，主要定义了图片数据获取的接口load()，**从不同的数据源获取图片需要实现不同的ImageProvider ，如AssetImage是实现了从Asset中加载图片的ImageProvider，而NetworkImage实现了从网络加载图片的ImageProvider**。

**Image**

Image widget有一个必选的image参数，<font color="#dd0000">**它对应一个ImageProvider**</font>。下面我们分别演示一下如何从asset和网络加载图片。

**从asset中加载图片**

1. 在工程根目录下创建一个images目录，并将图片avatar.png拷贝到该目录。

2. 在pubspec.yaml中的flutter部分添加如下内容：

   ```yaml
     assets:
       - images/avatar.png
   ```

   > 注意: 由于 yaml 文件对缩进严格，所以必须严格按照每一层两个空格的方式进行缩进，此处assets前面应有两个空格。

3. 加载该图片

   ```dart
   Image(
     image: AssetImage("images/avatar.png"),
     width: 100.0
   );
   ```

   <font color="#dd0000">**Image也提供了一个快捷的构造函数Image.asset用于从asset中加载、显示图片**</font>：

   ```dart
   Image.asset("images/avatar.png",
     width: 100.0,
   )
   ```

**从网络加载图片**

```dart
Image(
  image: NetworkImage(
      "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4"),
  width: 100.0,
)
```

<font color="#dd0000">**Image也提供了一个快捷的构造函数Image.network用于从网络加载、显示图片**</font>：

```dart
Image.network(
  "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4",
  width: 100.0,
)
```

**Image缓存**

<font color="#dd0000">**Flutter框架对加载过的图片是有缓存的（内存），默认最大缓存数量是1000，最大缓存空间为100M**</font>。

###3.5.2 ICON

在Flutter开发中，iconfont和图片相比有如下优势：

1. 体积小：可以减小安装包大小。
2. 矢量的：iconfont都是矢量图标，放大不会影响其清晰度。
3. 可以应用文本样式：可以像文本一样改变字体图标的颜色、大小对齐等。
4. 可以通过TextSpan和文本混用。

### 3.6 单选开关和复选框

Material 组件库中提供了Material风格的单选开关Switch和复选框Checkbox，<font color="#dd0000">**虽然它们都是继承自StatefulWidget，但它们本身不会保存当前选中状态，选中状态都是由父组件来管理的**</font>。当Switch或Checkbox被点击时，会触发它们的onChanged回调，我们可以在此回调中处理选中状态改变逻辑。

### 3.7.1 TextField

controller：编辑框的控制器，通过它可以设置/获取编辑框的内容、选择编辑内容、监听编辑文本改变事件。大多数情况下我们都需要显式提供一个controller来与文本框交互。如果没有提供controller，则TextField内部会自动创建一个。

focusNode：用于控制TextField是否占有当前键盘的输入焦点。它是我们和键盘交互的一个句柄（handle）。

InputDecoration：用于控制TextField的外观显示，如提示文本、背景颜色、边框等。

obscureText：是否隐藏正在编辑的文本，如用于输入密码的场景等，文本内容会用“•”替换。

onChange：输入框内容改变时的回调函数；注：内容改变事件也可以通过controller来监听。

###3.7.2 表单Form

实际业务中，在正式向服务器提交数据前，都会对各个输入框数据进行合法性校验，但是对每一个TextField都分别进行校验将会是一件很麻烦的事。还有，如果用户想清除一组TextField的内容，除了一个一个清除有没有什么更好的办法呢？为此，Flutter提供了一个Form 组件，<font color="#dd0000">**它可以对输入框进行分组，然后进行一些统一操作，如输入内容校验、输入框重置以及输入内容保存**</font>。

## 第四章 布局类组件

###4.1 布局类组件简介

布局类组件都会包含一个或多个子组件，不同的布局类组件对子组件排版(layout)方式不同。我们在前面说过Element树才是最终的绘制树，Element树是通过Widget树来创建的（通过Widget.createElement()），Widget其实就是Element的配置数据。在Flutter中，根据Widget是否需要包含子节点将Widget分为了三类，分别对应三种Element，如下表：

| Widget                        | 对应的Element                  | 用途                                                         |
| ----------------------------- | ------------------------------ | ------------------------------------------------------------ |
| LeafRenderObjectWidget        | LeafRenderObjectElement        | <font color="#dd0000">**Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类，如Image。**</font> |
| SingleChildRenderObjectWidget | SingleChildRenderObjectElement | <font color="#dd0000">**包含一个子Widget，如：ConstrainedBox、DecoratedBox等**</font> |
| MultiChildRenderObjectWidget  | MultiChildRenderObjectElement  | <font color="#dd0000">**包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等**</font> |

> 注意，Flutter中的很多Widget是直接继承自StatelessWidget或StatefulWidget，然后在build()方法中构建真正的RenderObjectWidget，如Text，它其实是继承自StatelessWidget，然后在build()方法中通过RichText来构建其子树，而RichText才是继承自MultiChildRenderObjectWidget。所以为了方便叙述，我们也可以直接说Text属于MultiChildRenderObjectWidget（其它widget也可以这么描述），这才是本质。读到这里我们也会发现，其实**StatelessWidget和StatefulWidget就是两个用于组合Widget的基类，它们本身并不关联最终的渲染对象（RenderObjectWidget）**。

布局类组件就是指直接或间接继承(包含)MultiChildRenderObjectWidget的Widget，它们一般都会有一个children属性用于接收子Widget。我们看一下继承关系 Widget > RenderObjectWidget > (Leaf/SingleChild/MultiChild)RenderObjectWidget 。

RenderObjectWidget类中定义了创建、更新RenderObject的方法，子类必须实现他们，关于RenderObject我们现在只需要知道它是最终布局、渲染UI界面的对象即可，也就是说，对于布局类组件来说，其布局算法都是通过对应的RenderObject对象来实现的，所以读者如果对接下来介绍的某个布局类组件的原理感兴趣，可以查看其对应的RenderObject的实现，比如Stack（层叠布局）对应的RenderObject对象就是RenderStack，而层叠布局的实现就在RenderStack中。

### 4.3 弹性布局（Flex）

弹性布局允许子组件按照一定比例来分配父容器空间。弹性布局的概念在其它UI系统中也都存在，如H5中的弹性盒子布局，Android中的FlexboxLayout等。Flutter中的弹性布局主要通过Flex和Expanded来配合实现。

**Flex**

Flex组件可以沿着水平或垂直方向排列子组件，如果你知道主轴方向，使用Row或Column会方便一些，因为Row和Column都继承自Flex，参数基本相同，所以能使用Flex的地方基本上都可以使用Row或Column。Flex本身功能是很强大的，它也可以和Expanded组件配合实现弹性布局。

**Expanded**

可以按比例“扩伸” Row、Column和Flex子组件所占用的空间。

### 4.4 流式布局

<font color="#dd0000">**Row默认只有一行，如果超出屏幕不会折行，超出屏幕会报错。我们把超出屏幕显示范围会自动折行的布局称为流式布局**</font>。Flutter中通过**Wrap和Flow**来支持流式布局，将上例中的Row换成Wrap后溢出部分则会自动折行

###4.4.1 Wrap

Wrap和Flex（包括Row和Column）除了超出显示范围后Wrap会折行外，其它行为基本相同。

###4.4.2 Flow

一般很少会使用Flow，因为其过于复杂，需要自己实现子widget的位置转换，在很多场景下首先要考虑的是Wrap是否满足需求。

Flow有如下优点：

- 性能好；Flow是一个对子组件尺寸以及位置调整非常高效的控件，Flow用转换矩阵在对子组件进行位置调整的时候进行了优化：在Flow定位过后，如果子组件的尺寸或者位置发生了变化，在FlowDelegate中的paintChildren()方法中调用context.paintChild 进行重绘，而context.paintChild在重绘时使用了转换矩阵，并没有实际调整组件位置。
- 灵活；由于我们需要自己实现FlowDelegate的paintChildren()方法，所以我们需要自己计算每一个组件的位置，因此，可以自定义布局策略。

缺点：

- 使用复杂。
- 不能自适应子组件大小，必须通过指定父容器大小或实现TestFlowDelegate的getSize返回固定大小。

### 4.5 层叠布局 Stack、Positioned

Flutter中使用Stack和Positioned这两个组件来配合实现绝对定位。Stack允许子组件堆叠，而Positioned用于根据Stack的四个角来确定子组件的位置。

###4.6 对齐与相对定位（Align）

如果我们只想简单的调整**一个**子元素在父元素中的位置的话，使用Align组件会更简单一些。

###4.6.2 Align和Stack对比

可以看到，Align和Stack/Positioned都可以用于指定子元素相对于父元素的偏移，但它们还是有两个主要区别：

1. 定位参考系统不同；Stack/Positioned定位的的参考系可以是父容器矩形的四个顶点；而Align则需要先通过alignment 参数来确定坐标原点，不同的alignment会对应不同原点，最终的偏移是需要通过alignment的转换公式来计算出。
2. Stack可以有多个子元素，并且子元素可以堆叠，而Align只能有一个子元素，不存在堆叠。

## 第五章 容器类组件

容器类Widget和布局类Widget都作用于其子Widget，不同的是：

- 布局类Widget一般都需要接收一个<font color="#dd0000">**widget数组（children）**</font>，他们直接或间接继承自（或包含）MultiChildRenderObjectWidget ；而容器类Widget一般只需要接收一个<font color="#dd0000">**子Widget（child）**</font>，他们直接或间接继承自（或包含）SingleChildRenderObjectWidget。
- 布局类Widget是按照一定的排列方式来对其子Widget进行排列；而容器类Widget一般只是包装其子Widget，对其添加一些修饰（补白或背景色等）、变换(旋转或剪裁等)、或限制(大小等)。

###5.1 填充（Padding）

Padding可以给其子节点添加填充（留白），和边距效果类似。我们在前面很多示例中都已经使用过它了，现在来看看它的定义：

```dart
Padding({
  ...
  EdgeInsetsGeometry padding,
  Widget child,
})
```

EdgeInsetsGeometry是一个抽象类，开发中，我们一般都使用EdgeInsets类，它是EdgeInsetsGeometry的一个子类，定义了一些设置填充的便捷方法。

### EdgeInsets

我们看看EdgeInsets提供的便捷方法：

- fromLTRB(double left, double top, double right, double bottom)：分别指定四个方向的填充。
- all(double value) : 所有方向均使用相同数值的填充。
- only({left, top, right ,bottom })：可以设置具体某个方向的填充(可以同时指定多个方向)。
- symmetric({ vertical, horizontal })：用于设置对称方向的填充，vertical指top和bottom，horizontal指left和right。

###5.2 尺寸限制类容器

尺寸限制类容器用于限制容器大小，Flutter中提供了多种这样的容器，如ConstrainedBox、SizedBox、UnconstrainedBox、AspectRatio等，本节将介绍一些常用的。

###5.2.1 ConstrainedBox

<font color="#dd0000">**onstrainedBox用于对子组件添加额外的约束**</font>。例如，如果你想让子组件的最小高度是80像素，你可以使用const BoxConstraints(minHeight: 80.0)作为子组件的约束。

#### 示例

我们先定义一个redBox，它是一个背景颜色为红色的盒子，不指定它的宽度和高度：

```dart
Widget redBox=DecoratedBox(
  decoration: BoxDecoration(color: Colors.red),
);
```

**BoxConstraints**

```dart
const BoxConstraints({
  this.minWidth = 0.0, //最小宽度
  this.maxWidth = double.infinity, //最大宽度
  this.minHeight = 0.0, //最小高度
  this.maxHeight = double.infinity //最大高度
})
```

### 5.2.2 SizedBox

实际上SizedBox只是ConstrainedBox的一个定制，上面代码等价于：

```dart
ConstrainedBox(
  constraints: BoxConstraints.tightFor(width: 80.0,height: 80.0),
  child: redBox, 
)
```

而BoxConstraints.tightFor(width: 80.0,height: 80.0)等价于：

```dart
BoxConstraints(minHeight: 80.0,maxHeight: 80.0,minWidth: 80.0,maxWidth: 80.0)
```

### 5.2.3 多重限制

```dart
ConstrainedBox(
    constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),
    child: ConstrainedBox(
      constraints: BoxConstraints(minWidth: 60.0, minHeight: 60.0),
      child: redBox,
    )
)
```

有多重限制时，对于minWidth和minHeight来说，是取父子中相应数值较大的。实际上，只有这样才能保证父限制与子限制不冲突。

###5.2.4 UnconstrainedBox

UnconstrainedBox不会对子组件产生任何限制，它允许其子组件按照其本身大小绘制。一般情况下，我们会很少直接使用此组件，但在"去除"多重限制的时候也许会有帮助

###5.3 装饰容器DecoratedBox

<font color="#dd0000">**可以在其子组件绘制前(或后)绘制一些装饰（Decoration），如背景、边框、渐变等。**</font>

DecoratedBox定义如下：

```dart
const DecoratedBox({
  Decoration decoration,
  DecorationPosition position = DecorationPosition.background,
  Widget child
})
```

- decoration：代表将要绘制的装饰，它的类型为Decoration。Decoration是一个抽象类，它定义了一个接口 createBoxPainter()，子类的主要职责是需要通过实现它来创建一个画笔，该画笔用于绘制装饰。
- position：此属性决定在哪里绘制Decoration，它接收DecorationPosition的枚举类型，该枚举类有两个值：
  - background：在子组件之后绘制，即背景装饰。
  - foreground：在子组件之上绘制，即前景。

**BoxDecoration**

我们通常会直接使用BoxDecoration类，它是一个Decoration的子类，实现了常用的装饰元素的绘制。

### 5.4 变换（Transform）

Transform可以在其子组件绘制时对其应用一些矩阵变换来实现一些特效。

Matrix4是一个4D矩阵，通过它我们可以实现各种矩阵操作，下面是一个例子：

```dart
Container(
  color: Colors.black,
  child: new Transform(
    alignment: Alignment.topRight, //相对于坐标系原点的对齐方式
    transform: new Matrix4.skewY(0.3), //沿Y轴倾斜0.3弧度
    child: new Container(
      padding: const EdgeInsets.all(8.0),
      color: Colors.deepOrange,
      child: const Text('Apartment for rent!'),
    ),
  ),
);
```

**RotatedBox**

RotatedBox和Transform.rotate功能相似，它们都可以对子组件进行旋转变换，但是有一点不同：<font color="#dd0000">**RotatedBox的变换是在layout阶段，会影响在子组件的位置和大小**</font>。

###5.5 Container

我们在前面的章节示例中多次用到过Container组件，本节我们就详细介绍一下Container组件。Container是一个组合类容器，它本身不对应具体的RenderObject，<font color="#dd0000">**它是DecoratedBox、ConstrainedBox、Transform、Padding、Align等组件组合的一个多功能容器，所以我们只需通过一个Container组件可以实现同时需要装饰、变换、限制的场景**</font>。

Container的大多数属性在介绍其它容器时都已经介绍过了，不再赘述，但有两点需要说明：

- 容器的大小可以通过width、height属性来指定，也可以通过constraints来指定；<font color="#dd0000">**如果它们同时存在时，width、height优先**</font>。实际上Container内部会根据width、height来生成一个constraints。
- <font color="#dd0000">**color和decoration是互斥的**</font>，如果同时设置它们则会报错！实际上，当指定color时，Container内会自动创建一个decoration。

##第六章 可滚动组件

###6.1 可滚动组件简介

可滚动组件都直接或间接包含一个Scrollable组件，因此它们包括一些共同的属性，为了避免重复介绍，我们在此统一介绍一下：

```dart
Scrollable({
  ...
  this.axisDirection = AxisDirection.down,
  this.controller,
  this.physics,
  @required this.viewportBuilder, //后面介绍
})
```

- axisDirection滚动方向。
- physics：此属性接受一个ScrollPhysics类型的对象，它决定可滚动组件如何响应用户操作，比如用户滑动完抬起手指后，继续执行动画；或者滑动到边界时，如何显示。默认情况下，Flutter会根据具体平台分别使用不同的ScrollPhysics对象，应用不同的显示效果，如当滑动到边界时，继续拖动的话，在iOS上会出现弹性效果，而在Android上会出现微光效果。如果你想在所有平台下使用同一种效果，可以显式指定一个固定的ScrollPhysics，Flutter SDK中包含了两个ScrollPhysics的子类，他们可以直接使用：
  - ClampingScrollPhysics：Android下微光效果。
  - BouncingScrollPhysics：iOS下弹性效果。
- controller：此属性接受一个ScrollController对象。**ScrollController的主要作用是控制滚动位置和监听滚动事件**。默认情况下，Widget树中会有一个默认的PrimaryScrollController，如果子树中的可滚动组件没有显式的指定controller，并且primary属性值为true时（默认就为true），可滚动组件会使用这个默认的PrimaryScrollController。

**Scrollbar**

Scrollbar是一个Material风格的滚动指示器（滚动条），如果要给可滚动组件添加滚动条，只需将Scrollbar作为可滚动组件的任意一个父级组件即可，如：

```dart
Scrollbar(
  child: SingleChildScrollView(
    ...
  ),
);
```

**ViewPort视口**

在很多布局系统中都有ViewPort的概念，在Flutter中，术语ViewPort（视口），如无特别说明，则是指一个Widget的实际显示区域。例如，一个ListView的显示区域高度是800像素，虽然其列表项总高度可能远远超过800像素，但是其ViewPort仍然是800像素。

**基于Sliver的延迟构建**

通常可滚动组件的子组件可能会非常多、占用的总高度也会非常大；如果要一次性将子组件全部构建出将会非常昂贵！为此，Flutter中提出一个Sliver（中文为“薄片”的意思）概念，如果一个可滚动组件支持Sliver模型，那么该滚动可以将子组件分成好多个“薄片”（Sliver），<font color="#dd0000">**只有当Sliver出现在视口中时才会去构建它，这种模型也称为“基于Sliver的延迟构建模型”**</font>。可滚动组件中有很多都支持基于Sliver的延迟构建模型，如ListView、GridView，但是也有不支持该模型的，如SingleChildScrollView。

###6.2 SingleChildScrollView

SingleChildScrollView类似于Android中的ScrollView，**它只能接收一个子组件**。

需要注意的是，通常SingleChildScrollView只应在期望的内容不会超过屏幕太多时使用，这是<font color="#dd0000">**因为SingleChildScrollView不支持基于Sliver的延迟实例化模型，所以如果预计视口可能包含超出屏幕尺寸太多的内容时，那么使用SingleChildScrollView将会非常昂贵（性能差）**</font>，此时应该使用一些支持Sliver延迟加载的可滚动组件，如ListView。

###6.3 ListView

ListView是最常用的可滚动组件之一，它可以沿一个方向线性排布所有子组件，并且它也支持基于Sliver的延迟构建模型。

```dart
ListView({
  ...  
  //可滚动widget公共参数
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  EdgeInsetsGeometry padding,

  //ListView各个构造函数的共同参数  
  double itemExtent,
  bool shrinkWrap = false,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double cacheExtent,

  //子widget列表
  List<Widget> children = const <Widget>[],
})
```

上面参数分为两组：第一组是可滚动组件的公共参数，本章第一节中已经介绍过，不再赘述；第二组是ListView各个构造函数（ListView有多个构造函数）的共同参数，我们重点来看看这些参数，：

- itemExtent：<font color="#dd0000">**该参数如果不为null，则会强制children的“长度”为itemExtent的值**</font>；这里的“长度”是指滚动方向上子组件的长度，也就是说如果滚动方向是垂直方向，则itemExtent代表子组件的高度；如果滚动方向为水平方向，则itemExtent就代表子组件的宽度。<font color="#dd0000">**在ListView中，指定itemExtent比让子组件自己决定自身长度会更高效**</font>，这是因为指定itemExtent后，滚动系统可以提前知道列表的长度，而无需每次构建子组件时都去再计算一下，尤其是在滚动位置频繁变化时（滚动系统需要频繁去计算列表高度）。
- shrinkWrap：**该属性表示是否根据子组件的总长度来设置ListView的长度，默认值为false **。默认情况下，ListView的会在滚动方向尽可能多的占用空间。当ListView在一个无边界(滚动方向上)的容器中时，shrinkWrap必须为true。
- addAutomaticKeepAlives：该属性表示是否将列表项（子组件）包裹在AutomaticKeepAlive 组件中；典型地，**在一个懒加载列表中，如果将列表项包裹在AutomaticKeepAlive中，在该列表项滑出视口时它也不会被GC（垃圾回收）**，它会使用KeepAliveNotification来保存其状态。如果列表项自己维护其KeepAlive状态，那么此参数必须置为false。
- addRepaintBoundaries：该属性表示是否将列表项（子组件）包裹在RepaintBoundary组件中。**当可滚动组件滚动时，将列表项包裹在RepaintBoundary中可以避免列表项重绘**，<font color="#dd0000">**但是当列表项重绘的开销非常小（如一个颜色块，或者一个较短的文本）时，不添加RepaintBoundary反而会更高效**</font>。和addAutomaticKeepAlive一样，如果列表项自己维护其KeepAlive状态，那么此参数必须置为false。

> 注意：上面这些参数并非ListView特有，在本章后面介绍的其它可滚动组件也可能会拥有这些参数，它们的含义是相同的。

**默认构造函数**

默认构造函数有一个children参数，它接受一个Widget列表（List）。<font color="#dd0000">**这种方式适合只有少量的子组件的情况，因为这种方式需要将所有children都提前创建好（这需要做大量工作）**</font>，而不是等到子widget真正显示的时候再创建，也就是说通过默认构造函数构建的ListView没有应用基于Sliver的懒加载模型。

```dart
ListView(
  shrinkWrap: true, 
  padding: const EdgeInsets.all(20.0),
  children: <Widget>[
    const Text('I\'m dedicating every day to you'),
    const Text('Domestic life was never quite my style'),
    const Text('When you smile, you knock me out, I fall apart'),
    const Text('And I thought I was so smart'),
  ],
);
```

**ListView.builder**

<font color="#dd0000">**ListView.builder适合列表项比较多（或者无限）的情况，因为只有当子组件真正显示的时候才会被创建，也就说通过该构造函数创建的ListView是支持基于Sliver的懒加载模型的。**</font>

```dart
ListView.builder({
  // ListView公共参数已省略  
  ...
  @required IndexedWidgetBuilder itemBuilder,
  int itemCount,
  ...
})
```

- itemBuilder：它是列表项的构建器，类型为IndexedWidgetBuilder，返回值为一个widget。**当列表滚动到具体的index位置时，会调用该构建器构建列表项**。
- itemCount：列表项的数量，**如果为null，则为无限列表**。

```dart
ListView.builder(
    itemCount: 100,
    itemExtent: 50.0, //强制高度为50.0
    itemBuilder: (BuildContext context, int index) {
      return ListTile(title: Text("$index"));
    }
);
```

**ListView.separated**

ListView.separated可以在生成的列表项之间添加一个分割组件，<font color="#dd0000">**它比ListView.builder多了一个separatorBuilder参数，该参数是一个分割组件生成器。**</font>

###6.4 GridView

GridView和ListView的大多数参数都是相同的，**唯一需要关注的是gridDelegate参数，类型是SliverGridDelegate，它的作用是控制GridView子组件如何排列(layout)**。

SliverGridDelegate是一个抽象类，定义了GridView Layout相关接口，<font color="#dd0000">**子类需要通过实现它们来实现具体的布局算法**</font>。Flutter中提供了两个SliverGridDelegate的子类SliverGridDelegateWithFixedCrossAxisCount和SliverGridDelegateWithMaxCrossAxisExtent。

**SliverGridDelegateWithFixedCrossAxisCount**

该子类实现了一个<font color="#dd0000">**横轴为固定数量**</font>子元素的layout算法

**SliverGridDelegateWithMaxCrossAxisExtent**

该子类实现了一个<font color="#dd0000">**横轴子元素为固定最大长度**</font>的layout算法

###6.5 CustomScrollView

CustomScrollView是可以使用Sliver来自定义滚动模型（效果）的组件。它可以包含多种滚动模型，举个例子，假设有一个页面，顶部需要一个GridView，底部需要一个ListView，而要求整个页面的滑动效果是统一的，即它们看起来是一个整体。如果使用GridView+ListView来实现的话，就不能保证一致的滑动效果，因为它们的滚动效果是分离的，所以这时就需要一个"胶水"，把这些彼此独立的可滚动组件"粘"起来，而CustomScrollView的功能就相当于“胶水”。

**可滚动组件的Sliver版**

Sliver在前面讲过，有细片、薄片之意，在Flutter中，Sliver通常指可滚动组件子元素（就像一个个薄片一样）。但是在CustomScrollView中，需要粘起来的可滚动组件就是CustomScrollView的Sliver了，**如果直接将ListView、GridView作为CustomScrollView是不行的，因为它们本身是可滚动组件而并不是Sliver**！因此，为了能让可滚动组件能和CustomScrollView配合使用，Flutter提供了一些可滚动组件的Sliver版，如SliverList、SliverGrid等。实际上Sliver版的可滚动组件和非Sliver版的可滚动组件最大的区别就是**前者不包含滚动模型（自身不能再滚动），而后者包含滚动模型** ，也正因如此，CustomScrollView才可以将多个Sliver"粘"在一起，这些Sliver共用CustomScrollView的Scrollable，所以最终才实现了统一的滑动效果。

## 第七章 功能型组件

# 7.1 导航返回拦截（WillPopScope）

为了避免用户误触返回按钮而导致APP退出，在很多APP中都拦截了用户点击返回键的按钮，然后进行一些防误触判断，比如当用户在某一个时间段内点击两次时，才会认为用户是要退出（而非误触）。<font color="#dd0000">**Flutter中可以通过WillPopScope来实现返回按钮拦截**</font>

```dart
const WillPopScope({
  ...
  @required WillPopCallback onWillPop,
  @required Widget child
})
```

onWillPop是一个回调函数，当用户点击返回按钮时被调用（包括导航返回按钮及Android物理返回按钮）。该回调需要<font color="#dd0000">**返回一个Future对象**</font>，**如果返回的Future最终值为false时，则当前路由不出栈(不会返回)；最终值为true时，当前路由出栈退出**。我们需要提供这个回调来决定是否退出。

###7.2 数据共享（InheritedWidget）

InheritedWidget是Flutter中非常重要的一个功能型组件，它提供了一种数据在widget树中从上到下传递、共享的方式，比如我们在应用的根widget中通过InheritedWidget共享了一个数据，那么我们便可以在任意子widget中来获取该共享的数据！这个特性在一些需要在widget树中共享数据的场景中非常方便！

> <font color="#dd0000">**InheritedWidget和React中的context功能类似，和逐级传递数据相比，它们能实现组件跨级传递数据。InheritedWidget的在widget树中数据传递方向是从上到下的，这和通知Notification（将在下一章中介绍）的传递方向正好相反。**</font>

**didChangeDependencies**

在之前介绍StatefulWidget时，我们提到State对象有一个didChangeDependencies回调，它会在“依赖”发生变化时被Flutter Framework调用。**而这个“依赖”指的就是子widget是否使用了父widget中InheritedWidget的数据**！<font color="#dd0000">**如果使用了，则代表子widget依赖有依赖InheritedWidget**</font>；如果没有使用则代表没有依赖。这种机制可以使子组件在所依赖的InheritedWidget变化时来更新自身！比如当主题、locale(语言)等发生变化时，依赖其的子widget的didChangeDependencies方法将会被调用。

**深入了解InheritedWidget**

**调用dependOnInheritedWidgetOfExactType() 和 getElementForInheritedWidgetOfExactType()的区别就是前者会注册依赖关系，而后者不会**

###7.3 跨组件状态共享（Provider）

在Flutter开发中，状态管理是一个永恒的话题。一般的原则是：**如果状态是组件私有的，则应该由组件自己管理；如果状态要跨组件共享，则该状态应该由各个组件共同的父元素来管理**。

在Flutter当中有没有更好的跨组件状态管理方式了呢？答案是肯定的，那怎么做的？我们想想前面介绍的InheritedWidget，**它的天生特性就是能绑定InheritedWidget与依赖它的子孙组件的依赖关系，并且当InheritedWidget数据发生变化时，可以自动更新依赖的子孙组件**！利用这个特性，我们可以将需要跨组件共享的状态保存在InheritedWidget中，然后在子组件中引用InheritedWidget即可，Flutter社区著名的<font color="#dd0000">**Provider包**</font>正是基于这个思想实现的一套跨组件状态共享解决方案，接下来我们便详细介绍一下Provider的用法及原理。

###7.5 异步UI更新（FutureBuilder、StreamBuilder）

很多时候我们会依赖一些异步数据来动态更新UI，比如在打开一个页面时我们需要先从互联网上获取数据，在获取数据的过程中我们显示一个加载框，等获取到数据时我们再渲染页面；又比如我们想展示Stream（比如文件流、互联网数据接收流）的进度。**Flutter专门提供了FutureBuilder和StreamBuilder两个组件来快速实现这种功能**。

## 7.5.1 FutureBuilder

FutureBuilder会依赖一个Future，<font color="#dd0000">**它会根据所依赖的Future的状态来动态构建自身**</font>。我们看一下FutureBuilder构造函数：

```dart
FutureBuilder({
  this.future,
  this.initialData,
  @required this.builder,
})
```

- future：FutureBuilder依赖的Future，**通常是一个异步耗时任务**。

- initialData：初始数据，用户设置默认数据。

- builder：Widget构建器；该构建器会在Future执行的不同阶段被多次调用，构建器签名如下：

  ```dart
  Function (BuildContext context, AsyncSnapshot snapshot)
  ```

  **snapshot会包含当前异步任务的状态信息及结果信息** ，比如我们可以通过snapshot.connectionState获取异步任务的状态信息、通过snapshot.hasError判断异步任务是否有错误等等，完整的定义读者可以查看AsyncSnapshot类定义。

  另外，FutureBuilder的builder函数签名和StreamBuilder的builder是相同的。

## 第八章 事件处理和通知

### 8.4 Notification

通知（Notification）是Flutter中一个重要的机制，在widget树中，每一个节点都可以分发通知，通知会沿着当前节点向上传递，所有父节点都可以通过NotificationListener来监听通知。Flutter中将这种由子向父的传递通知的机制称为**通知冒泡**（Notification Bubbling）。通知冒泡和用户触摸事件冒泡是相似的，但有一点不同：通知冒泡可以中止，但用户触摸事件不行。

> 通知冒泡和Web开发中浏览器事件冒泡原理是相似的，都是事件从出发源逐层向上传递，我们可以在上层节点任意位置来监听通知/事件，也可以终止冒泡过程，终止冒泡后，通知将不会再向上传递。

## 第九章 动画

###9.1 Flutter动画简介

**Flutter中动画抽象**

Flutter中也对动画进行了抽象，主要涉及Animation、Curve、Controller、Tween这四个角色。

**Animation**

**Animation是一个抽象类，它本身和UI渲染没有任何关系，而它主要的功能是保存动画的插值和状态**；Animation对象是一个在一段时间内依次生成一个区间(Tween)之间值的类。Animation对象在整个动画执行过程中输出的值可以是线性的、曲线的、一个步进函数或者任何其他曲线函数等等，这由Curve来决定。 

**动画通知**

我们可以通过Animation来监听动画每一帧以及执行状态的变化，Animation有如下两个方法：

1. addListener()；它可以用于给Animation添加帧监听器，在每一帧都会被调用。帧监听器中最常见的行为是改变状态后调用setState()来触发UI重建。
2. addStatusListener()；它可以给Animation添加“动画状态改变”监听器；动画开始、结束、正向或反向（见AnimationStatus定义）时会调用状态改变的监听器。

读者在此只需要知道帧监听器和状态监听器的区别，在后面的章节中我们将会举例说明。

**Curve**

动画过程可以是匀速的、匀加速的或者先加速后减速等。Flutter中通过Curve（曲线）来描述动画过程，我们把匀速动画称为线性的(Curves.linear)，而非匀速动画称为非线性的。

我们可以通过CurvedAnimation来指定动画的曲线，如：

```dart
final CurvedAnimation curve =
    new CurvedAnimation(parent: controller, curve: Curves.easeIn);
```

CurvedAnimation和AnimationController（下面介绍）都是Animation类型。CurvedAnimation可以通过包装AnimationController和Curve生成一个新的动画对象 ，我们正是通过这种方式来将动画和动画执行的曲线关联起来的。我们指定动画的曲线为Curves.easeIn，它表示动画开始时比较慢，结束时比较快。 [Curves](https://docs.flutter.io/flutter/animation/Curves-class.html) 类是一个预置的枚举类，定义了许多常用的曲线，下面列几种常用的：

| Curves曲线 | 动画过程                     |
| ---------- | ---------------------------- |
| linear     | 匀速的                       |
| decelerate | 匀减速                       |
| ease       | 开始加速，后面减速           |
| easeIn     | 开始慢，后面快               |
| easeOut    | 开始快，后面慢               |
| easeInOut  | 开始慢，然后加速，最后再减速 |

除了上面列举的， [Curves](https://docs.flutter.io/flutter/animation/Curves-class.html) 类中还定义了许多其它的曲线，在此便不一一介绍，读者可以自行查看Curves类定义。

当然我们也可以创建自己Curve，例如我们定义一个正弦曲线：

```dart
class ShakeCurve extends Curve {
  @override
  double transform(double t) {
    return math.sin(t * math.PI * 2);
  }
}
```

**AnimationController**

**AnimationController用于控制动画，它包含动画的启动forward()、停止stop() 、反向播放 reverse()等方法**。<font color="#dd0000">**AnimationController会在动画的每一帧，就会生成一个新的值**</font>。默认情况下，AnimationController在给定的时间段内线性的生成从0.0到1.0（默认区间）的数字。 例如，下面代码创建一个Animation对象（但不会启动动画）：

```dart
final AnimationController controller = new AnimationController(
    duration: const Duration(milliseconds: 2000), vsync: this);
```

AnimationController生成数字的区间可以通过lowerBound和upperBound来指定，如：

```dart
final AnimationController controller = new AnimationController( 
 duration: const Duration(milliseconds: 2000), 
 lowerBound: 10.0,
 upperBound: 20.0,
 vsync: this
);
```

AnimationController派生自Animation，因此可以在需要Animation对象的任何地方使用。 但是，AnimationController具有控制动画的其他方法，例如forward()方法可以启动正向动画，reverse()可以启动反向动画。在动画开始执行后开始生成动画帧，屏幕每刷新一次就是一个动画帧，在动画的每一帧，会随着根据动画的曲线来生成当前的动画值（Animation.value），然后根据当前的动画值去构建UI，当所有动画帧依次触发时，动画值会依次改变，所以构建的UI也会依次变化，所以最终我们可以看到一个完成的动画。 另外在动画的每一帧，Animation对象会调用其帧监听器，等动画状态发生改变时（如动画结束）会调用状态改变监听器。

duration表示动画执行的时长，通过它我们可以控制动画的速度。

> **注意**： 在某些情况下，动画值可能会超出AnimationController的[0.0，1.0]的范围，这取决于具体的曲线。例如，fling()函数可以根据我们手指滑动（甩出）的速度(velocity)、力量(force)等来模拟一个手指甩出动画，因此它的动画值可以在[0.0，1.0]范围之外 。也就是说，根据选择的曲线，CurvedAnimation的输出可以具有比输入更大的范围。例如，Curves.elasticIn等弹性曲线会生成大于或小于默认范围的值。

**Ticker**

当创建一个AnimationController时，需要传递一个<font color="#dd0000">**vsync参数，它接收一个TickerProvider类型的对象**</font>，它的主要职责是创建Ticker，定义如下：

```dart
abstract class TickerProvider {
  //通过一个回调创建一个Ticker
  Ticker createTicker(TickerCallback onTick);
}
```

Flutter应用在启动时都会绑定一个SchedulerBinding，通过SchedulerBinding可以给每一次屏幕刷新添加回调，而Ticker就是通过SchedulerBinding来添加屏幕刷新回调，这样一来，每次屏幕刷新都会调用TickerCallback。使用Ticker(而不是Timer)来驱动动画会防止屏幕外动画（动画的UI不在当前屏幕时，如锁屏时）消耗不必要的资源，因为Flutter中屏幕刷新时会通知到绑定的SchedulerBinding，而Ticker是受SchedulerBinding驱动的，由于锁屏后屏幕会停止刷新，所以Ticker就不会再触发。

通常我们会将SingleTickerProviderStateMixin添加到State的定义中，然后将State对象作为vsync的值，这在后面的例子中可以见到。

**Tween**

默认情况下，AnimationController对象值的范围是[0.0，1.0]。如果我们需要构建UI的动画值在不同的范围或不同的数据类型，则可以使用Tween来添加映射以生成不同的范围或数据类型的值。例如，像下面示例，Tween生成[-200.0，0.0]的值：

```dart
final Tween doubleTween = new Tween<double>(begin: -200.0, end: 0.0);
```

Tween构造函数需要begin和end两个参数。Tween的唯一职责就是定义从输入范围到输出范围的映射。输入范围通常为[0.0，1.0]，但这不是必须的，我们可以自定义需要的范围。

Tween继承自Animatable，而不是继承自Animation，Animatable中主要定义动画值的映射规则。

下面我们看一个ColorTween将动画输入范围映射为两种颜色值之间过渡输出的例子：

```dart
final Tween colorTween =
    new ColorTween(begin: Colors.transparent, end: Colors.black54);
```

Tween对象不存储任何状态，相反，它提供了evaluate(Animation animation)方法，它可以获取动画当前映射值。 Animation对象的当前值可以通过value()方法取到。evaluate函数还执行一些其它处理，例如分别确保在动画值为0.0和1.0时返回开始和结束状态。

**Tween.animate**

要使用Tween对象，需要调用其animate()方法，然后传入一个控制器对象。例如，以下代码在500毫秒内生成从0到255的整数值。

```dart
final AnimationController controller = new AnimationController(
    duration: const Duration(milliseconds: 500), vsync: this);
Animation<int> alpha = new IntTween(begin: 0, end: 255).animate(controller);
```

注意animate()返回的是一个Animation，而不是一个Animatable。

以下示例构建了一个控制器、一条曲线和一个Tween：

```dart
final AnimationController controller = new AnimationController(
    duration: const Duration(milliseconds: 500), vsync: this);
final Animation curve =
    new CurvedAnimation(parent: controller, curve: Curves.easeOut);
Animation<int> alpha = new IntTween(begin: 0, end: 255).animate(curve);
```

## 第十章 自定义组件

###10.1 自定义组件方法简介

**组合其它Widget**

**这种方式是通过拼装其它组件来组合成一个新的组件**。例如我们之前介绍的Container就是一个组合组件，它是由DecoratedBox、ConstrainedBox、Transform、Padding、Align等组件组成。

**自绘**

如果遇到无法通过现有的组件来实现需要的UI时，我们可以通过自绘组件的方式来实现。

**实现RenderObject**

Flutter提供的自身具有UI外观的组件，如文本Text、Image都是通过相应的RenderObject（我们将在“Flutter核心原理”一章中详细介绍RenderObject）渲染出来的，如Text是由RenderParagraph渲染；而Image是由RenderImage渲染。**RenderObject是一个抽象类，它定义了一个抽象方法paint(...)**：

```dart
void paint(PaintingContext context, Offset offset)
```

**PaintingContext代表组件的绘制上下文，通过PaintingContext.canvas可以获得Canvas，而绘制逻辑主要是通过Canvas API来实现**。子类需要重写此方法以实现自身的绘制逻辑，如RenderParagraph需要实现文本绘制逻辑，而RenderImage需要实现图片绘制逻辑。

可以发现，RenderObject中最终也是通过Canvas API来绘制的，那么通过实现RenderObject的方式和上面介绍的通过CustomPaint和Canvas自绘的方式有什么区别？其实答案很简单，**CustomPaint只是为了方便开发者封装的一个代理类**，它直接继承自SingleChildRenderObjectWidget，通过RenderCustomPaint的paint方法将Canvas和画笔Painter(需要开发者实现，后面章节介绍)连接起来实现了最终的绘制（绘制逻辑在Painter中）。

###10.2 组合现有组件

在Flutter中页面UI通常都是由一些低阶别的组件组合而成，当我们需要封装一些通用组件时，应该首先考虑是否可以通过组合其它组件来实现，如果可以，则应优先使用组合，因为直接通过现有组件拼装会非常简单、灵活、高效。

###10.4 自绘组件 （CustomPaint与Canvas）

对于一些复杂或不规则的UI，我们可能无法通过组合其它组件的方式来实现，比如我们需要一个正六边形、一个渐变的圆形进度条、一个棋盘等。当然，有时候我们可以使用图片来实现，但在一些需要动态交互的场景静态图片也是实现不了的，比如要实现一个手写输入面板，这时，我们就需要来自己绘制UI外观。

几乎所有的UI系统都会提供一个自绘UI的接口，这个接口通常会提供一块2D画布Canvas，Canvas内部封装了一些基本绘制的API，开发者可以通过Canvas绘制各种自定义图形。在Flutter中，提供了一个CustomPaint 组件，它可以结合画笔CustomPainter来实现自定义图形绘制。

### 第十一章 文件操作与网络请求

###11.1 文件操作

#### APP目录

Android和iOS的应用存储目录不同，[PathProvider](https://pub.dartlang.org/packages/path_provider) 插件提供了一种**平台透明**的方式来访问设备文件系统上的常用位置。该类当前支持访问两个文件系统位置：

- **临时目录:** 可以使用 getTemporaryDirectory() 来获取临时目录； 系统可随时清除的临时目录（缓存）。在iOS上，这对应于[NSTemporaryDirectory()](https://developer.apple.com/reference/foundation/1409211-nstemporarydirectory) 返回的值。在Android上，这是[getCacheDir()](https://developer.android.com/reference/android/content/Context.html#getCacheDir())返回的值。
- **文档目录:** 可以使用getApplicationDocumentsDirectory()来获取应用程序的文档目录，该目录用于存储只有自己可以访问的文件。只有当应用程序被卸载时，系统才会清除该目录。在iOS上，这对应于NSDocumentDirectory。在Android上，这是AppData目录。
- **外部存储目录**：可以使用getExternalStorageDirectory()来获取外部存储目录，如SD卡；由于iOS不支持外部目录，<font color="#dd0000">**所以在iOS下调用该方法会抛出UnsupportedError异常**</font>，而在Android下结果是android SDK中getExternalStorageDirectory的返回值。

## 第十二章 包与插件

### 12.1 开发Package

**package可以创建共享的模块化代码，一个最小的Package包括**：

- **一个pubspec.yaml文件**：声明了Package的名称、版本、作者等的元数据文件。
- **一个 lib 文件夹**：包括包中公开的(public)代码，最少应有一个.dart文件

Flutter Packages分为两类：

- Dart包：其中一些可能包含Flutter的特定功能，因此**对Flutter框架具有依赖性**，这种包仅用于Flutter，例如[fluro](https://pub.dartlang.org/packages/fluro)包。
- 插件包：一种专用的Dart包，其中包含用Dart代码编写的API，以及针对Android（使用Java或Kotlin）和针对iOS（使用OC或Swift）平台的特定实现，**也就是说插件包括原生代码**，一个具体的例子是[battery](https://pub.dartlang.org/packages/battery)插件包。

###12.2 插件开发：平台通道简介

Flutter中提供了一个平台通道（platform channel），用于Flutter和原生平台的通信。平台通道正是Flutter和原生之间通信的桥梁，它也是Flutter插件的底层基础设施。

Flutter使用了一个灵活的系统，允许您调用特定平台的API，无论在Android上的Java或Kotlin代码中，还是iOS上的ObjectiveC或Swift代码中均可用。

**Flutter与原生之间的通信依赖灵活的消息传递方式**：

- <font color="#dd0000">**应用的Flutter部分通过平台通道（platform channel）将消息发送到其应用程序的所在的宿主（iOS或Android）应用（原生应用）**</font>。
- <font color="#dd0000">**宿主监听平台通道，并接收该消息。然后它会调用该平台的API，并将响应发送回客户端，即应用程序的Flutter部分**</font>。

**平台通道**

使用平台通道在Flutter(client)和原生(host)之间传递消息，如下图所示：

![img](http://47.110.40.63:8080/img/blog/Flutter原生通信通道示意图.png)

当在Flutter中调用原生方法时，调用信息通过平台通道传递到原生，原生收到调用信息后方可执行指定的操作，如需返回数据，则原生会将数据再通过平台通道传递给Flutter。值得注意的是消息传递是异步的，这确保了用户界面在消息传递时不会被挂起。

在客户端，[MethodChannel API](https://docs.flutter.io/flutter/services/MethodChannel-class.html) 可以发送与方法调用相对应的消息。 在宿主平台上，MethodChannel 在[Android API](https://docs.flutter.io/javadoc/io/flutter/plugin/common/MethodChannel.html) 和 [FlutterMethodChannel iOS API](https://docs.flutter.io/objcdoc/Classes/FlutterMethodChannel.html)可以接收方法调用并返回结果。这些类可以帮助我们用很少的代码就能开发平台插件。

> **注意**: 如果需要，方法调用(消息传递)可以是反向的，即宿主作为客户端调用Dart中实现的API。 [quick_actions](https://pub.dartlang.org/packages/quick_actions)插件就是一个具体的例子。

**如何获取平台信息**

Flutter 中提供了一个全局变量defaultTargetPlatform来获取当前应用的平台信息，defaultTargetPlatform定义在"platform.dart"中，它的类型是TargetPlatform，这是一个枚举类，定义如下：

```dart
enum TargetPlatform {
  android,
  fuchsia,
  iOS,
}
```

可以看到目前Flutter只支持这三个平台。我们可以通过如下代码判断平台：

```dart
if(defaultTargetPlatform==TargetPlatform.android){
  // 是安卓系统，do something
  ...
}
...
```

<font color="#dd0000">**Flutter中提供了一种覆盖默认平台的机制，我们可以通过显式指定debugDefaultTargetPlatformOverride全局变量的值来指定应用平台**</font>。比如：

```dart
debugDefaultTargetPlatformOverride=TargetPlatform.iOS;
print(defaultTargetPlatform); // 会输出TargetPlatform.iOS
```

上面代码即在Android中运行后，Flutter APP就会认为是当前系统是iOS，Material组件库中所有组件交互方式都会和iOS平台对齐，defaultTargetPlatform的值也会变为TargetPlatform.iOS。

### 12.3 开发Flutter插件

下面我们通过一个**获取电池电量的插件**来介绍一下Flutter插件的开发流程。该插件中我们在Dart中通过getBatteryLevel 调用Android BatteryManager API和iOS device.batteryLevel API。

**创建一个新的应用程序项目**

首先创建一个新的应用程序:

- 在终端中运行：**flutter create batterylevel**

默认情况下，模板支持使用Java编写Android代码，或使用Objective-C编写iOS代码。要使用Kotlin或Swift，请使用-i和/或-a标志:

- 在终端中运行: flutter create -i swift -a kotlin batterylevel

**创建Flutter平台客户端**

该应用的State类拥有当前的应用状态。我们需要延长这一点以保持当前的电量

首先，我们构建通道。我们使用MethodChannel调用一个方法来返回电池电量。

通道的客户端和宿主通过**通道构造函数中传递的通道名称**进行连接。<font color="#dd0000">**单个应用中使用的所有通道名称必须是唯一的**</font>; 我们建议在通道名称前加一个唯一的“域名前缀”，例如samples.flutter.io/battery。

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
...
class _MyHomePageState extends State<MyHomePage> {
  static const platform = const MethodChannel('samples.flutter.io/battery');

  // Get battery level.
}
```

接下来，我们调用通道上的方法，指定通过字符串标识符调用方法getBatteryLevel。 该调用可能失败(平台不支持平台API，例如在模拟器中运行时)，所以我们将invokeMethod调用包装在try-catch语句中。

我们使用返回的结果，在setState中来更新用户界面状态batteryLevel。

```dart
  // Get battery level.
  String _batteryLevel = 'Unknown battery level.';

  Future<Null> _getBatteryLevel() async {
    String batteryLevel;
    try {
      final int result = await platform.invokeMethod('getBatteryLevel');
      batteryLevel = 'Battery level at $result % .';
    } on PlatformException catch (e) {
      batteryLevel = "Failed to get battery level: '${e.message}'.";
    }

    setState(() {
      _batteryLevel = batteryLevel;
    });
  }
```

最后，我们在build创建包含一个小字体显示电池状态和一个用于刷新值的按钮的用户界面。

```dart
@override
Widget build(BuildContext context) {
  return new Material(
    child: new Center(
      child: new Column(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
          new RaisedButton(
            child: new Text('Get Battery Level'),
            onPressed: _getBatteryLevel,
          ),
          new Text(_batteryLevel),
        ],
      ),
    ),
  );
}
```

至此Flutter部分的测试代码写好了，接下来我们需要实现Android和iOS平台下的API，由于平台API实现部分篇幅较大，我们将在接下来的两节中，分别介绍Android和iOS端API的实现。

###12.4 插件开发：Android端API实现

```java
package com.example.flutter_app;
import io.flutter.plugin.common.MethodCall;
import io.flutter.plugin.common.MethodChannel;
import io.flutter.plugin.common.MethodChannel.MethodCallHandler;
import io.flutter.plugin.common.MethodChannel.Result;
import android.os.Bundle;
import io.flutter.app.FlutterActivity;
import io.flutter.plugins.GeneratedPluginRegistrant;

import android.content.ContextWrapper;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.BatteryManager;
import android.os.Build.VERSION;
import android.os.Build.VERSION_CODES;

public class MainActivity extends FlutterActivity {
  private static final String CHANNEL = "samples.flutter.io/battery";

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    GeneratedPluginRegistrant.registerWith(this);
    new MethodChannel(getFlutterView(), CHANNEL).setMethodCallHandler(
      new MethodCallHandler() {
        @Override
        public void onMethodCall(MethodCall call, Result result) {
          if (call.method.equals("getBatteryLevel")) {
            int batteryLevel = getBatteryLevel();

            if (batteryLevel != -1) {
              result.success(batteryLevel);
            } else {
              result.error("UNAVAILABLE", "Battery level not available.", null);
            }
          } else {
            result.notImplemented();
          }
        }
      });
  }

  private int getBatteryLevel() {
    int batteryLevel = -1;
    if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
      BatteryManager batteryManager = (BatteryManager) getSystemService(BATTERY_SERVICE);
      batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY);
    } else {
      Intent intent = new ContextWrapper(getApplicationContext()).
              registerReceiver(null, new IntentFilter(Intent.ACTION_BATTERY_CHANGED));
      batteryLevel = (intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) * 100) /
              intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1);
    }
    return batteryLevel;
  }
}

```

###12.6 Texture和PlatformView

本节主要介绍原生和Flutter之间如何**共享图像**，以及如何在Flutter中嵌套原生组件。

###12.6.1 Texture（示例：使用摄像头）

Texture可以理解为GPU内保存将要绘制的图像数据的一个对象，Flutter engine会将Texture的数据在内存中直接进行映射（而无需在原生和Flutter之间再进行数据传递），Flutter会给每一个Texture分配一个id，同时Flutter中提供了一个Texture组件，Texture构造函数定义如下：

```dart
const Texture({
  Key key,
  @required this.textureId,
})
```

**Texture 组件正是通过textureId与Texture数据关联起来**；<font color="#dd0000">**在Texture组件绘制时，Flutter会自动从内存中找到相应id的Texture数据，然后进行绘制**</font>。可以总结一下整个流程：**图像数据先在原生部分缓存，然后在Flutter部分再通过textureId和缓存关联起来，最后绘制由Flutter完成**。

如果我们作为一个插件开发者，我们在原生代码中分配了textureId，那么在Flutter侧使用Texture组件时要如何获取textureId呢？这又回到了之前的内容了，textureId完全可以通过MethodChannel来传递。

另外，值得注意的是，当原生摄像头捕获的图像发生变化时，Texture 组件会自动重绘，这不需要我们写任何Dart 代码去控制。

### Texture用法

由于Flutter官方提供的相机（camera）插件和视频播放（video_player）插件都是使用Texture来实现的。

###12.6.2 PlatformView （示例：WebView）

如果我们在开发过程中需要使用一个原生组件，但这个原生组件在Flutter中很难实现时怎么办（如webview）？这时一个简单的方法就是将需要使用原生组件的页面全部用原生实现，在flutter中需要打开该页面时通过消息通道打开这个原生的页面。但是这种方法有一个最大的缺点，就是原生组件很难和Flutter组件进行组合。

在 Flutter 1.0版本中，Flutter SDK中新增了AndroidView和UIKitView 两个组件，<font color="#dd0000">**这两个组件的主要功能就是将原生的Android组件和iOS组件嵌入到Flutter的组件树中**</font>，这个功能是非常重要的，尤其是对一些实现非常复杂的组件，比如webview，这些组件原生已经有了，如果Flutter中要用，重新实现的话成本将非常高，所以如果有一种机制能让Flutter共享原生组件，这将会非常有用，也正因如此，Flutter才提供了这两个组件。

由于AndroidView和UIKitView 是和具体平台相关的，所以称它们为PlatformView。

## 第十四章 Flutter核心原理

###14.1 Flutter UI系统

当一帧图像绘制完毕后准备绘制下一帧时，显示器会发出一个垂直同步信号（如VSync）， 60Hz的屏幕就会一秒内发出 60次这样的信号。**而这个信号主要是用于同步CPU、GPU和显示器的**。一般地来说，计算机系统中，CPU、GPU和显示器以一种特定的方式协作：CPU将计算好的显示内容提交给 GPU，GPU渲染后放入帧缓冲区，然后视频控制器按照同步信号从帧缓冲区取帧数据传递给显示器显示。

###14.2.1 Element

UI树其实是由一个个独立的Element节点构成。我们也说过组件最终的Layout、渲染都是通过RenderObject来完成的，从创建到渲染的大体流程是：根据Widget生成Element，然后创建相应的RenderObject并关联到Element.renderObject属性上，最后再通过RenderObject来完成布局排列和绘制。

Element就是Widget在UI树具体位置的一个实例化对象，大多数Element只有唯一的renderObject，但还有一些Element会有多个子节点，如继承自RenderObjectElement的一些类，比如MultiChildRenderObjectElement。最终所有Element的RenderObject构成一棵树，我们称之为”Render Tree“即”渲染树“。总结一下，我们可以认为Flutter的UI系统包含三棵树：**Widget树、Element树、渲染树。他们的依赖关系是：Element树根据Widget树生成，而渲染树又依赖于Element树**

![img](http://47.110.40.63:8080/img/blog/Flutter三棵树依赖关系.png)

现在我们重点看一下Element，Element的生命周期如下：

1. Framework 调用Widget.createElement 创建一个Element实例，记为element
2. Framework 调用 element.mount(parentElement,newSlot) ，mount方法中首先调用element所对应Widget的createRenderObject方法创建与element相关联的RenderObject对象，然后调用element.attachRenderObject方法将element.renderObject添加到渲染树中插槽指定的位置（这一步不是必须的，一般发生在Element树结构发生变化时才需要重新attach）。插入到渲染树后的element就处于“active”状态，处于“active”状态后就可以显示在屏幕上了（可以隐藏）。
3. 当有父Widget的配置数据改变时，同时其State.build返回的Widget结构与之前不同，此时就需要重新构建对应的Element树。为了进行Element复用，在Element重新构建前会先尝试是否可以复用旧树上相同位置的element，element节点在更新前都会调用其对应Widget的canUpdate方法，如果返回true，则复用旧Element，旧的Element会使用新Widget配置数据更新，反之则会创建一个新的Element。Widget.canUpdate主要是判断newWidget与oldWidget的runtimeType和key是否同时相等，如果同时相等就返回true，否则就会返回false。根据这个原理，当我们需要强制更新一个Widget时，可以通过指定不同的Key来避免复用。
4. 当有祖先Element决定要移除element 时（如Widget树结构发生了变化，导致element对应的Widget被移除），这时该祖先Element就会调用deactivateChild 方法来移除它，移除后element.renderObject也会被从渲染树中移除，然后Framework会调用element.deactivate 方法，这时element状态变为“inactive”状态。
5. “inactive”态的element将不会再显示到屏幕。为了避免在一次动画执行过程中反复创建、移除某个特定element，“inactive”态的element在当前动画最后一帧结束前都会保留，如果在动画执行结束后它还未能重新变成“active”状态，Framework就会调用其unmount方法将其彻底移除，这时element的状态为defunct,它将永远不会再被插入到树中。
6. 如果element要重新插入到Element树的其它位置，如element或element的祖先拥有一个GlobalKey（用于全局复用元素），那么Framework会先将element从现有位置移除，然后再调用其activate方法，并将其renderObject重新attach到渲染树。

### 14.2.2 BuildContext

BuildContext就是widget对应的Element，所以我们可以通过context在StatelessWidget和StatefulWidget的build方法中直接访问Element对象。

widget树只是Element树的映射，我们完全可以直接通过Element来搭建一个UI框架。

###14.3 RenderObject和RenderBox

**每个Element都对应一个RenderObject，我们可以通过Element.renderObject 来获取。并且我们也说过RenderObject的主要职责是Layout和绘制，所有的RenderObject会组成一棵渲染树Render Tree**。

**RenderObject就是渲染树中的一个对象，它拥有一个parent和一个parentData 插槽（slot）**，所谓插槽，就是指预留的一个接口或位置，这个接口和位置是由其它对象来接入或占据的，这个接口或位置在软件中通常用预留变量来表示，而parentData正是一个预留变量，它正是由parent 来赋值的，parent通常会通过子RenderObject的parentData存储一些和子元素相关的数据，如在Stack布局中，RenderStack就会将子元素的偏移数据存储在子元素的parentData中（具体可以查看Positioned实现）。

**RenderObject类本身实现了一套基础的layout和绘制协议**，<font color="#dd0000">**但是并没有定义子节点模型**</font>（如一个节点可以有几个子节点，没有子节点？一个？两个？或者更多？）。<font color="#dd0000">**它也没有定义坐标系统**</font>（如子节点定位是在笛卡尔坐标中还是极坐标？）和<font color="#dd0000">**具体的布局协议**</font>（是通过宽高还是通过constraint和size?，或者是否由父节点在子节点布局之前或之后设置子节点的大小和位置等）。为此，**Flutter提供了一个RenderBox类，它继承自RenderObject，布局坐标系统采用笛卡尔坐标系，这和Android和iOS原生坐标系是一致的，都是屏幕的top、left是原点，然后分宽高两个轴**，大多数情况下，我们直接使用RenderBox就可以了，除非遇到要自定义布局模型或坐标系统的情况，下面我们重点介绍一下RenderBox。

###14.5 图片加载原理与缓存

Flutter框架对加载过的图片是有缓存的（内存），默认最大缓存数量是1000，最大缓存空间为100M。

###14.5.1 ImageProvider

我们已经知道Image 组件的image 参数是一个**必选参数，它是ImageProvider类型**。ImageProvider是一个抽象类，定义了图片数据获取和加载的相关接口。它的主要职责有两个：

1. **提供图片数据源**
2. <font color="#dd0000">**缓存图片**</font>

我们看看ImageProvider抽象类的详细定义：

```dart
abstract class ImageProvider<T> {
  ImageStream resolve(ImageConfiguration configuration) {
    // 实现代码省略
  }
  Future<bool> evict({ ImageCache cache,
                      ImageConfiguration configuration = ImageConfiguration.empty }) async {
    // 实现代码省略
  }

  Future<T> obtainKey(ImageConfiguration configuration); 
  @protected
  ImageStreamCompleter load(T key); // 需子类实现
}
```

**load(T key)方法**

加载图片数据源的接口，不同的数据源的加载方法不同，每个ImageProvider的子类必须实现它。比如NetworkImage类和AssetImage类，它们都是ImageProvider的子类，但它们需要从不同的数据源来加载图片数据：NetworkImage是从网络来加载图片数据，而AssetImage则是从最终的应用包里来加载（加载打到应用安装包里的资源图片）。 我们以NetworkImage为例，看看其load方法的实现：

```dart
@override
ImageStreamCompleter load(image_provider.NetworkImage key) {

  final StreamController<ImageChunkEvent> chunkEvents = StreamController<ImageChunkEvent>();

  return MultiFrameImageStreamCompleter(
    codec: _loadAsync(key, chunkEvents), //调用
    chunkEvents: chunkEvents.stream,
    scale: key.scale,
    ... //省略无关代码
  );
}
```

我们看到，load方法的返回值类型是ImageStreamCompleter ，它是一个抽象类，**定义了管理图片加载过程的一些接口**，Image Widget中正是通过它来监听图片加载状态的。

MultiFrameImageStreamCompleter 是 ImageStreamCompleter的一个子类，是flutter sdk预置的类，通过该类，我们以方便、轻松地创建出一个ImageStreamCompleter实例来做为load方法的返回值。

我们可以看到，MultiFrameImageStreamCompleter 需要一个codec参数，该参数类型为Future。**Codec 是处理图片编解码的类的一个handler，实际上，它只是一个flutter engine API 的包装类，也就是说图片的编解码逻辑不是在Dart 代码部分实现，而是在flutter engine中实现的**。Codec类部分定义如下：

```dart
@pragma('vm:entry-point')
class Codec extends NativeFieldWrapperClass2 {
  // 此类由flutter engine创建，不应该手动实例化此类或直接继承此类。
  @pragma('vm:entry-point')
  Codec._();

  /// 图片中的帧数(动态图会有多帧)
  int get frameCount native 'Codec_frameCount';

  /// 动画重复的次数
  /// * 0 表示只执行一次
  /// * -1 表示循环执行
  int get repetitionCount native 'Codec_repetitionCount';

  /// 获取下一个动画帧
  Future<FrameInfo> getNextFrame() {
    return _futurize(_getNextFrame);
  }

  String _getNextFrame(_Callback<FrameInfo> callback) native 'Codec_getNextFrame';
```

我们可以看到Codec最终的结果是一个或多个（动图）帧，而这些帧最终会绘制到屏幕上。

MultiFrameImageStreamCompleter 的 codec参数值为_loadAsync方法的返回值，我们继续看_loadAsync方法的实现：

```dart
 Future<ui.Codec> _loadAsync(
    NetworkImage key,
    StreamController<ImageChunkEvent> chunkEvents,
  ) async {
    try {
      //下载图片
      final Uri resolved = Uri.base.resolve(key.url);
      final HttpClientRequest request = await _httpClient.getUrl(resolved);
      headers?.forEach((String name, String value) {
        request.headers.add(name, value);
      });
      final HttpClientResponse response = await request.close();
      if (response.statusCode != HttpStatus.ok)
        throw Exception(...);
      // 接收图片数据 
      final Uint8List bytes = await consolidateHttpClientResponseBytes(
        response,
        onBytesReceived: (int cumulative, int total) {
          chunkEvents.add(ImageChunkEvent(
            cumulativeBytesLoaded: cumulative,
            expectedTotalBytes: total,
          ));
        },
      );
      if (bytes.lengthInBytes == 0)
        throw Exception('NetworkImage is an empty file: $resolved');
      // 对图片数据进行解码
      return PaintingBinding.instance.instantiateImageCodec(bytes);
    } finally {
      chunkEvents.close();
    }
  }
```

可以看到_loadAsync方法主要做了两件事：

1. **下载图片**。
2. <font color="#dd0000">**对下载的图片数据进行解码**</font>。

下载逻辑比较简单：通过HttpClient从网上下载图片，另外下载请求会设置一些自定义的header，开发者可以通过NetworkImage的headers命名参数来传递。

在图片下载完成后调用了PaintingBinding.instance.instantiateImageCodec(bytes)对图片进行解码，值得注意的是instantiateImageCodec(...)也是一个Native API的包装，实际上会调用Flutter engine的instantiateImageCodec方法，源码如下：

```dart
String _instantiateImageCodec(Uint8List list, _Callback<Codec> callback, _ImageInfo imageInfo, int targetWidth, int targetHeight)
  native 'instantiateImageCodec';
```

**obtainKey(ImageConfiguration)方法**

该接口主要是为了配合实现图片缓存，<font color="#dd0000">**ImageProvider从数据源加载完数据后，会在全局的ImageCache中缓存图片数据，而图片数据缓存是一个Map**</font>，而Map的key便是调用此方法的返回值，不同的key代表不同的图片数据缓存。

**resolve(ImageConfiguration) 方法**

resolve方法是ImageProvider的暴露的给Image的主入口方法，它接受一个ImageConfiguration参数，返回ImageStream，即图片数据流。我们重点看一下resolve执行流程：

```dart
ImageStream resolve(ImageConfiguration configuration) {
  ... //省略无关代码
  final ImageStream stream = ImageStream();
  T obtainedKey; //
  //定义错误处理函数
  Future<void> handleError(dynamic exception, StackTrace stack) async {
    ... //省略无关代码
    stream.setCompleter(imageCompleter);
    imageCompleter.setError(...);
  }

  // 创建一个新Zone，主要是为了当发生错误时不会干扰MainZone
  final Zone dangerZone = Zone.current.fork(...);

  dangerZone.runGuarded(() {
    Future<T> key;
    // 先验证是否已经有缓存
    try {
      // 生成缓存key，后面会根据此key来检测是否有缓存
      key = obtainKey(configuration);
    } catch (error, stackTrace) {
      handleError(error, stackTrace);
      return;
    }
    key.then<void>((T key) {
      obtainedKey = key;
      // 缓存的处理逻辑在这里，记为A，下面详细介绍
      final ImageStreamCompleter completer = PaintingBinding.instance
          .imageCache.putIfAbsent(key, () => load(key), onError: handleError);
      if (completer != null) {
        stream.setCompleter(completer);
      }
    }).catchError(handleError);
  });
  return stream;
}
```

ImageConfiguration 包含图片和设备的相关信息，如图片的大小、所在的AssetBundle(只有打到安装包的图片存在)以及当前的设备平台、devicePixelRatio（设备像素比等）。Flutter SDK提供了一个便捷函数createLocalImageConfiguration来创建ImageConfiguration 对象：

```dart
ImageConfiguration createLocalImageConfiguration(BuildContext context, { Size size }) {
  return ImageConfiguration(
    bundle: DefaultAssetBundle.of(context),
    devicePixelRatio: MediaQuery.of(context, nullOk: true)?.devicePixelRatio ?? 1.0,
    locale: Localizations.localeOf(context, nullOk: true),
    textDirection: Directionality.of(context),
    size: size,
    platform: defaultTargetPlatform,
  );
}
```

我们可以发现这些信息基本都是通过Context来获取。

上面代码A处就是处理缓存的主要代码，这里的**PaintingBinding.instance.imageCache 是 ImageCache的一个实例，它是PaintingBinding的一个属性，而Flutter框架中的PaintingBinding.instance是一个单例，imageCache事实上也是一个单例，也就是说图片缓存是全局的，统一由PaintingBinding.instance.imageCache 来管理**。

下面我们看看ImageCache类定义：

```dart
const int _kDefaultSize = 1000;
const int _kDefaultSizeBytes = 100 << 20; // 100 MiB

class ImageCache {
  // 正在加载中的图片队列
  final Map<Object, _PendingImage> _pendingImages = <Object, _PendingImage>{};
  // 缓存队列
  final Map<Object, _CachedImage> _cache = <Object, _CachedImage>{};

  // 缓存数量上限(1000)
  int _maximumSize = _kDefaultSize;
  // 缓存容量上限 (100 MB)
  int _maximumSizeBytes = _kDefaultSizeBytes;

  // 缓存上限设置的setter
  set maximumSize(int value) {...}
  set maximumSizeBytes(int value) {...}

  ... // 省略部分定义

  // 清除所有缓存
  void clear() {
    // ...省略具体实现代码
  }

  // 清除指定key对应的图片缓存
  bool evict(Object key) {
   // ...省略具体实现代码
  }


  ImageStreamCompleter putIfAbsent(Object key, ImageStreamCompleter loader(), { ImageErrorListener onError }) {
    assert(key != null);
    assert(loader != null);
    ImageStreamCompleter result = _pendingImages[key]?.completer;
    // 图片还未加载成功，直接返回
    if (result != null)
      return result;

    // 有缓存，继续往下走
    // 先移除缓存，后再添加，可以让最新使用过的缓存在_map中的位置更近一些，清理时会LRU来清除
    final _CachedImage image = _cache.remove(key);
    if (image != null) {
      _cache[key] = image;
      return image.completer;
    }
    try {
      result = loader();
    } catch (error, stackTrace) {
      if (onError != null) {
        onError(error, stackTrace);
        return null;
      } else {
        rethrow;
      }
    }
    void listener(ImageInfo info, bool syncCall) {
      final int imageSize = info?.image == null ? 0 : info.image.height * info.image.width * 4;
      final _CachedImage image = _CachedImage(result, imageSize);
      // 下面是缓存处理的逻辑
      if (maximumSizeBytes > 0 && imageSize > maximumSizeBytes) {
        _maximumSizeBytes = imageSize + 1000;
      }
      _currentSizeBytes += imageSize;
      final _PendingImage pendingImage = _pendingImages.remove(key);
      if (pendingImage != null) {
        pendingImage.removeListener();
      }

      _cache[key] = image;
      _checkCacheSize();
    }
    if (maximumSize > 0 && maximumSizeBytes > 0) {
      final ImageStreamListener streamListener = ImageStreamListener(listener);
      _pendingImages[key] = _PendingImage(result, streamListener);
      // Listener is removed in [_PendingImage.removeListener].
      result.addListener(streamListener);
    }
    return result;
  }

  // 当缓存数量超过最大值或缓存的大小超过最大缓存容量，会调用此方法清理到缓存上限以内
  void _checkCacheSize() {
   while (_currentSizeBytes > _maximumSizeBytes || _cache.length > _maximumSize) {
      final Object key = _cache.keys.first;
      final _CachedImage image = _cache[key];
      _currentSizeBytes -= image.sizeBytes;
      _cache.remove(key);
    }
    ... //省略无关代码
  }
}
```

有缓存则使用缓存，没有缓存则调用load方法加载图片，加载成功后:

1. 先判断图片数据有没有缓存，如果有，则直接返回ImageStream。
2. 如果没有缓存，则调用load(T key)方法从数据源加载图片数据，加载成功后先缓存，然后返回ImageStream。

另外，我们可以看到ImageCache类中有设置缓存上限的setter，所以，<font color="#dd0000">**如果我们可以自定义缓存上限**</font>：

```dart
 PaintingBinding.instance.imageCache.maximumSize=2000; //最多2000张
 PaintingBinding.instance.imageCache.maximumSizeBytes = 200 << 20; //最大200M
```

现在我们看一下缓存的key，因为Map中相同key的值会被覆盖，也就是说key是图片缓存的一个唯一标识，只要是不同key，那么图片数据就会分别缓存（即使事实上是同一张图片）。那么图片的唯一标识是什么呢？跟踪源码，很容易发现key正是ImageProvider.obtainKey()方法的返回值，而此方法需要ImageProvider子类去重写，这也就意味着不同的ImageProvider对key的定义逻辑会不同。其实也很好理解，比如对于NetworkImage，将图片的url作为key会很合适，而对于AssetImage，则应该将“包名+路径”作为唯一的key。下面我们以NetworkImage为例，看一下它的obtainKey()实现：

```dart
@override
Future<NetworkImage> obtainKey(image_provider.ImageConfiguration configuration) {
  return SynchronousFuture<NetworkImage>(this);
}
```

代码很简单，创建了一个同步的future，然后直接将自身做为key返回。因为Map中在判断key（此时是NetworkImage对象）是否相等时会使用“==”运算符，那么定义key的逻辑就是NetworkImage的“==”运算符：

```dart
@override
bool operator ==(dynamic other) {
  ... //省略无关代码
  final NetworkImage typedOther = other;
  return url == typedOther.url
      && scale == typedOther.scale;
}
```

很清晰，<font color="#dd0000">**对于网络图片来说，会将其“url+缩放比例”作为缓存的key。也就是说**如果两张图片的url或scale只要有一个不同，便会重新下载并分别缓存**。**</font>

另外，我们需要注意的是，图片缓存是在内存中，并没有进行本地文件持久化存储，这也是为什么网络图片在应用重启后需要重新联网下载的原因。

同时也意味着在应用生命周期内，如果缓存没有超过上限，相同的图片只会被下载一次。

