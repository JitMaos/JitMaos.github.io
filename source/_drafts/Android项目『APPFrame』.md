---
title: Android项目『APPFrame』
tags:
---

### GreenDao使用记录

一、3.2.2上目前只支持使用java类生成dao，不支持kotlin

二、报错：Caused by: org.greenrobot.greendao.DaoException: Could not init DAOConfig

解决：添加混淆规则：

-keep class org.greenrobot.greendao.**{*;}
-keepclassmembers class * extends org.greenrobot.greendao.AbstractDao {
public static java.lang.String TABLENAME;
}
-keep class **$Properties
三、



### 使用TitleBar

一、修改Activity的theme为NoActionBar

```java
<style name="NoActionBar" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
</style>
```

二、添加menu.xml

```java
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/toolbar_viewmode"
        android:title="练习"
        app:showAsAction="always"/>
    <item
        android:id="@+id/toolbar_quickclassify"
        android:title="分类"
        app:showAsAction="always"/>
    <item
        android:id="@+id/toolbar_sort"
        android:title="排序"
        app:showAsAction="always"/>
</menu>
```



三、添加布局元素

```java
<androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        android:elevation="4dp"
        android:layout_marginTop="0dp"
        android:layout_marginStart="0dp"
        android:layout_marginEnd="0dp"
        app:menu="@menu/base_main_activity_actions"
        />
```

四、设置supportActionBar和listener

```java
setSupportActionBar(toolbar)
toolbar.setNavigationOnClickListener(this)
```

五、重写onOptionsItemSelected和onCreateOptionsMenu



### 命令行查看数据库

1. 选择一个root过的手机，或者新建一个root过得模拟器，具体可以看命令行操作用户结尾的是#(已root)，还是$(未root)，或者提示pemisssion denied。
2. cd到databases文件目录。
3. 使用adb shell命令，进入shell模式。
4. 使用sqlite3命令选择数据库文件，如sqlite3 DBNAME.db。
5. 使用.table查看当前选中数据库包含的数据表。
6. 查看具体数据表的结构：.schema tableName
7. 退出shell，.exit。



## 优化设计

### 体验优化

+ 初始化数据库时，显示加载百分比进度条，暂定计算方式有两种

  + 当前处理行数 / 总行数
  + 下载文件时，已经知道了要处理的单词数量。

  