---
title: 使用Rxjava&Retrofit搭建网络层
date: 2018-12-08 19:57:18
tags:
- android
- rxjava
- retrofit
categories:
- Android开发
---

# 概述

之前上线了一个项目，姑且称为TestApp，开发过程中遇到了一些坑，决定对网络层做一点总结。项目的网络层使用了目前比较流行的Retrofit和Rxjava框架，编程语言使用了Kotlin。

---

<br>

# 配置

配置网络层Rxjava2.x和Retrofit最新的依赖
```gradle
    // RxJava
    implementation deps.rxjava2
    implementation deps.rx_android
    // Network
    implementation deps.retrofit.runtime
    implementation deps.retrofit.gson
    implementation deps.retrofit.rxjava_adapter
    implementation deps.okhttp_logging_interceptor // 打印网络层log
```

其中deps是*versions.gradle*文件中的一个变量, *versions.gradle*文件用于管理各个依赖的版本信息，可在全局gradle文件中引入。

---

<br>

# 一般实践

与后端通信的接口数据协议如下
```json
{
  "code": "0", // 0表示业务请求成功， 非0表示业务请求异常
  "message": "操作成功",
  "data": {
    "result": 1523326559000 // data可以为空、可以为单个数据、可以为规范的json数据
  },
  "responseTime": 1523326674474
}

```

对应的Model如下
```kotlin
data class ApiResponse<out T>(val code: String,
                              val data: T?,
                              val message: String?) : Serializable
```

Android端部分基础接口API如下
```kotlin

interface TestApi {
	// 登录
	// 这里需要说明下，因为data即这里的TokenModel可以为空，Rxjava里不允许传递空，而response是不能为空的，所以最后返回response作为处理的对象。
    @POST("auth/login")
    fun login(@Body loginRequest: Map<String, String>
    ): Flowable<ApiResponse<TokenModel>>
   
    @GET("salt")
    fun getSalt(): Flowable<ApiResponse<SaltModel>>

	// 获取用户基本信息
	// token: header里的动态Token值
    @GET("users/v1/baseInfo")
    fun getUserBaseInfo(@Header("X-Authorization") token: String
    ): Flowable<ApiResponse<UserModel>>

	// 修改头像后通知服务端
    @PUT("users/v1/avatar")
    fun notifyAvatarUpdate(@Header("X-Authorization") token: String
    ): Flowable<ApiResponse<Boolean?>>

	// 获取上传头像需要的信息，我们上传头像到阿里云服务器，之后会单独说明这部分
	@GET("aliyun/sts")
    fun getAliOSSSTS(@Header("X-Authorization") token: String,
                     @Query("right") right: String,
                     @Query("prod") prod: String
    ): Flowable<ApiResponse<AliOSSSTSModel>>
}
```


实现接口的服务如下
```kotlin

object TestService {
    private val netApi = createNetApi() 
    // 远程服务地址
    private val API_BASE_URL = BuildConfig.SERVICE_PLATFORM

    private val parameterInterceptor: Interceptor
        get() = Interceptor { chain ->
            val request = chain.request()
            val newUrlBuilder = request.url()
                    .newBuilder()
                    .scheme(request.url().scheme())
                    .host(request.url().host())

            val newRequest = request.newBuilder()
                    .method(request.method(), request.body())
                    .url(addParameters(newUrlBuilder, DataManager.baseRequestParams().get()).build()) // 这里是在Url的末尾添加基本请求参数，如版本号、平台类型、时间等
                    .removeHeader("Pragma")
                    .build()
            chain.proceed(newRequest)
        }

    private fun addParameters(builder: HttpUrl.Builder, params: Map<String, String>): HttpUrl.Builder {
        for ((key, value) in params) {
            builder.addQueryParameter(key, value)
        }
        return builder
    }

    fun api(): TestApi {
        return netApi
    }

    private fun createNetApi(): TestApi {
        val okHttpClient = OkHttpClient.Builder()
                .addInterceptor(parameterInterceptor) // 添加基本参数
                .addInterceptor(HttpLoggingInterceptor()
                       .setLevel(HttpLoggingInterceptor.Level.BODY))  // 打印log
                .readTimeout(5, TimeUnit.SECONDS)
                .connectTimeout(10, TimeUnit.SECONDS)
                .build()

        val retrofit = Retrofit.Builder()
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .baseUrl(API_BASE_URL)
                .client(okHttpClient)
                .build()
        return retrofit.create(TestApi::class.java)
    }
}

```

