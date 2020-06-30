---
title: 读书笔记『Kotlin实战』
date: 2019-03-21 17:10:20
tags: [Kotlin,读书笔记]
categories: Android
---

**P18**：在Kotlin中没有声明数组类型的特殊语法，**数组就是类**

**P19**：在Kotlin中if是有结果的表达式

```java
fun mac(a:Int,b:Int):Int{ return if(a>b) a else b}
```

**P25**：自定义访问器

```java
class Retangle(val height:Int, val width:Int) {
  val isSqual:Boolean
    gen() { return height == width }
}
```

**P30**：在"when"中使用任意对象

```java
fun mix(cl:Color, c2:Color) {
  when(setOf(c1, c2)) {
    setOf(RED, YELLOW) -> ORANGE
    setOf(YELLOW, BLUE) -> GREEN
    setOf(Blue, VIOLET) -> INDIGO
  }
}
```

**P37**：迭代数字：区间和数列

+ 区间：for( i in 1 .. 100 )  { print( i ) }
+ 步长区间：for( i in 100 <font color="#dd0000">**downTo 1 step 2**</font> )
+ 半闭合区间：for( x in 0 **util** size )，等同于for( x in 0 .. size-1 )

**P38**：迭代map

+ for( letter.binary ) in list.**withIndex()**  { "**\$index: \$element**"}

<!-- more -->

**P41**：Kotlin中throw结构是一个表达时刻，能作为另一个表达式的一部分使用：

```java
val percentPage = if( num in 0..100) number else IllegalArgumentExcetion()
```

**P49**：在Kotlin中方法参数可以指定默认值

```java
fun testDefaultArg(a:Int,b:Int = 11,c:Int=22) {
    println("$a,$b,$c")
}
>>>println("$a,$b,$c") //输出：9,11,22
```

**P51**：顶层函数和顶层方法都会编译成**静态成员**，如果想要public static final，可以使用const修饰顶层属性。

**P53**：扩展函数式一个类的成员函数，不过它定义在类的外面，扩展函数不能访问私有或受保护的成员

```java
fun String lastChar():char = this.get(this.lenght - 1)
```

**P54**：如果导入的函数存在重名，可以使用as来标识别名

```java
import strings.lastChar as last
```

**P57**：**扩展函数不存在重写**，因为Kotlin会把它们当做**静态函数**对待。另外**成员函数的优先级高于扩展函数**。

**P60**：中缀调用：var map = mapOf(1 to "one",2 to "two")；解构声明：val(number,name) = 1 to "one"

**P69**：Kotlin中接口是可以包含属性的。<font color="#dd0000">**Kotlin中嵌套的类默认并不是内部类：它们并没有包含对其他外部类的隐式引用。**</font>

**P72**：如果继承有两个相同的方法，需要使用"<>"显式地指定实现哪一个

```java
override fun showOff {
  super<Callable>.showOff()
  super<Focusable>.showOff()
}
```

**P73**：Kotlin中的类默认是final，如果希望这个类能够被继承，需要使用open修饰符修饰。**已经重写了的方法，如果没有final修饰，表明它是开放的。**

**P75**：Kotlin中默认的可见性是public；<font color="#dd0000">**Kotlin中一个外部类不能访问嵌套类的private成员。**</font>

**P77**：在Java中，在一个类中声明另一个类时，这个类默认是内部类，内部类持有外部类的引用。要去除这种引用，需要把内部类声明成static的。在Kotlin中没有显式修饰符的嵌套类与Java中的static是一样的，**要把它变成内部类，需要使用inner修饰。**

**P79**：密封类：用**sealed**修饰的类对可能创建的子类做出了严格的限制。所有<font color="#dd0000">**直接**</font>子类必须嵌套在父类中。**sealed修饰符隐含这个类是一个open类**。

**P82**：如果所有的构造方法参数都有默认值，编译器会**生成一个额外的不带参数的构造方法来使用这些默认值**。

**P85**：在Kotlin中，接口可以包含<font color="#dd0000">**抽象属性声明**</font>，者意味着实现User接口的类**需要提供一个取得抽象属性的方式。**

```java
interface User { val nickname:String}
class PrivateUser(override val nickname:String):User
```

**P88**：在**非空属性**上使用lateInit修饰符表明这个属性会将初始化推迟到构造方法被调用后。

**P90**：如果想进行引用比较，可以使用**===**运算符，这与Java中的**==**效果一样。

