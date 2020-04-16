---
title: Flutter基础学习
tags: [Flutter]
---

学习文档：<https://www.youtube.com/playlist?list=PLt85kdOx9ozWwcy6FVOquYNMah12FTWbP>

[https://book.flutterchina.club](https://book.flutterchina.club/)

1. Dart的入口是main()、Flutter的入口在main.dart。

2. Text Widget的样式会受父控件影响，可以自定义Text的样式：

   ```dart
   return Scaffold(
     body: Row(
       children: <Widget>[
         Text(
           'TextWidget',
           style: TextStyle(
             fontSize: 20.0,
             fontWeight: FontWeight.w600,
             color: Colors.red,
             fontStyle: FontStyle.italic),
         ),
       ],
     ));
   ```

3. Container组件，包含child时，其大小就是child的，而在不包含子组件的时候，Container会去尝试fit父控件。

4. 设定padding和margin

5. 使用了(`=>`)符号，这是Dart中单行函数或方法的简写。

6. 在Flutter中，大多数东西都是widget（后同“组件”或“部件”），包括对齐(alignment)、填充(padding)和布局(layout)等，它们都是以widget的形式提供。

7. Flutter在构建页面时，会调用组件的`build`方法，widget的主要工作是提供一个build()方法来描述如何构建UI界面（通常是通过组合、拼装其它基础widget）。

8. `Scaffold` 是Material库中提供的页面脚手架，它包含导航栏和Body以及`FloatingActionButton`（如果需要的话）。 本书后面示例中，路由默认都是通过`Scaffold`创建。

   <!-- more -->

9. `home` 为Flutter应用的首页，它也是一个widget。

10. StatefulWidget中不包含build()方法，包含createElement()和createState()；StatelessWidget中包含build()和createElement()方法，不包含createState()。**Stateful widget可以拥有状态，这些状态在widget生命周期中是可以变的，而Stateless widget是不可变的。**

11. `setState`方法的作用是通知Flutter框架，有状态发生了改变，Flutter框架收到通知后，会执行`build`方法来根据新的状态重新构建界面，** Flutter 对此方法做了优化，使重新执行变的很快，所以你可以重新构建任何需要更新的东西，而无需分别去修改各个widget。**

12. #### 为什么要将build方法放在State中，而不是放在StatefulWidget中？

    + 状态访问不便
    + 继承`StatefulWidget`不便

13. 添加按钮:

    ```javascript
    Column(
          children: <Widget>[
          ... //省略无关代码
          FlatButton(
             child: Text("open new route"),
             textColor: Colors.blue,
             onPressed: () {
              //导航到新路由   
              Navigator.push( context,
               MaterialPageRoute(builder: (context) {
                  return NewRoute();
               }));
              },
             ),
           ],
     )
    ```

14. `MaterialPageRoute`继承自`PageRoute`类，`PageRoute`类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。`MaterialPageRoute` 是Material组件库提供的组件，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画。下面我们介绍一下`MaterialPageRoute` 构造函数的各个参数的意义：

    ```dart
      MaterialPageRoute({
        WidgetBuilder builder,
        RouteSettings settings,
        bool maintainState = true,
        bool fullscreenDialog = false,
      })
    ```

    - `builder` 是一个WidgetBuilder类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。我们通常要实现此回调，返回新路由的实例。
    - `settings` 包含路由的配置信息，如路由名称、是否初始路由（首页）。
    - `maintainState`：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置`maintainState`为false。
    - `fullscreenDialog`表示新的路由页面是否是一个全屏的模态对话框，在iOS中，如果`fullscreenDialog`为`true`，新页面将会从屏幕底部滑入（而不是水平方向）

15. Navigator`是一个路由管理的组件，它提供了打开和退出路由页方法。`Navigator`通过一个栈来管理活动路由集合。通常当前屏幕显示的页面就是栈顶的路由。

    ### Future push(BuildContext context, Route route)

    将给定的路由入栈（即打开新的页面），返回值是一个`Future`对象，用以接收新路由出栈（即关闭）时的返回数据。

    ### bool pop(BuildContext context, [ result ])

    将栈顶路由出栈，`result`为页面关闭时返回给上一个页面的数据。

    Navigator类中第一个参数为context的**静态方法**都对应一个Navigator的**实例方法**， 比如`Navigator.push(BuildContext context, Route route)`等价于`Navigator.of(context).push(Route route)` ，下面命名路由相关的方法也是一样的。

    所谓“命名路由”（Named Route）即有名字的路由，我们可以先给路由起一个名字，然后就可以通过路由名字直接打开新的路由了，这为路由管理带来了一种直观、简单的方式。其实注册路由表就是给路由起名字，路由表的定义如下：

    ```dart
    Map<String, WidgetBuilder> routes;
    ```

    它是一个`Map`，key为路由的名字，是个字符串；value是个`builder`回调函数，用于生成相应的路由widget。我们在通过路由名字打开新路由时，应用会根据路由名字在路由表中查找到对应的`WidgetBuilder`回调函数，然后调用该回调函数生成路由widget并返回。

    ```dart
    MaterialApp(
      title: 'Flutter Demo',
      initialRoute:"/", //名为"/"的路由作为应用的home(首页)
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      //注册路由表
      routes:{
       "new_page":(context)=>NewRoute(),
       "/":(context)=> MyHomePage(title: 'Flutter Demo Home Page'), //注册首页路由
      } 
    );
    ```

16. 在Flutter最初的版本中，命名路由是不能传递参数的，后来才支持了参数；

    我们先注册一个路由：

    ```dart
     routes:{
       "new_page":(context)=>EchoRoute(),
      } ,
    ```

    在路由页通过`RouteSetting`对象获取路由参数：

    ```dart
    class EchoRoute extends StatelessWidget {
    
      @override
      Widget build(BuildContext context) {
        //获取路由参数  
        var args=ModalRoute.of(context).settings.arguments
        //...省略无关代码
      }
    }
    ```

    在打开路由时传递参数

    ```dart
    Navigator.of(context).pushNamed("new_page", arguments: "hi");
    ```

17. 包管理

    1. 注意`dependencies`和`dev_dependencies`的区别，前者的依赖包将作为APP的源码的一部分参与编译，生成最终的安装包。而后者的依赖包只是作为开发阶段的一些工具包，主要是用于帮助我们提高开发、测试效率，比如flutter的自动化测试包等。

18. 资源管理

    1. Flutter APP安装包中会包含代码和 assets（资源）两部分。**Assets是会打包到程序安装包中的，可在运行时访问**。常见类型的assets包括静态数据（例如JSON文件）、配置文件、图标和图片（JPEG，WebP，GIF，动画WebP / GIF，PNG，BMP和WBMP）等。<font color="#dd0000">**在构建期间，Flutter将asset放置到称为 *asset bundle* 的特殊存档中，应用程序可以在运行时读取它们（但不能修改）。**</font>

    2. 构建过程支持“asset变体”的概念：不同版本的asset可能会显示在不同的上下文中。 在`pubspec.yaml`的assets部分中指定assets路径时，**构建过程中，会在相邻子目录中查找具有相同名称的任何文件。这些文件随后会与指定的asset一起被包含在asset bundle中。**

       例如，如果应用程序目录中有以下文件:

       - …/pubspec.yaml
       - …/graphics/my_icon.png
       - …/graphics/background.png
       - …/graphics/dark/background.png
       - …etc.

       然后`pubspec.yaml`文件中只需包含:

       ```
       flutter:
         assets:
           - graphics/background.png
       ```

       那么这两个`graphics/background.png`和`graphics/dark/background.png` 都将包含在您的asset bundle中。前者被认为是*main asset* （主资源），后者被认为是一种变体（variant

    3. 您的应用可以通过[`AssetBundle`](https://docs.flutter.io/flutter/services/AssetBundle-class.html)对象访问其asset 。有两种主要方法允许从Asset bundle中加载字符串或图片（二进制）文件

       - 通过[`rootBundle`](https://docs.flutter.io/flutter/services/rootBundle.html) 对象加载：每个Flutter应用程序都有一个[`rootBundle`](https://docs.flutter.io/flutter/services/rootBundle.html)对象， 通过它可以轻松访问主资源包，直接使用`package:flutter/services.dart`中全局静态的`rootBundle`对象来加载asset即可。
       - 通过 [`DefaultAssetBundle`](https://docs.flutter.io/flutter/widgets/DefaultAssetBundle-class.html) 加载：建议使用 [`DefaultAssetBundle`](https://docs.flutter.io/flutter/widgets/DefaultAssetBundle-class.html) 来获取当前BuildContext的AssetBundle。 **这种方法不是使用应用程序构建的默认asset bundle，而是使父级widget在运行时动态替换的不同的AssetBundle，这对于本地化或测试场景很有用。**

       通常，可以使用`DefaultAssetBundle.of()`在应用运行时来间接加载asset（例如JSON文件），而在widget上下文之外，或其它`AssetBundle`句柄不可用时，可以使用`rootBundle`直接加载这些asset，例如：

       ```dart
       import 'dart:async' show Future;
       import 'package:flutter/services.dart' show rootBundle;
       
       Future<String> loadAsset() async {
         return await rootBundle.loadString('assets/config.json');
       }
       ```

    4. 类似于原生开发，Flutter也可以为当前设备加载适合其分辨率的图像。

       #### 声明分辨率相关的图片 assets

       [`AssetImage`](https://docs.flutter.io/flutter/painting/AssetImage-class.html) 可以将asset的请求逻辑映射到最接近当前设备像素比例（dpi）的asset。为了使这种映射起作用，必须根据特定的目录结构来保存asset：

       - …/image.png
       - …/**M**x/image.png
       - …/**N**x/image.png
       - …etc.

       其中M和N是数字标识符，对应于其中包含的图像的分辨率，也就是说，它们指定不同设备像素比例的图片。

       主资源默认对应于1.0倍的分辨率图片。看一个例子：

       - …/my_icon.png
       - …/2.0x/my_icon.png
       - …/3.0x/my_icon.png

       在设备像素比率为1.8的设备上，`.../2.0x/my_icon.png` 将被选择。对于2.7的设备像素比率，`.../3.0x/my_icon.png`将被选择。

       如果未在`Image` widget上指定渲染图像的宽度和高度，那么`Image` widget将占用与主资源相同的屏幕空间大小。 也就是说，如果`.../my_icon.png`是72px乘72px，那么`.../3.0x/my_icon.png`应该是216px乘216px; 但如果未指定宽度和高度，它们都将渲染为72像素×72像素（以逻辑像素为单位）。

       `pubspec.yaml`中asset部分中的每一项都应与实际文件相对应，**但主资源项除外。当主资源缺少某个资源时，会按分辨率从低到高的顺序去选择 ，也就是说1x中没有的话会在2x中找，2x中还没有的话就在3x中找。**

       包也可以选择在其`lib/`文件夹中包含未在其`pubspec.yaml`文件中声明的资源。在这种情况下，对于要打包的图片，应用程序必须在`pubspec.yaml`中指定包含哪些图像。

19. 指定iOS启动图标，在Flutter项目的根目录中，导航到`.../ios/Runner`。该目录中`Assets.xcassets/AppIcon.appiconset`已经包含占位符图片（见图2-9）， 只需将它们替换为适当大小的图片，保留原始文件名称。

    更新iOS启动页，要将图片添加到启动屏幕（splash screen）的中心，请导航至`.../ios/Runner`。在`Assets.xcassets/LaunchImage.imageset`， 拖入图片，并命名为`LaunchImage.png`、`LaunchImage@2x.png`、`LaunchImage@3x.png`。

20. 调试Flutter

    1. 使用flutter analyze分析代码，这个工具是一个静态代码检查工具，它是`dartanalyzer`工具的一个包装，主要用于分析代码并帮助开发者发现可能的错误，比如，Dart分析器大量使用了代码中的类型注释来帮助追踪问题，避免`var`、无类型的参数、无类型的列表文字等。

    2. 当使用Dart Observatory（或另一个Dart调试器，例如IntelliJ IDE中的调试器）时，可以使用该`debugger()`语句插入编程式断点。要使用这个，你必须添加`import 'dart:developer';`到相关文件顶部。

       `debugger()`语句采用一个可选`when`参数，您可以指定该参数仅在特定条件为真时中断，如下所示：

       ```dart
       void someFunction(double offset) {
         debugger(when: offset > 30.0);
         // ...
       }
       ```

    3. 如果你一次输出太多，那么Android有时会丢弃一些日志行。为了避免这种情况，您可以使用Flutter的`foundation`库中的[`debugPrint()`](https://docs.flutter.io/flutter/foundation/debugPrint.html)。 这是一个封装print，它将输出限制在一个级别，避免被Android内核丢弃。

    4. 要关闭调试模式并使用发布模式，请使用`flutter run --release`运行您的应用程序。 这也关闭了Observatory调试器。一个中间模式可以关闭除Observatory之外所有调试辅助工具的，称为“profile mode”，用`--profile`替代`--release`即可。

    5. 要转储Widgets树的状态，请调用[`debugDumpApp()`](https://docs.flutter.io/flutter/widgets/debugDumpApp.html)。 只要应用程序已经构建了至少一次（即在调用`build()`之后的任何时间），您可以在应用程序未处于构建阶段（即，不在`build()`方法内调用 ）的任何时间调用此方法（在调用`runApp()`之后）。

    6. 如果您尝试调试布局问题，那么Widget树可能不够详细。在这种情况下，您可以通过调用`debugDumpRenderTree()`转储渲染树。

21. Dart线程模型及异常捕获

    Java和OC都是多线程模型的编程语言，任意一个线程触发异常且该异常未被捕获时，就会导致整个进程退出。但Dart和JavaScript不会，它们都是单线程模型，运行机制很相似(但有区别)。

    [img](https://cdn.jsdelivr.net/gh/flutterchina/flutter-in-action/docs/imgs/2-12.png)

    Dart 在单线程中是以消息循环机制来运行的，其中包含两个任务队列，一个是“微任务队列” **microtask queue**，另一个叫做“事件队列” **event queue**。从图中可以发现，微任务队列的执行优先级高于事件队列。

    在Dart中，所有的外部事件任务都在事件队列中，如IO、计时器、点击、以及绘制事件等，而微任务通常来源于Dart内部，并且微任务非常少，之所以如此，是因为微任务队列优先级高，如果微任务太多，执行时间总和就越久，事件队列任务的延迟也就越久，对于GUI应用来说最直观的表现就是比较卡，所以必须得保证微任务队列不会太长。值得注意的是，我们可以通过`Future.microtask(…)`方法向微任务队列插入一个任务。

    **在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是当前任务的后续代码就不会被执行了，也就是说一个任务中的异常是不会影响其它任务执行的。**

    Flutter 框架为我们在很多关键的方法进行了异常捕获。这里举一个例子，当我们布局发生越界或不合规范时，Flutter就会自动弹出一个错误界面，这是因为Flutter已经在执行build方法时添加了异常捕获，最终的源码如下：

    这样我们就可以处理那些Flutter为我们捕获的异常了，接下来我们看看如何捕获其它异常。

    ```dart
    @override
    void performRebuild() {
     ...
      try {
        //执行build方法  
        built = build();
      } catch (e, stack) {
        // 有异常时则弹出错误提示  
        built = ErrorWidget.builder(_debugReportException('building $this', e, stack));
      } 
      ...
    }
    ```

    我们发现`onError`是`FlutterError`的一个静态属性，它有一个默认的处理方法 `dumpErrorToConsole`，到这里就清晰了，如果我们想自己上报异常，只需要提供一个自定义的错误处理回调即可，如：

    ```dart
    void main() {
      FlutterError.onError = (FlutterErrorDetails details) {
        reportError(details);
      };
     ...
    }
    ```

    我们最终的异常捕获和上报代码大致如下：

    ```dart
    void collectLog(String line){
        ... //收集日志
    }
    void reportErrorAndLog(FlutterErrorDetails details){
        ... //上报错误和日志逻辑
    }
    
    FlutterErrorDetails makeDetails(Object obj, StackTrace stack){
        ...// 构建错误信息
    }
    
    void main() {
      FlutterError.onError = (FlutterErrorDetails details) {
        reportErrorAndLog(details);
      };
      runZoned(
        () => runApp(MyApp()),
        zoneSpecification: ZoneSpecification(
          print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
            collectLog(line); // 收集日志
          },
        ),
        onError: (Object obj, StackTrace stack) {
          var details = makeDetails(obj, stack);
          reportErrorAndLog(details);
        },
      );
    }
    ```



### 组件基础

1. Widget简介

   在Flutter中，Widget的功能是“描述一个UI元素的配置数据”，它就是说，Widget其实并不是表示最终绘制在设备屏幕上的显示元素，而它只是描述显示元素的一个配置数据。

   实际上，<font color="#dd0000">**Flutter中真正代表屏幕上显示元素的类是`Element`，也就是说Widget只是描述`Element`的配置数据！**</font>**Widget只是UI元素的一个配置数据，并且一个Widget可以对应多个Element**。

2. 我们先来看一下Widget类的声明：

   ```dart
   @immutable
   abstract class Widget extends DiagnosticableTree {
     const Widget({ this.key });
     final Key key;
   
     @protected
     Element createElement();
   
     @override
     String toStringShort() {
       return key == null ? '$runtimeType' : '$runtimeType-$key';
     }
   
     @override
     void debugFillProperties(DiagnosticPropertiesBuilder properties) {
       super.debugFillProperties(properties);
       properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
     }
   
     static bool canUpdate(Widget oldWidget, Widget newWidget) {
       return oldWidget.runtimeType == newWidget.runtimeType
           && oldWidget.key == newWidget.key;
     }
   }
   ```

   - `Widget`类继承自`DiagnosticableTree`，`DiagnosticableTree`即“诊断树”，主要作用是提供调试信息。
   - `Key`: 这个`key`属性类似于React/Vue中的`key`，主要的作用是决定是否在下一次`build`时复用旧的widget，决定的条件在`canUpdate()`方法中。
   - `createElement()`：正如前文所述“一个Widget可以对应多个`Element`”；Flutter Framework在构建UI树时，会先调用此方法生成对应节点的`Element`对象。此方法是Flutter Framework隐式调用的，在我们开发过程中基本不会调用到。
   - `debugFillProperties(...)` 复写父类的方法，主要是设置诊断树的一些特性。
   - `canUpdate(...)`是一个静态方法，它主要用于在Widget树重新`build`时复用旧的widget，其实具体来说，应该是：是否用新的Widget对象去更新旧UI树上所对应的`Element`对象的配置；通过其源码我们可以看到，只要`newWidget`与`oldWidget`的`runtimeType`和`key`同时相等时就会用`newWidget`去更新`Element`对象的配置，否则就会创建新的`Element`。

   3. context

      `build`方法有一个`context`参数，它是`BuildContext`类的一个实例，表示当前widget在widget树中的上下文，每一个widget都会对应一个context对象（因为每一个widget都是widget树上的一个节点）。<font color="#dd0000">**实际上，`context`是当前widget在widget树中位置中执行”相关操作“的一个句柄，比如它提供了从当前widget开始向上遍历widget树以及按照widget类型查找父级widget的方法。**</font>

   4. StatefulWidget

      和`StatelessWidget`一样，`StatefulWidget`也是继承自`Widget`类，并重写了`createElement()`方法，不同的是返回的`Element` 对象并不相同；另外`StatefulWidget`类中添加了一个新的接口`createState()`。

   5. State

      一个StatefulWidget类会对应一个State类，State表示与其对应的StatefulWidget要维护的状态，State中的保存的状态信息可以：

      1. 在widget 构建时可以被同步读取。
      2. 在widget生命周期中可以被改变，**当State被改变时，可以手动调用其`setState()`方法通知Flutter framework状态发生改变**，Flutter framework在收到消息后，会重新调用其`build`方法重新构建widget树，从而达到更新UI的目的。
      3. State中有两个常用属性：
         1. `widget`，它表示与该State实例关联的widget实例，由Flutter framework动态设置。注意，这种关联并非永久的，因为在应用声明周期中，UI树上的某一个节点的widget实例在重新构建时可能会变化，但State实例只会在第一次插入到树中时被创建，当在重新构建时，如果widget被修改了，Flutter framework会动态设置State.widget为新的widget实例。
         2. `context`。StatefulWidget对应的BuildContext，作用同StatelessWidget的BuildContext。

   6. State生命周期

   7. 在widget树中获取State对象的三种方法

      1. 通过Context获取

         ```dart
         ScaffoldState _state = context.ancestorStateOfType(
                       TypeMatcher<ScaffoldState>());
         ```

      2. 通过静态方法of获取

         ```dart
         ScaffoldState _state=Scaffold.of(context);
         ```

      3. 通过GlobalKey获取

         ```dart
         static GlobalKey<ScaffoldState> _globalKey= GlobalKey();
         ...
         Scaffold(
             key: _globalKey , //设置key
             ...  
         )
         _globalKey.currentState.openDrawer()
         ```

   8. 状态管理

      如何决定使用哪种管理方法？下面是官方给出的一些原则可以帮助你做决定：

      - 如果状态是用户数据，如复选框的选中状态、滑块的位置，则该状态最好由父Widget管理。
      - 如果状态是有关界面外观效果的，例如颜色、动画，那么状态最好由Widget本身来管理。
      - 如果某一个状态是不同Widget共享的则最好由它们共同的父Widget管理。


### 基础组件

1. 文本、字体样式

   `textAlign`：文本的对齐方式；可以选择左对齐、右对齐还是居中。

   `maxLines`、`overflow`：指定文本显示的最大行数，默认情况下，文本是自动折行的，如果指定此参数，则文本最多不会超过指定的行。如果有多余的文本，可以通过`overflow`来指定截断方式，默认是直接截断，本例中指定的截断方式`TextOverflow.ellipsis`，它会将多余文本截断后以省略符“...”表示；TextOverflow的其它截断方式请参考SDK文档。

   `textScaleFactor`：代表文本相对于当前字体大小的缩放因子，相对于去设置文本的样式`style`属性的`fontSize`，它是调整字体大小的一个快捷方式。

   如果我们需要对一个Text内容的不同部分按照不同的样式显示，这时就可以使用`TextSpan`，它代表文本的一个“片段”。

2. DefaultTextStyle

   在Widget树中，文本的样式默认是可以被继承的（子类文本类组件未指定具体样式时可以使用Widget树中父级设置的默认样式），因此，如果在Widget树的某一个节点处设置一个默认的文本样式，那么该节点的子树中所有文本都会默认使用这个样式，而`DefaultTextStyle`正是用于设置默认文本样式的。

   ```dart
   DefaultTextStyle(
     //1.设置文本默认样式  
     style: TextStyle(
       color:Colors.red,
       fontSize: 20.0,
     ),
     textAlign: TextAlign.start,
     child: Column(
       crossAxisAlignment: CrossAxisAlignment.start,
       children: <Widget>[
         Text("hello world"),
         Text("I am Jack"),
         Text("I am Jack",
           style: TextStyle(
             inherit: false, //2.不继承默认样式
             color: Colors.grey
           ),
         ),
       ],
     ),
   );
   ```

3. 图片

   Flutter中，我们可以通过`Image`组件来加载并显示图片，`Image`的数据源可以是asset、文件、内存以及网络。

   ### ImageProvider

   `ImageProvider` 是一个抽象类，主要定义了图片数据获取的接口`load()`，从不同的数据源获取图片需要实现不同的`ImageProvider` ，如`AssetImage`是实现了从Asset中加载图片的ImageProvider，而`NetworkImage`实现了从网络加载图片的ImageProvider。

   ### Image

   `Image` widget有一个必选的`image`参数，它对应一个ImageProvider。下面我们分别演示一下如何从asset和网络加载图片。

   ```dart
   //从Assets中加载图片
   Image.asset("images/avatar.png",
     width: 100.0,
   )
   //从网络加载图片
   Image.network(
     "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4",
     width: 100.0,
   )
   ```

   ### Image缓存

   Flutter框架对加载过的图片是有缓存的（内存），默认最大缓存数量是1000，最大缓存空间为100M。关于Image的详细内容及原理我们将会在后面进阶部分深入介绍。

   Flutter中，可以像Web开发一样使用iconfont，iconfont即“字体图标”，它是将图标做成字体文件，然后通过指定不同的字符而显示不同的图片。

4. ICON

   Flutter中，可以像Web开发一样使用iconfont，iconfont即“字体图标”，它是将图标做成字体文件，然后通过指定不同的字符而显示不同的图片。

   > 在字体文件中，每一个字符都对应一个位码，而每一个位码对应一个显示字形，不同的字体就是指字形不同，即字符对应的字形是不同的。而在iconfont中，只是将位码对应的字形做成了图标，所以不同的字符最终就会渲染成不同的图标。

   在Flutter开发中，iconfont和图片相比有如下优势：

   1. 体积小：可以减小安装包大小。
   2. 矢量的：iconfont都是矢量图标，放大不会影响其清晰度。
   3. 可以应用文本样式：可以像文本一样改变字体图标的颜色、大小对齐等。
   4. **可以通过TextSpan和文本混用。**

   Flutter封装了`IconData`和`Icon`来专门显示字体图标，上面的例子也可以用如下方式实现：

   ```dart
   Row(
     mainAxisAlignment: MainAxisAlignment.center,
     children: <Widget>[
       Icon(Icons.accessible,color: Colors.green,),
       Icon(Icons.error,color: Colors.green,),
       Icon(Icons.fingerprint,color: Colors.green,),
     ],
   )
   ```

   `Icons`类中包含了所有Material Design图标的`IconData`静态变量定义。,

5. 单选开关和复选框

   Material 组件库中提供了Material风格的单选开关`Switch`和复选框`Checkbox`，<font color="#dd0000">**虽然它们都是继承自`StatefulWidget`，但它们本身不会保存当前选中状态，选中状态都是由父组件来管理的。**</font>当`Switch`或`Checkbox`被点击时，会触发它们的`onChanged`回调，我们可以在此回调中处理选中状态改变逻辑。

   值得一提的是`Checkbox`有一个属性`tristate` ，表示是否为三态，其默认值为`false` ，这时Checkbox有两种状态即“选中”和“不选中”，对应的value值为`true`和`false` 。如果`tristate`值为`true`时，value的值会增加一个状态`null`，读者可以自行了解。

6. 输入框及表单

   `TextField`用于文本输入，它提供了很多属性，我们先简单介绍一下主要属性的作用，然后通过几个示例来演示一下关键属性的用法。

7. 进度指示器

   Material 组件库中提供了两种进度指示器：`LinearProgressIndicator`和`CircularProgressIndicator`，它们都可以同时用于精确的进度指示和模糊的进度指示。

   ### LinearProgressIndicator

   `LinearProgressIndicator`是一个线性、条状的进度条，定义如下：

   ```dart
   LinearProgressIndicator({
     double value,
     Color backgroundColor,
     Animation<Color> valueColor,
     ...
   })
   ```

   - `value`：`value`表示当前的进度，取值范围为[0,1]；**如果`value`为`null`时则指示器会执行一个循环动画（模糊进度）；当`value`不为`null`时，指示器为一个具体进度的进度条。**
   - `backgroundColor`：指示器的背景色。
   - `valueColor`: 指示器的进度条颜色；值得注意的是，该值类型是`Animation<Color>`，这允许我们对进度条的颜色也可以指定动画。如果我们不需要对进度条颜色执行动画，换言之，我们想对进度条应用一种固定的颜色，此时我们可以通过`AlwaysStoppedAnimation`来指定。

### 布局类组件

布局类组件都会包含一个或多个子组件，不同的布局类组件对子组件排版(layout)方式不同。我们在前面说过**`Element`树才是最终的绘制树，`Element`树是通过Widget树来创建的（通过`Widget.createElement()`），Widget其实就是Element的配置数据。**在Flutter中，根据Widget**是否需要包含子节点**将Widget分为了三类，分别对应三种Element，如下表：

| Widget                        | 对应的Element                  | 用途                                                         |
| ----------------------------- | ------------------------------ | ------------------------------------------------------------ |
| LeafRenderObjectWidget        | LeafRenderObjectElement        | **Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类，如Image。** |
| SingleChildRenderObjectWidget | SingleChildRenderObjectElement | **包含一个子Widget，如：ConstrainedBox、DecoratedBox等**     |
| MultiChildRenderObjectWidget  | MultiChildRenderObjectElement  | **包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等** |

> 注意，Flutter中的很多Widget是直接继承自StatelessWidget或StatefulWidget，然后在`build()`方法中构建真正的RenderObjectWidget，如Text，它其实是继承自StatelessWidget，然后在`build()`方法中通过RichText来构建其子树，而RichText才是继承自MultiChildRenderObjectWidget。所以为了方便叙述，我们也可以直接说Text属于MultiChildRenderObjectWidget（其它widget也可以这么描述），这才是本质。读到这里我们也会发现，其实**StatelessWidget和StatefulWidget就是两个用于组合Widget的基类，它们本身并不关联最终的渲染对象（RenderObjectWidget）**。

<font color="#dd0000">**布局类组件就是指直接或间接继承(包含)`MultiChildRenderObjectWidget`的Widget，它们一般都会有一个`children`属性用于接收子Widget**</font>。我们看一下继承关系 Widget > RenderObjectWidget > (Leaf/SingleChild/MultiChild)RenderObjectWidget 。

`RenderObjectWidget`类中定义了创建、更新`RenderObject`的方法，子类必须实现他们，关于`RenderObject`我们现在只需要知道它是最终布局、渲染UI界面的对象即可，也就是说，对于布局类组件来说，其布局算法都是通过对应的`RenderObject`对象来实现的，所以读者如果对接下来介绍的某个布局类组件的原理感兴趣，可以查看其对应的`RenderObject`的实现，比如`Stack`（层叠布局）对应的`RenderObject`对象就是`RenderStack`，而层叠布局的实现就在`RenderStack`中。

1. 线性布局(Row和Column)

   所谓线性布局，即指沿水平或垂直方向排布子组件。Flutter中通过`Row`和`Column`来实现线性布局，类似于Android中的`LinearLayout`控件。**`Row`和`Column`都继承自`Flex`**。

   对于线性布局，有主轴和纵轴之分，如果布局是沿水平方向，那么主轴就是指水平方向，而纵轴即垂直方向；如果布局沿垂直方向，那么主轴就是指垂直方向，而纵轴就是水平方向。在线性布局中，有两个定义对齐方式的枚举类`MainAxisAlignment`和`CrossAxisAlignment`，分别代表主轴对齐和纵轴对齐。

2. Row

   主要属性：

   - `textDirection`：表示水平方向子组件的布局顺序(是从左往右还是从右往左)，默认为系统当前Locale环境的文本方向(如中文、英语都是从左往右，而阿拉伯语是从右往左)。
   - `mainAxisSize`：表示`Row`在主轴(水平)方向占用的空间，默认是`MainAxisSize.max`，表示尽可能多的占用水平方向的空间，此时无论子widgets实际占用多少水平空间，`Row`的宽度始终等于水平方向的最大宽度；而`MainAxisSize.min`表示尽可能少的占用水平空间，当子组件没有占满水平剩余空间，则`Row`的实际宽度等于所有子组件占用的的水平空间；
   - `mainAxisAlignment`：表示子组件在`Row`所占用的水平空间内对齐方式，如果`mainAxisSize`值为`MainAxisSize.min`，则此属性无意义，因为子组件的宽度等于`Row`的宽度。只有当`mainAxisSize`的值为`MainAxisSize.max`时，此属性才有意义，`MainAxisAlignment.start`表示沿`textDirection`的初始方向对齐，如`textDirection`取值为`TextDirection.ltr`时，则`MainAxisAlignment.start`表示左对齐，`textDirection`取值为`TextDirection.rtl`时表示从右对齐。而`MainAxisAlignment.end`和`MainAxisAlignment.start`正好相反；`MainAxisAlignment.center`表示居中对齐。读者可以这么理解：`textDirection`是`mainAxisAlignment`的参考系。
   - `verticalDirection`：**表示`Row`纵轴（垂直）的对齐方向，默认是`VerticalDirection.down`，表示从上到下。**
   - `crossAxisAlignment`：表示子组件在纵轴方向的对齐方式，`Row`的高度等于子组件中最高的子元素高度，它的取值和`MainAxisAlignment`一样(包含`start`、`end`、 `center`三个值)，<font color="#dd0000">**不同的是`crossAxisAlignment`的参考系是`verticalDirection`，即`verticalDirection`值为`VerticalDirection.down`时`crossAxisAlignment.start`指顶部对齐，`verticalDirection`值为`VerticalDirection.up`时，`crossAxisAlignment.start`指底部对齐；而`crossAxisAlignment.end`和`crossAxisAlignment.start`正好相反；**</font>

3. 弹性布局

   1. 弹性布局允许子组件按照一定比例来分配父容器空间。弹性布局的概念在其它UI系统中也都存在，如H5中的弹性盒子布局，Android中的`FlexboxLayout`等。Flutter中的弹性布局主要通过`Flex`和`Expanded`来配合实现。

      `Flex`组件可以沿着水平或垂直方向排列子组件，如果你知道主轴方向，使用`Row`或`Column`会方便一些，因为<font color="#dd0000">**`Row`和`Column`都继承自`Flex`**</font>，参数基本相同，所以能使用Flex的地方基本上都可以使用`Row`或`Column`。**`Flex`本身功能是很强大的，它也可以和`Expanded`组件配合实现弹性布局。**

   2. `Spacer`的功能是占用指定比例的空间，实际上它只是`Expanded`的一个包装类

4. 流式布局

   `Wrap`和`Flex`（包括`Row`和`Column`）除了超出显示范围后`Wrap`会折行外，其它行为基本相同。

   - `spacing`：主轴方向子widget的间距

   - `runSpacing`：纵轴方向的间距

   - `runAlignment`：纵轴方向的对齐方式

   我们一般很少会使用`Flow`，因为其过于复杂，需要自己实现子widget的位置转换，在很多场景下首先要考虑的是`Wrap`是否满足需求。`Flow`主要用于一些需要自定义布局策略或性能要求较高(如动画中)的场景。

   `Flow`有如下优点：

   - 性能好；`Flow`是一个对子组件尺寸以及位置调整非常高效的控件，`Flow`用转换矩阵在对子组件进行位置调整的时候进行了优化：在`Flow`定位过后，如果子组件的尺寸或者位置发生了变化，在`FlowDelegate`中的`paintChildren()`方法中调用`context.paintChild` 进行重绘，而`context.paintChild`在重绘时使用了转换矩阵，并没有实际调整组件位置。
   - 灵活；由于我们需要自己实现`FlowDelegate`的`paintChildren()`方法，所以我们需要自己计算每一个组件的位置，因此，可以自定义布局策略。

   缺点：

   - 使用复杂。

   - <font color="#dd0000">**不能自适应子组件大小，必须通过指定父容器大小或实现`TestFlowDelegate`的`getSize`返回固定大小。**</font>

5. 层叠布局

   层叠布局和Web中的绝对定位、Android中的Frame布局是相似的，子组件可以根据距父容器四个角的位置来确定自身的位置。绝对定位允许子组件堆叠起来（按照代码中声明的顺序）。Flutter中使用`Stack`和`Positioned`这两个组件来配合实现绝对定位。`Stack`允许子组件堆叠，而`Positioned`用于根据`Stack`的四个角来确定子组件的位置。

   1. Stack

      ```dart
      Stack({
        this.alignment = AlignmentDirectional.topStart,
        this.textDirection,
        this.fit = StackFit.loose,
        this.overflow = Overflow.clip,
        List<Widget> children = const <Widget>[],
      })
      ```

      - `alignment`：此参数决定如何去对齐没有定位（没有使用`Positioned`）或部分定位的子组件。所谓部分定位，在这里**特指没有在某一个轴上定位：**`left`、`right`为横轴，`top`、`bottom`为纵轴，只要包含某个轴上的一个定位属性就算在该轴上有定位。
      - `textDirection`：和`Row`、`Wrap`的`textDirection`功能一样，都用于确定`alignment`对齐的参考系，即：`textDirection`的值为`TextDirection.ltr`，则`alignment`的`start`代表左，`end`代表右，即`从左往右`的顺序；`textDirection`的值为`TextDirection.rtl`，则alignment的`start`代表右，`end`代表左，即`从右往左`的顺序。
      - `fit`：此参数用于确定**没有定位**的子组件如何去适应`Stack`的大小。`StackFit.loose`表示使用子组件的大小，`StackFit.expand`表示扩伸到`Stack`的大小。
      - `overflow`：此属性决定如何显示超出`Stack`显示空间的子组件；值为`Overflow.clip`时，超出部分会被剪裁（隐藏），值为`Overflow.visible` 时则不会。

   2. Positioned

      ```dart
      const Positioned({
        Key key,
        this.left, 
        this.top,
        this.right,
        this.bottom,
        this.width,
        this.height,
        @required Widget child,
      })
      ```

      `left`、`top` 、`right`、 `bottom`分别代表离`Stack`左、上、右、底四边的距离。`width`和`height`用于指定需要定位元素的宽度和高度。注意，`Positioned`的`width`、`height` 和其它地方的意义稍微有点区别，此处用于配合`left`、`top` 、`right`、 `bottom`来定位组件，**举个例子，在水平方向时，你只能指定`left`、`right`、`width`三个属性中的两个，如指定`left`和`width`后，`right`会自动算出(`left`+`width`)，**<font color="#dd0000">**如果同时指定三个属性则会报错，垂直方向同理**</font>。

      **未定位的Widget会占满Stack整个空间**：

      ```dart
      Stack(
        alignment:Alignment.center ,
        fit: StackFit.expand, //未定位widget占满Stack整个空间
        children: <Widget>[
          Positioned(
            left: 18.0,
            child: Text("I am Jack"),
          ),
          Container(child: Text("Hello world",style: TextStyle(color: Colors.white)),
            color: Colors.red,
          ),
          Positioned(
            top: 18.0,
            child: Text("Your friend"),
          )
        ],
      ),
      ```

      因为第二个组件没有指定位置，所以它会占满整个stack，且它位于第三个组件上层，所以会遮罩第三个组件。

6. 对齐与相对定位

### 容器类组件

容器类Widget和布局类Widget都作用于其子Widget，不同的是：

- 布局类Widget一般都需要接收一个widget数组（children），他们直接或间接继承自（或包含）MultiChildRenderObjectWidget ；而容器类Widget一般只需要接收一个子Widget（child），他们直接或间接继承自（或包含）SingleChildRenderObjectWidget。
- 布局类Widget是按照一定的排列方式来对其子Widget进行排列；而容器类Widget一般只是包装其子Widget，对其添加一些修饰（补白或背景色等）、变换(旋转或剪裁等)、或限制(大小等)。

#### 填充(Padding)

`Padding`可以给其子节点添加填充（留白），和边距效果类似。

```dart
Padding({
  ...
  EdgeInsetsGeometry padding,
  Widget child,
})
```

`EdgeInsetsGeometry`是一个抽象类，开发中，我们一般都使用`EdgeInsets`类，它是`EdgeInsetsGeometry`的一个子类，定义了一些设置填充的便捷方法。

##### EdgeInsets

我们看看`EdgeInsets`提供的便捷方法：

- `fromLTRB(double left, double top, double right, double bottom)`：分别指定四个方向的填充。
- `all(double value)` : 所有方向均使用相同数值的填充。
- `only({left, top, right ,bottom })`：可以设置具体某个方向的填充(可以同时指定多个方向)。
- `symmetric({ vertical, horizontal })`：**用于设置对称方向的填充**，`vertical`指`top`和`bottom`，`horizontal`指`left`和`right`。

#### 尺寸限制类容器

尺寸限制类容器用于限制容器大小，Flutter中提供了多种这样的容器，如`ConstrainedBox`、`SizedBox` 、`UnconstrainedBox`、`AspectRatio`等。

##### ConstrainedBox

`ConstrainedBox`用于对子组件添加<font color="#dd0000">**额外**</font>的约束。例如，如果你想让子组件的最小高度是80像素，你可以使用`const BoxConstraints(minHeight: 80.0)`作为子组件的约束。

##### BoxConstraints

BoxConstraints用于设置限制条件，它的定义如下：

```dart
const BoxConstraints({
  this.minWidth = 0.0, //最小宽度
  this.maxWidth = double.infinity, //最大宽度
  this.minHeight = 0.0, //最小高度
  this.maxHeight = double.infinity //最大高度
})
```

##### SizedBox

`SizedBox`用于给子元素指定<font color="#dd0000">**固定的宽高**</font>，**实际上`SizedBox`只是`ConstrainedBox`**的一个定制如：

```dart
SizedBox(
  width: 80.0,
  height: 80.0,
  child: redBox
)
```

##### 多重限制

如果某一个组件有多个父级`ConstrainedBox`限制，那么最终会是哪个生效？

```dart
ConstrainedBox(
    constraints: BoxConstraints(minWidth: 60.0, minHeight: 60.0), //父
    child: ConstrainedBox(
      constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),//子
      child: redBox,
    )
)
```

<font color="#dd0000">**对于`minWidth`和`minHeight`来说，是取父子中相应数值较大的。**</font>

##### UnconstrainedBox

`UnconstrainedBox`不会对子组件产生任何限制，它允许其子组件按照其本身大小绘制。一般情况下，我们会很少直接使用此组件，但在"去除"多重限制的时候也许会有帮助

#### 装饰容器DecoratedBox

`DecoratedBox`可以在其子组件绘制前(或后)绘制一些装饰（Decoration），如背景、边框、渐变等。

```dart
DecoratedBox(
    decoration: BoxDecoration(
      gradient: LinearGradient(colors:[Colors.red,Colors.orange[700]]), //背景渐变
      borderRadius: BorderRadius.circular(3.0), //3像素圆角
      boxShadow: [ //阴影
        BoxShadow(
            color:Colors.black54,
            offset: Offset(2.0,2.0),
            blurRadius: 4.0
        )
      ]
    ),
  child: Padding(padding: EdgeInsets.symmetric(horizontal: 80.0, vertical: 18.0),
    child: Text("Login", style: TextStyle(color: Colors.white),),
  )
)
```

#### 变换（Transform）

`Transform`可以在其子组件绘制时对其应用一些矩阵变换来实现一些特效。`Matrix4`是一个4D矩阵，通过它我们可以实现各种矩阵操作，下面是一个例子

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

##### 平移

`Transform.translate`接收一个`offset`参数，可以在绘制时沿`x`、`y`轴对子组件平移指定的距离。

```dart
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  //默认原点为左上角，左移20像素，向上平移5像素  
  child: Transform.translate(
    offset: Offset(-20.0, -5.0),
    child: Text("Hello world"),
  ),
)
```

##### 旋转

可以对子组件进行旋转变换，如：

```dart
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  child: Transform.rotate(
    //旋转90度
    angle:math.pi/2 ,
    child: Text("Hello world"),
  ),
)；
```

##### 缩放

`Transform.scale`可以对子组件进行缩小或放大，如：

```dart
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  child: Transform.scale(
      scale: 1.5, //放大到1.5倍
      child: Text("Hello world")
  )
);
```

**注意**：`Transform`的变换是应用在绘制阶段，而并不是应用在布局(layout)阶段，所以无论对子组件应用何种变化，其占用空间的大小和在屏幕上的位置都是固定不变的，因为这些是在布局阶段就确定的。

由于矩阵变化只会作用在绘制阶段，所以在某些场景下，在UI需要变化时，可以直接通过矩阵变化来达到视觉上的UI改变，而不需要去重新触发build流程，这样会节省layout的开销，所以性能会比较好。如之前介绍的`Flow`组件，它内部就是用矩阵变换来更新UI，除此之外，Flutter的动画组件中也大量使用了`Transform`以提高性能。

##### RotatedBox

`RotatedBox`和`Transform.rotate`功能相似，它们都可以对子组件进行旋转变换，但是有一点不同：<font color="#dd0000">**`RotatedBox`的变换是在layout阶段，会影响在子组件的位置和大小。**</font>

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    DecoratedBox(
      decoration: BoxDecoration(color: Colors.red),
      //将Transform.rotate换成RotatedBox  
      child: RotatedBox(
        quarterTurns: 1, //旋转90度(1/4圈)
        child: Text("Hello world"),
      ),
    ),
    Text("你好", style: TextStyle(color: Colors.green, fontSize: 18.0),)
  ],
),
```

