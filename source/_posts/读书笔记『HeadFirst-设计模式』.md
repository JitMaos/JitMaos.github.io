---
title: 读书笔记『HeadFirst-设计模式』
date: 2019-05-12 13:44:35
tags: [读书笔记,设计模式]
---


时隔5年，重读《Head First 设计模式》，这本书写的很很好，语言生动，举的例子很形象。看着很顺畅，下面按内容顺序略做笔记。

### 策略模式

#### 模式定义

定义了算法族，分别封装起来，让它们之间可以相互替换，**此模式让算法的变化独立于使用算法的客户。**

#### 个人理解

1. 封装所有可能发生变化的算法成一个独立的算法族。
2. 对象持有算法族接口对象的引用。
3. 对象方法委托算法族接口对象去实现。
4. **有一个**比**是一个**更好，或者说**多用组合，少用继承**。

<!-- more -->

#### 类图

场景说明：设计有很多鸭子(如橡皮鸭，红头鸭子，木头鸭子等)有2个行为，不同的鸭子有不同的**叫声**和**飞行模式**，使用**策略模式**设计类图结构。

![未使用策略模式实例](https://img-blog.csdnimg.cn/2019051213434533.jpg)

![策略模式实例](https://img-blog.csdnimg.cn/20190512134139882.jpg)

#### 笔记

**设计原则**：把会变化的部分取出并封装起来，以便以后可以轻易地改动或扩展此部分，而不影响不需要变化的其他部分。

**P9**：Java接口不包含具体实现代码，所以继承接口无法达到代码的复用。

### 观察者模式

#### 模式定义

定义了对象之间的<font color="#dd0000">**一对多**</font>依赖，这样一来当一个对象改变状态时，它所有的依赖者都会手动通知并自动更新。

#### 类图

场景说明：有一个气象站可以通过气象数据，有不同的公告牌(如：根据气压计显示天气预报的ForecastDisplay、显示平均值的统计显示板StaticsDisplay、基于当前观测值的显示板CurrentDisplay等)**都需要**监听天气数据的变化并刷新显示内容。

![观察者模式](https://img-blog.csdnimg.cn/20190512180629148.jpg)

#### 笔记

**P51：**当你试图 够了观察者模式时，可以利用**报纸订阅服务，以及出版者和订阅者比拟这一切。**

**P53：**松耦合的设计之所以能让我们建立有弹性的OO系统，能够应对变化，是因为对象之间的相互依赖降到了最低。

### 装饰者模式

#### 模式定义

动态地将责任附加到对象上，若要扩展功能，装饰者提供比继承更具有弹性的替代方案。

#### 场景举例

咖啡店里不同咖啡的有不同的价格，且用户自行选择添加不同的配料(牛奶，奶泡，摩卡等)，不同的配料价格不一样。而且还需要考虑后期的扩展，比如：咖啡店开发了一个新品种的咖啡、某种配料的价格改变了等等。**这种场景下使用装饰者比较合适**，可以这么理解：<font color="#dd0000">**首先咖啡有价格，咖啡加上配料后还是一杯咖啡，这里的配料只是起装饰的作用，同时配料也有价格属性。**</font>

#### 笔记

**P85：**利用继承设计子类的行为，是在**编译时静态决定的**，而且所有的子类都会继承到相同的行为。然而，如果能够利用组合的做法扩展对象的行为，就可以在**运行时动态地进行扩展。**

<font color="#dd0000">**P86：类应该对扩展开放，对修改关闭——开闭原则。**</font>

**P90：**<font color="#dd0000">**装饰者和被装饰对象有相同的超类型**</font>(这一点和Builder模式及Factory模式有显著的区别，可以作为区分的依据)。

**P93：**当我们将被装饰者与组件组合时，就是在加入新的行为。所得到的新的行为，**并不是继承自超类，而是由组合对象得来的。**

**P102：**Java I/O库就是使用装饰者模式实现的，可以以此编写自己的InputStream实现：

```java
//LowerCaseInputStream.java
import java.io.FilterInputStream;
import java.io.IOException;
import java.io.InputStream;
public class LowerCaseInputStream extends FilterInputStream {

    protected LowerCaseInputStream(InputStream in) {
        super(in);
    }
    public int read() throws IOException {
        int c = super.read();
        return (c == -1 ? c : Character.toLowerCase((char)c));
    }
    public int read(byte[] b,int offset,int len) throws IOException {
        int result = super.read(b,offset,len);
        for(int i=offset;i<offset+result;i++) {
            b[i] = (byte)Character.toLowerCase((char)b[i]);
        }
        return result;
    }
}
```

测试：

```java
 try {
   InputStream in = new LowerCaseInputStream(
     new BufferedInputStream(
       new FileInputStream("TestCustomInputStream.txt")));
   while((c = in.read()) >= 0) {
     System.out.print((char)c);
   }
   in.close();
 } catch (IOException e) {
   e.printStackTrace();
 }
```

文件TestCustomInputStream.txt：

```java
AAbbCCdDEfghi
```

输出：

```java
aabbccddefghi
```

**P104：**装饰者模式也有缺点，就是有时会在设计中加入大量的小类，者偶尔会导致别人不容易理解。以Java I/O库来说，人们第一次接触到这个库时，往往无法轻易地理解它。但是如果能认识到这些类都是用来包装InputStream的，一切都会变得简单多了。

#### 类图

![装饰者模式](https://img-blog.csdnimg.cn/20190513224943906.jpg)



### 工厂模式

#### 模式定义

**定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个，工厂方法让类把实例化推迟到子类。**

所谓**决定**并不是指模式允许子类本身在运行时做决定，**而是指创建类的时候，不需要知道实际创建的产品是哪个**，<font color="#dd0000">**选择使用哪个子类，自然就决定了实际创建的产品是什么。**</font>

**设计原则：要依赖抽象，不要依赖具体类。**

#### 场景举例

设有一个披萨品牌，在多个城市都有连锁店，不同的区域有不同的风味，每家店有多种披萨产品。

```java
public class PizzaTestDrive {
    public static void main(String[] args) {
        PizzaFactory nyFactory = new NYPizzaFactory();
        nyFactory.createPizza("cheese");
        
        PizzaFactory chicagoFactory = new ChicagoPizzaFactory();
        chicagoFactory.createPizza("cheese");
    }
}
```

#### 类图

![工厂模式](https://img-blog.csdnimg.cn/20190521071512403.jpg)

###抽象工厂

#### 模式定义

提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指明具体类。

#### 工厂VS抽象工厂

工厂模式和抽象工厂模式都是用来创建对象的，工厂模式使用<font color="#dd0000">**继承**</font>的方式，抽象工厂使用的是<font color="#dd0000">**组合**</font>的方式；工厂方法负责将客户从<font color="#dd0000">**具体类型**</font>中解耦，而抽象工厂可以把客户从<font color="#dd0000">**实际具体产品**</font>中解耦。 



### 单例模式

#### 模式定义

确保一个类只有一个实例，并提供一个全局访问点。**使用单例模式的一个好处是做到了延迟加载**。

一般如果在**非多线程**的情况下可以使用最简单的不加锁的形式：

```java
public class DBHelper {
    private static DBHelper instance;

    private DBHelper() {}
    public static DBHelper getInstance() {
        if(instance == null) {
            instance = new DBHelper();
        }
        return instance;
    }
}
```

上面的代码，如果在多线程环境下，可能还是会创建多个实例对象，此时对方法进行**同步**

```java
//多线程，但并发情况不多，可以对整个方法进行同步
public class DBHelper {
    private static DBHelper instance;

    private DBHelper() {}
    public static synchronized DBHelper getInstance() {
        if(instance == null) {
            instance = new DBHelper();
        }
        return instance;
    }
}
```

如果getInstance()的性能对于应用程序很关键，那么应该使用<font color="#dd0000">**双重检测**</font>的方法

```java
//getIntance()对性能影响很关键，使用"双重检测"
public class DBHelper {
    private volatile static DBHelper instance;

    private DBHelper() {}
    public synchronized DBHelper getInstance() {
        if(instance == null) {
            synchronized (DBHelper.class) {
                if(instance == null) {
                    instance = new DBHelper();
                }
            }
        }
        return instance;
    }
}
```

最后，一种更为简单有效的方法是**取消延迟实例化**的方式，避免多线程同步问题。

```java
//一种更为简单有效的方法是"取消延迟实例化"的方式，避免多线程同步问题。
public class DBHelper {
    private static DBHelper instance = new DBHelper();

    private DBHelper() {}
    public static DBHelper getInstance() {
        return instance;
    }
}
```

### 命令模式

**P196**：命令模式可以将"动作的请求者"从"动作的执行者"对象中解耦。

**P199**：以订单-服务员-厨师为例，**服务员其实不必担心订单的内容是什么，或者由谁来负责餐点。他只需要知道，订单有一个orderUp()方法可以调用，这就够了。**

#### 模式定义

**命令模式将"请求"封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。**



### 代理模式

#### 模式定义

代理模式为另一个对象提供一个替身或占位符以控制对整个对象的访问。

#### 静态代理

**P471**：和装饰者看起来很像，不过装饰者是为了给对象增加行为，而代理则是为了控制对对象的访问。

#### 动态代理



以为Java已经为你创建了Proxy类，所以你需要有办法来告诉Proxy类你需要什么。你不能像以前把代码放到Proxy类中，因为Proxy不是你直接实现的。既然这样的代码不能放在Proxy类中，那么要放在哪里？放在InvocationHandler中。InvocationHandler的工作是响应代理的任何调用。你可以把InvocationHandler想成是代理收到方法调用后，请求做实际工作的对象。

未完待续~~