**P92**：Kotlin中数据类(data class)会自动帮你生成通用方法，如:toString、equals、hashCode。另外还多生成了一个<font color="#dd0000">**copy**</font>方法，用于修改val参数。 

**P93**：当你需要向类中添加一些行为的时候，Kotlin中提供了类委托的支持，无论什么时候实现一个接口，都可以使用by关键字将接口的实现委托到另外一个对象。<font color="#dd0000">**好处是不需要实现一堆接口中定义的方法**</font>，如：

```java
class DelegatingCollection<T>(innerList:Collection<T> = ArrayList<T>()):Collection<T> by innerList{}
```

**P95**：Kotlin中object关键字的核心是：<font color="#dd0000">**定义一个类并同事创建一个实例。**</font>对象声明：定义单例的一种方式，对象声明中可以包含属性、方法、初始化语句块等的声明，<font color="#dd0000">**但是不能包含构造方法**</font>，因为对象声明在定义时就创建了对象，不需要构造方法；在Java中调用对象声明使用INSTANCE来访问静态对象。

**P98**：伴生对象：可以持有工厂方法和其他与这个类相关，但在调用时并不依赖类实现的方法。它们的成员可以通过类名来访问。

**P103**：对象表达式：用来**替换Java的匿名内部类**。和Java匿名内部类只能扩展一个类或**实现一个接口**不同，Kotlin的匿名对象可以实现多个接口或不实现接口。另外对象表达式中的代码可以访问创建它的函数中的变量，<font color="#dd0000">**访问并没有被限制在final变量，可以在代码中修改变量值。**</font>(对于非final变量来说，它的值被封装在一个特殊的包装器中，这样你就可以改变这个值，而这个包装器的引用会和lambda代码一起存储)

**P108**：Lambdab表达式的语法：{ x:Int，y:Int -> x + y }，实例：people.maxBy ({p:Person -> p.age})  =(如果lambda是函数调用的最后一个实参，它可以放在括号外面)> people.maxBy (){p:Persion -> p.age}  =(当lambda时唯一的实参，可以去掉括号)>  people.maxBy{p:person -> p.age} =(和局部变量一样，如果lambda参数的类型可以被推导出来，就可以省略)> people.maxBy{p -> p.age} =(如果当前上下文只有一个参数，且能被推导出类型，可以使用)> people.maxBy{ it.age }

**P114**：成员引用，提供了简明语法，来创建一个调用的单个方法(普通函数，构造函数等)或者访问单个属性的函数值。使用<font color="#dd0000">**双冒号：：**</font>来实现，如：

```java
fun salute() = println("Salute!")
>>> run(::salute) //引用顶级函数
```

**P117**：filter函数可以从集合中移除你不想要的元素，<font color="#dd0000">**但它不会改变这些元素**</font>；map函数对集合中的每一个元素应用给定的函数并把结果<font color="#dd0000">**收集到一个新集合中。**</font>

```java
val list = listOf(1,2,3,4)
>>> println(list.map{ it * it}) //输出[1,4,9,16]
```

**P118**：Kotlin中，可以使用all、any、count、find、!all来检查集合中的所有元素是否符合某个条件，如：

```java
val canBeInClub27 = { it.age <= 27} //条件
val people = listOf(Person("A",27),Person("B",22))
>>>println(peole.all(canBeInClub27)) //输出：false
```

**P119**：使用groupBy把元素按不同的特征划分为不同的分组。如：

```java
val p = listOf(Person("A",1),Person("B",1),Person("C",2))
>>>println(p.groupBy{ it.age }) //输出{1=[Person(name=A, age=1), Person(name=B, age=1)], 2=[Person(name=C, age=2)]}，有List转换成了Map<Int，List<Person>>
```

**P120**：**flatMap**根据实参给定的函数对集合中的每个元素做变换，然后<font color="#dd0000">**把多个列表合并(或者说平铺)**</font>成一个列表。而如果不需要做任何变换，只需要平铺一个集合，可以使用**flatter函数**。

**P121**：**Kotlin惰性集合操作的入口是Sequence接口**，这个接口中只包含一个iterator方法，用来从序列中获取值；Sequence的强大之处在于<font color="#dd0000">**序列中的元素求值是惰性的**</font>，因此使用序列可以更高效的对集合元素执行链式操作，而不需要创建额外的集合来保存过程中产生的中间结果。

