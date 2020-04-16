---
title: Android代码片段
tags:
---

#### 集成Arouter

1. 在lib_base的build.gradle中

   ```java
   dependencies {
     com.alibaba:arouter-api:1.4.1
   }
   ```

2. 在功能Module的build.gralde中

   ```java
   android {
       defaultConfig {
        ...
           //阿里路由框架配置
           javaCompileOptions {
               annotationProcessorOptions {
                   arguments = [AROUTER_MODULE_NAME: project.getName()]
               }
           }
       }
   }
   dependencies {
       annotationProcessor 'com.alibaba:arouter-compiler:1.1.4'
       implementation project(':lib_base') //引用lib_base
   }
   ```

#### 配置功能Module的共同引用的config.gradle

```java
if (isModule.toBoolean()) {
    //作为独立App应用运行
    apply plugin: 'com.android.application'
} else {
    //作为组件运行
    apply plugin: 'com.android.library'
}
android {
    ...
    defaultConfig {
        ...
        //阿里路由框架配置
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [AROUTER_MODULE_NAME: project.getName()]
            }
        }
    }
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
            if (isModule.toBoolean()) {
                //独立运行
                manifest.srcFile 'src/main/module/AndroidManifest.xml'
            } else {
                //合并到宿主
                manifest.srcFile 'src/main/AndroidManifest.xml'
                resources {
                    //正式版本时，排除alone文件夹下所有调试文件
                    exclude 'src/module/alone/*'
                }
            }
        }
    }
    buildTypes {
       ...
    }
    dataBinding {
        enabled true
    }
}
```

#### 配置lib_base的build.gradle

在lib_base中配置功能Module需要用到的support、Glide、Kotlin、Arouter等类库，其自身因为不需要进行页面跳转，所以不用引用Arouter。

```java
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-android'

android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion
    defaultConfig {
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName
    }
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
            if (isModule) {
            } else {
                resources {
                    //正式版本时，排除debug文件夹下所有调试文件
                    exclude 'src/debug/*'
                }
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

//Retrofit/RxJava/Glide/RxBinding/OKHttp/Kotlin/RecyclerViewHelper/EventBus/ButterKnife
dependencies {
    api files('libs/afinal_0.5.jar')

    //公用的support相关库在base中依赖
    api rootProject.ext.support["design"]
    api rootProject.ext.support["appcompat-v7"]
    api rootProject.ext.support["recyclerview-v7"]

    //Recycler-Helper
    api rootProject.ext.dependencies["recyclerHelper"]

    // Glide
    api rootProject.ext.dependencies["glide"]
    annotationProcessor rootProject.ext.dependencies["glide-compiler"]
    // OKHttp
    api rootProject.ext.dependencies["okhttp"]

    //kotlin
    api "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
      
    //阿里路由框架
    api rootProject.ext.dependencies["arouter-api"]

}
repositories {
    mavenCentral()
}

```

#### 集成Kotlin

1. 在项目根目录下的build.gradle添加

   ```java
   buildscript {
       ext.kotlin_version = '1.3.21'
       dependencies {
           classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
       }
   }
   ```

2. 在module的build.gradle中，注意是<font color="#dd0000">**jdk7**</font>，不是**jre7**。

   ```java
   apply plugin: 'kotlin-android'
   apply plugin: 'kotlin-android-extensions'
   dependencies {
     //${kotlin_version}会读取根目录的build.gralde的ext.kotlin_version值填入
   	api "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
   }
   ```

#### 集成Glide

