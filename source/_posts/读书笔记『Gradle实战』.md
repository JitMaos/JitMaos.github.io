---
title: 读书笔记『Gradle实战』
date: 2019-04-05 15:34:50
tags: 读书笔记
---

项目自动化的类型：按需构建、触发构建、预定构建，基于时间的程序调度方案。

**P15**：Ant的缺点

+ 使用XML作为构建逻辑的定义语言相比于其他简明的定义语言，会导致构建脚本过于臃肿和啰嗦。
+ 复杂的构建逻辑会导致又长又难维护的构建脚本。当尝试用标记语言去定义类似if-then/if-then-else的逻辑语言时，它完全就成了一种负担。
+ Ant没有提供任何指导来告诉你如何构建项目。在一个企业级配置中，这常常会导致一个build文件每一次看上去都不一样。常用功能时常被到处拷贝。项目中每一个新的开发人员都需要去理解构建中每一个独立的部分。
+ 在没有Ivy的情况下，使用Ant很难管理依赖。在通常情况下，你需要将JAR文件提交到版本控制系统中，并且手动管理组织结构。

**P16**：Maven选择约定优于配置的思想，同时是基于构建生命周期的思想。Maven有如下缺点：

+ Maven推荐一个默认的结构和生命周期，常常会太过限制，也许不适合你的项目。
+ 为Maven写定制的扩展过于累赘。你需要学习Mojos.

**P26**：Gradle提供了具有表达性的DSL，约定优于配置的方法和强大的依赖管理。摒弃了XML，引入了动态语言Groovy来定义构建逻辑。

**P33**：对于一个Java项目，Gradle已经提供了默认的有意义的任务。例如，你可以编译Java产品源代码，运行测试和组装JAR文件。每个Java项目都以一个标准的目录布局开始。它定义了在哪里可以找到源代码、资源文件和测试代码。可以通过约定属性改变它们的默认值。

<!-- more -->

**P42**：使用Gralde命令行

+ 执行一个任务：gradle taskname
+ 执行多个任务：gradle taskA taskB taskC(注意，**如果A/B/C之间有相互引用，那么涉及到引用任务只会执行一次**)
+ 声明task依taskA**.dependsOn** taskB，**taskB会先执行**
+ 列出所有task，gradle -q tasks **--all**
+ 支持任务名的驼峰缩写，如：gradle groupTherapy => gradle **gT**
+ 排除一个任务，gradle groupTherapy <font color="#dd0000">**-x**</font>  excludeTaskName

**P47**：守护进程以后台进程方式运行Gradle，可是在使用gradle命令时添加--daemon参数来启动，**守护进程只会创建一次，并会在3小时空闲时间后自动过期。**不使用守护进程：**--no-daemon**，停止守护进程：**--stop**

**P55***：Java插件式Gradle自身装载的一个插件。告诉Gradle要使用那个插件，比如告诉它要使用Java插件：

```java
apply plugin: 'java'
```

Java插件引入的约定之一就是源代码的位置。在默认情况下，插件回到src/main/java目录下查找。Java插件提供的一个任务叫做build。这个build任务会以正确的顺序**编译你的源代码，运行测试，组装JAR文件。**运行gradle build命令后，可能得到类似:compileTestJava <font color="#dd0000">**UP-TO-DATE**</font>，其中**UP-TO-DATE"表示该任务被跳过了。Gralde的增量式构建支持自动鉴别不需要被运行的任务。

**P60**：改变项目源代码目录及改变输出目录：

```java
sourceSets {
    main {
        java {
            srcDirs = ['src']
        }
    }
}
buildDir = 'out'
```

**P61**：**定义仓库**，Gradle要求你至少定义一个仓库(respository)：

```java
repositories {
    mavenCentral()
}
```

**定义依赖**：一个依赖是通过<font color="#dd0000">**group标识符、名字和一个指定版本**</font>来确定的。如：

