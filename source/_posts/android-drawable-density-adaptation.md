---
title: Drawable分辨率适配
date: 2019-01-15 15:10:24
tags:
- Android
- drawable
categories:
- Android开发
---

# 问题

定义了不同的`drawable-***dpi`的文件夹，但只在其中某些里面放置了图片，比如在`drawable-xhdpi`里放置了图片，其他的文件夹里没有放置图片，那么非xhdpi的设配如何适配图片资源？如何定位图片以及如何显示？

# 图片选择规则

当前->高分辨率->低分辨率。

- 先找当前分辨率的drawable文件夹，没有则进行下一步；
- 向上寻找高于当前分辨率的drawable文件夹，没有则进行下一步；
- 向下寻找低于当前分辨率的drawable文件夹。没有？不可能通过编译。

# 图片显示缩放规则

设备上显示的图片尺寸 =（设配分辨率 / 适配图片定义分辨率）* 适配图片尺寸。

示例。图片尺寸为144x144，位于`drawalbe-xxhdpi/ic_wechat`，在Nexus 6（1440 x 2560， 560dpi）显示尺寸为168 x 168，即 （560/480）* 144。

> Android Developer Guide对此语焉不详，以上结论来自我的实践，可写DEMO验证。
>
> 另外，mipmap资源适配规则与drawable资源保持一致。

------

# 参考

- [Android Developer Guide - Support different pixel densities](https://developer.android.com/training/multiscreen/screendensities)