---
title: OkHttp『源码学习-杂项』
tags: 
---

OKHttp是个优秀的开源项目，值得学习。但是代码量比EventBus、Retrofit之类的大多了。想要一下子没有遗漏的都搞清楚不是一件容易的事情。这篇博客，是我在学习过程中的一些问题，有些自觉找到了答案，有些还不知道如何回答，留待以后补充吧。

## 思考问题：

1. 异步，同步队列共用线程池吗？共用请求数量限制(maxRequests和maxRequestsPerHost)吗？

   答：`请求数量限制只对异步队列有效`，对于同步任务不做限制，只管添加执行就得了。

2. 为什么不直接使用线程池，而是自己另外维护了工作队列？

   答：首先这里使用了的线程池是基于同步队列的，而同步队列本身是不支持工作队列的，所以需要一个额外的工作队列来维护任务的提交工作。

3. 为什么OkHttp的异步任务选择使用两个队列维护？而不是使用一个工作队列实现？

   答：说下我自己的看法，首先为什么需要两个队列，因为在提交异步任务是需要统计当前的运行中的任务数量，使用一个队列的话，还需要遍历队列中的任务状态，而将等待运行和运行中分离开来可以避免这种困扰，减少问题发生的可能性。

4. 等待队列中的任务何时触发运行？

   答：不管是异步任务还是同步任务，任务的执行代码都包含在一个try…catch代码块中，该代码块的finally中执行dispatcher.finished(this)，在finished()方法中首先会从异步运行中队列中移除代表当前任务的Call对象，然后会去异步等待就绪队列中拿任务放到异步运行中队列中，并执行任务。看源码：

   ```java
   //Dispatcher.finished()
   private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
       int runningCallsCount;
       Runnable idleCallback;
       synchronized (this) {
         //首先从运行队列中移除当前已完成的Call对象
         if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
         //这个promoteCalls()方法中，会从异步就绪等待队列中拿取任务放到异步运行队列并执行
         if (promoteCalls) promoteCalls(); 
         runningCallsCount = runningCallsCount();
         idleCallback = this.idleCallback;
       }
   
       if (runningCallsCount == 0 && idleCallback != null) {
         //如果没有任务在运行时，触发回调，暂时不知道使用场景
         idleCallback.run();
       }
     }
   ```

   再看一下调度等待就绪任务的promoteCalls()方法：

   ```java
   //Dispatcher.promoteCalls()
   private void promoteCalls() {
       if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
       if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.
   
       for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
         AsyncCall call = i.next();
   
         if (runningCallsForHost(call) < maxRequestsPerHost) {
           //从当前等待就绪队列中移除当前Call对象
           i.remove();
           //添加Call到运行队列
           runningAsyncCalls.add(call);
           //执行任务
           executorService().execute(call);
         }
   
         if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
       }
     }
   ```

5. 如果okhttpclient不作为单例使用，那么线程池是否就有多个了，Dispatcher呢？

   **答**：每个OkHttpClient对象实例对应一个Dispatcher，因此也就意味着多个OkHttpClient对应着多个线程池。

6. RetryAndFollowUpInterceptor中的重试指的是什么？是重试连接还是跳转？还是包含了这两个动作：重连次数和跳转数限制？

   **答**：

7. 缓存需要手动指定才能生效吗？缓存机制是什么？

8. DiskLruCache算法的具体细节是什么？缓存清理线程的工作原理是什么？

9. 为什么要把AsyncCall定义成RealCall的内部类？有什么好处吗？

10. 为什么缓存管理需要另外一个日志文件？

11. 为什么DiskLruCache的内部类Entry需要一个cleanFiles文件数组和一个dirtyFiles文件数组？

<!-- more -->

## 我们发送一个简单请求的过程：

1. 构造一个OkHttpClient对象，注意每个OkHttpClient都会有一个线程池和连接池，所以想要做到线程池和连接池的复用，最好使用一个全局的OkHttpClient单例对象。
2. 构造一个Request，这里使用了Builder设计模式。
3. 使用OkHttpClient的newCall方法构造一个Call接口对象，这个Call接口对象的具体实现在RealCall类中。
4. 调用Call接口的enqueue(Callback)或execute()执行异步或同步请求。

## 流程分析

总得来说，一个请求的发送响应流程分为几个主要步骤：

+ 任务调度
+ 拦截器处理(责任链机制)
+ 缓存机制
+ 连接复用。

### 任务调度

1. 在构造OkHttpClient对象的Builder时，会在Builder的构造方法中创建一个Dispatcher()，Dispatcher在OkHttp中扮演了一个管理调度器的角色。

2. **Dispatcher中包含三个任务队列，分别是：readyAsyncCalls(异步就绪等待队列)、runningAsyncCalls(运行中的异步任务队列)和runningSyncCalls(运行中的同步任务队列)。**同时，Dispatcher中还包含了一个线程池，<font color="#dd0000">**这是一个基于同步队列(SynchronousQueue)的线程池**</font>，Dispatcher在提交任务到线程池时会对当前线程执行情况做校验：

   ```java
   synchronized void enqueue(AsyncCall call) {
       if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
         runningAsyncCalls.add(call);
         executorService().execute(call);
       } else {
         readyAsyncCalls.add(call);
       }
     }
   ```

   **注意，maxRequests和maxRequestPerHost只对异步任务做限制。**

3. <font color="#dd0000">**不管是同步任务还是异步任务，最后都会执行execute()**</font>，这段代码被包含在一个catch块中，该catch块后面的finally中都执行了Dispatcher.finished(Call)方法

   ```java
   @Override public Response execute() throws IOException {
       synchronized (this) {
         if (executed) throw new IllegalStateException("Already Executed");
         executed = true;
       }
       captureCallStackTrace();
       eventListener.callStart(this);
       try {
         client.dispatcher().executed(this);
         Response result = getResponseWithInterceptorChain();
         if (result == null) throw new IOException("Canceled");
         return result;
       } catch (IOException e) {
         eventListener.callFailed(this, e);
         throw e;
       } finally {
         client.dispatcher().finished(this);
       }
     }
   ```

4. 在finished方法中会根据isPromoteCall来决定是否会触发promoteCalls方法(Dispatcher有多个重载的finished()方法，**同步任务不触发promoteCalls，异步方法触发promoteCalls方法**)：

   ```java
     private void promoteCalls() {
       if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
       if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.
   
       for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
         AsyncCall call = i.next();
   
         if (runningCallsForHost(call) < maxRequestsPerHost) {
           i.remove();
           runningAsyncCalls.add(call);
           executorService().execute(call);
         }
         if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
       }
     }
   ```

   从代码中可以看出promoteCalls方法是用于**从就绪等待队列中拿去任务到异步运行队列的**。

### Okhttp3中的拦截器责任链

**责任链模式**可以查看『补充』中的查看，在OkHttp3中实现了对请求和响应的层层拦截处理，每个拦截器按照自己的职责添加处理方法。

Okhttp中有这几个内置的拦截器，看一张别人的流程图