**P123**：序列操作：sequence.map{ ... }.filter{ ... }.toList()，其中map和filter是中间操作，是惰性的，toList()是末端操作，返回结果。所谓惰性就是，<font color="#dd0000">**只有在请求结果时才会真的进行中间操作**</font>。另外，**序列中时逐元素处理的，处理完一个元素的全部中间操作，才会进行下一个，意味着不会发生任何变换**。

**P125**：使用generateSequence函数给定序列中的前一个元素，这个函数会计算出下一个元素。

```java
val naturalNumbers = generateSequence(0){ it + 1}
val numbersTo100 = naturalNumbers.takeWhile{ it <= 100}
>>>println(numbersTo100.sum()) //同样的，numbersTo100也是惰性操作，只有调用sum()请求结果是才会进行计算
```

**P130**：注意lambda内部没有匿名对象那样的this：没有办法引用到lambda转换成的匿名类实例。从编译器的角度来看，**lambda是一个代码块，而不是一个对象**，而且也不能把它当成对象引用。Lambda中的this引用指向的是包围它的类。

**P131**：带接收者的lambda：在lambda函数体内可以调用一个不同对象的方法，而且无须借助任何额外限定符；如with函数：

```java
fun alphabet():String{
    val _sb = StringBuilder()
    return with(_sb) { //实际上是一个接收两个参数的函数，第一个函数时_sb，第二个函数时lambda，with函数把它的第一个参数转换成第二个参数的接收者，可以使用this指代接收者，this也可以省略。
        for(letter in 'a' .. 'z') {
            this.append(letter) //this
        }
        this.toString()
    }
}
```

**P134**：apply被声明成一个扩展函数。它的接收者变成了作为实参的lambda的接收者。类似于Builder，如：

```java
TextView(context).apply { 
    text = "Sample Text"
	textSize = 20.0
}
```

**P138**：Kotlin中的可空类型特性是为了把运行的空指针异常转变为编译期的错误检测。**问号可以加在任何类型的后面来表示这个类型的变量可以存储null引用**：

```java
Type? = Type or null
```

可空标识符（？）最重要的操作是和null进行比较，**一旦你进行了比较，编译器会记住，并且再这次比较发生的作用域内把这个值当做非空对象类对待。**

**P141**：安全调用符（**?.**)，它允许你把一次null检查和一次方法调用合并成一个操作。如：

```java
s?.toUpperCase() => if(s != null) s.toUpperCase() else null
```

**P143**：**Elvis**(**?:**)运算符接收两个运算数，如果第一个运算数不为null，运算结果就是第一个运算数；如果第一个运算数为null，结果就是第二个运算数。如：

```java
fun strLenSafe(s:String?):Int = s ?. length ?: 0
>>>println(strLenSafe(null)) //输出：0
//结合Exception
val address = person.company?.address ?: throw IllegalArgumentException("No address")
```

**P145**：安全转换：**as?**经常Elvis运算符结合使用，如：

```java
val otherPerson = o as? Person?: return false
```

**P146**：非空断言(**!!**)，可以将任何值转换成非空类型。如果对null值做非空断言，则会抛出异常。明确告诉编译器知道可能为空，也愿意接受异常。

**P148**：let函数让处理可空表达式变的更容易。`安全调用"let"只在表达式不为null时执行lambda`。如：

```java
if(email!= null) sendEmailTo(email) -> 
email？.let{ email -> sendEmailTo(email)}
```

**P149**：Kotlin通常要求你在构造方法中初始化所有属性，如果**某个属性是非空类型，你就必须提供非空的初始化值**。如果你使用可空类型，意味着该属性的每一次访问都需要null检查或者**!!**运算符；延迟初始化标识符(**lateinit**)只能修饰var，因为val是final的，在编译期就确定了其变量值，也不存在延迟初始化的需求。

**P153**：`Kotlin中所有泛型类和泛型函数的类型参数默认都是可空的。`要是类型参数非空，必须要为它指定一个非空的上界，如：

```java
fun <T:Any> printHashCode(t:T) {
    println(t.hashCode())
}
```

**P154**：**Kolin把那些定义在Java代码中的类型看成平台类型**，当调用java中的方法时处理参数，或者重写java接口时处理参数。参数可以是可空也可以是非空的，需要小心处理。

**P160**：Kotlin不会自动地把数字从一种类型转换成另外一种，即使是转换成范围更大的数据类型。每种接触类型都有转换函数，如：toInt()，toLong()，toShort()等。**equals除了会比较两个元素中存储的值，还会比较装箱类型**。

