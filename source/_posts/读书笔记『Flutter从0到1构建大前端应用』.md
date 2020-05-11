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

**P18**：Dart是一门强类型语言，在第一次赋值时，**如果已经确定了是字符串类型，则不能更改为别的类型**。如果真的想要改变，可以使用**dynamic**关键字。

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

**P25**：<font color="#dd0000">**使用dynamic时会告诉编译器，我们不用做类型检测，并且知道自己在做什么。**</font>

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
void userSettings({@required int age,String name})
```

可选的位置参数

```dart
void userSettings({int age,String name,[String interests]}) {
  if(interests != null) {
    //...
  }
}
```

默认参数，默认值是编译时常量

```dart
void userSettings({int age = 21,String name='小米'})
```

**P31**：一个Future表示一个异步操作产生的的结果。

```java
Future<int> future = getFuture();
future.then((value) => handleValue(value)) //then用于接受异步处理的结果
  .catchError((error) => handleError(error)) //catchError用于捕获异常
  .whenComplete() => handleComplete(); //whenComplete用于处理执行结束的回调，无论是否异常都会执行
```

then用于接受异步处理的结果；catchError用于捕获异常；whenComplete用于处理执行结束的回调，无论是否异常都会执行

当有需要有延迟的运算(async)时，可以将其放到队列(await)中去，注意：<font color="#dd0000">**await必须在async标记的函数中运行，否则会报错。**</font>async和await用于解决嵌套回调等问题：

```dart
steps() async {
  try {
    String step1Res = await step1('step1');
    String step2Res = await step2(step1Res);
    String step3Res = await step1(step2Res);
  } catch(e) {}
}
```

**P32**：**Flutter的继承也是单继承的(extends)**，因为Flutter没有访问修饰符，所以子类可以访问父类中所有的变量和方法；<font color="#dd0000">**Flutter中每个类都是一个隐式的接口，这个接口包含类中所有的成员变量和方法**</font>。当类被当做接口使用时，类中的方法需要在子类中被实现。

<font color="#dd0000">**P34：**</font>Dart新增的mixins语法特性，作用是**可以在类中混入其他功能。**具体的讲，就是把<font color="#dd0000">**自己的方法提供给其他类使用，但却不需要成为其他类的父类，它以非继承的方式来复用代码**</font>。使用mixins，需要使用with关键字。

```dart
abstract class CanFixComputer {
  ...
  void fixComputer() {
    print('软件工程师修电脑')
  }
}
class ITTeacher with CanFixComputer {
  @override
  void fixComputer() {
    print('IT老师修电脑')
  }
}
```

###第三章 一切皆组件

P40**：从布局来看，官方把Flutter布局分为Basic Widget、Single-Child、Multi-Child。

**P58**：Scaffold是基于**Material库**的一个**与路由相关**的模板组件，可以认为是Flutter提供的标准化的布局容器。

```dart
Widget build(BuildContext context) {
  return Scaffold(
  	AppBar:...
    body:...
    bottomNavigationBar:...
    floatingActionButton:...
    drawer:...
    ...
  )
}
```

**P62**：Column是不支持滚动的，如果需要实现滚动功能，则需要考虑使用ListView。

**P76**：使用Wrap替换Row可以实现自动换行的效果。

**P79**：Context表示组件上下文的意思。**通过Context可以遍历和查找当前Widget树**。

<font color="#dd0000">**P80：**</font>StatelessWidget即无状态的Widget，它<font color="#dd0000">**无法通过setState设置组件的状态**</font>。对于其内部属性，应该被声明为final，防止被意外修改。

```dart
class MyStatelessWidget extends StatelessWidget {
  MyStatelessWidget({
    Key key,
    this.parameter,
  }):super(key:Key),
  final parameter;
  @override
  Widget build(BuildContext context){}
}
```

StatefulWidget即有状态Widget。创建一个StatefulWidget组件时，**会同时创建一个State对象**，并且StatefulWidget通过与State关联可以达到刷新UI的目的。

```dart 
class MyStatefulWidget extends StatefulWidget {
  MyStatefulWidget({
    Key key,
    this.color,
  }):super(key:key),
  
  final Color color;
  
