---
title: OKHttp3『使用』
date: 2019-04-28 10:49:52
tags:
---


#### 一、介绍

OKHttp是一个高效的开源网络请求类库，具有以下特性：

+ 支持HTTP/2，**允许所有同一个主机地址的请求共享同一个socket连接**
+ 使用连接池，提高请求效率
+ 透明的GZIp压缩减少响应数据的大小
+ 缓存响应数据，避免一些完全重复的请求
+ 当网络出现状态时，OKHTTP会进行尝试，如果服务有多个地址，OKHTTP会进行不同地址的请求尝试；**注意，这里的重试指的是网络连接失败时的重试，而读取超时，连接超时不在范围内！**

#### 二、配置环境

##### 网络权限

```java
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
```

##### 类库依赖

```java
api 'com.squareup.okhttp3:okhttp:3.10.0'
api 'com.squareup.okio:okio:1.14.0'
```

#### 三、基础使用

```java
//创建一个OKHttpClient对象
OkHttpClient okHttpClient = new OkHttpClient();
//创建一个Request
Request request = new Request.Builder()
  .url("https://api.github.com/users/jitmaos")
  .addHeader("key","value")
  .get()
  .build();
//使用request，同步请求会返回一个Response，异步请求会回调成功、失败函数
Response response = okHttpClient.newCall(request).execute();
Log.d(TAG,response.body().string());
```

#### 三、常用请求

```java
//Get请求
OkHttpClient okHttpClient = new OkHttpClient();
//创建一个Request
Request request = new Request.Builder()
  .url("https://api.github.com/users/JakeWharton")
  .get()
  .build();
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
//Post提交文件
MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
OkHttpClient okHttpClient = new OkHttpClient();
File file = new File("test.md");
Request request = new Request.Builder()
  .url("https://api.github.com/markdown/raw")
  .post(RequestBody.create(mediaType, file))
  .build();
okHttpClient.newCall(request).enqueue(new Callback() {
  @Override
  public void onFailure(Call call, IOException e) {
    Log.d(TAG, "onFailure: " + e.getMessage());
  }
  @Override
  public void onResponse(Call call, Response response) throws IOException {
    Log.d(TAG, response.protocol() + " " +response.code() + " " + response.message());
    Headers headers = response.headers();
    for (int i = 0; i < headers.size(); i++) {
      Log.d(TAG, headers.name(i) + ":" + headers.value(i));
    }
    Log.d(TAG, "onResponse: " + response.body().string());
  }
});
```

### 四、拦截器

拦截器分为应用拦截器和网络拦截器

##### 应用拦截器

+ 不用关心中间过程的响应，如重定向和重试
+ 总是只调用一次，即使HTTP响应式从缓存中获取的
+ 观察应用程序的初衷，不关心OKHTTP注入的头信息，如If-None-Match
+ 允许短路，而且不用调用Chain.proceed()
+ 允许重试，使得Chain.proceed()可以调用多次

##### 网路拦截器

+ 能够操作中间过程的响应，如重定向和重试
+ 当网络短路不会调用缓存响应
+ 只观察网络上传输的数据
+ 访问包含请求信息的连接

##### 自定义拦截器

```java
public class LoggingInterceptor implements Interceptor {
  private static final String TAG = "LoggingInterceptor";
  @Override
  public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    long startTime = System.nanoTime();
    Log.d(TAG, String.format("Sending request %s on %s%n%s",
                             request.url(), chain.connection(), request.headers()));
    Response response =  chain.proceed(request);
    long endTime = System.nanoTime();
    Log.d(TAG, String.format("Received response for %s in %.1fms%n%s",
       response.request().url(), (endTime - startTime) / 1e6d, response.headers()));
    return response;
  }
}
```

### 其他

##### 缓存响应

本段参考：[OkHttp3-使用进阶](<https://www.jianshu.com/p/25e89116847c>)

要缓存响应数据，首先需要指定一个可以**读写并确定大小的缓存目录**，OKHttp会根据响应头的配置信息对数据进行缓存。服务器会在返回的响应数据中配置这个数据缓存多久的信息，如：Cache-Control:max-age=10000。我们也可以在请求头中添加自定义的缓存有效时间信息，如：Cache-control:max-stale=3600。**可以通过Cache Control.FORCE_NETWORK指定不使用缓存直接从网络读取数据，也可以使用CacheControl.FORCE_CACHE指定不使用网络，直接从缓存读取数据。**

```java
private final OkHttpClient client;
public CacheResponse(File cacheDirectory) throws Exception {
  int cacheSize = 10 * 1024 * 1024; // 10 MiB
  Cache cache = new Cache(cacheDirectory, cacheSize);

  client = new OkHttpClient.Builder()
    .cache(cache)
    .build();
}

public void run() throws Exception {
  Request request = new Request.Builder()
    .url("http://publicobject.com/helloworld.txt")
    .build();

  Response response1 = client.newCall(request).execute();
  if (!response1.isSuccessful()) throw new IOException("Unexpected code " + response1);

  String response1Body = response1.body().string();
  System.out.println("Response 1 response:          " + response1);
  System.out.println("Response 1 cache response:    " + response1.cacheResponse());
  System.out.println("Response 1 network response:  " + response1.networkResponse());

  Response response2 = client.newCall(request).execute();
  if (!response2.isSuccessful()) throw new IOException("Unexpected code " + response2);

  String response2Body = response2.body().string();
  System.out.println("Response 2 response:          " + response2);
  System.out.println("Response 2 cache response:    " + response2.cacheResponse());
  System.out.println("Response 2 network response:  " + response2.networkResponse());
  System.out.println("Response 2 equals Response 1? " + 					   response1Body.equals(response2Body));
}
```

##### 取消一个请求

在Android中适时的取消一个请求是有必要的，比如当一个请求还在返回中，关闭了按钮，此时就应该及时的取消这个页面所有未完成的请求动作。

 服务器指定一个服务延时5秒执行

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
  throws ServletException, IOException {
  System.out.println(System.currentTimeMillis() + "-收到请求");
  try {
    Thread.sleep(5000);
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  new BaseCURDOperator<Book>().search(this,req,resp,null,"id desc",null, Book.class,true);
  System.out.println(System.currentTimeMillis() + "-完成处理");
}
```

Android端测试取消请求

```java
val executor = Executors.newScheduledThreadPool(1)
val client = OkHttpClient()
  object:Thread(){
    override fun run() {
      super.run()
        val request = Request.Builder()
        .url("http://192.168.1.6:8080/TP_S/BookList") // 服务器延时了5秒返回数据
        .build()
        val call = client.newCall(request)
        // 1秒后取消请求
        executor.schedule(Runnable { call.cancel() }, 1, TimeUnit.SECONDS)

        try {
          val response = call.execute()
        } catch (e: IOException) {
          //取消请求后，再尝试去拿response，报Socket closed，说明请求被取消了
          e.printStackTrace()
        }
    }
  }.start()
```

参考：[OkHttp3最佳入门使用](<https://blog.csdn.net/dingshuhong_/article/details/80335499>)