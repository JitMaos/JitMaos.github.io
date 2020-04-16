---
title: Retrofit『使用』
date: 2019-04-29 20:17:25
tags:
---


#### Retrofit的优点

1. 可以配置不同HTTP client来实现网络请求，如okhttp、httpclient等 
2. 将接口的定义与使用分离开来，实现结构。
3. 支持多种返回数据解析的Converter可以快速进行数据转换。
4. 和RxJava集成的很好
5. 因为容易和RxJava结合使用，所以对于异步请求，同步请求也不需要做额外的工作。
6. Retrofit是基于OKHttp

#### 简单使用

##### 配置依赖

在module的build.gradle中添加

```java
// Retrofit
api "com.squareup.retrofit2:retrofit:2.3.0"
api "com.squareup.retrofit2:converter-gson:2.3.0"
api "com.squareup.retrofit2:adapter-rxjava2:2.3.0"
    
// OkHttp3
api "com.squareup.okhttp3:okhttp:3.10.0"
api "com.squareup.okhttp3:logging-interceptor:3.10.0"

// RxJava2
api "io.reactivex.rxjava2:rxjava:2.1.9"
api "io.reactivex.rxjava2:rxandroid:2.0.2"

// RxLifecycle
api "com.trello.rxlifecycle2:rxlifecycle:2.2.1"
api "com.trello.rxlifecycle2:rxlifecycle-android:2.2.1"
api "com.trello.rxlifecycle2:rxlifecycle-components:2.2.1"
```

##### 定义Retrofit单例

在Application中初始化Retrofit，因为一个Retrofit对象本身就包含一个线程池，所以我们可以初始化一个Retrofit对象，并将其做成一个全局单例对象

<!-- more -->

```java
/**
 * Retrofit单例管理
 * Created by Leon.W on 2019/4/28
 */
public class RetrofitManager {
    private final String BASE_URL = "https://api.github.com";
    private static RetrofitManager sInstance;
    private Retrofit mRetrofit;
    public static RetrofitManager getInstance() {
        if (null == sInstance) {
            synchronized (RetrofitManager.class) {
                if (null == sInstance) {
                    sInstance = new RetrofitManager();
                }
            }
        }
        return sInstance;
    }

    public void init() {
        if(mRetrofit == null) {
            //初始化一个OkHttpClient
            OkHttpClient.Builder builder = new OkHttpClient.Builder()
                    .connectTimeout(30000, TimeUnit.MILLISECONDS)
                    .readTimeout(30000, TimeUnit.MILLISECONDS)
                    .writeTimeout(30000, TimeUnit.MILLISECONDS);
            builder.addInterceptor(new LoggingInterceptor());
            OkHttpClient okHttpClient = builder.build();

            //使用该OkHttpClient创建一个Retrofit对象
            mRetrofit = new Retrofit.Builder()
                    //添加Gson数据格式转换器支持
                    .addConverterFactory(GsonConverterFactory.create())
                    //添加RxJava语言支持
                    .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                    //指定网络请求client
                    .client(okHttpClient)
                    .baseUrl(BASE_URL)
                    .build();
        }
    }

    public Retrofit getRetrofit() {
        if(mRetrofit == null) {
            throw  new IllegalStateException("Retrofit instance hasn't init!");
        }
        return mRetrofit;
    }
}
```

##### 定义ApiService

```java
//ApiService.java
public interface ApiService {
    @GET("/TP_S/BookList")
    Observable<JsonArrayBase<Book>> queryBookList();
}
```

##### 定义接口方法实现方法

```java
//GithubAPI.java
public class GithubAPI {
    Observable<GithubUserInfo> queryJakeWhartonInfo() {
        return RetrofitManager.getInstance().getRetrofit()
                //动态代理创建GithubAPI对象
                .create(ApiService.class)
                .queryJakeWhartonInfo()
                //指定上游发送事件线程
                .subscribeOn(Schedulers.computation())
                //指定下游接收事件线程
                .observeOn(AndroidSchedulers.mainThread());
    }
}
```

##### 定义返回数据实体类

```java
public class GithubUserInfo {
    String login,url,name,company;
    int id,public_repos,followers;

    @Override
    public String toString() {
        return "GithubUserInfo{" +
                "login='" + login + '\'' +
                ", url='" + url + '\'' +
                ", name='" + name + '\'' +
                ", company='" + company + '\'' +
                ", id=" + id +
                ", public_repos=" + public_repos +
                ", followers=" + followers +
                '}';
    }
}
```

