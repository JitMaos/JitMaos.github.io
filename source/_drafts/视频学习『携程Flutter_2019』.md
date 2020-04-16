---
title: 2019』
tags:
---

### 第二章

1. DartPad是Dart的一个线上playground，提供Dart线上playground的还有：Online Dart Compiler.

### 第三章

1. 在Dart中只有"true"被视为true。其他的都被视为false。

2. 在Dart中数字也是对象。

3. 从Dart1.2开始,null-aware运算符可以帮助我们做null检测。?.运算符在左边为null的情况下阻止右侧的代码的执行。

4. 在Dart中，async函数返回一个Futrue，函数的主体是稍后运行。await运算符用于等待Futrue.

5. 声明式UI

   在声明式UI中，视图配置(例如Flutter的Widget)是不可变的，并且只是轻量级的"蓝图"。要更改UI，**Widget会在自身上触发重建(最常见的是通过Flutter的StatefulWidgets上调用setState())并构建一个新的Widget子树**。RenderObjects在帧之间保持不变，Flutter的轻量级Widgets告诉框架在状态之间改变RenderObjects，接下来Flutter框架会处理其余部分。

6. 在Flutter中，几乎所有东西都是Widget。Widget是用户界面的基础构建块，每个窗口widget都嵌套在父窗口Widget中，**并从其父窗口集成属性。**

7. 一个自定义Widget的实例代码：

   ```dart
   class CustomCard extends StatelessWidget {
     CustomCard({@required this.index,@required
     this.onPress});
   
     final index;
     final Function onPress;
   
     @override
     Widget build(BuildContext context) {
       // TODO: implement build
       return Card(
         child: Column(
           children: <Widget>[
             Text('Card $index'),
             FlatButton(
               child: const Text('Press'),
               onPressed: this.onPress,
             )
           ],
         ),
       );
     }
   }
   ```

8. **Flutter中的View**，Widget和View是有一些区别。首先，Widget具有不同的生命周期：**它们是存在于状态被改变之前。**<font color="#dd0000">**每当Widget或者其状态发生变化时，Flutter的框架都会创建一个新的Widget实例树。**</font>相比之下，Android/iOS视图被绘制一次，并且在调用invalidate/setNeedsDisplay之前不会重绘。

   此外，与View不同，Flutter的Widget很轻巧，部分原因在于他的不变性。因为它本身不是试图，并且不是直接绘制的任何东西，而是对UI及其语义的描述。

9. **如何更新Widgets**，无状态Widget和有状态Widget之间的重要区别在于StatefulWidgets具有一个State对象，该对象存储状态数据并将其传递到树重建中，因此状态不会丢失。

10. **如何在布局中添加或删除组件？**在Flutter中，因为Widget是不可变的，我可以传入一个函数或表达式，该函数或表达式返回一个Widget给父项，并通过布尔值控制该Widget的创建。

11. **如何对Widget做动画？**在Flutter中，**使用AnimationController，这是一个可以暂停、寻找、停止、反转动画的Android类型。它需要一个Ticker当vsync发生时发送信号，并且在每帧运行时创建一个介于0和1之间的线性插值(interpolation)**。我们可以创建一个或多个Animation并附加给一个controller。<font color="#dd0000">**[代码]**</font>

12. **如何绘图(Canvas draw/paint)?** Flutter基于底层渲染引擎Skia。Flutter有两个类CustomPaint和CustomPainter可以帮助我们绘制画布。<font color="#dd0000">**[代码]**</font>

13. **如何构建自定义Widgets？**在Flutter中，推荐组合多个小的widgets来构建一个自定义的widget。<font color="#dd0000">**[代码]**</font>

14. **如何设置Widget的透明度？**

    在Flutter中如果要改变透明度，可以给widget包裹一个Opacity widget来实现。

    ```dart
    Opacity(
    	opacity:0.5,
      child:Text('透明度50%')
    )
    ```

15. 