**P165**：List<Int?> 表示列表中的元素可空，List<Int>? 表示整个列表可空，但是其中的元素不可空。

**P167**：Kotlin的集合设计和Java有所不同，它把访问集合数据的接口(**kotlin.collections.Collection**)和修改集合数据的接口分开了(**kotlin.collections.MutableCollection**，继承了Collection接口)

**P168**：<font color="#dd0000">**只读集合不一定是不可变的**</font>，当一个变量同时被只读集合和可变集合引用时，他就可能被改变。

**P173**：**Kotlin中的数组是一个带有类型参数的类**，创建数组的方式有包含：arrayOf、arrayOfNulls、以及Array构造方法接收数组大小和一个lambda表达式，调用lambda表达式来创建每一个数组元素。如：

```java
val letters = Array<String>(26) {i -> ('a' + i).toString() }
```

**P174**：使用toTypedArray可以将集合中的元素按照元素类型转换成对应的数组；Kotlin中数组类型包含的类型参数始终会变成对象类型，想要在数组中包含基础类型，需要使用特殊的数组类，如IntArray、BooleanArray等。

**P180**：重载算数运算符，不能定义自己的运算符。Kolin限定了你能重载哪些运算符，以及你需要在你的类中定义的对应名字的函数(如：**plus、minus、times、div、mod**)。具体如下：

```java
data class Point(val x:Int,val y:Int) {
    operator fun plus(other:Point):Point {
        return Point(x + other.x,y + other.y)
    }
}
>>> println(Point(2,4) + Point(3,6)) //输出：Point(x=5,y=10)
```

**P189**：在Kotlin中，下标运算符是一个约定。使用下标运算符读取元素会被转换为get运算符方法的调用，并且写入元素将调用set。如下，**其实是个扩展函数**：

```java
operator fun Point.get(index:Int):Int {
    return when(index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invaild coordinate $index")
    }
}
>>>println(Point(11,22)[0]) //输出：11
```

**P195**：**属性委托**，可以把一个值的读取委托给别的辅助对象，会为辅助对象生成get、set方法，当读取属性值的时候调用get方法。属性委托的一种常见用法是懒加载：

```java
class Person(val name:String) {
    val emails by lazy {loadEmails(this)}
}
```

**P207**：**高阶函数**就是以另一个函数作为参数或者返回值的函数。在Kotlin中，函数可以用**lambda**或者**函数引用**来表示。高阶函数语法：**(参数类型列表) -> 返回值类型**，需要注意的是，一个函数类型声明总是需要一个显式的返回类型，<font color="#dd0000">**在这种场景下Unit是不能省略的**</font>。如下：

```java
(int,String) -> Unit
```

高阶函数作为参数使用一：

```java
fun twoAndThree(operation:(Int,Int) -> Int) {
    val result = operation(2,3)
    println("The result is $result")
}
>>>twoAndThree({a,b -> a * b}) //输出：The result is 6
>>>twoAndThree({a,b -> a + b}) //输出：The result is 5
```

高阶函数作为参数使用二：

```java
fun String.filter(predicate:(Char) -> Boolean):String {
    var sb = StringBuilder()
    for(index in 0 until length) {
        val element = get(index)
        if(predicate(element)) sb.append(element)
    }
    return sb.toString()
}
>>>println("ab1c".filter { it in 'a' .. 'z' }) //输出：abc
```

**P211**：一个函数类型的变量时FunctionN接口的一个实现，Kotlin标准库定义了一系列的接口，Function0<R>，Function1<P1,R)，Function<P1,P2,R>等。每个接口定义了一个invoke方法，调用这个方法就会执行函数。

**P214**：返回函数的函数，**really cool！**声明一个返回另一个函数的函数，**需要指定一个函数类型作为返回类型**。

```java
enum class Delivery {STANDARD,EXPEDITED}
class Order(val itemCount:Int)
fun getShippingCostCalculator(delivery: Delivery):(Order) -> Double {
    if(delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount}
    }
    return { order -> 1.2 * order.itemCount}
}
```

**P218**：lambda表达式会被正常的编译成匿名类，这表明每调用一次lambda表达式，一个额外的类就会被创建。**并且如果lambda捕捉了某个变量，那么每次调用的时候都会创建一个新的对象**。这会带来运行时的额外开销，导致使用lambda比使用一个直接执行的相同代码的函数效率更低。<font color="#dd0000">**当一个函数被声明为inline时，它的函数体是内联的，换句话说，函数体会直接替换到函数被调用的地方，而不是被正常调用。**</font>

