---
title: 开发笔记-理解FileProvider
date: 2018-10-21 11:53:40
updated: 2018-10-21 17:52:48
tags:
- Android
- FileProvider
categories:
- Android开发
---



# 介绍

用于更安全地在APP之间实现文件共享的一种机制，适用于Android 7.0及以上。

当应用A（如相册）允许其他应用（以应用B为例）调用自己并获取自己的文件时，应用A需要使用`FileProvider`来转化自己的文件路径为`content://Uri`，并提供给需要文件的应用B。其中

1. Android 7.0以上如果不使用`FileProvider`，应用A将返回`file:///` `Uri`，而报错。
2. 返回的`content://Uri`有生命周期，即应用B接收文件的`Activity`的生命周期，如果这个`Activity`被销毁了，则返回的`content://Uri`也将失效而不能获取文件。

-- -- --

<br>



# 场景-从相册获取图片

这是经常遇到的场景。当我们的APP（开发者APP）需要从相册获取图片时，一般使用如下代码

```kotlin
    private fun getPicture() {
        val intent = Intent()
        intent.type = "image/*"
        intent.action = Intent.ACTION_GET_CONTENT
        intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true)
        startActivityForResult(Intent.createChooser(intent, "Select Picture"), REQUEST_CODE_ADD_PICTURE)
    }
```

并在`onActivityResult`处理获取到Uri，但需要记住的是，这个Uri是相册应用使用`FileProvider`处理后的`content://Uri`，其获取图片的权限是暂时的。

这个场景中，是开发者APP从相册获取图片，无需在开发者APP中配置`FileProvider`。

-- -- --

<br>



# 场景-拍照

开发者APP调用相机拍照并获取照片，官方文档[TakePhoto](https://developer.android.com/training/camera/photobasics#kotlin)对这一块有详细的说明。

开发者APP调用相机拍照后需要保存在一个文件中，这个文件地址由开发者APP规定。也就是说，相机在拍照后需要读写开发者APP规定的文件，那么这个时候，开发者APP就需要提供文件共享功能，这就需要用到`FileProvider`了。

在这个场景中，开发者APP中需要配置`FileProvider`。

-- -- --

<br>



# 参考

1. [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider)
2. [TakePhoto](https://developer.android.com/training/camera/photobasics#kotlin)