title: Retrofit+RxJava2+OkHttp统一处理封装
author: liompei
tags:
  - retrofit
  - rxjava
  - okhttp
categories:
  - Android
date: 2017-12-27 09:20:00
---
## 准备

本文需要的框架

Retrofit https://square.github.io/retrofit/

OkHttp https://square.github.io/okhttp/

RxJavahttps://github.com/ReactiveX/RxJava

进入后引用最新版本

Retrofit

```glide
dependencies{
    implementation 'com.squareup.retrofit2:retrofit:2.3.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.3.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.3.0'
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.1'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.9.0'
    implementation 'com.trello.rxlifecycle2:rxlifecycle-components:2.2.0'
}
```
`com.squareup.retrofit2:retrofit:2.3.0`是retrofit的主体

`com.squareup.retrofit2:converter-gson:2.3.0`是它的json解析器,使用gson解析结果,因此网络请求的参数类型都必须是json,若想解析String类型,则可替换成`com.squareup.retrofit2:converter-scalars:2.3.0`

`com.squareup.retrofit2:adapter-rxjava2:2.3.0`引入了RxJava2

`io.reactivex.rxjava2:rxandroid:2.0.1`则可指定事件发生的线程

`com.squareup.okhttp3:logging-interceptor:3.9.0`是okHttp的拦截器,打印日志用的

`com.trello.rxlifecycle2:rxlifecycle-components:2.2.0`是用于绑定生命周期,解决RxJava内存泄漏


Manifast.xml加入权限

```xml
    <uses-permission android:name="android.permission.INTERNET"/>
```

## 开始

先写一个HttpConfig ,用于基础的参数配置

```java
/**
 * Created by Liompei
 * time : 2017/12/27 9:59
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:
 */

public class HttpConfig {

    public static int HTTP_TIME_OUT=3000;

    public static String BASE_URL="http://192.168.31.144:8080/gangup/";

    public boolean isShowLoading = false;  //是否显示加载动画

    public boolean isCancelable = true;  //是否允许取消,默认允许

    public boolean isCanceledOnTouchOutside = false;  //是否允许点击屏幕取消,默认不允许

    public String mContent="请稍后...";  //progress显示的文字

    public DialogInterface.OnCancelListener mOnCancelListener;  //取消事件

    public Context mContext;

    //使用默认配置项
    public HttpConfig() {
    }

    //显示progress配置项
    public HttpConfig showLoading(Context context){
        mContext=context;
        isShowLoading=true;
        return this;
    }

    public boolean isShowLoading() {
        return isShowLoading;
    }

    public void setShowLoading(boolean showLoading) {
        isShowLoading = showLoading;
    }

    public boolean isCancelable() {
        return isCancelable;
    }

    public void setCancelable(boolean cancelable) {
        isCancelable = cancelable;
    }

    public boolean isCanceledOnTouchOutside() {
        return isCanceledOnTouchOutside;
    }

    public void setCanceledOnTouchOutside(boolean canceledOnTouchOutside) {
        isCanceledOnTouchOutside = canceledOnTouchOutside;
    }

    public String getContent() {
        return mContent;
    }

    public void setContent(String content) {
        mContent = content;
    }

    public DialogInterface.OnCancelListener getOnCancelListener() {
        return mOnCancelListener;
    }

    public void setOnCancelListener(DialogInterface.OnCancelListener onCancelListener) {
        mOnCancelListener = onCancelListener;
    }

    public Context getContext() {
        return mContext;
    }

    public void setContext(Context context) {
        mContext = context;
    }

}
```

新建基础解析类,比如我要解析一个
```json
{
    "code":"200",
    "msg":"请求成功",
    "result":{
        "username":"张三",
        "age":20,
        "adress":"北京"
    }
}
```
新建HttpResult.class
```java
/**
 * Created by Liompei
 * time : 2017/12/27 10:10
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:基础解析类
 */

public class HttpResult<T> {

    @SerializedName("code")
    private int code;
    @SerializedName("message")
    private String message;
    @SerializedName("result")
    private T result;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getResult() {
        return result;
    }

    public void setResult(T result) {
        this.result = result;
    }

    @Override
    public String toString() {

        return "code: " + code + "\nmessage: " + message + "\nresult: " + result;
    }
}
```
解析的结果
```java
/**
 * Created by Liompei
 * time : 2017/12/27 10:31
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:
 */

public class UserBean {
    
    private String username;
    private int age;
    private String address;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

新建HttpCallback
```java
/**
 * Created by Liompei
 * time : 2017/12/27 10:08
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:
 */