**P221**：如果lambda作为参数被调用，这样的代码能被容易的内联，**如果lambda在某个地方被保存起来，此时lambda表达式将不能被内联，因为必须要有一个包含这些代码的对象存在**。

**P226**：只有在以lambda作为参数的函数是内联函数的时候才能从更外层的函数返回(**return**)。一个非内联函数可以把它的lambda保存在变量中，可能在函数返回后继续使用。可以使用标签，从局部返回：

```java
fun lookForAlice2(boys:List<Boy>) {
    boys.forEach aa@{
        if(it.name == "Alice") {
            println("Found!")
            return@aa
        }
    }
    println("Alice is not found")
}
>>>forEach是一个Kotlin自带的内联函数，这里使用标签从局部返回，不会执行println语句代码
```

**229**：**return从最近使用fun关键字声明的函数返回**，在匿名函数中，不带标签的return会从匿名函数返回，而不是从包含匿名函数的函数返回。

```kotlin
fun lookForAlice(people:List<People>) { 
  people.forEach(fun(people){//从这Return
    if(person.name == "Alice") return
  })
}

fun lookForAlice(people:List<People>) { //从这Return
  people.forEach {
    if(it.name == "Alice") return
  }
}
```

**P236**：和Java中使用关键字extends来约束泛型形参上界不同，Kotlin中使用**:**来约束上界，如下：

```java
fun <T : Number> List<T>.sum() : T
```

一旦指定了类型形参T的上界，你就可以调用定义在上界类中的方法。如：

```java
fun <T:Number> oneHalf(value:T):Double {
    return value.toDouble() / 2.0
}
```

**P237**：默认的泛型参数的上界是Any，如果想限定实参为非空的，可以指定**T : Any**；可以为一个类型参数指定多个约束，如下：

```java
fun <T> ensureTrailingPeriod(seq:T):T where T:CharSequence,T:Appendable {
    if(!seq.endsWith('.')) {
        seq.append('.')
    }
    return seq
}
>>>println(ensureTrailingPeriod(StringBuilder("ABC"))) //输出：ABC.
```

**P239**：泛型的类型擦除，泛型之所以能限制容器或者方法实参的类型，是因为在**编译期**做了校验，但是到了**运行时**泛型类型会被擦出掉，也就是编译期不会区分List&lt;String&gt;和List&lt;Student&gt;；不能判断一个对象是否是List&lt;String&gt;，只能使用List<?>判断该对象是否是List；在Kotlin中使用星号投影 ：

```java
if(value is List<*>){ ... }
```

**P241**：Kotlin中内联函数的类型形参能够被**实化**，意味着可以在运行时引用实际的类型实参。把函数声明成inline并且**用relified标记类型参数**:

```java
inline fun <reified T> isA(value:Any) = value is T
>>>println(isA<String>("abc")) //输出：true
```

<font color="#dd0000">**注意，带relified类型参数的inline函数不能在java代码中调用。**</font>

**P244**：使用实化类型参数代替**类引用**(class)，如Activity中的startActivity函数：

```java
inline fun<relified T:Activity> Context.startActivity() {
  	val intent = Intent(this,T::class.java)
    startActivity(intent)
}
>>>startActivity<DetailActivity>()
```

**P246**：变型描述拥有相同基础类型和不同类型实参的类型之间是如何关联的(如List<String>和List<Any>)；当一个可变字符串列表(MutableList<String>)传递给一个期望Any对象列表的函数是否安全，取决于该字符串列表中的元素是否可变，如果是可变的，意味着列表中的元素类型可能不一致，则此时它是不安全的，编译器会禁止编译通过。

**P248**：协变：保留子类型化关系。如果Cat是Animal的子类型，那么如果List<Cat>也是List<Animal>的子类型，我们说子类型化被保留了；**类型参数T上用关键字out修饰有两层含义**：

1. 子类型化会被保留(Producer<Cat>是Producer<Animal>的子类型)
2. <font color="#dd0000">**T只能用在out位置**</font>

```java
interface Transformer<T>{
  fun transform(t:T):T  //前面的T称为"in"位置，只能消费类型为T的值；后面的T称为"out"位置，只能生产类型为T的值
}
```

需要注意，位置规则只覆盖了类外部可见的(public、protected和internal)API。私有方法的参数既不在in位置也不在out位置。in和out有点类似于java中的<? extend T>和<? super T>指定类型的上、下界限。