#### Container

`Container`是一个组合类容器，它本身不对应具体的`RenderObject`，它是`DecoratedBox`、`ConstrainedBox、Transform`、`Padding`、`Align`等组件组合的一个多功能容器，**所以我们只需通过一个`Container`组件可以实现同时需要装饰、变换、限制的场景**。

```dart
Container({
  this.alignment,
  this.padding, //容器内补白，属于decoration的装饰范围
  Color color, // 背景色
  Decoration decoration, // 背景装饰
  Decoration foregroundDecoration, //前景装饰
  double width,//容器的宽度
  double height, //容器的高度
  BoxConstraints constraints, //容器大小的限制条件
  this.margin,//容器外补白，不属于decoration的装饰范围
  this.transform, //变换
  this.child,
})
```

`Container`的大多数属性在介绍其它容器时都已经介绍过了，不再赘述，但有两点需要说明：

- 容器的大小可以通过`width`、`height`属性来指定，也可以通过`constraints`来指定；如果它们同时存在时，`width`、`height`优先。实际上Container内部会根据`width`、`height`来生成一个`constraints`。
- `color`和`decoration`是互斥的，如果同时设置它们则会报错！实际上，当指定`color`时，`Container`内会自动创建一个`decoration`。

#### Scaffold、TabBar、底部导航