```java
dependencies {
    compile group: 'org.apache.commons',name:'commons-lang3',version:'3.1'
}
```

**P65**：应用插件是一个**幂等操作**，在编程中一个幂等操作的特点是**其任意多次执行所产生的影响均与一次执行的影响相同**。

**P68**：修改Gradle插件提供的默认值

```java
apply plugin 'jetty'
jettyRun { //Jetty运行Web应用的Task是jettyRun
	httpPort = 9090
    contextPath = 'todo'
}
```

**P69**：<font color="#dd0000">**Gradle包装器**</font>，它是Gradle的核心特性，能够让机器在没有安装Gradle运行时的情况下运行Gradle构建。它也让构建脚本运行在一个指定的Gradle版本上。它是通过自动从中心仓库下载Gradle运行时，解压和使用来实现的。**最终的目标是创建一个独立于系统、系统配置和Gradle版本的可靠和可重复的构建。**其实就是将Gradle的配置信息写入到项目的task中，运行该task会自动下载解压运行对应版本的Gradle，免去了自己去下载安装的过程以及可能产生的版本不一致问题。

**P78**：Gralde使用领域驱动设计(DDD)的原理为其自己的领域构建软件模型。因此，在Gradle API中有相应的类来表示project和task。**Gradle基于build.gradle中的配置实例化org.gralde.api.Project类，并且能够通过project变量使其隐式可用。**部分API接口及重要方法：

```java
<<Interface>> Project
//构建脚本配置
apply(options: Map<String,?>)
buildscript(config: Closure)
//依赖管理
dependencies(config: Closure)
configurations(config: Closure)
getDependencies()
getConfigurations()
//getter、setter
getAnt()
getName()
getDescription()
getGroup()
getPath()
getVersion()
getLogger()
setDescription(descritption: String)
setVersion(version: Object)
//创建文件
file(path: Object)
files(paths: Object...) 
fileTree(baseDir: Obejct)
//创建task
task(args: Map<String,?>,name: String,c: Closure)
task(name: String,c: Closure)
```

**P80**：Task接口

```java
//Task依赖
depnedson(tasks: Object...)
//动作定义
doFirst(action: Closure)
doLast(action: Closre)
getActions()
//输入、输出数据声明
getInputs()
getOutputs()
//getter、setter
getAnt()
getDescription()
getEnabled()
getGroup()
setDescription(descritpion: String)
setEnabled(enabled: boolean)
setGroup(group: String)
```

**P83**：动作(action)就是在task中合适的地方放置构建逻辑。Task接口提供了两个相关的方法来声明task动作：doFirst(Closure)和doLast(Closure)。左移操作符是doLast方法的快捷版本。

**P86**：理解Gradle**并不能保证task依赖的执行顺序**是很重的。

**P86**：Gradle的三个生命周期阶段：<font color="#dd0000">**初始化阶段、配置阶段、执行阶段**</font>

+ 初始化阶段，Gradle为项目创建一个Project实例。
+ 配置阶段，Gradle构建了一个模型来表示任务，并参与到构建中来。增量式构建特性决定了模型中的task是否需要被运行。
+ 执行阶段，任务按顺序执行。

**P90**：Gradle通过比较两个构建task的inputs和outpus来决定task是否是最新的，只有当inputs和outputs不同时，task才运行。

**P101**：有两种方式可以编写回调生命周期时间：在闭包中，或者是通过Gradle API所提供的监听器接口实现。

**P107**：使用dependencies脚本来定义构建所依赖的类库。使用repositories闭包告诉构建从哪里获取依赖。

**P117**：当依赖管理器在仓库中查找一个依赖时，会通过属性组合来找到它。

```java
dependencies {
  	org.hibernate  :  hibernamte_core  :  3.6.3-Final
  	// group              name              version
}
```

这里没有classifier，classifier用来在相同的group、name、version时区分不同的依赖。**这些属性以键值对的形式存在**。

