---
title: Android『AnnotationProcessor基础』
tags: [Android,AnnotationProcessor]
---

看看AnnotationProcessor，是Android替代APT(个人开发者开发的，已停止维护)的编译器生成代码的库。注入Dagger2、EventBus、ButterKnife都使用了AnnotationProcessor来实现功能。先了解一下基本知识：

#### 元注解

元注解指的是**用来修饰注解的注解**，包括：

+ @Retention：定义注解的保留策略
+ @Target：定义注解的作用目标
+ @Inherited：说明该注解将被包含在javadoc中
+ @Documented：说明子类可以继承父类中的该注解

#### RetentionPolicy

```java
//java.lang.annotation.RetentionPolicy.java
/**
 * Annotation retention policy.  The constants of this enumerated type
 * describe the various policies for retaining annotations.  They are used
 * in conjunction with the {@link Retention} meta-annotation type to specify
 * how long annotations are to be retained.
 *
 * @author  Joshua Bloch
 * @since 1.5
 */
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

这个枚举表示注解的保留策略，他们会在元注解Retention中使用到，用来指定对应的注解能够保留多久。

+ SOURCE，注解会被编译器移除
+ CLASS，注解可以编译器的保留在class文件中，**但是不能在虚拟机运行时保留**。这也是默认的注解保留行为。
+ RUNTIME，注解可以在编译器及虚拟机的运行都保留下来，所以这时注解可以用反射读取。

#### ElementType

```java
//java.lang.annotation.ElementType.java
/**
 * 这些常量用来表明注解的注解范围。在元注解Target中使用得到。
 */
public enum ElementType {
    /** 类，接口(含注解)*/
    TYPE,
    /** 成员变量(含枚举常量) */
    FIELD,
    /** 方法声明 */
    METHOD,

    /** 参数声明 */
    PARAMETER,

    /** 构造方法声明 */
    CONSTRUCTOR,

    /** 局部变量声明 */
    LOCAL_VARIABLE,

    /** 注解声明 */
    ANNOTATION_TYPE,

    /** 包声明 */
    PACKAGE,

    /**
     * 类型变量
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * 任何使用类型的地方
     * @since 1.8
     */
    TYPE_USE
}
```

总结来说你需要三个文件：

一个自定义的注解、一个自定义的AbstractProcessor用于基于自定义注解生成java文件、以及一个工具类Tool用于绑定/触发生成java文件动作。

大概过程是当我们Activity A中调用Tool的action()方法触发绑定动作时，会连接到AbstractPRocessor，而AbstractProcessor中有自定义的处理逻辑，大致流程是遍历Activity A中所有的复合条件的注解修饰对象，然后对这些对象调用方法。

