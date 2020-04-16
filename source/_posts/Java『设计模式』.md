---
title: Java『设计模式』
tags:
  - Java
  - 设计模式
date: 2019-04-28 11:30:22
---

在看源码的过程中经常会遇到一些设计模式，下面做一下记录

#### 责任链

当你想要让一个以上的对象**有机会**能够处理某个请求的时候，可以使用责任链模式。**链中的每个对象扮演处理器，并且有一个后继对象。它可以处理请求，也可以把请求转发给后继者。**

责任链的优点：

- 将请求的发送者和接收者解耦
- 可以简化你的对象，因为它不需要知道链的结构。
- 通过改变链内的成员或调动他们的顺序，允许你动态地新增或删除责任。

<!-- moe-->

责任链的缺点：

- 可能不容易观察运行时的特征，有碍于除错。
- 并不能保证请求一定会被执行。

![img](<https://img-blog.csdnimg.cn/2019042814181333.jpg>)

看一下实例代码：

```java
public class Translator {
    Translator next;
    void translate(String s) {
        if(next != null) {
            next.translate(s);
        } else {
            System.out.println("Can't translate Word :" + s.toString());
        }
    }
    //!!setNext返回next对象，为了测试调动方便可以链式调用
    Translator setNext(Translator next) {
        this.next = next;
        return next;
    }
}
public class BaiDuTranslator extends Translator  {
    @Override
    void translate(String s) {
        if(s.equals("BaiDu")) { //自定义逻辑
            System.out.println("Processed!");
            return ;
        } else {
            super.translate(s);
        }
    }
}
public class YouDaoTranslator extends Translator {
    @Override
    void translate(String s) {
        if(s.equals("YouDao")) { //自定义逻辑
            System.out.println("Processed!");
            return ;
        } else {
            super.translate(s);
        }
    }
}
public class BeiKeTranslator extends Translator {
    @Override
    void translate(String s) {
        if(s.equals("BeiKe")) { //自定义逻辑
            System.out.println("Processed!");
            return ;
        } else {
            super.translate(s);
        }
    }
}
//测试：
BaiDuTranslator translator = new BaiDuTranslator();
translator.setNext(new YouDaoTranslator()).setNext(new BeiKeTranslator());
translator.translate("Rampage");
translator.translate("BeiKe");
//输出：
Can't translate Word :Rampage
Processed!
```

#### 代理模式

动态代理模式提供了一种通过代理对象访问目标对象的方式，目的是为了在不改变目标对象的代码的前提下**增加**额外的功能。Java中有三种代理模式，分别是静态代理、动态代理、Cglib代理。

##### 静态代理

静态代理中代理对象(Proxy)和被代理对象(Target)需要实现相同的接口或继承相同的父类。**这样在外部调用看来他们不知道自己访问的是Target还是Proxy，可以保证原有业务逻辑的连续性。**

```java
public class TestDesignPattern {
    public static void main(String[] args) {
        iTranslate translator = new EngTranslatorProxy(new EngTranslator());
        System.out.println(translator.translate("我叫大强My name is DaQiang."));
    }
    interface iTranslate {
        String translate(String source);
    }
    static class EngTranslator implements iTranslate {
        @Override
        public String translate(String source) {
            return "翻译结果:" + source;
        }
    }
    static class EngTranslatorProxy implements iTranslate {
        EngTranslator engTranslator;

        public EngTranslatorProxy(EngTranslator engTranslator) {
            this.engTranslator = engTranslator;
        }
        @Override
        public String translate(String source) {
            //添加过滤中文的逻辑
            Pattern pat = Pattern.compile("[\u4e00-\u9fa5]");
            Matcher mat = pat.matcher(source);
            source = mat.replaceAll("");
            return engTranslator.translate(source);
        }
    }
}
>>>输出：
翻译结果：My name is DaQiang.
```

##### 动态代理

静态代理每新建一个代理逻辑，就需要新建一个代理对象，这样可能导致产生很多的代理类。和静态代理不同，动态代理<font color="#dd0000">**在内存中构建代理对象**</font>，从而不需要创建很多代理类(Proxy)。

```java
public static void main(String[] args) {
        System.out.println(((iTranslate)new EngTranslatorProxy(
                new EngTranslator()).getProxyInstance())
                .translate("我叫大强My name is DaQiang."));
    }
    interface iTranslate {
        String translate(String source);
    }
    static class EngTranslator implements iTranslate {
        @Override
        public String translate(String source) {
            return "翻译结果:" + source;
        }
    }
    static class EngTranslatorProxy  {
        EngTranslator engTranslator;
        public EngTranslatorProxy(EngTranslator engTranslator) {
            this.engTranslator = engTranslator;
        }
        public Object getProxyInstance() {
            return Proxy.newProxyInstance(engTranslator.getClass().getClassLoader(),
                engTranslator.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method,
                                         Object[] args) throws Throwable {
                        //添加过滤中文的逻辑
                        Pattern pat = Pattern.compile("[\u4e00-\u9fa5]");
                        Matcher mat = pat.matcher(args[0].toString());
                        String filterSource = mat.replaceAll("");
                        Object returnValue = method.invoke(engTranslator,filterSource);
                        return returnValue;
                    }
                });
        }
    }
>>>输出：
翻译结果:My name is DaQiang.
```

##### Cglib代理

静态代理和动态代理都需要目标对象实现一个接口。然而有时候目标对象没法实现接口，就是一个类。此时<font color="#dd0000">**可以使用继承的方式扩展其子类的功能**</font>，在Java中，可以使用Cglib这个库，在内存中构建一个目标对象的子类对象。

```java
/**
 * 目标对象,没有实现任何接口
 */
public class UserDao {
    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

Cglib代理工厂：ProxyFactory.java

```java
/**
 * Cglib子类代理工厂
 * 对UserDao在内存中动态构建一个子类对象
 */
public class ProxyFactory implements MethodInterceptor{
    //维护目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();

    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("开始事务...");

        //执行目标对象的方法
        Object returnValue = method.invoke(target, args);

        System.out.println("提交事务...");

        return returnValue;
    }
}
```

测试类：

```java
/**
 * 测试类
 */