一般到达这一步就可以直接调用API了，如：
```kotlin
// 检查版本
TestService.api().checkVersion()
                .compose(apply())
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread()
                .subscribe({ response->
	                // on success
                    // do something for response successful
                    // has new version or not
                }, { t ->
	                // on error
	                // do something for net error
                })
```

但这样写需要考虑一下几个问题
1. 代码重复。每次都要写线程切换等重复代码。
2. 异常处理。subscribe()的onSuccess处理的是响应成功的结果，包括业务正常code==0和业务异常code!=0两种情况，我们一般希望业务异常和网络异常放在一起统一处理。另外，网络异常的话，提示信息不友好，我们希望对其进行统一处理。
3. 链式调用。比如请求登录的时候，需要对密码做加盐处理，即现请求盐值接口，之后请求登录接口，直接在调用的地方写则比较臃肿，多次调用会出现代码重复，增加维护难度。
4. 接口统一。这里只显示了一个远程服务，如果有多个远程服务以及多个网络数据协议的话(比如上传头像到阿里云服务器)则需要分别写接口以及接口实现的服务，导致接口调用分散。

解决以上几点，我们需要对基础的API方法做一层封装

-- -

<br>

# API封装

下面对接口做封装
```kotlin

object WrapNetApi {

    // version
    fun checkVersion(): Flowable<ApiResponse<VersionModel?>> {
        return TestService.api()
                .checkVersion()
                .compose(apply())
    }

	// 登录。链式请求
    fun loginWithSalt(mobile: String, password: String): Flowable<ApiResponse<TokenModel?>> {
        return TestService.api()
                .getSalt()
                .flatMap { response ->
                   // 从response里获取盐值并对mobile进行加密，得到参数map
                   // ----省略部分代码
                    return@flatMap TestService.api().login(map)
                }
                .compose(apply())
    }
	
	// 获取基本信息
    fun getUserBaseInfo(): Flowable<ApiResponse<UserModel?>> {
        return TestService.api()
                .getUserBaseInfo(token())
                .compose(apply())
    }

    // 上传头像。后面有具体说明
    fun uploadAvatar(context: Context, avatarData: ByteArray): Flowable<ApiResponse<Boolean?>> {
        return TestService.api()
                .getAliOSSSTS(token(),"WRITE", "OSS")
                .flatMap { response ->
                    AliOSSService.uploadFile(context, response.data!!, avatarData)
                }
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.io())
                .flatMap {
                    TestService.api().notifyAvatarUpdate(token())
                }
                .compose(apply())
    }

	// 异常处理
    private fun <T> apply(): FlowableTransformer<ApiResponse<T?>, ApiResponse<T?>> {
        return FlowableTransformer { flowable ->
            flowable.map({ response ->
                if (ApiCode.SUCCESS.code != response.code) {
                    // 业务异常。
                    throw ApiException(response)
                }
                response
            })
                    .onErrorResumeNext { t: Throwable ->
                        // 非业务异常
                        // ExceptionEngine.handleException将网络等非业务异常封装成ApiException类并赋予友好的提示message
                        Flowable.error(ExceptionEngine.handleException(t))
                    }
                    .subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread())// 线程调度
        }
    }

	// 获取Token
    private fun token(): String {
	    // 省略
    }
}

```

