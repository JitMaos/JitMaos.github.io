---
Stitle: Flutter『问题记录』
date: 2019-11-14 11:14:08
tags:
---



### Tips

1. AS中clean Flutter项目

   在命令行中输入：

   ```java
   flutter clean
   ```

2. 环境搭建

   1. 检查项目中Android中build.gradle中gradle版本、gradle/wrapper中的gradle版本，以及fluttersdk/packages/gradle/flutter.gradle中的版本是否一致
   2. 检查AS中Preferences中flutter的SDK是否指定
   
3. Dart 中没有 修饰符用来修饰私有方法，在定义方法和属性时候 添加 _  用来区分私有方法和属性。

4. Dart中的get/set

   ```dart
   class Rect{
     num height;
     num width;   
     Rect(this.height,this.width);
     get area{
       return this.height*this.width;
     }
      set areaHeight(value){
       this.height=value;
     }
   }
   
   void main(){
     Rect r=new Rect(10,2);
     print("面积:${r.area}");      //注意调用直接通过访问属性的方式访问area
   }
   ```

5. Dart中多个构造函数

   只能有一个默认构造函数

   Student(this.name,this.age);

   可以有多个命名构造函数

   Student.fromJson(String json);

   Student.fromDB(String name,String age);

6. 关闭Dialog

   ```dart
   Navigator.of(context).pop();
   ```

7. Flutter中的Key

8. Navigator.pop可以带参数

   Navigator.pot(context,"result"); 其中的"result"就可以在外部回调处理了

9. 传递互调的一种实现

   ```dart
   //定义
   Container customButtom(String text,Color startColor,Color endColor,VoidCallback onTap) {
       return Container(
         margin: EdgeInsets.all(3),
         child: DecoratedBox(
           decoration: BoxDecoration(
               gradient: LinearGradient(colors: [startColor,endColor]),
               borderRadius: BorderRadius.all(Radius.circular(3.0)),
               boxShadow: [
                 BoxShadow(
                     color:Colors.black,
                     offset:Offset(3,3),
                     blurRadius: 4.0
                 )
               ]
           ),
           child: FlatButton(
             child: Text(text),
             onPressed:(){
               onTap.call();
               print(text);
             },
           ),
         ),
       );
     }
   
   //调用
   customButtom("Search", Colors.purple,Colors.blueAccent,(){
                     TRecordDbProvider provider = new TRecordDbProvider();
                     Future<List<TRecord>> future = provider.getRecordList();
                     future.then((value) => print(value));
                   }),
   ```

   

10. 取TextField的值

   从TextEditingController.text中取的

11. 界面顶部TabBar

    ```dart
        return DefaultTabController(
            length: 3,
            child: Scaffold(
              appBar: AppBar(
               	...
                bottom: TabBar(
                  tabs:<Widget>[
                    Tab(icon:Icon(Icons.local_florist)),
                    Tab(icon:Icon(Icons.change_history)),
                    Tab(icon:Icon(Icons.directions_bike)),
                  ]
                ),
              ),
              body: TabBarView(
              	children:<Widget>[
                  Icon(Icons.xxx,size:)
                ]
              )
            ));
    
    ```

12. 使用Expanded导致组件位置异常

    解决：将Expended的flex设置为0

13. Row均分内容

    ```dart 
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    ```

14. 去除右上角debug标记，在App中添加

    ```
    debugShowCheckedModeBanner: false,
    ```

15. 获取屏幕宽高信息，使用MediaQuery.of(context).xxx

    ```dart
    MediaQuery.of(context).size.width //获取屏幕宽度
    ```

16. <font color="#dd0000">**传递函数**</font>

    ```
    //定义
    Widget slideItem(Null Function() onTap){}
    
    //调用
    slideItem("Listening", "Books", "assets/slide_1.png", (){}),
    ```

17. 组件的<font color="#dd0000">**Transform属性**</font>，改变组件的位置属性，实现层叠效果。

    ```dart
    transform: Matrix4.translationValues(0.0, -25.0, 0.0), //x,y,z
    ```

18. 修改RaisedButton的形状，使用shape来修饰实现

    ```dart
    child:RaisedButton(
    	...
      shape: new RoundedRectangleBorder(borderRadius: new BorderRadius.circular(30.0))
    )
    ```

19. 实现评分组件

    ```dart
    Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Icon(Icons.star, color: Theme.of(context).primaryColor ,),
        Icon(Icons.star, color: Theme.of(context).primaryColor ,),
        Icon(Icons.star, color: Theme.of(context).primaryColor ,),
        Icon(Icons.star, color: Theme.of(context).primaryColor ,),
        Icon(Icons.star, color: Colors.grey[400] ,),
      ],
    ),
    ```