public class App {
    @Test
    public void test(){
        //目标对象
        UserDao target = new UserDao();
        //代理对象
        UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();
        //执行代理对象的方法
        proxy.save();
    }
}
```

本段落参考：[Java的三种代理模式](<https://www.cnblogs.com/cenyu/p/6289209.html>)

#### 门面模式

门面模式(外观模式)简单说就是隐藏了系统的复杂性，把一些复杂的流程封装成一个简单的接口让外部用户更简单的使用。门面模式中涉及3个角色：

+ 门面角色：外观模式的核心。它被客户角色调用，它熟悉子系统的功能。内部根据客户角色的需求预定了几种功能的组合。
+ 子系统角色:实现了子系统的功能。它对客户角色和Facade时未知的。它内部可以有系统内的相互交互，也可以由供外界调用的接口。
+ 客户角色:通过调用Facede来完成要实现的功能。

看一个例子，以计算机启动为例，将一系列CPU、内存、硬盘的启动放到一个总的启动方法里，**用户只需要调用该方法而不需要知道计算机内部的具体启动流程。**

```java
//Computer.java
public class Computer {

    CPU cpu;
    Disk dis;
    Memory memory;
    public Computer() {
        cpu = new CPU();
        dis = new Disk();
        memory = new Memory();
    }
    public void start() {
        System.out.println("Computer start begin---------");
        cpu.start();
        dis.start();
        memory.start();
        System.out.println("Computer start end---------");
    }
    public void shutdown() {
        System.out.println("Computer shutdown begin---------");
        cpu.shutdown();
        dis.shutdown();
        memory.shutdown();
        System.out.println("Computer shutdown end---------");
    }
}
//CPU.java
public class CPU {
    private String TAG = "CPU";
    public void start() {
        System.out.println(TAG + " start");
    }
    public void shutdown() {
        System.out.println(TAG + " shutdown");
    }
}
//Disk.java
public class Disk {
    private String TAG = "Disk";
    public void start() {
        System.out.println(TAG + " start");
    }
    public void shutdown() {
        System.out.println(TAG + " shutdown");
    }
}
//Memory.java
public class Memory {
    private String TAG = "Memory";
    public void start() {
        System.out.println(TAG + " start");
    }
    public void shutdown() {
        System.out.println(TAG + " shutdown");
    }
}
```

测试一下：

```java
public class Test {
    public static void main(String[] args) {
        Computer computer = new Computer();
        computer.start();
        computer.shutdown();
    }
}
>>>输出：
Computer start begin---------
CPU start
Disk start
Memory start
Computer start end---------
Computer shutdown begin---------
CPU shutdown
Disk shutdown
Memory shutdown
Computer shutdown end---------
```

#### 建造者模式(Builder)

建造者模式在Java中很常见，通常在某各类的构造方法有很多参数，且参数后期可能还会增加的情况下使用。**核心思想就是使用一个Builder来实现动态添加一个参数/配置项，返回自身对象。**

```java
public class Rocket {
    private String name;
    private int spead;
    private int weight;
    private String dest;

    public Rocket(Builder builder) {
        this.name = builder.name;
        this.spead = builder.spead;
        this.weight = builder.weight;
        this.dest = builder.dest;
    }
    public String getName() {
        return name;
    }
    public int getSpead() {
        return spead;
    }
    public int getWeight() {
        return weight;
    }
    public String getDest() {
        return dest;
    }

    //建造者
    public static class Builder {
        private String name;
        private int spead;
        private int weight;
        private String dest;

        public Builder name(String name) {
            this.name = name;
            return this;
        }
        public Builder spead(int spead) {
            this.spead = spead;
            return this;
        }
        public Builder weight(int weight) {
            this.weight = weight;
            return this;
        }
        public Builder dest(String dest) {
            this.dest = dest;
            return this;
        }
        public Rocket build() {
            return new Rocket(this);
        }
    }
}
```

测试一下：

```java
Rocket rocket = new Rocket.Builder().name("BlueStar").spead(1000).weight(2200).dest("moon").build();
```

#### 未完待续。。。