##### 调用接口

```java
new GithubAPI().queryJakeWhartonInfo().subscribe(new Observer<GithubUserInfo>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(GithubUserInfo githubUserInfo) {
        Log.d(TAG,githubUserInfo.toString());
    }
    @Override
    public void onError(Throwable e) {
        e.printStackTrace();
        Log.e(TAG,e.getMessage());
    }
    @Override
    public void onComplete() {
    }
});

>>>输出：
D/TestRetrofit: GithubUserInfo{login='JakeWharton', url='https://api.github.com/users/JakeWharton', name='Jake Wharton', company='Google, Inc.', id=66577, public_repos=102, followers=52467}
```

#### 异常处理

请求过程中的异常一般分为2种类型，一种是类似网络异常、服务器这种环境问题；另一种比如请求参数错误、登录超时、Token失效等异常。分别做如下处理

##### 环境问题

环境异常诸如404、500、502等服务器状态异常，或者设备本身网路异常造成的，这种时候的Exception会在onError方法中得到响应。

##### 数据问题

数据问题诸如请求参数异常、对象为空、登录超时等数据相关异常，这种情况Response还是会走onNext方法，只是我们需要在里面根据自定义的code，来处理各种数据异常。

下面是一个具体的基础Observer类，在其onNext中解析

一般服务器接口返回数据会约定一个简单的格式：

```java
{
    code:int,
    msg:String,
    data:{} //可能是对象，有可能是数组data:[]
}
```

**对应的建议解析类，一般当接口返回先解析其code是否为成功，如果不是，那看看是否是特定的错误码，把错误码code和错误信息msg包装成一个自定义的Exception进行处理。如果成功，则对返回结果进行进一步的解析，针对不同的接口解析成“对象”或者“数组”。**

```java
//JsonBase.java，解析code和msg
public class JsonBase implements Serializable{
	private static final long serialVersionUID = -6182189632617616248L;
	@SerializedName("msg")
	private String msg;
	private int code = -1;

	public int getCode() {
		return code;
	}
	public void setCode(int code) {
		this.code = code;
	}
	public String getMsg() {
		return msg;
	}
	public void setMsg(String msg) {
		this.msg = msg;
	}
}
```

```java
//JsonArrayBase.java，解析数组类型
public class JsonArrayBase<T> extends JsonBase {
	@SerializedName("data")
	List<T> data;
	public List<T> getData() {
		return data;
	}
	public void setData(List<T> data) {
		this.data = data;
	}

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder();
		for(int i = 0; i< data.size(); i++) {
			sb.append(data.get(i).toString());
		}
		return sb.toString();
	}
}
```

```java
//JsonObjBase.java，解析对象类型
public class JsonObjBase<T> extends JsonBase {
	@SerializedName("data")
	T data;
	public T getData() {
		return data;
	}
	public void setData(T data) {
		this.data = data;
	}
}
```

基础Observer，用于处理数据异常(即code不是SUCC的情况)；网络异常(在onError方法中进行toast提示)，**具体使用界面可以通过重写onError来得到回调(比如分页加载失败，需要隐藏加载进度条，此时需要得到失败回调)**

