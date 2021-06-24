---
title: Android草稿
date: 2019-05-05 10:22:02
tags:
- Android
- 草稿
categories:
- Android开发
---

**文件上传和下载**

包括。分片上传、分片下载、断点续上传、断点续下载。<br>原理。利用HTTP头 Range和Content-Range字段。

注意点：

- 断点续传。判断文件是否改变，ETag头来放置文件的唯一标识，如文件的MD5值。

参考

- [文件分片与断点续传原理与具体实现]([http://lcodecorex.github.io/2016/08/01/%E6%96%87%E4%BB%B6%E5%88%86%E7%89%87%E4%B8%8E%E6%96%AD%E7%82%B9%E7%BB%AD%E4%BC%A0%E5%8E%9F%E7%90%86%E4%B8%8E%E5%85%B7%E4%BD%93%E5%AE%9E%E7%8E%B0/](http://lcodecorex.github.io/2016/08/01/文件分片与断点续传原理与具体实现/))
- [android-upload-service](https://github.com/gotev/android-upload-service)
- [okdownload](https://github.com/lingochamp/okdownload)