20. Scaffold中可以通过<font color="#dd0000">**floatingActionButton**</font>配置悬浮层

    ```
    A button displayed floating above [body], in the bottom right corner.
    floatingActionButton: Row( ... )
    ```

21. ScrollPhysics

    ```dart
    BouncingScrollPhysics ：允许滚动超出边界，但之后内容会反弹回来。
    ClampingScrollPhysics ： 防止滚动超出边界，夹住 。
    AlwaysScrollableScrollPhysics ：始终响应用户的滚动。
    NeverScrollableScrollPhysics ：不响应用户的滚动。
    ```

22. 进度条组件，使用foregroundDecoration实现

    ```dart
    Container(
      width: MediaQuery.of(context).size.width / 10 * 7,
      height: 10.0,
      decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(10.0),
        color: Color(getColorHexFromStr("#ECECEC"))
      ),
      foregroundDecoration: BoxDecoration(
        borderRadius: BorderRadius.circular(10.0),
        gradient: LinearGradient(
          begin: Alignment.centerLeft,
          end: Alignment.centerRight,
          stops: [0.1, 0.7, 0.7],
          colors: [Color(getColorHexFromStr("#FF974D")), Color(getColorHexFromStr("#FF6A02")), Color(getColorHexFromStr("#ECECEC"))],
        ),
      ),
    ),
    ```

23. 为<font color="#dd0000">**Container添加点击**</font>事件，通过GestureDetector包含实现

    ```dart
    return GestureDetector(
        onTap: () {
          Navigator.pushNamed(
            context,
            Account.routeName,
          );
        },
        child: Container(
          width: 40.0,
          height: 40.0,
          decoration: BoxDecoration(
            image: DecorationImage(
              fit: BoxFit.contain,
              image: AssetImage("assets/account.png"),          
            ),
          ),
        ),
      )
    ```

24. Container添加Border

    ```dart
    decoration: BoxDecoration(
      border: Border.all(
        color: Colors.orange,
        width: 3.0,
      ),
      borderRadius: BorderRadius.circular(12.0),
    ),
    ```

25. 在Scaffold中使用bottomNavigationBar新建底部对齐的组件。

    ```dart
    bottomNavigationBar:Container( ... )
    ```

26. 创建对称的padding/margin

    ```dart
    margin: EdgeInsets.symmetric(vertical: 30),
    padding: EdgeInsets.symmetric(horizontal: 30, vertical: 5),
    ```

27. 指定TextField背景，使用border

    ```dart
    TextField(
      ...
      decoration: InputDecoration(
        fillColor: Colors.grey[200],
        filled: true,
        border: OutlineInputBorder(
          borderSide: BorderSide.none,
          borderRadius: BorderRadius.circular(8),
        )),
    ),
    ```

28. 指定TextField内边距，使用contentPadding

    ```dart
    
    TextField(
      ...
      decoration: InputDecoration(
        ...
        contentPadding: const EdgeInsets.symmetric(vertical: -10, horizontal: 15),),
    ),
    ```

29. Flutter实现返回页面刷新

    ```dart
    Navigator.pushNamed（context,"RouteName").then((value) => {
      refreshData()
    });
    ```

30. 



### 问题记录

1. Gradle Build error: versionCode not found. Define flutter.versionCode in the local.properties file.

   解决：在Android目录的local.properties文件中添加flutter的版本相关信息

   ```dart
   flutter.buildMode=debug
   flutter.versionName=1.0.0
   flutter.versionCode=1
   ```

2. AAPT: error: resource android:attr/fontVariationSettings not found after Flutter Upgrade.

   解决：设置compileSdkVersion到对应的版本

3. # No connected devices found;

   检查项目设置，JDK位置、Dart SDK是否正确

4. Column嵌套ListView不显示

   将 ListView 用 Expand 包裹起来。

5. 使用json_serializable生成反序列化对象失败

   注意文件名要完全相同，大小写也是

6. 生成json序列化文件

   参考：https://www.jianshu.com/p/b307a377c5e8

   1. 使用part "File.g.dart"
   2. 命令：flutter packages pub run build_runner build

7. flutter Vertical viewport was given unbounded height

   解决：每一层都需要嵌套在Expanded中
   
8. Flutter Incorrect use of ParentDataWidget

   保持：Expanded、Flexible只在Row、Column等组件内，不在其他组件内使用。

9. 