```java
//BaseObserver.java
public abstract class BaseObserver<T> implements Observer<T> {
    private String TAG = "BaseObserver";

    @Override
    public void onNext(T t) {
        if (t == null) {
            onError(HttpCode.ERROR_EMPTY_OBJ, getErrorMessage(HttpCode.ERROR_EMPTY_OBJ));
        } else {
            onSuccess(t);
        }
    }

    /**
     * 外部想要处理异常(比如分页加载失败，需要隐藏加载中效果)时，可以重写该方法
     */
    public void onError(int errorCode, String message) {
    }

    /**
     * 外部重写，接受数据
     * @param t
     */
    public abstract void onSuccess(T t);

    /**
     * 不显示服务器返回错误信息(部分接口返回不规范)
     */
    public boolean isShowErrorToast() {
        return true;
    }

    @Override
    public void onError(Throwable e) {
        int errorCode = -1;
        String errMsg = "";
        //自定义异常
        if (e instanceof MyException) {
            MyException exception = (MyException) e;
            errorCode = exception.getErrorCode();
            errMsg = exception.getMessage();
            handleIybErrorCode(errorCode);
            if (isShowErrorToast()) {
                Toast.makeText(TestAPP.getInstance().getApplicationContext(), errMsg,Toast.LENGTH_LONG).show();
            }
        } else if (e instanceof NullPointerException) { // RxJava2 发送值为null时，不执行 onNext，直接走 onError
            errorCode = HttpCode.ERROR_EMPTY_OBJ;
            errMsg = getErrorMessage(HttpCode.ERROR_EMPTY_OBJ);
        } else if (e instanceof SocketTimeoutException) {
            errorCode = HttpCode.ERROR_TIMEOUT;
            errMsg = getErrorMessage(HttpCode.ERROR_TIMEOUT);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), errMsg,Toast.LENGTH_LONG).show();
        } else if (e instanceof NetworkErrorException) {
            errorCode = HttpCode.ERROR_NETWORK;
            errMsg = getErrorMessage(HttpCode.ERROR_NETWORK);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), errMsg,Toast.LENGTH_LONG).show();
        } else if (e instanceof HttpException) {
            HttpException httpException = (HttpException) e;
            errMsg = httpException != null ? httpException.getMessage() : getErrorMessage(HttpCode.ERROR_SERVER_EXCEPTION);
            int httpErrorCode = httpException != null ? httpException.code() : HttpCode.ERROR_UNKNOWN;
            Log.d(TAG, "Http request error:" + "message=" + errMsg + " :::: " + "httpErrorCode=" + httpErrorCode);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), getErrorMessage(HttpCode.ERROR_SERVER_EXCEPTION),Toast.LENGTH_LONG).show(); // 统一提示服务器异常
        } else {
            errMsg = e != null ? e.getMessage() : getErrorMessage(HttpCode.ERROR_UNKNOWN);
        }
        onError(errorCode, errMsg);
    }

    /**
     * 处理数据异常code
     * @param errorCode
     */
    private void handleIybErrorCode(int errorCode) {
        if (errorCode == HttpCode.ERROR_ALREADY_REGISTER) {
            //已注册处理逻辑
        } else if (errorCode == HttpCode.ERROR_LOGIN_EXPIRED ) {
            //登录超时处理逻辑
        }
    }
    
    private String getErrorMessage(int errorCode) {
        String message = HttpCode.ERRORS.get(errorCode);
        if (TextUtils.isEmpty(message)) {
            message = HttpCode.ERRORS.get(HttpCode.ERROR_UNKNOWN);
        }
        return message;
    }
}
```

```java
public abstract class BaseObserver<T> implements Observer<T> {
    private String TAG = "BaseObserver";

    @Override
    public void onNext(T t) {
        if (t == null) {
            onError(HttpCode.ERROR_EMPTY_OBJ, getErrorMessage(HttpCode.ERROR_EMPTY_OBJ));
        } else {
            onSuccess(t);
        }
    }
    /**外部想要处理异常时(比如分页加载失败，需要隐藏加载中效果)，可以重写该方法*/
    public void onError(int errorCode, String message) {
    }

    public abstract void onSuccess(T t);

    /** 不显示服务器返回错误信息(部分接口返回不规范) */
    public boolean isShowErrorToast() {
        return true;
    }

    @Override
    public void onError(Throwable e) {
        int errorCode = -1;
        String errMsg = "";
        //自定义异常
        if (e instanceof MyException) {
            MyException exception = (MyException) e;
            errorCode = exception.getErrorCode();
            errMsg = exception.getMessage();
            handleErrorCode(errorCode);
            if (isShowErrorToast()) {
                Toast.makeText(TestAPP.getInstance().getApplicationContext(), errMsg,Toast.LENGTH_LONG).show();
            }
        } else if (e instanceof NullPointerException) { // RxJava2 发送值为null时，不执行 onNext，直接走 onError
            errorCode = HttpCode.ERROR_EMPTY_OBJ;
            errMsg = getErrorMessage(HttpCode.ERROR_EMPTY_OBJ);
        } else if (e instanceof SocketTimeoutException) {
            errorCode = HttpCode.ERROR_TIMEOUT;
            errMsg = getErrorMessage(HttpCode.ERROR_TIMEOUT);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), errMsg,Toast.LENGTH_LONG).show();
        } else if (e instanceof NetworkErrorException) {
            errorCode = HttpCode.ERROR_NETWORK;
            errMsg = getErrorMessage(HttpCode.ERROR_NETWORK);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), errMsg,Toast.LENGTH_LONG).show();
        } else if (e instanceof HttpException) {
            HttpException httpException = (HttpException) e;
            errMsg = httpException != null ? httpException.getMessage() : getErrorMessage(HttpCode.ERROR_SERVER_EXCEPTION);
            int httpErrorCode = httpException != null ? httpException.code() : HttpCode.ERROR_UNKNOWN;
            Log.d(TAG, "Http request error:" + "message=" + errMsg + " :::: " + "httpErrorCode=" + httpErrorCode);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), getErrorMessage(HttpCode.ERROR_SERVER_EXCEPTION),Toast.LENGTH_LONG).show(); // 统一提示服务器异常
        } else {
            errMsg = e != null ? e.getMessage() : getErrorMessage(HttpCode.ERROR_UNKNOWN);
            Toast.makeText(TestAPP.getInstance().getApplicationContext(), e.getMessage(),Toast.LENGTH_LONG).show(); // 统一提示服务器异常
        }
        onError(errorCode, errMsg);
    }

    /**
     * 处理数据异常code
     * @param errorCode
     */
    private void handleErrorCode(int errorCode) {
        if (errorCode == HttpCode.ERROR_ALREADY_REGISTER) {
            //已注册处理逻辑
        } else if (errorCode == HttpCode.ERROR_LOGIN_EXPIRED ) {
            //登录超时处理逻辑
        }
    }
    private String getErrorMessage(int errorCode) {
        String message = HttpCode.ERRORS.get(errorCode);
        if (TextUtils.isEmpty(message)) {
            message = HttpCode.ERRORS.get(HttpCode.ERROR_UNKNOWN);
        }
        return message;
    }
}
```