  @override 
  _MyStatefulWidgetState createState() => new _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  ...
  @override
  Widget build(BuildContext context) {...}
}
```

**P82**：生命周期

**initState**是State生命周期里第一个被执行的方法，可以在该方法进行一些初始化动作。<font color="#dd0000">**在initState里，Framework层还没有把Context和State关联在一起，所以还不能访问Context。初始化方法在生命周期里只会被执行一次。**</font>

**didChangeDependencies**：在执行完initState之后就可以访问Context了。如果Widget使用了InheritedWidget的数据，并且在InheritedWidget的数据发生改变时，Flutter Framework就会触发didChangeDependencies的回调。

**build**：在执行完didChangeDependencies之后，<font color="#dd0000">**每次调用setState都会触发build方法。**</font>

**dispose**：在组件被销毁时调用。

![img](http://47.110.40.63:8080/img/blog/Flutter State生命周期.png)

<font color="#dd0000">**P88**</font>：在Flutter中，每个Widget都有一个唯一标识Key，在创建和渲染时生成，可以手动指定。在某些场景下，需要保存Key，**可以保存到GlobalKey、LocalKey、UniqueKey、ObjectKey中**，可以根据key找到的对应的Widget。

**P89**：被InheritedWidget暴露出来的数据，可以高效地在Widget树中**从上往下**传递和共享，并支持**跨级**数据传递。

### 第四章 事件处理

**P100**：原始指针事件，把Listener包裹在需要监听的组件外来实现。

```dart
...
  body:Center(
  	child:Listener(
    	child:Container()
    )
  )
```

**P102**：可以使用IgnorePoiner和AbsorbPointer对事件进行忽略，前者本身可以接受事件，后者本身也不可接收事件。

**P105**：GestureDetector可以支持更丰富的事件，如缩放、双击、垂直、水平的手势。同样需要包裹在要监听的组件外面。

```dart
GestureDetector (
	onTap:() {
    print('tap');
  },
  child:Container()...
)
```

**P112**：事件竞争和手势冲突

Flutter加入了**手势竞技场**的概念。在给同一个组件同时添加水平和垂直回调时，若用户将指针水平移动超过一定的逻辑像素，则判定水平获胜。反之判定垂直获胜。

### 第五章 动画

**P129**：当创建AnimationController时，需要传入vsync参数，这个参数接受的是TickerProvider类型的对象，<font color="#dd0000">**作用是阻止在屏幕锁屏时执行动画以避免不必要的资源浪费。**</font>

**P136**：AnimatedWidget对addListener、setState等动画相关动作进行了封装，使用起来更方便。

**P141**：通过Hero，可以在路由之间做出流程的转场动画。**Hero的组件需要同时定义源组件和目标组件，其中源组件和目标组件被Hero包裹在需要动画控制的组件外面。**

### 第六章 使用网络技术和异步编程

**P184**：在Flutter里，异步使用Futrue来修饰的，并运行在event loop里。Flutter中一个很重要的概念是isolate，通过Flutter Engine层面的**一个线程**来实现的，而实现isolate的线程又是由Flutter管理和创建的。所有的Dart代码都运行在isolate上。通常情况下，我们的应用都运行在main isolate中，在有必要时，我们可以创建新的isolate。<font color="#dd0000">**多个isolate无法共享内存，必须通过相关的API通信才可以。**</font>

**P185**：event loop

![img](http://47.110.40.63:8080/img/blog/Flutter eventloop运行流程图.png)

1. 运行APP并执行main方法
2. 开始ging优先处理microtask queue，知道队列为空
3. 当microtask queue为空后，开始处理event queue。如果event queue里面有event，则执行，<font color="#dd0000">**每执行一条再判断此时新的microtask queue是否为空**</font>，并且每次只取出一条来执行。可以这么理解，在处理所有event之前，我们会做一些市区内给，并且会把这些事情放在microtask queue中。

4. microtask queue 和 event queue都为空，则APP可以正常退出。

注意：当处理microtask queue时，event queue会被阻塞，所以microtask queue中应避免任务太多或长时间处理，**否则将导致APP的绘制和交互等行为被卡住**。

**P192**：Future表示“将来”一次异步获取得到的数据，而Stream是多次异步获取得到的数据；Future将返回一个值，而Stream将返回多次值。

### 第七章 路由

**P197**：一个界面跳转到另一个界面使用Navigator.push方法，返回上一个界面使用Navigator.pop方法。路由分为静态路由和动态路由。

**204**：参数回传

1. 等待回传数据

   ```dart
     _navigateAndDisplaySelection(BuildContext context) async {
       final result = await Navigator.push(
         context,
         MaterialPageRoute(builder: (context) => SelectionScreen()),
       );
     }
   ```

2. 回传数据

   ```dart
   Navigator.pop(context, 'Flutter');
   ```

**P207**：路由栈

类似Android的launchMode功效

+ pushReplacementNamed：替换当前栈顶页面为新的页面
+ popAndPushNamed：和上面的一样，只是多了pop的交互效果
+ pushNamedAndRemoveUntil：添加一个元素，并且移除历史，直到named对象
+ popUnil：没有push，移除历史，直到named对象

### 第八章 持久化

**P215**：推荐使用shared_preferences插件，这是一个异步的key-value存储的插件。

+ 写入

  ```dart
  SharedPreferences prefs = await SharedPreferences.getInstance();
  prefs.setString(k,v);
  ```

+ 读取

  ```dart
  SharedPreferences prefs = await SharedPreferences.getInstance();
  prefs.getString(k);
  ```

+ 清除

  ```dart
  SharedPreferences prefs = await SharedPreferences.getInstance();
  prefs.remove(k);
  ```

**P220**：sqflite

+ 支持事务和批处理
+ 支持自动version管理
+ 支持增、删、改、查的Helper工具类
+ **支持Android/iOS后台线程的运行**

**获取和删除**

```dart
var databasesPath = await getDatabasesPath();
String path = join(databasePath,'demo.db');
await deleteDatabase(path);
```

**打开并创建数据库**

```dart
Database database = await openDatabase(path,version:1,oncreate:(
	Database db,int version) async {
  	await db.execute('CREATE TABLE Test(id INTEGER ...)')
	}
));
```

**插入数据的两种方式**

//方式一，直接使用sql

```dart
Future<int> rawInsert(String sql,[List<dynamic> arguments]);
int id = await txn.rawInsert(
	'INSERT INTO Test(name,value,num) VALUES(?,?,?)',['another name',1213,3.1212])
);
```

//方式二，使用map

```dart
Future<int> insert(String table,Map<String,dynamic> values,{String nullColumnHack,ConflictAlgorithm conflictAlgorithm});
```

**修改的两种方式**，和插入一样，只是方法名变了

```dart
Future<int> rawUpdate(String sql,[List<dynamic> arguments]);
```

```dart
Future<int> rawUpdate(String table,Map<String,dynamic> values,{String nullColumnHack,ConflictAlgorithm conflictAlgorithm});
```

**查询的两种方式**

```dart
Future<List<Map<String,dynamic>>> rawQuery(String sql,[List<dynamic> arguments]);
```

```dart
Future<List<Map<String,dynamic>>> Query(String table,{
  bool distinct,List<String> columns,String where,List<dynamic> whereArgs,
  String groupBy,String having,String orderBy,int limit,int offset
});
```

**物理删除的两种方式**

```dart
Future<int> rawDelete(String sql,[List<dynamic> arguments]);
```

```dart
Future<int> delete(String table,{String where,List<dynamic> whereArgs})
```

**计算总记录数**

```dart
Sqflite.firstIntValue(await database.rawQuery('SELECT COUNT(*) FROM Test'))
```

**关闭数据库**

```dart
await databse.close()
```

### 第九章

**P238**：添加未发布的package，编辑pubspec.yaml

```dart
dependencies:
	plugin1:
		path: ../plugin1/