**P253**：![image](https://img-blog.csdnimg.cn/20190402061628299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2Mzg4ODM=,size_16,color_FFFFFF,t_70)

**P256**：可以为类型参数指定in、out来限制其为消费(可写)、生产(只读)。

**P257**：星号投射类似于java中的MyType<?>，你不知道里面存的是什么类型，所以**不能往里存数据，但是你可以往外取东西**，因为里面的元素总是Any?的子类型。

**P264**：Kotlin中注解指定实参的语法和Java有所不同

1. 要把一个类指定为注解实参，在类名后面加上**::class**，如：@MyAnnotation(MyClass**::class**)
2. 要把另一个注解指定为一个实参，去掉该注解名称前面的@，如@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))，其中ReplaceWith也是一个注解。
3. 要把一个数组指定为一个实参，使用arrayOf函数，如：@RequestMapping(path=arrayOf("A","B"))
4. 要把属性当做注解的实参使用，需要使用const修饰符修饰。

**P274**：在Kotlin中使用反射，有两种反射API，一种是标准的Java反射，定义在java.lang.reflect；第二种是Kotlin反射API，定义在kotlin.reflect中，Kotlin反射API被打包成独立的jar文件，kotlin-reflect.jar，需要自己添加依赖导入进来。**在Kotlin中使用反射需要先使用javaClass获取它的Java类，然后再通过.kotlin扩展属性，完成从Java到Kotlin的过渡。**

```java
val child = Child("Lucy")
val kClass = child.javaClass.kotlin
println(kClass.simpleName) //输出：Child
kClass.memberProperties.forEach{ println(it.name) } //输出：name
```

**P293**：Kotlin的DSL(领域特定语言，专注在特定任务或领域上，并放弃与该领域无关的功能，如SQL)是完全静态类型的，就像语言的其他功能一样。这意味着静态类型的所有优势，比如编译时错误的检测，以及更好的IDE支持，在你的API上使用DSL模式时仍然有效。

**299**：带接收者的lambda是Kotlin的一个强大特性，它可以让你使用一个结构来构建API，拥有结构是区分DSL和普通API的关键特征。带接收者的lambda格式：

**String.(Int, Int) -> Unit；String为接收者类型，(Int, Int)为参数类型**

```java
fun buildString(builderAction:StringBuilder.()-> Unit):String {
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}
val s2 = buildString {
    this.append("Hello")
    append("World")
}
```

当你将一个普通函数类型转换为扩展函数类型时，其调用方式也发生了变化。想调用一个扩展函数那样调用lambda，而不是将对象作为参数传递给lambda。

```java
val appendExcl :StringBuilder.() -> Unit = {this.append("!")}
val s3 = StringBuilder("Hi")
s3.appendExcl()
println(s3) //输出：Hi!
```

可以看到这里的带接收者的Lambda表达式，有点像一个匿名的扩展函数，且该匿名扩展函数的对象时调用者自己

**P310**：<font color="#dd0000">**invoke约定允许把自定义类型的对象当做函数一样调用**</font>。你已经见过函数类型的对象，它们可以作为函数调用；**使用invoke约定，也可以定义支持同样语法的对象**。get是我讨论过的约定之一，它允许你使用下标运算符来访问一个对象。

```java
class G(val g:String) {
    operator fun invoke(name:String) {
        println("$g,$name!")
    }
}
G("x")("w") //输出：x,w!
```

**P311**：Lambda除非是内联的，都是被编译成实现了函数式接口(Function1等)的类，而这些接口定义了具有对应数量参数的invoke方法。

**P312**：将lambda转换成一个实现了函数类型接口的类，并重写接口的invoke方法；是进行重构的一种方式。这种方法的优点是从lambda函数体重抽取的方法的作用域被金可能缩小了。

```java
data class Dog(val name:String,val color:String,val height:Int)

class GoodDogPredicate():(Dog) -> Boolean {
    override fun invoke(dog: Dog): Boolean {
        return dog.isGoodColor() && dog.isGoodHeight()
    }
    private fun Dog.isGoodColor():Boolean {
        return color == "White"
    }
    private fun Dog.isGoodHeight():Boolean {
        return height < 50
    }
}

val dog1 = Dog("Tom","Yellow",40)
val dog2 = Dog("Yame","White",40)
val dog3 = Dog("Petty","White",100)
val predicate = GoodDogPredicate()
for(d in listOf(dog1,dog2,dog3).filter(predicate)) {
    println(d.name + " is a good dog!") //Yame is a good dog
}

```