```java
//MyException.java
public class MyException extends Exception {
    private int mErrorCode;
    private String mMessage;

    public MyException(int errorCode, String message) {
        super();
        mErrorCode = errorCode;
        mMessage = message;
    }
    public int getErrorCode() {
        return mErrorCode;
    }
    public void setErrorCode(int mErrorCode) {
        this.mErrorCode = mErrorCode;
    }
    public String getMessage() {
        return mMessage;
    }
    public void setMessage(String message) {
        this.mMessage = message;
    }
}
```

```java
public class HttpCode {
    public static final int ERROR_UNKNOWN = -1;
    public static final int ERROR_SUCCESS = 0;
    // 1000~1099 自定义错误码
    public static final int ERROR_NETWORK = 1000;
    public static final int ERROR_TIMEOUT = 1001;
    public static final int ERROR_SERVER_EXCEPTION = 1002;
    public static final int ERROR_EMPTY_OBJ = 1011;
    public static final int ERROR_ALREADY_REGISTER = 100001; // 已经注册过
    public static final int ERROR_LOGIN_EXPIRED = 100002; // 登录cookie超时,需要重新登录
    public static final SparseArray<String> ERRORS = new SparseArray<>();

    static {
        ERRORS.append(ERROR_UNKNOWN, "未知错误");
        ERRORS.append(ERROR_SUCCESS, "请求成功");
        ERRORS.append(ERROR_NETWORK, "网络错误");
        ERRORS.append(ERROR_TIMEOUT, "连接超时");
        ERRORS.append(ERROR_SERVER_EXCEPTION, "服务器异常");
        ERRORS.append(ERROR_ALREADY_REGISTER, "您的手机号已经注册过i云保帐号");
        ERRORS.append(ERROR_LOGIN_EXPIRED, "登录超时,需要重新登录");
        ERRORS.append(ERROR_EMPTY_OBJ, "返回对象为空!");
    }
}
```

#### 生命周期绑定

有时我们关闭一个页面时，希望该页面上正在进行以及准备开始进行的请求能够及时关闭、取消掉。以免造成内存溢出或空指针异常等问题。此时就需要将请求与页面的生命周期相关联。因为Retrofit和RxJava2集成，由RxJava2控制上下游的时间分发，所以**处理RxJava2的生命周期问题就是处理Retrofit请求的声明周期问题。**可以看到在上面的”依赖配置“段落中导入了三个rxlifiecycle相关的依赖，其中rxliftcycle-components包含我们将要使用到的RxAppCompatActivity，而他有依赖另外两个依赖，可以查看RxAppCompatActivity的源码验证：

```java
//RxAppCompatActivity.java
import com.trello.rxlifecycle2.LifecycleProvider;
import com.trello.rxlifecycle2.LifecycleTransformer;
//来自基础包rxlifecycle
import com.trello.rxlifecycle2.RxLifecycle; 
//来自android包rxlifecycle-android
import com.trello.rxlifecycle2.android.ActivityEvent; 
//来自android包rxlifecycle-android
import com.trello.rxlifecycle2.android.RxLifecycleAndroid;
import io.reactivex.Observable;
import io.reactivex.subjects.BehaviorSubject;

public abstract class RxAppCompatActivity extends AppCompatActivity implements LifecycleProvider<ActivityEvent> { 
        ......
}
```