```

添加git上的依赖

```dart
dependencies:
	plugin1:
		git:
			url: git://github.com/flutter/plugin1.git
```

**P239**：调用flutter packages get之后，会生成pubspec.lock。该文件确保更新的package不会影响现有代码。

**P240**：创建自己的package

```dart
flutter create --org com.example --template=plugin -i swift -a kotlin hello
```

--org：指定包名；-i：指定iOS语言；-a：指定Android语言

**P241**：Platform Channel

Platform Channel 是Flutter与Platform指定的通信机制，包括3种

1. BasicMessageChannel：**用于传递字符串和半结构化的信息（在大内存数据块传递的情况使用）**
2. MethodChannel：**用于传递方法的调用**
3. EventChannel：**用于数据流(event streams)的通信**

**P242**：Flutter的消息传递工具是BinaryMessager，传递的消息格式是二进制的。二进制格式的消息通过消息编解码器(Codec)解码为能识别的消息，并传递给Handler来进行处理。

**P253**：在Android里每初始化一个FlutterView就会有一个FlutterEngine被初始化，这样就会有新的线程在Dart上运行。如果每个Activity中都有一个FlutterView，则会创建多个Flutter Engine实例，这会导致每个Flutter Engine实例加载的代码都独立运行在ioslate中，这种模式被称为多引擎模式。存在问题

1. 冗余的资源问题。
2. 插件注册问题。插件依赖Messenger传递消息，多个FlutterView时，插件的注册和通信将变得混乱和难以维护。
3. Flutter Widget和Native页面的差异化问题。
4. 增加页面之间通信的复杂度。

**P254**：FlutterBoost的思想是把Flutter容器做成浏览器的样子，然后填写一个页面地址，再由容器去管理页面的绘制。在Native端，如果初始化容器，就设置容器对应页面的标志即可。



















































































