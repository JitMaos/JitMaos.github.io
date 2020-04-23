---
title: Dragger2「基础使用」
tags:
---

#### 基本使用

首先Dagger是一个Android上用于实现依赖注入功能的开源库，之所以要使用依赖注入是**为了解耦**。今天简单聊一下Dagger的基本使用。Dagger的注入主要有2中方式：

+ @Injector注释构造方法提供实例；

1. 在app的build.gradle中添加依赖：

   ```java
   compile 'com.google.dagger:dagger:2.22.1'
   annotationProcessor 'com.google.dagger:dagger-compiler:2.22.1'
   ```

2. 使用依赖注解@Inject

   Man有一辆Car，现在要把Car注入到Man中去，新建Car和Man，注意这里的注入注解@Inject

   ```java
   //Car.java
   public class Car {
       @Inject
       Car() {}
   }
   ```

   ```java
   //Man.java
   public class Man {
       @Inject
       Car car;
   }
   ```

   编译一下代码，会在build/generated/source/apt/debug/package下生成Car_Factory和Man_MembersInjector。

   ```java
   //Car_Factory.java
   //Car的提供者
   public final class Car_Factory implements Factory<Car> {
     private static final Car_Factory INSTANCE = new Car_Factory();
   
     @Override
     public Car get() {
       return new Car();
     }
   
     public static Car_Factory create() {
       return INSTANCE;
     }
   
     public static Car newInstance() {
       return new Car();
     }
   }
   ```

   ```java
   //Man_MembersInjector.java
   //Car的注入目标
   public final class Man_MembersInjector implements MembersInjector<Man> {
     private final Provider<Car> carProvider;
   
     public Man_MembersInjector(Provider<Car> carProvider) {
       this.carProvider = carProvider;
     }
   
     public static MembersInjector<Man> create(Provider<Car> carProvider) {
       return new Man_MembersInjector(carProvider);
     }
   
     @Override
     public void injectMembers(Man instance) {
       injectCar(instance, carProvider.get());
     }
   
     public static void injectCar(Man instance, Car car) {
       instance.car = car;
     }
   }
   ```

   从上面的代码可以看出来，，好吧，，可能一下子不容易看出来。Car_Factory是Car的提供者；Man_MembersInjector是Man上面的Car的注入器，在其中有Provider\<Car\>用于提供Car的实例。问题是前面明明是Factory，怎么这里需要的是Provider呢？其实**Factory是一种特殊的Provider，它继承自Provider**。可以看源码：

   <!--more-->
   
   ```java
   /**
    * An {@linkplain Scope unscoped} {@link Provider}. While a {@link Provider} <i>may</i> apply scoping semantics while providing an instance, a factory implementation is guaranteed to exercise the binding logic ({@link Inject} constructors, {@link Provides} methods) upon each call to {@link #get}.
    *
    * <p>Note that while subsequent calls to {@link #get} will create new instances for bindings such as those created by {@link Inject} constructors, a new instance is not guaranteed by all bindings. For example, {@link Provides} methods may be implemented in ways that return the same instance for each call.
    */
   public interface Factory<T> extends Provider<T> {
   }
   ```
   
   现在，只有**Car_Factory**和**Man_MembersInjector**是不能完成注入动作的，**因为没有触发注入动作的时机**，那么此时我们需要一个**桥梁**来连接**Car_Factory**和**Man_MembersInjector**，那就是Componant。新建一个ManComponant(**名字其实可以随意**)
   
   ```java
   @Component
   public interface ManComponant {
       void injectMan(Man man);
   }
   
   ```
   
   Build或make之后，会自动生成DaggerManComponant：
   
   ```java
   public final class DaggerManComponant implements ManComponant {
     private DaggerManComponant() {}
   
     public static Builder builder() {
       return new Builder();
     }
   
     public static ManComponant create() {
       return new Builder().build();
     }
   
     @Override
     public void injectMan(Man man) {
       injectMan2(man);
     }
   
     private Man injectMan2(Man instance) {
       //这里的new Car()其实是Car_Factory.get()里面的代码逻辑
       Man_MembersInjector.injectCar(instance, new Car());
    		return instance;
     }
   
     public static final class Builder {
       private Builder() {}
   
       public ManComponant build() {
         return new DaggerManComponant();
       }
     }
   }
   ```
   


   最后，可以使用注入了：

   ```kotlin
   findViewById<View>(R.id.btnTestInject).setOnClickListener {
       val man = Man()
       Log.d(TAG, man.car.toString())
   }
   ```

