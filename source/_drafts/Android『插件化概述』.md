---
title: Android『插件化概述』
tags:
---

## 开源项目

### [didi](https://github.com/didi)/**[VirtualAPK](https://github.com/didi/VirtualAPK)**

#### 问题记录

1. I implemented VirtualAPK library and getting Execution failed for task ':app:compileDebugJavaWithJavac'](https://stackoverflow.com/questions/56184722/i-implemented-virtualapk-library-and-getting-execution-failed-for-task-appcom)

VirtualApk is doing this.....and computeBuildMapping is not supported(based on error in your post) in gradle 3.4.1.

```
if (project.extensions.extraProperties.get(Constants.GRADLE_3_1_0)) {
                    ImmutableMap<String, String> buildMapping = Reflect.on('com.android.build.gradle.internal.ide.ModelBuilder')
                            .call('computeBuildMapping', project.gradle)
                            .get()
                    compileArtifacts = ArtifactDependencyGraph.getAllArtifacts(
                            applicationVariant.variantData.scope, AndroidArtifacts.ConsumedConfigType.COMPILE_CLASSPATH, null, buildMapping)
                } else {
                    compileArtifacts = ArtifactDependencyGraph.getAllArtifacts(
                            applicationVariant.variantData.scope, AndroidArtifacts.ConsumedConfigType.COMPILE_CLASSPATH, null)
                }
```

<font color="#dd0000">**so, try building your project with gradle plugin 3.1.0 and gradle version 4.4**</font>

```
dependencies {
        classpath 'com.android.tools.build:gradle:3.1.0'


        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
```

gradle-wrapper.properties

```
distributionUrl=https\://services.gradle.org/distributions/gradle-4.4-all.zip
```







# 实践记录

## Hook 替换Activity

### 生成Dex文件

**命令**：

> dx  --dex  --output=输出路劲/output.dex(文件名可自定义)  目标class路径(如com\example\lenovo\testwidget\widget\TestDex.class)

**问题**：.class文件编译成.dex文件does not match path问题解决

解决：https://blog.csdn.net/u012911421/article/details/89705502

### 多版本问题

摘自：

```java
import java.io.File;
import java.io.IOException;
import java.lang.reflect.Array;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;

public class ClassLoaderHookHelper {

    //23和19的差别，就是 makeXXXElements 方法名和参数要求不同
    //后者是 makeDexElements(ArrayList<File> files, File optimizedDirectory,ArrayList<IOException> suppressedExceptions)
    //前者是 makePathElements(List<File> files, File optimizedDirectory,List<IOException> suppressedExceptions)
    public static void hookV23(ClassLoader classLoader, File outDexFilePath, File optimizedDirectory)
            throws IllegalAccessException, InvocationTargetException {
//        1.取得PathClassLoader的pathList的属性
        Field pathList = ReflectionUtil.getField(classLoader, "pathList");
//        2.取得PathClassLoader的pathList的属性真实值（得到一个DexPathList对象）
        Object dexPathListObj = pathList.get(classLoader);
//        3.获得DexPathList中的dexElements 属性
        Field dexElementsField = ReflectionUtil.getField(dexPathListObj, "dexElements");
//        4.获得DexPathList对象中dexElements 属性的真实值
        Object[] oldElements = (Object[]) dexElementsField.get(dexPathListObj);

        // 第二阶段
        List<File> files = new ArrayList<>();//开始构建makeDexElements的实参
        files.add(outDexFilePath);
        List<IOException> ioExceptions = new ArrayList<>();
        //获得 DexPathList 的 makePathElements 方法
        Method makePathElementsMethod = ReflectionUtil.getMethod(
                dexPathListObj, "makePathElements", List.class, File.class, List.class);
        assert makePathElementsMethod != null;
        //构建出一个新的Element数组
        Object[] newElements = (Object[]) makePathElementsMethod.invoke(
                null, files, optimizedDirectory, ioExceptions);
        //这个方法是静态方法，所以不需要传实例,直接invoke;这里取得的返回值就是 我们外部的dex文件构建成的 Element数组

        //下面把新数组和旧数组合并，注意新数组放前面
        Object[] dexElements = null;
        if (newElements != null && newElements.length > 0) {
            dexElements = (Object[]) Array.newInstance(oldElements.getClass().getComponentType(),
                    oldElements.length + newElements.length);//先建一个新容器
            //这5个参数解释一下, 如果是将A,B 你找AB的顺序组合成数组C，
            // 那么参数的含义，依次是 A对象，A数组开始复制的位置，C对象，C对象的开始存放的位置，数组A中要复制的元素个数
            System.arraycopy(
                    newElements, 0, dexElements, 0, newElements.length);//新来的数组放前面
            System.arraycopy(
                    oldElements, 0, dexElements, newElements.length, oldElements.length);
        }
        //最后把合并之后的数组设置到 dexElements里面
        dexElementsField.set(dexPathListObj, dexElements);
    }

    public static void hookV19(ClassLoader classLoader, File outDexFilePath, File optimizedDirectory)
            throws IllegalAccessException, InvocationTargetException {
        //1、获得DexPathList pathList 属性
        Field pathList = ReflectionUtil.getField(classLoader, "pathList");
        //2、获得DexPathList pathList对象
        Object dexPathListObj = pathList.get(classLoader);
        //3、获得DexPathList的dexElements属性
        Field dexElementsField = ReflectionUtil.getField(dexPathListObj, "dexElements");
        //4、获得pathList对象中 dexElements 的属性值
        Object[] oldElements = (Object[]) dexElementsField.get(dexPathListObj);

        //开始构建makeDexElements的实参
        List<File> files = new ArrayList<>();
        files.add(outDexFilePath);
        List<IOException> ioExceptions = new ArrayList<>();
        //获得 DexPathList 的 makeDexElements 方法
        Method makePathElementsMethod = ReflectionUtil.getMethod(
                dexPathListObj, "makeDexElements", ArrayList.class, File.class, ArrayList.class);//别忘了后面的参数列表
        //这个方法是静态方法，所以不需要传实例,直接invoke;这里取得的返回值就是 我们外部的dex文件构建成的 Element数组
        Object[] newElements = (Object[]) makePathElementsMethod.invoke(
                null, files, optimizedDirectory, ioExceptions);

        //下面把新数组和旧数组合并，注意新数组放前面
        Object[] dexElements = null;
        if (newElements != null && newElements.length > 0) {
            //先建一个新容器
            dexElements = (Object[]) Array.newInstance(oldElements.getClass().getComponentType(),
                    oldElements.length + newElements.length);
            //这5个参数解释一下, 如果是将A,B 你找AB的顺序组合成数组C，那么参数的含义，依次是 A对象，A数组开始复制的位置，C对象，C对象的开始存放的位置，数组A中要复制的元素个数
            System.arraycopy(
                    newElements, 0, dexElements, 0, newElements.length);//新来的数组放前面
            System.arraycopy(
                    oldElements, 0, dexElements, newElements.length, oldElements.length);
        }

        //最后把合并之后的数组设置到 dexElements里面
        dexElementsField.set(dexPathListObj, dexElements);
    }

    //14和19的区别，是这个方法 makeDexElements(ArrayList<File> files,File optimizedDirectory)···它又少了一个参数
    public static void hookV14(ClassLoader classLoader, File outDexFilePath, File optimizedDirectory)
            throws IllegalAccessException, InvocationTargetException {
        //1、获得DexPathList pathList 属性
        Field pathList = ReflectionUtil.getField(classLoader, "pathList");
        //2、获得DexPathList pathList对象
        Object dexPathListObj = pathList.get(classLoader);
        //3、获得DexPathList的dexElements属性
        Field dexElementsField = ReflectionUtil.getField(dexPathListObj, "dexElements");
        //4、获得pathList对象中 dexElements 的属性值
        Object[] oldElements = (Object[]) dexElementsField.get(dexPathListObj);

        //开始构建makeDexElements的实参
        List<File> files = new ArrayList<>();
        files.add(outDexFilePath);
        List<IOException> ioExceptions = new ArrayList<>();
        //获得 DexPathList 的 makeDexElements 方法
        Method makePathElementsMethod = ReflectionUtil.getMethod(
                dexPathListObj, "makeDexElements", ArrayList.class, File.class);//别忘了后面的参数列表
        //这个方法是静态方法，所以不需要传实例,直接invoke;这里取得的返回值就是 我们外部的dex文件构建成的 Element数组
        Object[] newElements = (Object[]) makePathElementsMethod.invoke(
                null, files, optimizedDirectory, ioExceptions);

        //下面把新数组和旧数组合并，注意新数组放前面
        Object[] dexElements = null;
        if (newElements != null && newElements.length > 0) {
            //先建一个新容器
            dexElements = (Object[]) Array.newInstance(oldElements.getClass().getComponentType(),
                    oldElements.length + newElements.length);
            //这5个参数解释一下, 如果是将A,B 你找AB的顺序组合成数组C，那么参数的含义，依次是 A对象，A数组开始复制的位置，C对象，C对象的开始存放的位置，数组A中要复制的元素个数
            System.arraycopy(
                    newElements, 0, dexElements, 0, newElements.length);//新来的数组放前面
            System.arraycopy(
                    oldElements, 0, dexElements, newElements.length, oldElements.length);
        }

        //最后把合并之后的数组设置到 dexElements里面
        dexElementsField.set(dexPathListObj, dexElements);
    }
}

```

