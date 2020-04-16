---
title: Glide『基础使用』
tags: 
---

今天开始学习Glide的源码，大概浏览了一下代码量挺大的，虽然其中有很多只是重复的不同的Loader、Encoder、Transformation等代码。但是乍一看代码量还是多，比OKHttp还要多。今天先看一下一些常见的用法。

<font color="#dd0000">**！！代码基于 Glide 4.8.0**</font>

#### 基础用法

```java
RequestOptions options = new RequestOptions()
  .placeholder(R.drawable.s) //占位符图片
  .error(R.drawable.s) //加载失败回调图片
  .override(300,300) //指定加载图片大小
  .fitCenter() //指定图片缩放类型为fitCenter
  .centerCrop() //指定图片缩放类型为fitCenter
  .centerInside() //指定图片缩放类型为centerInside
  .skipMemoryCache(true) //跳过内存缓存，默认是使用内存缓存的
  .diskCacheStrategy(DiskCacheStrategy.ALL) //缓存所有版本额图片
  .diskCacheStrategy(DiskCacheStrategy.AUTOMATIC) //默认选项，自动选择一个图片进行缓存
  .diskCacheStrategy(DiskCacheStrategy.DATA) //将解码之前的数据直接写到磁盘
  .diskCacheStrategy(DiskCacheStrategy.NONE) //跳过磁盘缓存
  .diskCacheStrategy(DiskCacheStrategy.RESOURCE) //只缓存最终显示的图片
  .priority(Priority.IMMEDIATE) //加载优先级,不保证完全按指定执行
  .priority(Priority.HIGH) //加载优先级,不保证完全按指定执行
  .priority(Priority.NORMAL) //加载优先级,不保证完全按指定执行
  .priority(Priority.LOW); //加载优先级,不保证完全按指定执行

//设定转换动画
TransitionOptions transitionOptions = new BitmapTransitionOptions();
transitionOptions.transition(R.anim.zoom_exit);

Glide.with(this)
  //.asGif() //显示Gif
  .asBitmap()//总是尝试以Bitmap去加载图片资源，即使实际上是一个动画图片
  .load("${IMG_URL}")
  .listener(new RequestListener<Bitmap>() { //设置加载回调
    @Override
    public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Bitmap> target, boolean isFirstResource) {
      return false;
    }

    @Override
    public boolean onResourceReady(Bitmap resource, Object model, Target<Bitmap> target, DataSource dataSource, boolean isFirstResource) {
      return false;
    }
  })
  .apply(options) //应用RequestOptions
  .transition(transitionOptions) //应用transition
  .into(iv);
```

一般我们只使用with,load,into方法，这三个方法也是Glide的主要流程所在：

```java
Glide.with(this).load("${IMG_URL}").into(iv);
```

**with**方法的参数是一个Context，对应的是Glide绑定界面生命周期的实现。具体看这里：



**load**方法对应的是任务的构造，负责管理加载任务的过程。具体看这里：



**into**方法对应的是开始执行加载任务的过程。具体看这里：