public abstract class HttpCallback<T> {
    
    public void onSubscribe(Disposable d) {

    }

    public abstract void onNext(HttpResult<T> httpResult);

    public void onError(Throwable e) {
        Zx.d(e.getMessage());
        Zx.show(e.getMessage());
    }

    public void onFinish() {

    }

}

```

MyObserver

```java
/**
 * Created by Liompei
 * time : 2017/12/27 10:26
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:
 */

public class MyObserver<T> implements Observer<HttpResult<T>> {

    private HttpCallback<T> mHttpCallback;
    private Disposable mDisposable;
    private HttpConfig mHttpConfig;
    private ProgressDialog mProgressDialog;

    //默认不显示progress
    public MyObserver(HttpCallback<T> httpCallback) {
        mHttpCallback = httpCallback;
        mHttpConfig = new HttpConfig();
    }

    public MyObserver(HttpCallback<T> httpCallback, HttpConfig httpConfig) {
        mHttpCallback = httpCallback;
        mHttpConfig = httpConfig;
        if (null != mHttpConfig.mContext && mHttpConfig.isShowLoading) {
            mProgressDialog = new ProgressDialog(mHttpConfig.mContext);
            mProgressDialog.setMessage(mHttpConfig.getContent());
            mProgressDialog.setCancelable(mHttpConfig.isCancelable);
            mProgressDialog.setCanceledOnTouchOutside(mHttpConfig.isCanceledOnTouchOutside);
            mProgressDialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
                @Override
                public void onCancel(DialogInterface dialog) {
                    if (!mDisposable.isDisposed()) {
                        mDisposable.dispose();  //释放资源
                    }
                    if (!(null == mHttpConfig.getOnCancelListener())) {
                        //点击事件
                        mHttpConfig.getOnCancelListener().onCancel(dialog);
                    }
                }
            });
        }
    }

    @Override
    public void onSubscribe(Disposable d) {
        mDisposable = d;
        mHttpCallback.onSubscribe(d);
        if (mHttpConfig.isShowLoading && mProgressDialog != null) {
            if (!mProgressDialog.isShowing()) {
                mProgressDialog.show();
            }
        }
    }

    @Override
    public void onNext(HttpResult<T> tHttpResult) {
        mHttpCallback.onNext(tHttpResult);
    }

    @Override
    public void onError(Throwable e) {
        if (null != mProgressDialog) {
            if (mProgressDialog.isShowing()) {
                mProgressDialog.dismiss();
            }
        }
        mHttpCallback.onError(e);
        mHttpCallback.onFinish();
    }

    @Override
    public void onComplete() {
        if (null != mProgressDialog) {
            if (mProgressDialog.isShowing()) {
                mProgressDialog.dismiss();
            }
        }
        mHttpCallback.onFinish();
    }
}
```

新建接口类,放置要请求的参数
```java
/**
 * Created by Liompei
 * time : 2017/12/27 10:29
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:模拟接口
 */

public interface HttpService {

    @GET("login")
    Observable<HttpResult<UserBean>> getLogin(@Query("username") String username, @Query("password") String password);

    @FormUrlEncoded
    @POST("login")
    Observable<HttpResult<UserBean>> postLogin(@Field("username") String username, @Field("password") String password);

}

如果返回结果是list,则将`Observable<HttpResult<UserBean>>`改为`Observable<HttpResult<List<UserBean>>>`即可
```
新建HttpRequest,放置要请求的参数配置

第一是单例的实例化,第二是请求的接口内容

```java
/**
 * Created by Liompei
 * time : 2017/12/27 10:44
 * 1137694912@qq.com
 * https://github.com/liompei
 * remark:请求
 */

public class HttpRequest {

    private Retrofit mRetrofit;
    private OkHttpClient mHttpClient;
    private HttpService mHttpService;

    private static HttpRequest instance;

    public static HttpRequest getInstance() {
        if (instance == null) {
            synchronized (HttpRequest.class) {
                instance = new HttpRequest();
            }
        }
        return instance;
    }