使用Factory方式注入有以下几个问题：

- 接口没有构造函数
- 第三方库的类不能被标注
- 构造函数中的参数必须配置

+ 使用@Module提供实例;

##### Provider方式

假设有一个火箭Rocket，需要注入一个发动机(Engine)，使用Provider+Module实现

1. 新建Engine

   ```java
   //Engine.java
   public class Engine {
       String name;
   }
   ```

2. 新建Rocket

   ```java
   //Rocket.java
   public class Rocket {
       String name;
       @Inject
       Engine engine;
   }
   ```

3. 新建EngineModule，用于提供Engine对象

   ```java
   @Module
   public class EngineModule {
       @Provides
       static Engine provideEngine() {
           return new Engine();
       }
   }
   ```

4. 新建RocketComponent，用于关联Rocket和EngineModule
   ```java
   @Component(modules = EngineModule.class)
   public interface RocketComponent {
       void inject(Rocket rocket);
   }
   ```

5. make一下代码，生成编译类DaggerCommonComponent

```java
//输出
D/Main: test.dagger.method2_provider.Rocket@99fb92f
D/Main: test.dagger.method2_provider.Rocket@5def28
D/Main: test.dagger.method2_provider.Rocket@f77d37d
D/Main: test.dagger.method2_provider.Rocket@15023be
```

还有Factory接口，注意这里Factory和Provider是继承关系的，后面还会聊到Provider接口。

```java

/**
 * 用于提供T的实例，通常由injector使用。对于可以注入的类型T，你可以注入一个Provider<T>，而不是直接注入一个T，这样有以下好处：
 	接收多个实例对象
 	懒加载或者选择只接收一个实例对象
 	中断循环依赖
 	虚拟的作用域使你可以从一个作用域中的更小的作用域中查找实例
 * Provides instances of {@code T}. Typically implemented by an injector. For
 * any type {@code T} that can be injected, you can also inject
 * {@code Provider<T>}. Compared to injecting {@code T} directly, injecting
 * {@code Provider<T>} enables:
 *
 * <ul>
 *   <li>retrieving multiple instances.</li>
 *   <li>lazy or optional retrieval of an instance.</li>
 *   <li>breaking circular dependencies.</li>
 *   <li>abstracting scope so you can look up an instance in a smaller scope
 *      from an instance in a containing scope.</li>
 * </ul>
 *
 * <p>For example:
 *
 * <pre>
 *   class Car {
 *     &#064;Inject Car(Provider&lt;Seat> seatProvider) {
 *       Seat driver = seatProvider.get();
 *       Seat passenger = seatProvider.get();
 *       ...
 *     }
 *   }</pre>
 */
public interface Provider<T> {

    /**
     * Provides a fully-constructed and injected instance of {@code T}.
     *
     * @throws RuntimeException if the injector encounters an error while
     *  providing an instance. For example, if an injectable member on
     *  {@code T} throws an exception, the injector may wrap the exception
     *  and throw it to the caller of {@code get()}. Callers should not try
     *  to handle such exceptions as the behavior may vary across injector
     *  implementations and even different configurations of the same injector.
     */
    T get();
}

```

从注释中可以看到：一个Factory是一个未指定Scope的Provider，一个Factory是为了保证当调用get方法时，能够实现构造方法的Intect绑定逻辑；注意后续每次在构造方法上使用Inject都会创建新的对象；而Provider不一样，他可能每次调用都返回相同的对象。

#### Scope、Lazy、Qualifier



#### Component、SubComponent



#### MVP结合Dagger2

