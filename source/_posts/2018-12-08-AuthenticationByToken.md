---
title: 使用Token进行身份验证
date: 2018-12-08 17:20:50
tags:
- android
- authentication
- token
categories:
- Android开发
---

# 概述

客户端中有些数据是需要登录后才能查看的，这时候就需要在接口中加入凭证，如Cookie 或Token。我们这里主要说下Token。更多关于身份验证的详细情况请见参考链接。

[身份验证的几种方案](https://www.jianshu.com/p/eaf9197abb6b)

> 典型的用户身份验证标准（方案）：
> HTTP BASIC Authentication
> HTTP Digest Authentication
> Form-based Authentication
> Token Based Authentication
> X.509 Certificate Authentication

[Token/Session/Cookie的区别](https://www.jianshu.com/p/5ac166c5fe76)

>其中Token身份验证如下:
>1）客户端使用账号密码请求登录
>2）服务端收到请求，验证账号密码
>3）验证通过，服务端签发一个token给客户端
>4）客户端收到token存储起来（例：存在cookie）
>5）客户端每次请求服务端时带着服务端签发的token
>6）服务端收到请求，验证token。验证成功则返回数据给客户端。

----

<br>

# 具体实践

## 认证逻辑

1. 客户端在登录页面执行登录请求，服务端通过后返回Token
2. 客户端本地存储Token，并且在需要身份认证的接口中添加Token(一般放在header里)
3. 服务端对每个Token赋予一个有效期，过了有效期之后，返回给客户端一个特殊的错误码，让其重新登录或者其他的方式刷新Token(如通过RefreshToken刷新Token)

>注意: 以下代码使用了Retrofit和Rxjava框架

## 登录接口

1. 执行加盐请求。利用盐加密密码，盐和密码拼接后使用SHA512加密。
2. 执行登录请求。

```kotlin
fun loginWithSalt(mobile: String, password: String): Flowable<ApiResponse<TokenModel?>> {
        return BossService.api()
                .getSalt() // 获取盐
                .flatMap { response ->
                    val enPassword = PasswordEncryptHelper.getWhenLogin(password, response.data!!.salt) // 加密
                    val map = hashMapOf(
                            Pair("mobile", mobile),
                            Pair("saltKey", response.data.saltkey),
                            Pair("password", enPassword))
                    return@flatMap BossService.api().login(map) // 登录
                }
                .compose(apply())
    }
```

## 登录操作及更新Token

```kotlin
private fun attemptLogin(mobile:String, password:String) {
        NetApi.loginWithSalt(mobile, password)
                .subscribe({ data ->
                    loginSuccess(data.data!!, mobile)
                }, { t ->
                    loginFail(t, mobile)
                })
    }

    private fun loginSuccess(any: TokenModel, mobile: String) {
	    // 更新当前用户信息
        DataManager.updateCurrentUser(mobile, UserModel(mobile))
        // 更新Token
        DataManager.currentUserSettings()!!.setToken(any.token)
		// 更新refreshToken。不过我们没有使用refreshToken,当Token过期后我们会提示用户去登录。        
        DataManager.currentUserSettings()!!.setRefreshToken(any.refreshToken)
    }

// TokenModel实体
class TokenModel(val token: String,
                 val refreshToken: String) : BaseModel()
```

## 需要身份认证的接口

```kotlin
	// TestApi中。接口，token放在header的"X-Authorization"字段中
    @GET("users/v1/baseInfo")
    fun getUserBaseInfo(@Header("X-Authorization") token: String
    ): Flowable<ApiResponse<UserModel>>
	
	// WrapTestApi中。
    fun getUserBaseInfo(): Flowable<ApiResponse<UserModel?>> {
        return BossService.api()
                .getUserBaseInfo(token())
                .compose(apply())
    }

	// WrapTestApi中
    private fun token(): String {
        val currentUser = DataManager.currentUserSettings()
        val token = currentUser?.getToken() ?: ""
        return "Bearer ".plus(token) // 一般不直接传递Token，需要在前面加点盐，为了简单起见，我们的项目固定写了一个字符串。
    }
```

## Token过期处理

```kotlin
// 在需要的地方调用
private fun fetchRemoteData(): Disposable {
        val loading = LoadingHelper.create(context!!)
        return NetApi.getUserBaseInfo()
                .compose(RxTransformers.applyLoading(loading))
                .subscribe({ user ->
	                // deal for success 
                }, { t ->
	                // deal for fail. 先判断是否认证失败
                    if (!NetErrorHelper.handleAuth(t, activity!!)) {
                        Util.toast().showShort("fail:${t.message}")
                    }
                })
    }

/**
 * 处理网络返回统一的错误码
 */
object NetErrorHelper {

    /**
     * @return true则进行认证失败处理；false则非认证错误
     */
    fun handleAuth(t: Throwable, activity: Activity): Boolean {
        if (t !is ApiException) {
            return false
        }
        when (t.code) {
	        // Token过期，弹出对话框去登录
            ApiCode.ERROR_JWT_TOKEN_EXPIRED.code -> {
                Util.dialog().warn(activity)
                        .content(t.message ?: ApiCode.ERROR_AUTHENTICATION_FAILED.description)
                        .positiveText(R.string.common_confirm)
                        .onPositive { _, _ ->
                            ARouter.getInstance().build(RouteTable.LOGIN).navigation()
                            activity.finish()
                        }
                        .cancelable(false)
                        .show()
                return true
            }
            else -> {
            }
        }
        return false
    }
}
```



----

<br>

# 参考

1. [身份验证的几种方案](https://www.jianshu.com/p/eaf9197abb6b)

2. [Token/Session/Cookie的区别](https://www.jianshu.com/p/5ac166c5fe76)