##### Scaffold

`Scaffold`是一个路由页的骨架，我们使用它可以很容易地拼装出一个完整的页面。一个完整的数路由页可能会包含导航栏、抽屉菜单(Drawer)以及底部Tab导航菜单等。

##### AppBar

`AppBar`是一个Material风格的导航栏，通过它可以设置导航栏标题、导航栏菜单、导航栏底部的Tab标题等。

##### TabBar

Material组件库中提供了一个`TabBar`组件，它可以快速生成`Tab`菜单。

##### TabBarView

通过`TabBar`我们只能生成一个静态的菜单，真正的Tab页还没有实现。由于`Tab`菜单和Tab页的切换需要同步，我们需要通过`TabController`去监听Tab菜单的切换去切换Tab页。

##### 抽屉菜单Drawer

`Scaffold`的`drawer`和`endDrawer`属性可以分别接受一个Widget来作为页面的左、右抽屉菜单。

##### 底部Tab导航栏

我们可以通过`Scaffold`的`bottomNavigationBar`属性来设置底部导航

#### 剪裁（Clip）

Flutter中提供了一些剪裁函数，用于对组件进行剪裁。

| 剪裁Widget | 作用                                                     |
| ---------- | -------------------------------------------------------- |
| ClipOval   | 子组件为正方形时剪裁为内贴圆形，为矩形时，剪裁为内贴椭圆 |
| ClipRRect  | 将子组件剪裁为圆角矩形                                   |
| ClipRect   | 剪裁子组件到实际占用的矩形大小（溢出部分剪裁）           |

