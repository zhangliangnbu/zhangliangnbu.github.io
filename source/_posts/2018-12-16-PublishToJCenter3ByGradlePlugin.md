---
title: 项目发布-BintrayRelease
date: 2018-12-16 22:52:56
tags:
- maven
- jcenter
- bintray
categories:
- 其他
---

# 前言

使用Gradle插件上传Android项目到Bintray平台是目前通用的做法，很方便。目前常用的Gradle插件有两个，一个是官方的[gradle-bintray-plugin](https://github.com/bintray/gradle-bintray-plugin#readme)，另一个是第三方开源的[bintray-release](https://github.com/novoda/bintray-release)。既然官方已经发布了自己的Gradle插件，那为什么还有人发布另外一个呢？可能因为官方自己的插件使用起来比较繁琐吧。

这篇文章简要介绍如何使用[bintray-release](https://github.com/novoda/bintray-release)发布Android项目到Bintray，并最终发布到JCenter。

bintray-release使用起来非常简单，具体详情请见[bintray-release wiki](https://github.com/novoda/bintray-release/wiki)。

**源码地址**。本文涉及到的`nicelogger`项目Github地址：https://github.com/zhangliangnbu/nice-logger

---

# 准备工作

参考上一篇文章，如果已经做了，可以跳过。默认你已经有了一个本地项目，已经创建了Bintray平台账号和Maven仓库。

**定义参数**

- Bintray平台仓库名称。`android`。
- Bintray平台Package名称。`nicelogger`。
- POM文件`groupId`。`com.liang.android`。
- POM文件`artifactId`。`nicelogger`。
- POM文件`version`。取`0.0.3`。

**准备本地项目**。有的话就不用创建。

**配置Bintray平台**。创建package，如果已经有了就不用创建了。

---

# 发布到Bintray

这一步使用插件，做如下工作：

* 在本地生成构件文件。
* 在Bintray平台创建版本。
* 上传文件到Bintray平台。
* 发布到Bintray平台仓库中。

具体使用请见[bintray-release wiki](https://github.com/novoda/bintray-release/wiki)，我的配置与wiki略有差异，本质上是一样的。

一，在工程目录`build.gradle`中添加插件地址，其中版本号请用最新的：

```groovy
buildscript {
    dependencies {
        // A helper for releasing from gradle up to bintray
        classpath 'com.novoda:bintray-release:0.9'
    }
}
```

二，在`nicelogger`module目录`build.gradle`中添加参数配置：

```groovy
apply plugin: 'com.novoda.bintray-release'

publish {
    userOrg = 'zhangliang'
    repoName = 'android'
    groupId = 'com.liang.android'
    artifactId = 'nicelogger'
    publishVersion = '0.0.3'
    uploadName = 'nicelogger'
    desc = 'Oh hi, this is a nice description for nicelogger, right?'
    website = 'https://github.com/zhangliangnbu/nice-logger'
}
```

> 可以在module目录中创建`bintrayReleaseUpload.gradle`文件，并将上述参数配置写入其中，然后在module目录`build.gradle`中通过`apply from: './bintrayReleaseUpload.gradle'`引入。这样做便于管理。

三，执行上传任务

```bash
./gradlew clean build bintrayUpload -PbintrayUser=BINTRAY_USERNAME -PbintrayKey=BINTRAY_KEY -PdryRun=false
```

>BINTRAY_USERNAME和BINTRAY_KEY填写自己的。

上传完后，即可在Bintray平台`nicelogger`包下看到发布的`0.0.3`版本。

从Bintray仓库发布到JCenter操作较简单，见上篇文章。

----

# 参考

1. [bintray-release](https://github.com/novoda/bintray-release)
2. [bintray-release wiki](https://github.com/novoda/bintray-release/wiki)
3. [bintray-release中文文档](https://github.com/novoda/bintray-release/wiki/%E4%B8%AD%E6%96%87%E6%96%87%E6%A1%A3HOME)
4. [gradle-bintray-plugin](https://github.com/bintray/gradle-bintray-plugin#readme)

---

