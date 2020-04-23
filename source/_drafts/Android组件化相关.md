---
title: Android组件化相关
date: 2019-03-29 06:09:54
tags:
---

一、新建需要的module，appModule、ModuleA、ModuleB、lib_common、lib_base

二、配置Arouter

1. 在gradle.properies配置isModule

   ```java
   isModule = true
   ```

2. 在各个组件Module的build.gradle中配置

   ```java
   if(isModule.toBoolean()) {
       apply plugin: 'com.android.application'
   } else {
       apply plugin: 'com.android.library'
   }
   ```

3. 配置组件Module的开发AndroidManifest.xml

   ```java
   android {
       sourceSets {
           main {
               if (isModule.toBoolean()) {
                   //指定组件Module独立开发时的Manifest
                   manifest.srcFile 'src/main/module/AndroidManifest.xml'
               } else {
                   manifest.srcFile 'src/main/AndroidManifest.xml'
                   //集成开发模式下排除debug文件夹中的所有Java文件
                   java {
                       //集成开发时需排除Application文件
                       exclude 'debug/**'
                   }
               }
           }
       }
   }
   ```

4. 在module的build.gradle中添加

   ```java
   android {
       defaultConfig {
           //ARouter
           javaCompileOptions {
               annotationProcessorOptions {
                   arguments = [moduleName: project.getName()]
               }
           }
       }
   }
   dependencies {
       annotationProcessor deps.arouter.compiler
   }
   ```

   

三、配置编译条件

四、测试跳转

五、继承MVVM，(Kotlin+双向绑定)

六、功能开发

七、基础library完善



八、其他

1. **资源冲突**

   可以在module的gradle中配置前缀，可以使用自定义名称，或者直接使用module名称，**不过这个设置只能在作为library编译时起作用**，如：

   ```java
   android {
       resourcePrefix 'im_'
       // resourcePrefix '${project.name}_'
   }
   ```

2. 

3. 

   

   