```dart
Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              ClipRect(//将溢出部分剪裁
                child: Align(
                  alignment: Alignment.topLeft,
                  widthFactor: .5,//宽度设为原来宽度一半
                  child: avatar,
                ),
              ),
              Text("你好世界",style: TextStyle(color: Colors.green))
            ],
          )
```

##### CustomClipper

我们可以使用`CustomClipper`来自定义剪裁区域

```dart
class MyClipper extends CustomClipper<Rect> {
  @override
  Rect getClip(Size size) => Rect.fromLTWH(10.0, 15.0, 40.0, 30.0);

  @override
  bool shouldReclip(CustomClipper<Rect> oldClipper) => false;
}
```

- `getClip()`是用于获取剪裁区域的接口，由于图片大小是60×60，我们返回剪裁区域为`Rect.fromLTWH(10.0, 15.0, 40.0, 30.0)`，及图片中部40×30像素的范围。
- `shouldReclip()` 接口决定是否重新剪裁。如果在应用中，剪裁区域始终不会发生变化时应该返回`false`，这样就不会触发重新剪裁，避免不必要的性能开销。如果剪裁区域会发生变化（比如在对剪裁区域执行一个动画），那么变化后应该返回`true`来重新执行剪裁。

### 可滚动组件

**当组件内容超过当前显示视口(ViewPort)时，如果没有特殊处理，Flutter则会提示Overflow错误。为此，Flutter提供了多种可滚动组件（Scrollable Widget）用于显示列表和长布局。**

