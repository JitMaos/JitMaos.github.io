---
title: 源码学习『OKHTTP——概括』
tags:
---

网上有很多讲OKHTTP源码分析的代码，我也自己写一个，主要是因为最近一直在做Android基础架构相关的工作。对好的开源项目比较有学习研究的兴趣。

本来想看懂以后写在一篇博客里，尽量概括通俗一点，但是写着写着发现内容太多了。而且揉在一起反而有些混乱，所以现在拆成四篇，分别是

+ 源码学习『OKHTTP——概括』

+ 源码学习『OKHTTP——概括』

+ 源码学习『OKHTTP——概括』

+ 源码学习『OKHTTP——概括』

今天先分享一下我所了解到的OKHTTP的主要内容。细节的东西放到其他几篇中详细讨论，今天只说主体流程。

### 简单使用

之前也写过一篇OKHTTP使用的博客，感兴趣的朋友可以看一下：[OKHttp3『使用』](<https://www.jianshu.com/p/81241f792e89>)，这里还是再简单拎一下代码

````java
//Post提交表单
OkHttpClient okHttpClient = new OkHttpClient();
//创建一个Request
RequestBody requestBody = new FormBody.Builder()
  .add("key","value")
  .build();
Request request = new Request.Builder()
  .url("https://api.github.com/users/JakeWharton")
  .post(requestBody)
  .build();
//enqueue表示执行一个异步任务，如果是同步任务使用execute()
okHttpClient.newCall(request).enqueue(new Callback() {
  @Override
  public void onFailure(Call call, IOException e) {
    Log.d(TAG,e.getMessage());
  }
  @Override
  public void onResponse(Call call, Response response) throws IOException {
    Log.d(TAG,response.body().string());
  }
});
````

### 我们发送一个简单请求的过程：

1. 构造一个OkHttpClient对象，注意每个OkHttpClient都会有一个线程池和连接池，所以想要做到线程池和连接池的复用，最好使用一个全局的OkHttpClient单例对象。
2. 构造一个Request，这里使用了Builder设计模式。
3. 使用OkHttpClient的newCall方法构造一个Call接口对象，这个Call接口对象的具体实现在RealCall类中。
4. 调用Call接口的enqueue(Callback)或execute()执行异步或同步请求。



### 任务调度





### 拦截器流程







### 