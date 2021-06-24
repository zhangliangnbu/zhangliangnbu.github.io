---
title: Android集成TBS填坑记录
date: 2019-11-15 09:01:50
tags:
- Android
- tbs
- x5
- h5
categories:
- Android开发
---

#前言

最近得空，记录下这几月Android开发中遇到的坑。

这篇文章主要记录集成腾讯浏览服务（TBS Tencent Browse Server）时遇到的问题以及解决方案。

---

# 问题：H5加载异常

**具体情况**

我在公司内网环境下，使用几个测试机，有的可以正常加载，有的则加载失败。我的接入步骤皆是按照官方接入文档以及Demo做的，检查了几遍，认为接入是没有问题的。

**解决**

结论：一是换外网；二是在入口Activity里请求外部读写权限并初始化`QbSdk.initX5Environment(context, callback)`。

分析：要明白TBS在初始化的时候做的事情。Tbs sdk有一个初始化的方法`QbSdk.initX5Environment(context, callback)`，按照官方Demo，是放置于`Application.onCreate()`中。这个方法首先会检测到sdcard里是否有x5内核所在的文件夹，有的话就会加载内核，如果文件夹为空则加载失败，总之不会去下载 内核；如果检测到没有相关文件夹，则会下载内核。一般安装了腾旭系的APP，在sdcard里都会有相对应的内核及文件夹。

x5内核的下载走外网，因此用公司内网测试时是不可能下载成功的。之前有的机器能在测试环境下通过，是因为其sdcard中已经存在内核了，或者是之前下载的，或者是由于安装了其他腾讯系APP。

在外网下测试，部分测试机依旧偶尔不能加载x5内核。这是因为初始化是在`Application.onCreate()`中的，此时并没有读写权限，所以每次都要下载，但完全下载成功需要一定的时间，因此如果快速打开H5页面，此时x5内核没有完全下载完成，就会出现加载失败。

---



# 参考

- [TBS官网](https://x5.tencent.com/)