    //日志打印
    HttpLoggingInterceptor logging = new HttpLoggingInterceptor(new HttpLoggingInterceptor.Logger() {
        @Override
        public void log(String message) {
            Zx.i(message);
        }
    });

    //默认方法
    private HttpRequest() {

        logging.setLevel(HttpLoggingInterceptor.Level.BODY);
        //手动创建一个OkHttpClient并设置超时时间
        mHttpClient = new OkHttpClient.Builder()
                .connectTimeout(HttpConfig.HTTP_TIME_OUT, TimeUnit.SECONDS)
                .addInterceptor(logging)
                .build();

        mRetrofit = new Retrofit.Builder()
                .baseUrl(HttpConfig.BASE_URL)
                .client(mHttpClient)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();

        mHttpService = mRetrofit.create(HttpService.class);
    }

}
```

最后加入请求
```java
    //****************请求*********************
    public void getLogin(RxAppCompatActivity activity, String username, String password, HttpCallback<UserBean> httpCallback) {
        Observable<HttpResult<UserBean>> observable = mHttpService.getLogin(username, password);
        observable.subscribeOn(Schedulers.io());
        observable.unsubscribeOn(Schedulers.io());
        observable.compose(RxLifecycle.bindUntilEvent(activity.lifecycle(), ActivityEvent.DESTROY));
        observable.observeOn(AndroidSchedulers.mainThread());
        observable.filter(new Predicate<HttpResult<UserBean>>() {
            @Override
            public boolean test(HttpResult<UserBean> httpResult) throws Exception {
                if (!httpResult.isSuccess()) {
                    throw new Exception(httpResult.getMessage() + ",错误码: " + httpResult.getCode());
                }
                return true;
            }
        });
        observable.subscribe(new MyObserver<UserBean>(httpCallback));
    }
```

代码太多了,每次都要写这么多,可以新建一个方法调用,比如

```java
    private void toSubscribe(RxAppCompatActivity activity, Observable observable, MyObserver observer) {

        observable.subscribeOn(Schedulers.io())
                .unsubscribeOn(Schedulers.io())
                .compose(RxLifecycle.bindUntilEvent(activity.lifecycle(), ActivityEvent.DESTROY))
                .observeOn(AndroidSchedulers.mainThread())
                .filter(new Predicate<HttpResult>() {
                    @Override
                    public boolean test(HttpResult httpResult) throws Exception {
                        if (!httpResult.isSuccess()) {
                            throw new Exception(httpResult.getMessage() + ",错误码: " + httpResult.getCode());
                        }
                        return true;
                    }
                })
                .subscribe(observer);
    }
    
   
```

```java
    public void postLogin(RxAppCompatActivity activity,String username,String password,HttpCallback<UserBean> httpCallback){
        Observable observable=mHttpService.postLogin(username, password);
        toSubscribe(activity,observable,new MyObserver(httpCallback));
    }
```

调用的方法比较简单,

```java
HttpRequest.getInstance().getLogin(this, "", "", new HttpCallback<UserBean>() {
            @Override
            public void onNext(HttpResult<UserBean> httpResult) {
                
            }
        });
```



## 上传文件


在HttpService中加入以下

```java
    @Multipart
    @POST("upload")
    Observable<HttpResult<UserBean>> upload(@Part MultipartBody.Part file,
                                            @Part("id")RequestBody id);
```
这里是假设通过id和头像文件修改头像

在HttpRequest中加入以下代码

```java
    public void upload(RxAppCompatActivity activity, String id, File file,HttpCallback<UserBean> httpCallback){
        RequestBody requestFile =
                RequestBody.create(MediaType.parse("multipart/form-data"), file);
        MultipartBody.Part filePart = MultipartBody.Part.createFormData("file", file.getName(), requestFile);

        RequestBody idBody = RequestBody.create(MediaType.parse("text/plain"), id);

        Observable observable = mHttpService.upload(filePart, idBody);
        toSubscribe(activity, observable, new MyObserver(httpCallback, new HttpConfig().showLoading(activity)));
    }

```

## 其他注意事项
```
@GET("")  //get请求

@FormUrlEncoded
@POST("")  //post表单请求

@POST("")  //此种情况下post可无参请求

@Multipart
@POST("")  //上传文件

```

## 项目源码

https://github.com/liompei/RetrofitDemo