其中，ApiException如下
```kotlin

open class ApiException(val code: String, message: String?) : Exception(message) {
    var data: Any? = null

    constructor(code: String, message: String?, data: Any?) : this(code, message) {
        this.data = data
    }

    constructor(apiResponse: ApiResponse<*>) : this(apiResponse.code, apiResponse.message, apiResponse.data)
}

```

ExceptionEngine代码如下
```kotlin
object ExceptionEngine {
    fun handleException(e: Throwable): ApiException {
        if (e is ApiException) {
            return e
        }

        val apiCode: ApiCode = if (e is HttpException) {
            ApiCode.ERROR_HTTP
        } else if (e is JsonParseException
                || e is JSONException
                || e is ParseException) {
            ApiCode.ERROR_PARSE
        } else if (e is SocketException
                || e is UnknownHostException) {
            ApiCode.ERROR_NETWORK
        } else {
            ApiCode.ERROR_UNKNOWN
        }
        return ApiException(apiCode.code, apiCode.description)
    }
}
```

如此，则实现了对API接口的封装，解决了上面提到的几个问题。
下面说一下上传头像到阿里云的业务及代码实现。

---

<br>

# 上传文件到阿里云

阿里云文件服务是单独的服务，需要另外写接口类和实现接口的服务类，但阿里文件服务有自己的接口API，因此不需要写接口类，只需要写一个服务类封装即可，如下。

阿里云文件服务
```kotlin
object AliOSSService {
    private val clientConfiguration = config()

	// 基础配置
    private fun config(): ClientConfiguration {
        val conf = ClientConfiguration()
        conf.connectionTimeout = 15 * 1000 // 连接超时，默认15秒
        conf.socketTimeout = 15 * 1000 // socket超时，默认15秒
        conf.maxConcurrentRequest = 5 // 最大并发请求数，默认5个
        conf.maxErrorRetry = 2 // 失败后最大重试次数，默认2次
        return conf
    }

	// 上传文件，并改造API为Rxjava格式
    fun uploadFile(context: Context, stsModel: AliOSSSTSModel, fileData: ByteArray): Flowable<ApiResponse<Boolean?>> {
        val credentialProvider = OSSStsTokenCredentialProvider(
                stsModel.credentials.accessKeyId,
                stsModel.credentials.accessKeySecret,
                stsModel.credentials.securityToken)
        val oss = OSSClient(context, stsModel.endpoint, credentialProvider, clientConfiguration)
        val put = PutObjectRequest(stsModel.bucket, stsModel.objectKey, fileData)
        return Flowable.create({ subscriber ->
            oss.asyncPutObject(put, object : OSSCompletedCallback<PutObjectRequest, PutObjectResult> {
                override fun onSuccess(request: PutObjectRequest?, result: PutObjectResult?) {
                    subscriber.onNext(ApiResponse(ApiCode.SUCCESS.code, true, "success"))
                    subscriber.onComplete()
                }

                override fun onFailure(request: PutObjectRequest?, clientException: ClientException?, serviceException: ServiceException?) {
                    subscriber.onError(ApiException(
                            code = if (serviceException == null) "10" else serviceException.errorCode,
                            message = serviceException?.rawMessage
                    ))
                }
            })
        }, BackpressureStrategy.LATEST)
    }
}
```


在WrapNetApi里进一步封装，增加Test服务相关请求

```kotlin
	// 上传头像。链式调用，先从Test服务获取上传头像需要的凭证，之后上传头像到阿里云服务，上传成功后通知Test服务
    fun uploadAvatar(context: Context, avatarData: ByteArray): Flowable<ApiResponse<Boolean?>> {
        return TestService.api()
                .getAliOSSSTS(token(),"WRITE", "OSS") // 获取凭证
                .flatMap { response ->
	                // 上传头像
                    AliOSSService.uploadFile(context, response.data!!, avatarData)
                }
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.io())
                .flatMap {
	                // 通知“上传成功”
                    TestService.api().notifyAvatarUpdate(token())
                }
                .compose(apply())
    }
```