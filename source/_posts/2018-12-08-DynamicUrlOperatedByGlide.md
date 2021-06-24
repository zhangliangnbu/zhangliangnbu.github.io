---
title: 动态Url过期处理
date: 2018-12-08 19:43:00
tags:
- Android
- URL
- Glide
categories:
- Android开发
---

# 场景描述

之前做项目时遇到这样的场景，我们的用户头像保存在阿里云OSS上，涉及到动态URL以及过期处理的问题。

**上传头像的逻辑**

1. 首先从APP服务端获取Token
2. 通过Token上传图片到阿里云OSS
3. 成功后通知APP服务端。

如下图所示，其中省略了APP服务端和阿里云OSS的交互。
![APP上传头像流程](https://upload-images.jianshu.io/upload_images/2658578-45788c6866229c0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

**获取头像逻辑**

1. 从APP服务端获取URL链接。
2. 根据URL链接获取图片。如果本地有缓存且没有过期则从本地拉取缓存，不然就从阿里云OSS获取


**动态的URL**
由于头像是私密的，每次获取用户头像时，url中会有一个动态的Token，如

>https://xxx.aliyuncs.com/avatar/xxx-d-avatar?token=yyyy&v=1523326538071

其中，阿里云OSS识别的是https://xxx.aliyuncs.com/avatar/xxx-d-avatar?token=yyyy， *v=1523326538071*是APP服务端为了控制头像版本而附加的字段。



**问题**

1. 需要缓存头像。当前用户的头像URL中，token的值“yyyy”每次都会改变，因此做缓存的时候，不能直接使用URL作为key，需要过滤掉Token。
2. 需要定期让缓存失效。阿里云OSS上保存的URL只是https://xxx.aliyuncs.com/avatar/xxx-d-avatar?token=yyyy 部分，因此有这样的问题，如果APP客户端上传头像成功，但由于某种原因（比如服务崩溃了）APP服务端并没有成功更新版本，即v的值，则下次APP客户端从服务端获取的仍然是旧的版本值，APP客户端会取本地缓存，而不向阿里云OSS拉取图片，虽然这种问题出现的概率较小，但不得不考虑到。

---

<br>

# 解决方法

项目使用了[Glide](https://github.com/bumptech/glide)图片框架。具体代码如下

**动态URL处理类，去掉动态Token**

```kotlin
class GlideDynamicAvatarUrl(private val dynamicUrl: String) : GlideUrl(dynamicUrl) {
    companion object {
        private const val IMAGE_VERSION = "v"
    }
    override fun getCacheKey(): String {
        val uri = Uri.parse(dynamicUrl)
        return StringBuilder()
                .append(uri.path)
                .append(uri.getQueryParameter(IMAGE_VERSION))
                .toString()
    }

    override fun toString(): String {
        return super.getCacheKey()
    }
}
```

**定期失效处理，添加额外的key**

```kotlin
object GlideAvatarHelper {
    private const val INVALID_TIME = 7 * 24 * 3600 * 1000L
    fun execute(context:Context, url: String?, iv: ImageView) {
        if (TextUtils.isEmpty(url)) {
            return iv.setImageResource(R.drawable.ic_avatar_default)
        }

        Glide.with(context)
                .load(GlideDynamicAvatarUrl(url!!))
                .apply(RequestOptions.signatureOf(ObjectKey(getSignatureKey())))
                .apply(RequestOptions.placeholderOf(R.drawable.ic_avatar_default))
                .apply(RequestOptions.errorOf(R.drawable.ic_avatar_default))
                .into(iv)
    }

    // 为URL添加额外的key, 定期让key失效即可
    // 这里设置的有效期是一周时间
    private fun getSignatureKey(): Long {
        val key = System.currentTimeMillis() / INVALID_TIME
        return key
    }
}
```