可滚动组件都直接或间接包含一个`Scrollable`组件，因此它们包括一些共同的属性，为了避免重复介绍，我们在此统一介绍一下：

```dart
Scrollable({
  ...
  this.axisDirection = AxisDirection.down,
  this.controller,
  this.physics,
  @required this.viewportBuilder, //后面介绍
})
```

- `axisDirection`滚动方向。

- ```
  physics
  ```

  ：此属性接受一个

  ```
  ScrollPhysics
  ```

  类型的对象，它决定可滚动组件如何响应用户操作，比如用户滑动完抬起手指后，继续执行动画；或者滑动到边界时，如何显示。默认情况下，Flutter会根据具体平台分别使用不同的

  ```
  ScrollPhysics
  ```

  对象，应用不同的显示效果，如当滑动到边界时，继续拖动的话，在iOS上会出现弹性效果，而在Android上会出现微光效果。如果你想在所有平台下使用同一种效果，可以显式指定一个固定的

  ```
  ScrollPhysics
  ```

  ，Flutter SDK中包含了两个

  ```
  ScrollPhysics
  ```

  的子类，他们可以直接使用：

  - `ClampingScrollPhysics`：Android下微光效果。
  - `BouncingScrollPhysics`：iOS下弹性效果。

- `controller`：此属性接受一个`ScrollController`对象。`ScrollController`的主要作用是控制滚动位置和监听滚动事件。默认情况下，Widget树中会有一个默认的`PrimaryScrollController`，如果子树中的可滚动组件没有显式的指定`controller`，并且`primary`属性值为`true`时（默认就为`true`），可滚动组件会使用这个默认的`PrimaryScrollController`。这种机制带来的好处是父组件可以控制子树中可滚动组件的滚动行为，例如，`Scaffold`正是使用这种机制在iOS中实现了点击导航栏回到顶部的功能。我们将在本章后面“滚动控制”一节详细介绍`ScrollController`。