**P119**：运行：**gradle dependencies**，可以得到依赖的树形图。**树形图中依赖后面的(*)表示被排除，这意味着依赖管理器选择的相同的或者另一个版本的类库；-> 表示自动使用新版本。**

```java
...
  	+--- xml-apis:xml-apis:1.0.b2 -> 1.3.03
  	...
	+--- msv:msv:2002044 (*)
...
```

**P120**：使用exclude来排除指定的传递性依赖，除了module关键字外，还可以用/或者group和/

```java
dependencies {
  	cargo('org.codehaus.cargo:cargo-ant:1.3.1') {
    	exclude group:'xml-apis',module:'xml-apis'
  	}
    cargo 'xml-apis:xml-apis:2.0.2'
}
```

​	或者可以使用transitive来排除所有的传递性依赖

```java
dependencies {
  	cargo('org.codehaus.cargo:cargo-ant:1.3.1') {
    		transitive = false	
  	}
}
```

**P121**：动态版本声明可以使用占位符latest-integration或者加号(+)

```java
dependencies {
  	cargo 'org.codehaus.cargo:cargo-ant:1.+'
}
```

**P128**：gradle的本地缓存目录在/Users/xxx/.gradle下，可以通过下面这个task打印出来

```java
task printDependencies << {
  configurations.getByName('libname').each {
    dependency -> println dependency
  }
}
```

**129**：Gradle通过比较本地和远程的**校验和**来检测仓库中的工件是否发生变化。没有发生变化的工件不会再次下载，并且会重用本地缓存中的。

**P145**：Project接口

```java
//特定的项目配置
project(path: String)
project(path: String,config: Closure)
//公共的项目配置
allprojects(action: Action<? super Project>)
allprojects(action: config: Clousre)
subprojects(action: Action<? super Project>)
subprojects(action: config: Clousre)
//项目解析顺序
evaluationDependsOn(Path: String)
evaluationDependsOnChildren()
```

**P154**：配置公共项目行为

```java
//为根项目和所有子项目设置group和version属性
allprojects {
  	group = "com.a.b"
    version = '0.1'
}
//设置Java插件只应用与子项目
subprojects {
	  apply plugin: 'java'
}
```

**P160**：如果你想构建可靠的、高质量的软件，自动化测试时开发工具箱的关键组成部分。自动化测试分为三种类型——单元测试、集成测试、功能测试。

+ 单元测试与产品代码实现一起作为一个task来执行，**目的是测试代码的最小单元**。在单元测试中，你需要避免与其他类或外部系统交互。在被测试代码中对其他组件的引用，通常用**测试替身**独立出来，这是测试中替代组件的一个通用术语，如Stub(打桩)或Mock(模拟)。
+ 集成测试用来测试一个完整的组件或子系统。集成测试的一个典型场景就是验证产品代码和数据库之间的交互。
+ 功能测试通常用于测试应用程序的端到端功能，包括从用户的角度与所有外部系统的交互。

**P162**：Java插件引入了两个新的配置，用来声明测试代码**编译或执行**所需的依赖类库：testCompile和testRuntime。比如声明一个**编译器**的依赖JUnit：

```java
dependencies {
  	testCompile 'junit:junit:4.11'
}
```

**P164**：自动化测试检测

+ 继承junit.framework.TestCase或groovy.unitGrooveTestCase的任何类或父类
+ 使用@RunWith注解的任何类或父类
+ 至少包含一个使用@Test注解的方法的任何类或父类

**P184**：每个source set定义中的源代码都需要在执行之前被编译，并且拷贝到正确的目录下。

**P200**：Gradle将插件分为两类：脚本插件和对象插件。一个脚本插件只不过是一个普通的Gradle构建脚本，它可以被导入到其他的构建脚本中。使用脚本插件，你可以做你学到的任何事情。对象插件需要实现org.gralde.api.Plugin接口。对象插件的源代码通常放在buildSrc目录下，要么和项目在一起，哟啊么是一个独立的项目，并且作为独立的JAR文件发布。

