---
title: Retrofit『源码学习』
date: 2019-10-28 10:15:24
tags: [源码学习,Retrofit]
---

Retrofit这个开源库代码量并不多，但是别人都说这个库很值得学习，主要是因为其中用了一些设计模式，设计的比较巧妙，值得学习揣摩。闲话少叙，书归正传，管理带着问题看源码，先看下面几个问题：

### 几个问题：

1. 只是声明定了一个接口对象，怎么实现的最后具体的解析成请求方法的？

2. 如果结合RxJava2，那么返回对象是怎么转换成RxJava2的Observable的？

3. 这里几个Factory(CallFactory、Converter.Factory、CallAdapter.Factory）分别什么用处？动态代理模式呢？

   答：

   CallFactory默认就是okhttp3.Call.Factory，是OkHttp3的Call接口的内部接口，包含一个newCall(Request）方法用于返回一个Call对象。可以用于指定自己的网络请求框架。

   Converter.Factory用于指定HTTP响应和请求内容的转换，比如服务器返回一个json字符串，需要转成JSON对象，可以通过..addConverterFactory(**GsonConverterFactory**.create())来实现，其中**GsonConverterFactory**

4. 使用Retrofit具体有什么好处？

   一是将接口定义和使用分离开来，但这实际上也可以通过自己的方法实现。我觉得可能更大的优势在于，**方便切换网络框架、自定义数据解析规则(可能你使用了一套自己的加密解密算法)、以及和RxJava的集成使用**。

5. Retrofit只能使用OKHttp作为网络请求框架吗？

   答：是的，目前代码中是写死的关联的是okhttp3.Call.Factory来创建对应的Call对象。但是可以自定义Call的Factory用于提供自定义的Call实现。

   ```java
   //Retrofit.Builder.callFactory()
   public Builder callFactory(okhttp3.Call.Factory factory) {
         this.callFactory = checkNotNull(factory, "factory == null");
         return this;
   }
   ```

6. 为什么要代理okhttp3.call，而不是直接使用？

7. 既然还是不能切换线程的话，为什么要关联主线程Looper？

   Retrofit2中也可以执行异步任务，也是enqueue(Callback<T/>)，实际上也是代理了一下OKHttp3中的Call的enqueue方法。回调需要在主线程中执行。

<!--more-->

### 动态代理模式

使用动态代理可以实现

在开始之前需要先了解一下动态代理的实现机制，首先这里有几个类：一个InvocationHandler接口、一个Proxy类、



### 使用Retrofit管理一个请求的过程

1. 定义一个包含请求方法的接口类，假设是MyInterface。

2. 调用Retrofit.create(MyInterface)方法，通过动态代理模式创建一个MyInterface的实现对象

   ```java
   /**
      * Create an implementation of the API endpoints defined by the {@code service} interface.
      * <p>
      * The relative path for a given method is obtained from an annotation on the method describing
      * the request type. The built-in methods are {@link retrofit2.http.GET GET},
      * {@link retrofit2.http.PUT PUT}, {@link retrofit2.http.POST POST}, {@link retrofit2.http.PATCH
      * PATCH}, {@link retrofit2.http.HEAD HEAD}, {@link retrofit2.http.DELETE DELETE} and
      * {@link retrofit2.http.OPTIONS OPTIONS}. You can use a custom HTTP method with
      * {@link HTTP @HTTP}. For a dynamic URL, omit the path on the annotation and annotate the first
      * parameter with {@link Url @Url}.
      * <p>
      * Method parameters can be used to replace parts of the URL by annotating them with
      * {@link retrofit2.http.Path @Path}. Replacement sections are denoted by an identifier
      * surrounded by curly braces (e.g., "{foo}"). To add items to the query string of a URL use
      * {@link retrofit2.http.Query @Query}.
      * <p>
      * The body of a request is denoted by the {@link retrofit2.http.Body @Body} annotation. The
      * object will be converted to request representation by one of the {@link Converter.Factory}
      * instances. A {@link RequestBody} can also be used for a raw representation.
      * <p>
      * Alternative request body formats are supported by method annotations and corresponding
      * parameter annotations:
      * <ul>
      * <li>{@link retrofit2.http.FormUrlEncoded @FormUrlEncoded} - Form-encoded data with key-value
      * pairs specified by the {@link retrofit2.http.Field @Field} parameter annotation.
      * <li>{@link retrofit2.http.Multipart @Multipart} - RFC 2388-compliant multipart data with
      * parts specified by the {@link retrofit2.http.Part @Part} parameter annotation.
      * </ul>
      * <p>
      * Additional static headers can be added for an endpoint using the
      * {@link retrofit2.http.Headers @Headers} method annotation. For per-request control over a
      * header annotate a parameter with {@link Header @Header}.
      * <p>
      * By default, methods return a {@link Call} which represents the HTTP request. The generic
      * parameter of the call is the response body type and will be converted by one of the
      * {@link Converter.Factory} instances. {@link ResponseBody} can also be used for a raw
      * representation. {@link Void} can be used if you do not care about the body contents.
      * <p>
      * For example:
      * <pre>
      * public interface CategoryService {
      *   &#64;POST("category/{cat}/")
      *   Call&lt;List&lt;Item&gt;&gt; categoryList(@Path("cat") String a, @Query("page") int b);
      * }
      * </pre>
      */
     @SuppressWarnings("unchecked") // Single-interface proxy creation guarded by parameter safety.
     public <T> T create(final Class<T> service) {
       Utils.validateServiceInterface(service);
       if (validateEagerly) {
         eagerlyValidateMethods(service);
       }
       return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
           new InvocationHandler() {
             private final Platform platform = Platform.get();
   
             @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
                 throws Throwable {
               // If the method is a method from Object then defer to normal invocation.
               if (method.getDeclaringClass() == Object.class) {
                 return method.invoke(this, args);
               }
               if (platform.isDefaultMethod(method)) {
                 return platform.invokeDefaultMethod(method, service, proxy, args);
               }
               ServiceMethod<Object, Object> serviceMethod =
                   (ServiceMethod<Object, Object>) loadServiceMethod(method);
               OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
               return serviceMethod.callAdapter.adapt(okHttpCall);
             }
           });
     }
   ```

   

2. 使用Builder模式构造一个Retrofit实例对象。期间可以指定Converter.Factory、CallAdapter.Factory，如：

   ```java
   .addConverterFactory(GsonConverterFactory.create())
   //添加RxJava语言支持
   .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
   ```

3. 通过Retrofit代理一个请求的发送：

   ```java
   RetrofitManager.getInstance().getRetrofit()
       .create(ApiService.class)
     	//具体的请求方法，定义在接口类中
       .queryBookList()
       //指定上游发送事件线程
       .subscribeOn(Schedulers.computation())
       //指定下游接收事件线程
       .observeOn(AndroidSchedulers.mainThread())
       .map(new ArrayFunc<Book>());
   ```

4. 思考一个问题，为什么上面的.create(ApiService.class).queryBookList()就能发送一个请求呢？

   首先.create(ApiService.class)中使用动态代理的方式创建了一个OkHttpCall<T\>的对象，看源码：

   ```java
    public <T> T create(final Class<T> service) {
       Utils.validateServiceInterface(service);
       if (validateEagerly) {
         eagerlyValidateMethods(service);
       }
       return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
           new InvocationHandler() {
             private final Platform platform = Platform.get();
   
             @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
                 throws Throwable {
               // If the method is a method from Object then defer to normal invocation.
               if (method.getDeclaringClass() == Object.class) {
                 return method.invoke(this, args);
               }
               if (platform.isDefaultMethod(method)) {
                 return platform.invokeDefaultMethod(method, service, proxy, args);
               }
               ServiceMethod<Object, Object> serviceMethod =
                   (ServiceMethod<Object, Object>) loadServiceMethod(method);
               OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
               return serviceMethod.callAdapter.adapt(okHttpCall);
             }
           });
     }
   ```





### 几个关键类

#### CallFactory

#### Converter.Factory

#### CallAdapter.Factory

```java
import java.lang.annotation.Annotation;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import javax.annotation.Nullable;

/**
 * 将一个类型为R的Call转换成目标类型T。
 */
public interface CallAdapter<R, T> {
  /**
   * 用来表示将HTTP响应体转成成Java对象时的类型，比如Call<Repo>对应的类型是Repo。
   */
  Type responseType();

  /**
   * 返回完成Call委托工作的目标类型T的实例。在Retrofit结合RxJava2时，就是返回一个Observable<Response<R>>。
   */
  T adapt(Call<R> call);

  /**
   * Creates {@link CallAdapter} instances based on the return type of {@linkplain
   * Retrofit#create(Class) the service interface} methods.
   */
  abstract class Factory {
    /**
     * Returns a call adapter for interface methods that return {@code returnType}, or null if it
     * cannot be handled by this factory.
     */
    public abstract @Nullable CallAdapter<?, ?> get(Type returnType, Annotation[] annotations,
        Retrofit retrofit);

    /**
     * Extract the upper bound of the generic parameter at {@code index} from {@code type}. For
     * example, index 1 of {@code Map<String, ? extends Runnable>} returns {@code Runnable}.
     */
    protected static Type getParameterUpperBound(int index, ParameterizedType type) {
      return Utils.getParameterUpperBound(index, type);
    }

    /**
     * 从type中提取原始类型，比如：如果type=List<? extends Runnable>,那么该方法将返回List.class。
     */
    protected static Class<?> getRawType(Type type) {
      return Utils.getRawType(type);
    }
  }
}
```

#### ServiceMethod

这个类可以说是Retrofit实现从简单的定义一个接口到具体实现调用Okhttp3进行网络请求的关键实现类了。可以自己了解一下。

#### Platform

```java
  static class Android extends Platform {
    @Override public Executor defaultCallbackExecutor() {
      //返回和持有主线程Handler的Executor
      return new MainThreadExecutor();
    }

    @Override CallAdapter.Factory defaultCallAdapterFactory(@Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    static class MainThreadExecutor implements Executor {
      //关联主线程Handler
      private final Handler handler = new Handler(Looper.getMainLooper());
		
      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
  }
```

### 用到的设计模式

####建造者模式

#### 动态代理模式

#### 适配器模式

#### 观察者模式

####门面模式

####策略模式



####

### 结语

简单概述Retrofit的实现原理：

定义一个包含请求方法的接口类，使用反射从请求方法中得到请求相关的参数、HTTP方法、返回类型等信息保存到一个<font color="#dd0000">**ServiceMethod**</font>中，再将ServiceMethod保存到一个ConcurrentHashMap<<Method,ServiceMethod<?,?\>>中，通过loadServiceMethod(Method)尝试从这个缓存中加载ServiceMethod信息。

使用得到的serviceMethod和请求的参数args构造一个Retrofit的OkHttpCall，注意这是Retrofit中的OkHttpCall，在其中会**代理**真正的okhttp3.Call的request、enqueue、execute等方法。**不管是enqueue还是execute方法中最后都会调用ServiceMethod.toResponse(catchingBody)来调用Retrofit具体关联的ConverterFactory生成的Converter进行内容转换**。