#### ViewPort视口

在很多布局系统中都有ViewPort的概念，在Flutter中，术语ViewPort（视口），如无特别说明，则是指一个Widget的实际显示区域。例如，一个`ListView`的显示区域高度是800像素，虽然其列表项总高度可能远远超过800像素，但是其ViewPort仍然是800像素。

#### 基于Sliver的延迟构建

通常可滚动组件的子组件可能会非常多、占用的总高度也会非常大；如果要一次性将子组件全部构建出将会非常昂贵！为此，Flutter中提出一个Sliver（中文为“薄片”的意思）概念，如果一个可滚动组件支持Sliver模型，那么该滚动可以将子组件分成好多个“薄片”（Sliver），只有当Sliver出现在视口中时才会去构建它，这种模型也称为“基于Sliver的延迟构建模型”。可滚动组件中有很多都支持基于Sliver的延迟构建模型，如`ListView`、`GridView`，但是也有不支持该模型的，如`SingleChildScrollView`。



### 使用记录

1. 分割线

   ```dar
   Divider(height:10.0,color:Colors.red)
   ```

2. Chip

   碎片，一般做标签用

3. CircleAvatar

   表示用户资料的圆形组件

4. 最大宽度、最大高度

   this.maxWidth = double.infinity

   this.maxHeight = double.infinity


5. MainAxisAlignment和CrossAxisAlignment

   MainAxisAlignment表示主轴方向，CrossAxisAlignment表示与主轴垂直的方向。

   //将子控件放在主轴的开始位置
     start,  
      //将子控件放在主轴的结束位置
     end,
     //将子控件放在主轴的中间位置
     center,
     //**将主轴空白位置进行均分，排列子元素**，<font color="#dd0000">**首尾没有空隙**</font>。
     spaceBetween,
     //**将主轴空白区域均分，使中间各个子控件间距相等**，<font color="#dd0000">**首尾子控件间距为中间子控件间距的一半**</font>
     spaceAround,
     //将主轴空白区域均分，使各个子控件间距相等
     spaceEvenly

6. 

### 异常记录

1. Invalid constant value 

   You are declaring your `Text` widget as a `const`, which requires all of its children to be `const` as well.

2. xxx.dart were declared as an inputs, but did not exist. Check the definition of target:kernel_snapshot for errors

   解决：这是文件重命名后造成的。在命令行中输入：Flutter clean，清理缓存即可。

3. Waiting for another flutter command to release the startup lock… 异常解决

   进入到你的flutter sdk目录中，然后找到`bin/cache/lockfile`文件，删除它即可。