下面开始声明周期相关的Demo，演示一下**未绑定页面生命周期导致内存溢出的问题**，假设有一个Activity中有一个operator每隔1秒发送一个事件：

```java
public class TestMemLeakActivity extends Activity {
    private String TAG = "TestMemLeak";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.rx_act_test_leak);
        Observable.interval(1000l, TimeUnit.MILLISECONDS)
                .subscribeOn(Schedulers.computation())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                    }
                    @Override
                    public void onNext(Long aLong) {
                        Log.d(TAG,aLong + "");
                    }
                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG,e.getMessage());
                    }
                    @Override
                    public void onComplete() {
                        Log.d(TAG,"onComplete");
                    }
                });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG,"onDestory------");
    }
}
>>>输出：
04-29 19:38:00.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 0
04-29 19:38:01.101 18964-18964/com.ebm.rxjava D/TestMemLeak: 1
04-29 19:38:02.101 18964-18964/com.ebm.rxjava D/TestMemLeak: 2
04-29 19:38:03.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 3
04-29 19:38:04.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 4
04-29 19:38:05.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 5
04-29 19:38:05.110 18964-18964/com.ebm.rxjava D/TestMemLeak: onDestory------
04-29 19:38:06.105 18964-18964/com.ebm.rxjava D/TestMemLeak: 6
04-29 19:38:07.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 7
04-29 19:38:08.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 8
04-29 19:38:09.102 18964-18964/com.ebm.rxjava D/TestMemLeak: 9
```

从Log可以看出，TestMemLeakActivity关闭调用onDestory()之后，事件没有随着界面关闭而停止发送，这样**会导致Activity无法回收，进而导致内存泄露**。下面使用RxAppCompatActivity进行将RxJava绑定到Activity的声明周期。

```java
//1.继承自RxAppCompatActivity
public class TestMemLeakActivity extends RxAppCompatActivity {
    private String TAG = "TestMemLeak";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.rx_act_test_leak);
        Observable.interval(1000l, TimeUnit.MILLISECONDS)
                .subscribeOn(Schedulers.computation())
                .observeOn(AndroidSchedulers.mainThread())
                //2.绑定声明周期
                .compose(this.<Long>bindToLifecycle())
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                    }
                    @Override
                    public void onNext(Long aLong) {
                        Log.d(TAG,aLong + "");
                    }
                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG,e.getMessage());
                    }
                    @Override
                    public void onComplete() {
                        Log.d(TAG,"onComplete");
                    }
                });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG,"onDestory------");
    }
}
>>>输出：
04-29 19:41:41.254 20080-20080/com.ebm.rxjava D/TestMemLeak: 0
04-29 19:41:42.254 20080-20080/com.ebm.rxjava D/TestMemLeak: 1
04-29 19:41:43.254 20080-20080/com.ebm.rxjava D/TestMemLeak: 2
04-29 19:41:44.253 20080-20080/com.ebm.rxjava D/TestMemLeak: 3
04-29 19:41:44.762 20080-20080/com.ebm.rxjava D/TestMemLeak: onComplete
04-29 19:41:44.762 20080-20080/com.ebm.rxjava D/TestMemLeak: onDestory------
```

从上面的Log可以看出，界面关闭之前先发送了onComplete事件，关闭了事件流的发送。

#### 更新

**19-08-19**：如果要使用Log，需要注意Log会强制要求将所有响应内容加载到内存中，以便进行打印操作。这可能会引发<font color="#dd0000">**OOM**</font> 问题，参考JakeWharton的评论回复。

```java
JakeWharton commented on Jun 24, 2014 
By calling .setLogLevel(RestAdapter.LogLevel.FULL) you are forcing Retrofit to buffer the entire request body into memory so that it can log. That’s what the call to readBodyToBytesIfNecessary in the stack trace is doing. 
Enabling logging like this should only be done when debugging.
```

**19-08-19**：使用<font color="#dd0000">**@Streaming**</font>处理大文件下载，避免造成内存溢出。(边下载边写入磁盘，而不是全部下载完成后在写入磁盘)

参考：[Retrofit（重构——下载大文件）](<https://blog.csdn.net/qqyanjiang/article/details/51076940>)