![avatar](https://upload-images.jianshu.io/upload_images/5713484-35a9989e893171f1.png)

#### Interceptor接口

```java
/**
 * Observes, modifies, and potentially short-circuits requests going out and the corresponding
 * responses coming back in. Typically interceptors add, remove, or transform headers on the request
 * or response.
 */
public interface Interceptor {
  Response intercept(Chain chain) throws IOException;

  interface Chain {
    Request request();

    Response proceed(Request request) throws IOException;

    /**
     * 返回用于当前Request执行的Connection对象。这个方法只在网络层拦截器中有用到，对于应用程序的拦截器，总是返回null。
     */
    @Nullable Connection connection();

    Call call();

    int connectTimeoutMillis();

    Chain withConnectTimeout(int timeout, TimeUnit unit);

    int readTimeoutMillis();

    Chain withReadTimeout(int timeout, TimeUnit unit);

    int writeTimeoutMillis();

    Chain withWriteTimeout(int timeout, TimeUnit unit);
  }
}
```

#### RetryAndFollowUpInterceptor

<font color="#dd0000">**这个拦截器用于实现失败重试以及追踪重定向**</font>(默认最多20次重定向)。如果Call被取消可能会抛出IOException。

整个Call相关的StreamAllocation在RetryAndFollowUpInterceptor的intercept方法中创建。先看一下intercept方法：

```java
@Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Call call = realChain.call();
    EventListener eventListener = realChain.eventListener();

    streamAllocation = new StreamAllocation(client.connectionPool(), createAddress(request.url()),
        call, eventListener, callStackTrace);

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      if (canceled) {
        streamAllocation.release();
        throw new IOException("Canceled");
      }

      Response response;
      boolean releaseConnection = true;
      try {
        response = realChain.proceed(request, streamAllocation, null, null);
        releaseConnection = false;
      } catch (RouteException e) {
        // 路由异常后的重试，要求request还未发送出去
        if (!recover(e.getLastConnectException(), false, request)) {
          throw e.getLastConnectException();
        }
        releaseConnection = false;
        continue;
      } catch (IOException e) {
        // 连接到服务器的异常重试，要求request还未发送出去。
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, requestSendStarted, request)) throw e;
        releaseConnection = false;
        continue;
      } finally {
        // We're throwing an unchecked exception. Release any resources.
        if (releaseConnection) {
          streamAllocation.streamFailed(null);
          streamAllocation.release();
        }
      }

      // Attach the prior response if it exists. Such responses never have a body.
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }

      Request followUp = followUpRequest(response);

      if (followUp == null) {
        if (!forWebSocket) {
          streamAllocation.release();
        }
        return response;
      }

      closeQuietly(response.body());

      if (++followUpCount > MAX_FOLLOW_UPS) {
        streamAllocation.release();
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }

      if (followUp.body() instanceof UnrepeatableRequestBody) {
        streamAllocation.release();
        throw new HttpRetryException("Cannot retry streamed HTTP body", response.code());
      }

      if (!sameConnection(response, followUp.url())) {
        streamAllocation.release();
        streamAllocation = new StreamAllocation(client.connectionPool(),
            createAddress(followUp.url()), call, eventListener, callStackTrace);
      } else if (streamAllocation.codec() != null) {
        throw new IllegalStateException("Closing the body of " + response
            + " didn't close its backing stream. Bad interceptor?");
      }

      request = followUp;
      priorResponse = response;
    }
  }
```

看一下其中的followUpRequest(Response)方法，可以看到这里主要是根据Http请求响应的状态码来决定是否有重定向，如果有就使用

```java
/**
   * 根据响应的状态码，判断是否需要进行重定向跟踪还是处理超时。如果不需要或不能重定向，则返回Null.
   */
  private Request followUpRequest(Response userResponse) throws IOException {
    if (userResponse == null) throw new IllegalStateException();
    Connection connection = streamAllocation.connection();
    Route route = connection != null
        ? connection.route()
        : null;
    int responseCode = userResponse.code();

    final String method = userResponse.request().method();
    switch (responseCode) {
      case HTTP_PROXY_AUTH:
        Proxy selectedProxy = route != null
            ? route.proxy()
            : client.proxy();
        if (selectedProxy.type() != Proxy.Type.HTTP) {
          throw new ProtocolException("Received HTTP_PROXY_AUTH (407) code while not using proxy");
        }
        return client.proxyAuthenticator().authenticate(route, userResponse);

      case HTTP_UNAUTHORIZED:
        return client.authenticator().authenticate(route, userResponse);

      case HTTP_PERM_REDIRECT:
      case HTTP_TEMP_REDIRECT:
        // "If the 307 or 308 status code is received in response to a request other than GET
        // or HEAD, the user agent MUST NOT automatically redirect the request"
        if (!method.equals("GET") && !method.equals("HEAD")) {
          return null;
        }
        // fall-through
      case HTTP_MULT_CHOICE:
      case HTTP_MOVED_PERM:
      case HTTP_MOVED_TEMP:
      case HTTP_SEE_OTHER:
        // Does the client allow redirects?
        if (!client.followRedirects()) return null;

        String location = userResponse.header("Location");
        if (location == null) return null;
        HttpUrl url = userResponse.request().url().resolve(location);

        // Don't follow redirects to unsupported protocols.
        if (url == null) return null;

        // If configured, don't follow redirects between SSL and non-SSL.
        boolean sameScheme = url.scheme().equals(userResponse.request().url().scheme());
        if (!sameScheme && !client.followSslRedirects()) return null;

        // Most redirects don't include a request body.
        Request.Builder requestBuilder = userResponse.request().newBuilder();
        if (HttpMethod.permitsRequestBody(method)) {
          final boolean maintainBody = HttpMethod.redirectsWithBody(method);
          if (HttpMethod.redirectsToGet(method)) {
            requestBuilder.method("GET", null);
          } else {
            RequestBody requestBody = maintainBody ? userResponse.request().body() : null;
            requestBuilder.method(method, requestBody);
          }
          if (!maintainBody) {
            requestBuilder.removeHeader("Transfer-Encoding");
            requestBuilder.removeHeader("Content-Length");
            requestBuilder.removeHeader("Content-Type");
          }
        }

        // When redirecting across hosts, drop all authentication headers. This
        // is potentially annoying to the application layer since they have no
        // way to retain them.
        if (!sameConnection(userResponse, url)) {
          requestBuilder.removeHeader("Authorization");
        }

        return requestBuilder.url(url).build();

      case HTTP_CLIENT_TIMEOUT:
        // 408's are rare in practice, but some servers like HAProxy use this response code. The
        // spec says that we may repeat the request without modifications. Modern browsers also
        // repeat the request (even non-idempotent ones.)
        if (!client.retryOnConnectionFailure()) {
          // The application layer has directed us not to retry the request.
          return null;
        }

        if (userResponse.request().body() instanceof UnrepeatableRequestBody) {
          return null;
        }

        if (userResponse.priorResponse() != null
            && userResponse.priorResponse().code() == HTTP_CLIENT_TIMEOUT) {
          // We attempted to retry and got another timeout. Give up.
          return null;
        }

        return userResponse.request();

      default:
        return null;
    }
  }
```

关于失败重试部分，需要注意的是**OKHttp里默认的重试机制可只捕获RouteException和IOException，并且重试还有更多的限制要求，比如不能重复发送未buffered的request、带body的request需要在发送之前才能重试等**。具体看recover方法：

```java
/**
   * 试着从一个失败中恢复与服务器的通信。如果Exception是可恢复的那么返回true，否则返回false。带有请求体body的request只有在body被缓存(buffered)或者失败发生于request被发送出去之前才能尝试恢复。
   */
  private boolean recover(IOException e, boolean requestSendStarted, Request userRequest) {
    streamAllocation.streamFailed(e);

    // 如果设置了不需要重试，直接返回false
    if (!client.retryOnConnectionFailure()) return false;

    // 如果网络请求已经开始，并且body内容只可以发送一次
    if (requestSendStarted && userRequest.body() instanceof UnrepeatableRequestBody) return false;

    // 不会重试的类型：协议异常、Socketet异常并且网络情况还没开始，ssl认证异常
    if (!isRecoverable(e, requestSendStarted)) return false;

    // 没有更多的路由可供尝试
    if (!streamAllocation.hasMoreRoutes()) return false;

    // 对于失败重试，使用在一个新的connection上使用同一个路由选择器Route Selector.
    return true;
  }
```

看一下路由相关的几个类Route、RouteSelector和RouteDatabase

#### Route

首先明白一点，IP和DNS之间并没有直接关系，IP是一个网络传输协议，DNS是域名解析服务。**一个域名使用DNS可能会解析到多台服务器IP，如果服务器做了负载冗余，那么当其中一台服务器挂了以后，可以无痕切换到别的服务器。**<font color="#dd0000">**一个域名对应多个IP是OKHttp重试及路由的基础**</font>。

```java
/**
 * 路由实体类用来和一个抽象的原始服务器建立连接。当创建一个连接时，客户端可能有多个选择：
 	 HTTP Proxy：一个代理服务器可以多种形式：
 	  HTTP代理：一个客户端可能指定了一个代理服务器。否则将使用ProxySelector，ProxySelector可能会返回多个需要重试的代理。
 	  IP地址：不管是通过IP直连到一个代理服务器还是原始服务器，每打开一个Sokcet连接就需要一个IP地址。而DNS服务器可能会返回多个IP地址。
 */
public final class Route {
  final Address address;
  final Proxy proxy;
  final InetSocketAddress inetSocketAddress;

  public Route(Address address, Proxy proxy, InetSocketAddress inetSocketAddress) {
    if (address == null) {
      throw new NullPointerException("address == null");
    }
    if (proxy == null) {
      throw new NullPointerException("proxy == null");
    }
    if (inetSocketAddress == null) {
      throw new NullPointerException("inetSocketAddress == null");
    }
    this.address = address;
    this.proxy = proxy;
    this.inetSocketAddress = inetSocketAddress;
  }

  public Address address() {
    return address;
  }

  /**
   * 返回当前路由的代理。Returns the {@link Proxy} of this route.
   *
   * <strong>Warning:</strong> This may disagree with {@link Address#proxy} when it is null. When
   * the address's proxy is null, the proxy selector is used.
   */
  public Proxy proxy() {
    return proxy;
  }

  public InetSocketAddress socketAddress() {
    return inetSocketAddress;
  }

  /**
   * Returns true if this route tunnels HTTPS through an HTTP proxy. See <a
   * href="http://www.ietf.org/rfc/rfc2817.txt">RFC 2817, Section 5.2</a>.
   */
  public boolean requiresTunnel() {
    return address.sslSocketFactory != null && proxy.type() == Proxy.Type.HTTP;
  }

  @Override public boolean equals(@Nullable Object other) {
    return other instanceof Route
        && ((Route) other).address.equals(address)
        && ((Route) other).proxy.equals(proxy)
        && ((Route) other).inetSocketAddress.equals(inetSocketAddress);
  }

  @Override public int hashCode() {
    int result = 17;
    result = 31 * result + address.hashCode();
    result = 31 * result + proxy.hashCode();
    result = 31 * result + inetSocketAddress.hashCode();
    return result;
  }

  @Override public String toString() {
    return "Route{" + inetSocketAddress + "}";
  }
}

```

Route是描述网络数据包传输的路径，最主要还是描述直接与其建立TCP连接的目标端点。对于设置了HTTP代理，且安全的连接 (SSL) 需要请求代理服务器建立一个到目标HTTP服务器的隧道连接，客户端与HTTP代理建立TCP连接，以此请求HTTP代理服务在客户端与HTTP服务器之间进行数据的盲转发。

#### RouteSelector

HTTP请求处理过程中所需的TCP连接建立过程，主要是找到一个Route，然后依据代理协议规则与特定目标建立TCP连接。对于无代理的情况，是与HTTP服务器建立TCP连接，对于SOCKS代理和http代理，是与代理服务器建立tcp连接，虽然都是与代理服务器建立tcp连接，但是SOCKS代理协议和http代理协议又有一定的区别。
 而且借助于域名做负均衡已经是网络中非常常见的手法了，因而，常常会有域名对应不同IP地址的情况。同时相同系统也可以设置多个代理，这使Route的选择变得非常复杂。
 在OKHTTP中，对Route连接有一定的错误处理机制。OKHTTP会逐个尝试找到Route建立TCP连接，直到找到可用的哪一个。这样对Route信息有良好的管理。OKHTTP中借助RouteSelector类管理所有路由信息，并帮助选择路由。

```java
private final Address address;
  private final RouteDatabase routeDatabase;

  /* The most recently attempted route. */
  private Proxy lastProxy;
  private InetSocketAddress lastInetSocketAddress;

  /* State for negotiating the next proxy to use. */
  private List<Proxy> proxies = Collections.emptyList();
  private int nextProxyIndex;

  /* State for negotiating the next socket address to use. */
  private List<InetSocketAddress> inetSocketAddresses = Collections.emptyList();
  private int nextInetSocketAddressIndex;

  /* State for negotiating failed routes */
  private final List<Route> postponedRoutes = new ArrayList<>();

  public RouteSelector(Address address, RouteDatabase routeDatabase) {
    this.address = address;
    this.routeDatabase = routeDatabase;

    resetNextProxy(address.url(), address.proxy());
  }

  /** Prepares the proxy servers to try. */
  private void resetNextProxy(HttpUrl url, Proxy proxy) {
    if (proxy != null) {
     //第一种方式
      // If the user specifies a proxy, try that and only that.
      proxies = Collections.singletonList(proxy);
    } else {
     //第二种方式
      // Try each of the ProxySelector choices until one connection succeeds.
      List<Proxy> proxiesOrNull = address.proxySelector().select(url.uri());
      proxies = proxiesOrNull != null && !proxiesOrNull.isEmpty()
          ? Util.immutableList(proxiesOrNull)
          : Util.immutableList(Proxy.NO_PROXY);
    }
    nextProxyIndex = 0;
  }
```

#### RouteDatabase

当在创建与目标地址的链接时，为了避免重复出现路由故障而创建的黑名单，如果尝试链接特定的IP或者代理服务器最后失败了，将记住这些故障。**之后进行IP重试时，会通过shouldPostpone()方法将RouteDatabase中的路由路径延迟执行。**

```java
 private final Set<Route> failedRoutes = new LinkedHashSet<>();

  /** Records a failure connecting to {@code failedRoute}. */
  public synchronized void failed(Route failedRoute) {
    failedRoutes.add(failedRoute);
  }

  /** Records success connecting to {@code failedRoute}. */
  public synchronized void connected(Route route) {
    failedRoutes.remove(route);
  }

  /** Returns true if {@code route} has failed recently and should be avoided. */
  public synchronized boolean shouldPostpone(Route route) {
    return failedRoutes.contains(route);
  }
```

#### BridgeInterceptor

作为用户代码和网络请求的代码的桥梁。首先它会基于用户的request创建一个网络request。然后它会处理网络调用。最后它会将网络response构成用户需要的Reponse。简单来说，就是<font color="#dd0000">**处理请求的编码、Cookie、GZip解压缩等问题**</font>。

```java
/**
 * 作为用户代码和网络请求的代码的桥梁。首先它会基于用户的request创建一个网络request。然后它会处理网络调用。最后它会将网络response构成用户需要的Reponse。
 Bridges from application code to network code. First it builds a network request from a user request. Then it proceeds to call the network. Finally it builds a user response from the network response.
 */
public final class BridgeInterceptor implements Interceptor {
  private final CookieJar cookieJar;

  public BridgeInterceptor(CookieJar cookieJar) {
    this.cookieJar = cookieJar;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();

    RequestBody body = userRequest.body();
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }

    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }

    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }

    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }

    List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }

    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }

    Response networkResponse = chain.proceed(requestBuilder.build());

    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);

    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      String contentType = networkResponse.header("Content-Type");
      responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
    }

    return responseBuilder.build();
  }

  /** Returns a 'Cookie' HTTP request header with all cookies, like {@code a=b; c=d}. */
  private String cookieHeader(List<Cookie> cookies) {
    StringBuilder cookieHeader = new StringBuilder();
    for (int i = 0, size = cookies.size(); i < size; i++) {
      if (i > 0) {
        cookieHeader.append("; ");
      }
      Cookie cookie = cookies.get(i);
      cookieHeader.append(cookie.name()).append('=').append(cookie.value());
    }
    return cookieHeader.toString();
  }
}
```

#### CacheInterceptor

原理和注意事项：

1、原理
 (1)、okhttp的网络缓存是基于http协议，不清楚请仔细看上一篇文章
 (2)、使用DiskLruCache的缓存策略，具体请看本片文章的第一章节
 2、注意事项：
 1、目前只支持GET，其他请求方式需要自己实现。
 2、需要服务器配合，通过head设置相关头来控制缓存
 3、创建OkHttpClient时候需要配置Cache

(二)流程：

1、如果配置了缓存，则从缓存中取出(可能为null)
 2、获取缓存的策略.
 3、监测缓存
 4、如果禁止使用网络(比如飞行模式),且缓存无效，直接返回
 5、如果缓存有效，使用网络，不使用网络
 6、如果缓存无效，执行下一个拦截器
 7、本地有缓存、根据条件判断是使用缓存还是使用网络的response
 8、把response缓存到本地

```java
@Override public Response intercept(Chain chain) throws IOException {
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }

    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }

    return response;
  }
```

#### ConnecInterceptor

```java
/** Opens a connection to the target server and proceeds to the next interceptor. */
public final class ConnectInterceptor implements Interceptor {
  public final OkHttpClient client;

  public ConnectInterceptor(OkHttpClient client) {
    this.client = client;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    StreamAllocation streamAllocation = realChain.streamAllocation();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    HttpCodec httpCodec = streamAllocation.newStream(client, doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();

    return realChain.proceed(request, streamAllocation, httpCodec, connection);
  }
}
```

首先通过streamAllocation的newStream方法获取一个流(HttpCodec 是个接口，根据协议的不同，由具体的子类的去实现)，第二步就是获取对应的RealConnection。

StreamAllocation的newStream()内部其实是通过findHealthyConnection()方法获取一个RealConnection，而在findHealthyConnection()里面通过一个while(true)死循环不断去调用findConnection()方法去找RealConnection.而在findConnection()里面其实是真正的寻找RealConnection，而上面提到的findHealthyConnection()里面主要就是调用findConnection()然后去验证是否是"健康"的。

在findConnection()里面主要是通过3重判断：1如果有已知连接且可用，则直接返回，2如果在连接池有对应address的连接，则返回，3切换路由再在连接池里面找下，如果有则返回，如果上述三个条件都没有满足，则直接new一个RealConnection。

然后开始握手，握手结束后，把连接加入连接池，如果在连接池有重复连接，和合并连接。

#### HttpCodec

真正的操作网络请求的流对象的接口，具体实现由Http1Codec、Http2Codec。

```java
import java.io.IOException;
import okhttp3.Request;
import okhttp3.Response;
import okhttp3.ResponseBody;
import okio.Sink;

/** 编码HTTP请求，解密HTTP响应 */
public interface HttpCodec {
  /**
   * 废弃一个输入流的耗时超时时间。因为本身是为了复用connection，所以这个超时时间应该小于建立一次连接的耗时才有意义。
   */
  int DISCARD_STREAM_TIMEOUT_MILLIS = 100;

  /** 返回一个输出流Sink，用来存放request body */
  Sink createRequestBody(Request request, long contentLength);

  /** This should update the HTTP engine's sentRequestMillis field. */
  void writeRequestHeaders(Request request) throws IOException;

  /** 将request刷入底层的socket. */
  void flushRequest() throws IOException;

  /** Flush the request to the underlying socket and signal no more bytes will be transmitted. */
  void finishRequest() throws IOException;

  /**
   * Parses bytes of a response header from an HTTP transport.
   *
   * @param expectContinue true to return null if this is an intermediate response with a "100"
   *     response code. Otherwise this method never returns null.
   */
  Response.Builder readResponseHeaders(boolean expectContinue) throws IOException;

  /** Returns a stream that reads the response body. */
  ResponseBody openResponseBody(Response response) throws IOException;

  /**
   * Cancel this stream. Resources held by this stream will be cleaned up, though not synchronously.
   * That may happen later by the connection pool thread.
   */
  void cancel();
}
```

#### CallServerInterceptor

在OkHttp里面读取数据主要是通过以下四个步骤来实现的

- 1 写入请求头
- 2 写入请求体
- 3 读取响应头
- 4 读取响应体

OkHttp的流程是完全独立的。同样读写数据月是交给相关的类来处理，就是**HttpCodec**(解码器)来处理。

```java
 @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    HttpCodec httpCodec = realChain.httpStream();
    StreamAllocation streamAllocation = realChain.streamAllocation();
    RealConnection connection = (RealConnection) realChain.connection();
    Request request = realChain.request();
    
    long sentRequestMillis = System.currentTimeMillis();
    //写入请求头
    httpCodec.writeRequestHeaders(request);

    Response.Builder responseBuilder = null;
    if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
      // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
      // Continue" response before transmitting the request body. If we don't get that, return what
      // we did get (such as a 4xx response) without ever transmitting the request body.
      if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
        httpCodec.flushRequest();
        responseBuilder = httpCodec.readResponseHeaders(true);
      }
     //写入请求体
      if (responseBuilder == null) {
        // Write the request body if the "Expect: 100-continue" expectation was met.
        Sink requestBodyOut = httpCodec.createRequestBody(request, request.body().contentLength());
        BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);
        request.body().writeTo(bufferedRequestBody);
        bufferedRequestBody.close();
      } else if (!connection.isMultiplexed()) {
        // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1 connection from
        // being reused. Otherwise we're still obligated to transmit the request body to leave the
        // connection in a consistent state.
        streamAllocation.noNewStreams();
      }
    }

    httpCodec.finishRequest();
    //读取响应头
    if (responseBuilder == null) {
      responseBuilder = httpCodec.readResponseHeaders(false);
    }

    Response response = responseBuilder
        .request(request)
        .handshake(streamAllocation.connection().handshake())
        .sentRequestAtMillis(sentRequestMillis)
        .receivedResponseAtMillis(System.currentTimeMillis())
        .build();
    //读取响应体
    int code = response.code();
    if (forWebSocket && code == 101) {
      // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
      response = response.newBuilder()
          .body(Util.EMPTY_RESPONSE)
          .build();
    } else {
      response = response.newBuilder()
          .body(httpCodec.openResponseBody(response))
          .build();
    }

    if ("close".equalsIgnoreCase(response.request().header("Connection"))
        || "close".equalsIgnoreCase(response.header("Connection"))) {
      streamAllocation.noNewStreams();
    }

    if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
      throw new ProtocolException(
          "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
    }
    return response;
  }
```

### 缓存机制

缓存实际上是一个比较复杂的逻辑，单独的功能块，实际上不属于OKhttp上的功能，实际上是通过是http协议和DiskLruCache做了处理。

LinkedHashMap可以实现LRU算法，并且在这个case里，它被用作对DiskCache的内存索引
 告诉你们一个秘密，Universal-Imager-Loader里面的DiskLruCache的实现跟这里的一模一样，除了io使用inputstream/outputstream
 使用LinkedHashMap和journal文件同时记录做过的操作，其实也就是有索引了，这样就相当于有两个备份，可以互相恢复状态
 通过dirtyFiles和cleanFiles，可以实现更新和读取同时操作，在commit的时候将cleanFiles的内容进行更新就好了

还是先从缓存拦截器说起吧

```java
/** Serves requests from the cache and writes responses to the cache. */
public final class CacheInterceptor implements Interceptor {
  final InternalCache cache;

  public CacheInterceptor(InternalCache cache) {
    this.cache = cache;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }

    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }

    return response;
  }

  private static Response stripBody(Response response) {
    return response != null && response.body() != null
        ? response.newBuilder().body(null).build()
        : response;
  }

  /**
   * Returns a new source that writes bytes to {@code cacheRequest} as they are read by the source
   * consumer. This is careful to discard bytes left over when the stream is closed; otherwise we
   * may never exhaust the source stream and therefore not complete the cached response.
   */
  private Response cacheWritingResponse(final CacheRequest cacheRequest, Response response)
      throws IOException {
    // Some apps return a null body; for compatibility we treat that like a null cache request.
    if (cacheRequest == null) return response;
    Sink cacheBodyUnbuffered = cacheRequest.body();
    if (cacheBodyUnbuffered == null) return response;

    final BufferedSource source = response.body().source();
    final BufferedSink cacheBody = Okio.buffer(cacheBodyUnbuffered);

    Source cacheWritingSource = new Source() {
      boolean cacheRequestClosed;

      @Override public long read(Buffer sink, long byteCount) throws IOException {
        long bytesRead;
        try {
          bytesRead = source.read(sink, byteCount);
        } catch (IOException e) {
          if (!cacheRequestClosed) {
            cacheRequestClosed = true;
            cacheRequest.abort(); // Failed to write a complete cache response.
          }
          throw e;
        }

        if (bytesRead == -1) {
          if (!cacheRequestClosed) {
            cacheRequestClosed = true;
            cacheBody.close(); // The cache response is complete!
          }
          return -1;
        }

        sink.copyTo(cacheBody.buffer(), sink.size() - bytesRead, bytesRead);
        cacheBody.emitCompleteSegments();
        return bytesRead;
      }

      @Override public Timeout timeout() {
        return source.timeout();
      }

      @Override public void close() throws IOException {
        if (!cacheRequestClosed
            && !discard(this, HttpCodec.DISCARD_STREAM_TIMEOUT_MILLIS, MILLISECONDS)) {
          cacheRequestClosed = true;
          cacheRequest.abort();
        }
        source.close();
      }
    };

    String contentType = response.header("Content-Type");
    long contentLength = response.body().contentLength();
    return response.newBuilder()
        .body(new RealResponseBody(contentType, contentLength, Okio.buffer(cacheWritingSource)))
        .build();
  }

  /** Combines cached headers with a network headers as defined by RFC 7234, 4.3.4. */
  private static Headers combine(Headers cachedHeaders, Headers networkHeaders) {
    Headers.Builder result = new Headers.Builder();

    for (int i = 0, size = cachedHeaders.size(); i < size; i++) {
      String fieldName = cachedHeaders.name(i);
      String value = cachedHeaders.value(i);
      if ("Warning".equalsIgnoreCase(fieldName) && value.startsWith("1")) {
        continue; // Drop 100-level freshness warnings.
      }
      if (!isEndToEnd(fieldName) || networkHeaders.get(fieldName) == null) {
        Internal.instance.addLenient(result, fieldName, value);
      }
    }

    for (int i = 0, size = networkHeaders.size(); i < size; i++) {
      String fieldName = networkHeaders.name(i);
      if ("Content-Length".equalsIgnoreCase(fieldName)) {
        continue; // Ignore content-length headers of validating responses.
      }
      if (isEndToEnd(fieldName)) {
        Internal.instance.addLenient(result, fieldName, networkHeaders.value(i));
      }
    }

    return result.build();
  }

  /**
   * Returns true if {@code fieldName} is an end-to-end HTTP header, as defined by RFC 2616,
   * 13.5.1.
   */
  static boolean isEndToEnd(String fieldName) {
    return !"Connection".equalsIgnoreCase(fieldName)
        && !"Keep-Alive".equalsIgnoreCase(fieldName)
        && !"Proxy-Authenticate".equalsIgnoreCase(fieldName)
        && !"Proxy-Authorization".equalsIgnoreCase(fieldName)
        && !"TE".equalsIgnoreCase(fieldName)
        && !"Trailers".equalsIgnoreCase(fieldName)
        && !"Transfer-Encoding".equalsIgnoreCase(fieldName)
        && !"Upgrade".equalsIgnoreCase(fieldName);
  }
}
```



#### DiskLruCache

#### Okio

#### 清除线程



### 连接复用机制

连接方式分为隧道连接和Socket连接，几个关键类：

+ Call
+ StreamAllocation
+ RealConnection
+ ConnectionPool

##### Call

```java
/**
 * 一个Call代表一个请求已经准备好可以请求了。Call可以被取消。因为这个对象代表了一个请求/响应对，所以它不能被执行两次。
 */
public interface Call extends Cloneable {
  /** 返回当前Call对应的最原始的请求对象 */
  Request request();

  /**
   * 立刻触发请求，并且会阻塞知道返回响应或者发生错误。
   */
  Response execute() throws IOException;

  /**
   * 安排请求在未来的时间中执行。OkHttpClient#Dispatch会安排好任务在合适执行。这个方法会通过responseCallback回调响应结果给调用方。
   */
  void enqueue(Callback responseCallback);

  /** 在可能的情况下取消请求，如果请求已经完成则不能被取消。 */
  void cancel();

  /**
   * 不管Call被执行了execute还是enqueue都会返回true。执行一个Call两次是不对的。
   */
  boolean isExecuted();

  boolean isCanceled();

  /**
   * 复制一个已经存在的Call，得到一个可以执行的Call。
   Create a new, identical call to this one which can be enqueued or executed even if this call has already been.
   */
  Call clone();

  interface Factory {
    Call newCall(Request request);
  }
}
```

##### StreamAllocation

如何理解StreamAllocation这个OKHttp3中自定义的类？先看一段官方的翻译：

[**翻译-Start**：StreamAllocation协调Connections、Streams、Calls三者之间的关系：

Connections：**物理的Socket连接**。因为建立一个连接可能会很慢，所以需要有能力来取消一个已经建立连接的连接。

Streams：**逻辑上的HTTP 请求/响应对**，位于Connections层智商。每一个连接都有它自己的分配(allocation)限制，用于定义同一个连接上最多支持的流的数量。HTTP/1.x最多一个分配1个流。而HTTP/2可以分配多个，这也就是所谓的连接的多路复用。

Calls：**逻辑上的流的序列**，通常是一个初始请求已经之后触发的多个请求。我们提倡在一个连接上管理一个Call的所有Stream。

StreamAllocation的实例可以用来代表了Call，在一个或多个Connection上使用一个或多个Stream。这个类提供了API用来释放这些资源：

- noNewStreams()：防止一个连接被新创建的流使用。
- streamFinished()：释放分配池(allocation)中的活跃的stream。注意每次只有一个stream处于活跃状态，如果想要新启一个stream，需要先调用streamFinished()来释放当前stream。
- release()：移除Call持有的connection。注意当一个流还在使用，那么connection不会立即被释放。**翻译-FIN**]

他封装了网络请求相关的信息：连接池，地址信息，网络请求，事件回调，负责网络连接的连接、关闭，释放等操作。好像还是不太懂，因为翻译的不好，或者不够简洁。<font color="#dd0000">**简单的用白话表述就是StreamAllocation是Connections、Streams、Calls之间的桥梁。一个Call可能对应多个Stream，而一个Stream必须依赖于一个Connection。StreamAllocation就是负责为请求的Call寻找合适的Connection，并管理Stream**</font>

```java
public final class StreamAllocation {
  public final Address address;
  private RouteSelector.Selection routeSelection;
  private Route route;
  private final ConnectionPool connectionPool;
  public final Call call;
  public final EventListener eventListener;
  private final Object callStackTrace;

  // State guarded by connectionPool.
  private final RouteSelector routeSelector;
  private int refusedStreamCount;
  private RealConnection connection;
  private boolean reportedAcquired;
  private boolean released;
  private boolean canceled;
  private HttpCodec codec;

  public StreamAllocation(ConnectionPool connectionPool, Address address, Call call,
      EventListener eventListener, Object callStackTrace) {
    this.connectionPool = connectionPool;
    this.address = address;
    this.call = call;
    this.eventListener = eventListener;
    this.routeSelector = new RouteSelector(address, routeDatabase(), call, eventListener);
    this.callStackTrace = callStackTrace;
  }

  public HttpCodec newStream(
      OkHttpClient client, Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
    int connectTimeout = chain.connectTimeoutMillis();
    int readTimeout = chain.readTimeoutMillis();
    int writeTimeout = chain.writeTimeoutMillis();
    boolean connectionRetryEnabled = client.retryOnConnectionFailure();

    try {
      RealConnection resultConnection = findHealthyConnection(connectTimeout, readTimeout,
          writeTimeout, connectionRetryEnabled, doExtensiveHealthChecks);
      HttpCodec resultCodec = resultConnection.newCodec(client, chain, this);

      synchronized (connectionPool) {
        codec = resultCodec;
        return resultCodec;
      }
    } catch (IOException e) {
      throw new RouteException(e);
    }
  }

  /**
   * 寻找一个健康的Connection，如果没找到，则会继续循环直到找到一个健康的Connection.
   */
  private RealConnection findHealthyConnection(int connectTimeout, int readTimeout,
      int writeTimeout, boolean connectionRetryEnabled, boolean doExtensiveHealthChecks)
      throws IOException {
    while (true) {
      RealConnection candidate = findConnection(connectTimeout, readTimeout, writeTimeout,
          connectionRetryEnabled);

      // If this is a brand new connection, we can skip the extensive health checks.
      synchronized (connectionPool) {
        if (candidate.successCount == 0) {
          return candidate;
        }
      }

      // Do a (potentially slow) check to confirm that the pooled connection is still good. If it
      // isn't, take it out of the pool and start again.
      if (!candidate.isHealthy(doExtensiveHealthChecks)) {
        noNewStreams();
        continue;
      }

      return candidate;
    }
  }

  /**
   * 返回一个用于装载stream的connection。优先使用已经存在的connection，没有的话就从连接池中获取，再没有就重新构建一个。
   */
  private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
      boolean connectionRetryEnabled) throws IOException {
    boolean foundPooledConnection = false;
    RealConnection result = null;
    Route selectedRoute = null;
    Connection releasedConnection;
    Socket toClose;
    synchronized (connectionPool) {
      if (released) throw new IllegalStateException("released");
      if (codec != null) throw new IllegalStateException("codec != null");
      if (canceled) throw new IOException("Canceled");

      // Attempt to use an already-allocated connection. We need to be careful here because our
      // already-allocated connection may have been restricted from creating new streams.
      releasedConnection = this.connection;
      toClose = releaseIfNoNewStreams();
      if (this.connection != null) {
        // We had an already-allocated connection and it's good.
        result = this.connection;
        releasedConnection = null;
      }
      if (!reportedAcquired) {
        // If the connection was never reported acquired, don't report it as released!
        releasedConnection = null;
      }

      if (result == null) {
        // Attempt to get a connection from the pool.
        Internal.instance.get(connectionPool, address, this, null);
        if (connection != null) {
          foundPooledConnection = true;
          result = connection;
        } else {
          selectedRoute = route;
        }
      }
    }
    closeQuietly(toClose);

    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);
    }
    if (result != null) {
      // If we found an already-allocated or pooled connection, we're done.
      return result;
    }

    // If we need a route selection, make one. This is a blocking operation.
    boolean newRouteSelection = false;
    if (selectedRoute == null && (routeSelection == null || !routeSelection.hasNext())) {
      newRouteSelection = true;
      routeSelection = routeSelector.next();
    }

    synchronized (connectionPool) {
      if (canceled) throw new IOException("Canceled");

      if (newRouteSelection) {
        // Now that we have a set of IP addresses, make another attempt at getting a connection from
        // the pool. This could match due to connection coalescing.
        List<Route> routes = routeSelection.getAll();
        for (int i = 0, size = routes.size(); i < size; i++) {
          Route route = routes.get(i);
          Internal.instance.get(connectionPool, address, this, route);
          if (connection != null) {
            foundPooledConnection = true;
            result = connection;
            this.route = route;
            break;
          }
        }
      }

      if (!foundPooledConnection) {
        if (selectedRoute == null) {
          selectedRoute = routeSelection.next();
        }

        // Create a connection and assign it to this allocation immediately. This makes it possible
        // for an asynchronous cancel() to interrupt the handshake we're about to do.
        route = selectedRoute;
        refusedStreamCount = 0;
        result = new RealConnection(connectionPool, selectedRoute);
        acquire(result, false);
      }
    }

    // If we found a pooled connection on the 2nd time around, we're done.
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);
      return result;
    }

    // Do TCP + TLS handshakes. This is a blocking operation.
    result.connect(
        connectTimeout, readTimeout, writeTimeout, connectionRetryEnabled, call, eventListener);
    routeDatabase().connected(result.route());

    Socket socket = null;
    synchronized (connectionPool) {
      reportedAcquired = true;

      // Pool the connection.
      Internal.instance.put(connectionPool, result);

      // If another multiplexed connection to the same address was created concurrently, then
      // release this connection and acquire that one.
      if (result.isMultiplexed()) {
        socket = Internal.instance.deduplicate(connectionPool, address, this);
        result = connection;
      }
    }
    closeQuietly(socket);

    eventListener.connectionAcquired(call, result);
    return result;
  }

  /**
   * Releases the currently held connection and returns a socket to close if the held connection
   * restricts new streams from being created. With HTTP/2 multiple requests share the same
   * connection so it's possible that our connection is restricted from creating new streams during
   * a follow-up request.
   */
  private Socket releaseIfNoNewStreams() {
    assert (Thread.holdsLock(connectionPool));
    RealConnection allocatedConnection = this.connection;
    if (allocatedConnection != null && allocatedConnection.noNewStreams) {
      return deallocate(false, false, true);
    }
    return null;
  }

  public void streamFinished(boolean noNewStreams, HttpCodec codec, long bytesRead, IOException e) {
    eventListener.responseBodyEnd(call, bytesRead);

    Socket socket;
    Connection releasedConnection;
    boolean callEnd;
    synchronized (connectionPool) {
      if (codec == null || codec != this.codec) {
        throw new IllegalStateException("expected " + this.codec + " but was " + codec);
      }
      if (!noNewStreams) {
        connection.successCount++;
      }
      releasedConnection = connection;
      socket = deallocate(noNewStreams, false, true);
      if (connection != null) releasedConnection = null;
      callEnd = this.released;
    }
    closeQuietly(socket);
    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }

    if (e != null) {
      eventListener.callFailed(call, e);
    } else if (callEnd) {
      eventListener.callEnd(call);
    }
  }

  public HttpCodec codec() {
    synchronized (connectionPool) {
      return codec;
    }
  }

  private RouteDatabase routeDatabase() {
    return Internal.instance.routeDatabase(connectionPool);
  }

  public synchronized RealConnection connection() {
    return connection;
  }

  public void release() {
    Socket socket;
    Connection releasedConnection;
    synchronized (connectionPool) {
      releasedConnection = connection;
      socket = deallocate(false, true, false);
      if (connection != null) releasedConnection = null;
    }
    closeQuietly(socket);
    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }
  }

  /** Forbid new streams from being created on the connection that hosts this allocation. */
  public void noNewStreams() {
    Socket socket;
    Connection releasedConnection;
    synchronized (connectionPool) {
      releasedConnection = connection;
      socket = deallocate(true, false, false);
      if (connection != null) releasedConnection = null;
    }
    closeQuietly(socket);
    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }
  }

  /**
   * Releases resources held by this allocation. If sufficient resources are allocated, the
   * connection will be detached or closed. Callers must be synchronized on the connection pool.
   *
   * <p>Returns a closeable that the caller should pass to {@link Util#closeQuietly} upon completion
   * of the synchronized block. (We don't do I/O while synchronized on the connection pool.)
   */
  private Socket deallocate(boolean noNewStreams, boolean released, boolean streamFinished) {
    assert (Thread.holdsLock(connectionPool));

    if (streamFinished) {
      this.codec = null;
    }
    if (released) {
      this.released = true;
    }
    Socket socket = null;
    if (connection != null) {
      if (noNewStreams) {
        connection.noNewStreams = true;
      }
      if (this.codec == null && (this.released || connection.noNewStreams)) {
        release(connection);
        if (connection.allocations.isEmpty()) {
          connection.idleAtNanos = System.nanoTime();
          if (Internal.instance.connectionBecameIdle(connectionPool, connection)) {
            socket = connection.socket();
          }
        }
        connection = null;
      }
    }
    return socket;
  }

  public void cancel() {
    HttpCodec codecToCancel;
    RealConnection connectionToCancel;
    synchronized (connectionPool) {
      canceled = true;
      codecToCancel = codec;
      connectionToCancel = connection;
    }
    if (codecToCancel != null) {
      codecToCancel.cancel();
    } else if (connectionToCancel != null) {
      connectionToCancel.cancel();
    }
  }

  public void streamFailed(IOException e) {
    Socket socket;
    Connection releasedConnection;
    boolean noNewStreams = false;

    synchronized (connectionPool) {
      if (e instanceof StreamResetException) {
        StreamResetException streamResetException = (StreamResetException) e;
        if (streamResetException.errorCode == ErrorCode.REFUSED_STREAM) {
          refusedStreamCount++;
        }
        // On HTTP/2 stream errors, retry REFUSED_STREAM errors once on the same connection. All
        // other errors must be retried on a new connection.
        if (streamResetException.errorCode != ErrorCode.REFUSED_STREAM || refusedStreamCount > 1) {
          noNewStreams = true;
          route = null;
        }
      } else if (connection != null
          && (!connection.isMultiplexed() || e instanceof ConnectionShutdownException)) {
        noNewStreams = true;

        // If this route hasn't completed a call, avoid it for new connections.
        if (connection.successCount == 0) {
          if (route != null && e != null) {
            routeSelector.connectFailed(route, e);
          }
          route = null;
        }
      }
      releasedConnection = connection;
      socket = deallocate(noNewStreams, false, true);
      if (connection != null || !reportedAcquired) releasedConnection = null;
    }

    closeQuietly(socket);
    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }
  }

  /**
   * Use this allocation to hold {@code connection}. Each call to this must be paired with a call to
   * {@link #release} on the same connection.
   */
  public void acquire(RealConnection connection, boolean reportedAcquired) {
    assert (Thread.holdsLock(connectionPool));
    if (this.connection != null) throw new IllegalStateException();

    this.connection = connection;
    this.reportedAcquired = reportedAcquired;
    connection.allocations.add(new StreamAllocationReference(this, callStackTrace));
  }

  /** Remove this allocation from the connection's list of allocations. */
  private void release(RealConnection connection) {
    for (int i = 0, size = connection.allocations.size(); i < size; i++) {
      Reference<StreamAllocation> reference = connection.allocations.get(i);
      if (reference.get() == this) {
        connection.allocations.remove(i);
        return;
      }
    }
    throw new IllegalStateException();
  }

  /**
   * Release the connection held by this connection and acquire {@code newConnection} instead. It is
   * only safe to call this if the held connection is newly connected but duplicated by {@code
   * newConnection}. Typically this occurs when concurrently connecting to an HTTP/2 webserver.
   *
   * <p>Returns a closeable that the caller should pass to {@link Util#closeQuietly} upon completion
   * of the synchronized block. (We don't do I/O while synchronized on the connection pool.)
   */
  public Socket releaseAndAcquire(RealConnection newConnection) {
    assert (Thread.holdsLock(connectionPool));
    if (codec != null || connection.allocations.size() != 1) throw new IllegalStateException();

    // Release the old connection.
    Reference<StreamAllocation> onlyAllocation = connection.allocations.get(0);
    Socket socket = deallocate(true, false, false);

    // Acquire the new connection.
    this.connection = newConnection;
    newConnection.allocations.add(onlyAllocation);

    return socket;
  }

  public boolean hasMoreRoutes() {
    return route != null
        || (routeSelection != null && routeSelection.hasNext())
        || routeSelector.hasNext();
  }

  @Override public String toString() {
    RealConnection connection = connection();
    return connection != null ? connection.toString() : address.toString();
  }

  public static final class StreamAllocationReference extends WeakReference<StreamAllocation> {
    /**
     * Captures the stack trace at the time the Call is executed or enqueued. This is helpful for
     * identifying the origin of connection leaks.
     */
    public final Object callStackTrace;

    StreamAllocationReference(StreamAllocation referent, Object callStackTrace) {
      super(referent);
      this.callStackTrace = callStackTrace;
    }
  }
}
```



##### RealConnection

##### ConnectionPool

管理HTTP或HTTP/2上连接的重用，指向相同Address的请求将共享一个连接。这个类实现了保持连接打开以备将来重用。<font color="#dd0000">**从源码中看到，默认的maxIdleConnections=5，默认的连接空闲存活时间是5分钟**</font>

```java
  public ConnectionPool() {
    this(5, 5, TimeUnit.MINUTES);
  }
```

内部使用基于SynchronousQueue的线程池用于存放清理任务(cleanupRunnable)。看一下cleanupRunnable的代码：

```java
private final Runnable cleanupRunnable = new Runnable() {
    @Override public void run() {
      while (true) {
        long waitNanos = cleanup(System.nanoTime());
        if (waitNanos == -1) return;
        if (waitNanos > 0) {
          long waitMillis = waitNanos / 1000000L;
          waitNanos -= (waitMillis * 1000000L);
          synchronized (ConnectionPool.this) {
            try {
              ConnectionPool.this.wait(waitMillis, (int) waitNanos);
            } catch (InterruptedException ignored) {
            }
          }
        }
      }
    }
  };
```

可以看到cleanupRunnable是一个死循环，不断的执行cleanup方法

```java
/**
   * 维护连接池操作，当超过keep alive限制或者空闲连接限制时执行，移除一个空闲时间最长的connection。
   * 返回下次执行cleanup方法的时间，如果不再需要cleanup则返回-1
   */
  long cleanup(long now) {
    int inUseConnectionCount = 0;
    int idleConnectionCount = 0;
    RealConnection longestIdleConnection = null;
    long longestIdleDurationNs = Long.MIN_VALUE;

    // 查找并移除一个connection，或者返回下次执行cleanup任务的时间
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();

        // 如果连接正在使用，跳过继续搜索下一个连接
        if (pruneAndGetAllocationCount(connection, now) > 0) {
          inUseConnectionCount++;
          continue;
        }

        idleConnectionCount++;

        // 交换对象，找出空闲时间最长的connection
        long idleDurationNs = now - connection.idleAtNanos;
        if (idleDurationNs > longestIdleDurationNs) {
          longestIdleDurationNs = idleDurationNs;
          longestIdleConnection = connection;
        }
      }

      if (longestIdleDurationNs >= this.keepAliveDurationNs
          || idleConnectionCount > this.maxIdleConnections) {
        // 找到了一个可以移除的connection，从列表中移除，稍后在循环外关闭它
        //注意，走到这个分支，会return 0，意味着会立即执行下一次cleanup任务。
        connections.remove(longestIdleConnection);
      } else if (idleConnectionCount > 0) {
        // 返回下一次进行cleanup的时间距离
        return keepAliveDurationNs - longestIdleDurationNs;
      } else if (inUseConnectionCount > 0) {
        // 所有的connection都在使用中，因此返回一个完整的cleanup任务间隔时间
        return keepAliveDurationNs;
      } else {
        // 没有任何连接，返回-1
        cleanupRunning = false;
        return -1;
      }
    }

    closeQuietly(longestIdleConnection.socket());

    // 立即重新执行cleanup任务
    return 0;
  }
```





### 补充

#### 责任链模式

当你想要让一个以上的对象**有机会**能够处理某个请求的时候，可以使用责任链模式。**链中的每个对象扮演处理器，并且有一个后继对象。它可以处理请求，也可以把请求转发给后继者。**

责任链的优点：

- 将请求的发送者和接收者解耦
- 可以简化你的对象，因为它不需要知道链的结构。
- 通过改变链内的成员或调动他们的顺序，允许你动态地新增或删除责任。

责任链的缺点：

- 可能不容易观察运行时的特征，有碍于除错。
- 并不能保证请求一定会被执行。

![img](<https://img-blog.csdnimg.cn/20190424000014296.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2Mzg4ODM=,size_16,color_FFFFFF,t_70>)

#### SynchronousQueue

#### TLS

安全传输层协议（TLS）用于在两个通信应用程序之间提供保密性和数据完整性。该协议由两层组成： **TLS 记录协议（TLS Record）和 TLS 握手协议（TLS Handshake）**。传输层安全性协议（英语：Transport Layer Security，缩写作**TLS**），及其<font color="#dd0000">**前身**</font>**安全套接层**（Secure Sockets Layer，缩写作**SSL**）是一种安全协议，目的是互联网通信提供安全及数据完整性保障。

SSL包含记录层（Record Layer）和传输层，记录层协议确定传输层数据的封装格式。传输层安全协议使用[X.509](https://baike.baidu.com/item/X.509)认证，之后利用非对称加密演算来对通信方做身份认证，之后交换对称密钥作为会谈密钥（[Session key](https://baike.baidu.com/item/Session%20key)）。这个会谈密钥是用来将通信两方交换的数据做加密，保证两个应用间通信的保密性和可靠性，使客户与服务器应用之间的通信不被攻击者窃听。

TLS协议采用[主从式架构](https://baike.baidu.com/item/%E4%B8%BB%E4%BB%8E%E5%BC%8F%E6%9E%B6%E6%9E%84)模型，用于在两个应用程序间透过网络创建起安全的连线，防止在交换数据时受到[窃听](https://baike.baidu.com/item/%E7%AA%83%E5%90%AC)及篡改。

TLS协议的优势是与高层的[应用层](https://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E5%B1%82)协议（如[HTTP](https://baike.baidu.com/item/HTTP)、[FTP](https://baike.baidu.com/item/FTP)、[Telnet](https://baike.baidu.com/item/Telnet)等）无耦合。应用层协议能透明地运行在TLS协议之上，由TLS协议进行创建加密通道需要的协商和认证。应用层协议传送的数据在通过TLS协议时都会被加密，从而保证通信的私密性。

TLS协议是可选的，必须配置客户端和服务器才能使用。主要有两种方式实现这一目标：一个是使用统一的TLS协议通信端口（例如：用于[HTTPS](https://baike.baidu.com/item/HTTPS)的端口443）；另一个是客户端请求服务器连接到TLS时使用特定的协议机制（例如：邮件、新闻协议和[STARTTLS](https://baike.baidu.com/item/STARTTLS)）。<font color="#dd0000">**一旦客户端和服务器都同意使用TLS协议，他们通过使用一个握手过程协商出一个有状态的连接以传输数据**</font>。通过握手，客户端和服务器协商各种参数用于创建安全连接：

- 当客户端连接到支持TLS协议的服务器要求创建安全连接并列出了受支持的密码组合（加密密码算法和加密哈希函数），握手开始。
- 服务器从该列表中决定加密和散列函数，并通知客户端。
- 服务器发回其数字证书，此证书通常包含服务器的名称、受信任的证书颁发机构（CA）和服务器的公钥。
- 客户端确认其颁发的证书的有效性。
- 为了生成会话密钥用于安全连接，客户端使用服务器的公钥加密随机生成的密钥，并将其发送到服务器，只有服务器才能使用自己的私钥解密。
- 利用随机数，双方生成用于加密和解密的对称密钥。这就是TLS协议的握手，握手完毕后的连接是安全的，直到连接（被）关闭。如果上述任何一个步骤失败，TLS握手过程就会失败，并且断开所有的连接。





#### 其他类

##### EventListener

提供可量化的指标，用于衡量应用中的HTTP请求的质量。提供如下方法：

```java
public abstract class EventListener {
  public static final EventListener NONE = new EventListener() {
  };

  static EventListener.Factory factory(final EventListener listener) {
    return new EventListener.Factory() {
      public EventListener create(Call call) {
        return listener;
      }
    };
  }
  public void callStart(Call call) {
  }
  public void dnsStart(Call call, String domainName) {
  }
  public void dnsEnd(Call call, String domainName, @Nullable List<InetAddress> inetAddressList) {
  }
  public void connectStart(Call call, InetSocketAddress inetSocketAddress, Proxy proxy) {
  }
  public void secureConnectStart(Call call) {
  }
  public void secureConnectEnd(Call call, @Nullable Handshake handshake) {
  }
  public void connectEnd(Call call, InetSocketAddress inetSocketAddress,
      @Nullable Proxy proxy, @Nullable Protocol protocol) {
  }
  public void connectFailed(Call call, InetSocketAddress inetSocketAddress,
      @Nullable Proxy proxy, @Nullable Protocol protocol, @Nullable IOException ioe) {
  }
  public void connectionAcquired(Call call, Connection connection) {
  }
  public void connectionReleased(Call call, Connection connection) {
  }
  public void requestHeadersStart(Call call) {
  }
  public void requestHeadersEnd(Call call, Request request) {
  }
  public void requestBodyStart(Call call) {
  }
  public void requestBodyEnd(Call call, long byteCount) {
  }
  public void responseHeadersStart(Call call) {
  }
  public void responseHeadersEnd(Call call, Response response) {
  }
  public void responseBodyStart(Call call) {
  }
  public void responseBodyEnd(Call call, long byteCount) {
  }
  public void callEnd(Call call) {
  }
  public void callFailed(Call call, IOException ioe) {
  }
}
```

##### NamedRunnable

是一个抽象类，继承了Runnable，为其扩展了一个name字段，并定义了抽象方法execute()。<font color="#dd0000">**注意！！这里的run方法中调用了抽象方法execute()，因此在Dispatcher调度任务时执行executorService().execute(call)，其实就是执行NamedRunnable的抽象方法execute()**</font>

```java
/**
 * Runnable implementation which always sets its thread name.
 */
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

##### AsyncCall & RealCall

两个都继承了**NamedRunnalbe**，并实现了execute()方法，而AsyncCall是RealCall的内部类。两个的不同在于execute的实现，AsyncCall因为是异步任务，所以在**execute方法中通过responseCallback接口对象中的onResponse和onFailure方法像调用方回传了请求结果**。

下面看一下AsyncCall的源码，注意finally中的finished调用。

```java
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;

    AsyncCall(Callback responseCallback) {
      super("OkHttp %s", redactedUrl());
      this.responseCallback = responseCallback;
    }

    String host() {
      return originalRequest.url().host();
    }

    Request request() {
      return originalRequest;
    }

    RealCall get() {
      return RealCall.this;
    }

    @Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        //调用finished方法，从运行中队列中移除当前Call，同时调用promoteCalls方法从等待队列中取得任务添加到运行中队列，并开始执行任务。
        client.dispatcher().finished(this);
      }
    }
  }
```



### 结语

聊一下看完OKHTTP源码之后印象深刻的地方吧！

1. 要看懂OKHTTP的源码，首先还得比较了解Http协议，比如缓存机制需要知道的响应头有：**expires，Cache-Control，Last-Modified，If-Modified-Since，ETag/If-None-Match**；而在判断**重定向**时，如果服务器返回401，那么会回调Authenticator的authenticate方法，我们可以在构造OkHttpClient时重写该方法实现自己的验证逻辑。
2. 使用队列管理同步任务、异步任务。控制并发数。
3. 关于StreamAllocation、Call、Stream之间的关系，以及为什么要这么设计的理解
4. 关于缓存的实现
5. 关于连接复用的实现



### 参考：

1. [OKHttp源码解析(一)—初阶](https://www.jianshu.com/p/82f74db14a18)

2. [okhttp源码解析（四）：重试机制](<https://www.jianshu.com/p/1c271d91529a>)
