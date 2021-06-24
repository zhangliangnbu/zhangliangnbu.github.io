---
title: ZeroCICE使用初步
date: 2018-12-08 22:25:42
tags:
- ZeroC
categories:
- 其他
---

# 概述

最近编译一个行情Android APP，里面使用到[ZeroC ICE](https://zeroc.com/)工具，将ice文件转换成Java文件(当然不止于此)。初次使用，做一点总结。

<br>

# macOS上安装

[利用homebrew安装最新版本](https://zeroc.com/downloads/ice#macos)
```bash
brew install ice
```
由于要编译的代码很老旧，使用的ice需要3.6.3版本的，但我根据官网文档，只能安装3.6.4的，最后也能跑的通
```bash
brew uninstall ice // 首先卸载之前最新版本
brew install zeroc-ice/tap/ice@3.6 // ice
brew install zeroc-ice/tap/icetouch@3.6  //ice touch  是为iphone和ipod touch开发的版本，开发Android的话可以不用安装
```
<br>

# Android项目配置

[具体可以见官网 using-ice-on-android](https://doc.zeroc.com/ice/3.7/release-notes/using-ice-on-android)
我自己的相关配置如下

```gradle
// 项目根目录gradle文件
buildscript {
    repositories {
        google()
        jcenter()
        maven {
            url 'https://repo.zeroc.com/nexus/content/repositories/releases'
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.2'
        classpath group: 'com.zeroc.gradle.ice-builder', name: 'slice', version: '1.3.14'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        maven {
            url 'https://repo.zeroc.com/nexus/content/repositories/releases'
        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
<br>

```gradle
// 应用的gradle文件
apply plugin: 'com.android.application'
apply plugin: 'slice'
slice {
    java {
        srcDir = '.'
        // 这是ice文件中需要引入的文件，可以拉出来放在工程目录，我这里图方便就这样写了
        include = ["/usr/local/Cellar/ice@3.6/3.6.4_1/share/Ice-3.6.4/slice/"] 
    }
}
// 这是ice文件转成Java文件后的输出目录
project.slice.output = project.file("${project.buildDir}/generated/source/ice")

android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "com.slm.tradeclient"
        minSdkVersion 16
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    sourceSets {
        main {
            java {
                // 这是ice文件的源目录，也就是把这个目录里的ice文件转成Java文件
                srcDirs = ['src/main/java', project.slice.output]
            }
        }
    }
    buildTypes {
        debug {
            debuggable true
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation 'com.zeroc:ice:3.6.4'
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    //  ...
}

```

在Gradle Projects里
1. 先编译slice。->compileSlice
2. 然后生成应用项。->assembleDebug

最后安装无误，顺利通过。

----

<br>

# 参考

1. [中文简单介绍](https://baike.baidu.com/item/ZeroC%20Ice)
2. [官网](https://zeroc.com/)
3. [某个博客](https://blog.csdn.net/robertaqi/article/details/5900695)