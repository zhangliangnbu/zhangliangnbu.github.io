---
title: Android构建06-Android应用构建基础
date: 2018-10-24 17:29:48
tags:
- Android
- Gradle
- Build
categories:
- Android构建系统
---



# 简介

Android构建流程是指将Android源代码转换成Apk（Android Application Package）这一过程，里面涉及到许多步骤和[工具](https://developer.android.com/studio/command-line/)。构建流程由Gradle和Android Gradle Plugin插件来管理，既可以通过IDE方式，也可以通过命令行方式来进行。

-- -- --

<br>



# 构建流程

官网有一个简单的[流程图](https://developer.android.com/studio/build/#build-process)，比较简略，图中主要展示了两步，且都是由Gradle和Andorid插件管理的：

1. 编译。编译器将源代码编译成DEX文件和供前者直接使用的资源文件。
2. 打包并签名。将上述DEX文件和资源文件合并成APK，之后进行签名生成最终可以用于安装、测试或发布的APK。



在许多博客中，都提到一个详细的流程图，说是来自官网，但我没有找到，可能以前存在，现在已经删除了，如下：

![Android构建流程](/images/android_build_process_detail.png)

上述流程分为七个步骤，涉及许多工具，如AAPT、AIDL等，有些工具现在已经升级或改变名称了，如AAPT现在升级为AAPT2。

> AAPT等工具位于{Android SDK}/build-tools/{版本号}。

七个步骤具体如下：

**1. 编译资源文件**

利用`aapt`编译资源文件生成`R.java`和`resources.arsc`

```bash
# R.java
$ aapt p -f -m -J app/out/R \
-M app/src/main/AndroidManifest.xml \
-S app/src/main/res \
-I $SDK/platforms/android-28/android.jar 
```

```bash
# 生成包含resources.arsc的apk，之后添加dex文件
$ aapt p -f -F app/out/res/resource.apk \
-M app/src/main/AndroidManifest.xml \
-S app/src/main/res \
-I $SDK/platforms/android-28/android.jar 
```

> 说明:
>
> * 上面的示例工程是利用Android Studio生成的，并去掉了自动引入的第三方依赖。
> * 需要生成必要的文件夹`$ mkdir -p app/out/R`和`$ mkdir -p app/out/res`
> * `\`表示命令行换行。
> * `$SDK`可用命令`$ export SDK=~/Library/Android/sdk`设置。
> * 命令的参数说明请见[Android AAPT](https://elinux.org/Android_aapt)，或`$ aapt`查看输出信息。
> * 如果有第三方依赖，用-I继续添加， 可参考[命令行编译Android](https://amorypepelu.github.io/2016/02/10/bash-Compile-Android/)。但我添加了com.android.support.constraint依赖，并用-I添加进命令行，执行命令后报错。



**2. 编译AIDL文件**

略



**3. 编译Java文件**

利用`javac`编译`.java`文件为`.class`文件

```bash
$ javac -target 1.8 -source 1.8 \
-bootclasspath $SDK/platforms/android-28/android.jar \
-d app/out/class \
app/out/R/com/liang/clidemo/R.java app/src/main/java/com/liang/clidemo/*.java 
```

> 生成`class`文件夹`$ mkdir app/out/class`



**4. 生成Dex文件**

```bash
$ dx --dex --output=app/out/dex app/out/class
```

> 需要生成`dex`文件夹`$ mkdir app/out/dex`



**5. 打包生成apk**

```bash
$ java -classpath $SDK/tools/lib/sdklib-26.0.0-dev.jar com.android.sdklib.build.ApkBuilderMain app/out/demo.apk \
-v -u -z app/out/res/resource.apk \
-f app/out/dex/classes.dex
```



**6. 签名**

签名debug版本的秘钥

```bash
$ jarsigner -verbose -keystore ~/.android/debug.keystore \
-storepass android \
-keypass android \
app/out/demo.apk androiddebugkey
```



**7. 对齐**

略



以上七个命令可以用来手动打包，但比较繁琐，而Gradle将这些命令封装起来作为任务，只需要执行`$ ./gradlew assemleDebug`等即可，提高了工作效率。当然了，Gradle的功能不止如此。

执行`./gradlew assembleDebug --console=plain`，可以看Gradle的任务执行顺序与用命令行打包顺序基本是一致的：

```bash
:app:checkDebugClasspath UP-TO-DATE
:app:preBuild UP-TO-DATE
:app:preDebugBuild UP-TO-DATE
# 编译aidl
:app:compileDebugAidl NO-SOURCE
:app:compileDebugRenderscript UP-TO-DATE
:app:checkDebugManifest UP-TO-DATE
# 编译BuildConfig.java
:app:generateDebugBuildConfig UP-TO-DATE
:app:prepareLintJar UP-TO-DATE
:app:mainApkListPersistenceDebug UP-TO-DATE
# 生成R.java和res资源文件
:app:generateDebugResValues UP-TO-DATE
:app:generateDebugResources UP-TO-DATE
:app:mergeDebugResources UP-TO-DATE
:app:createDebugCompatibleScreenManifests UP-TO-DATE
:app:processDebugManifest UP-TO-DATE
:app:splitsDiscoveryTaskDebug UP-TO-DATE
:app:processDebugResources UP-TO-DATE
:app:generateDebugSources UP-TO-DATE
:app:javaPreCompileDebug UP-TO-DATE
# javac编译java文件为class文件
:app:compileDebugJavaWithJavac UP-TO-DATE
:app:compileDebugNdk NO-SOURCE
:app:compileDebugSources UP-TO-DATE
:app:mergeDebugShaders UP-TO-DATE
:app:compileDebugShaders UP-TO-DATE
:app:generateDebugAssets UP-TO-DATE
:app:mergeDebugAssets UP-TO-DATE
# 生成dex文件
:app:transformClassesWithDexBuilderForDebug UP-TO-DATE
:app:transformDexArchiveWithExternalLibsDexMergerForDebug UP-TO-DATE
:app:transformDexArchiveWithDexMergerForDebug UP-TO-DATE
:app:mergeDebugJniLibFolders UP-TO-DATE
:app:transformNativeLibsWithMergeJniLibsForDebug UP-TO-DATE
:app:transformNativeLibsWithStripDebugSymbolForDebug UP-TO-DATE
:app:checkDebugLibraries UP-TO-DATE
:app:processDebugJavaRes NO-SOURCE
:app:transformResourcesWithMergeJavaResForDebug UP-TO-DATE
:app:validateSigningDebug UP-TO-DATE
# 打包
:app:packageDebug UP-TO-DATE
:app:assembleDebug UP-TO-DATE
```



-- -- --

<br>



# Android Plugin DSL Reference

[Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/index.html)详细描述了Android module构建脚本`build.gradle`所用到的参数。

包括四个扩展内容，下面以应用型项目的`build.gradle`为例，说下`AppExtension`的语法。

简单的应用型项目的`build.gradle`脚本：

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.liang.clidemo"
        minSdkVersion 21
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

解释如下：

* `Project`实例的三个方法。最外层的`apply`、`android`和`dependencies`都是`Project`实例的方法。
* `apply`方法。全称为`void apply(Map<String, ?> options)`，位于接口`PluginAware.java`中（`Project`实现了`PluginAware`），用于增加插件。示例中表示当前项目为应用型项目。
* `android`方法。原始的`Project`中并没有这个方法，应该位于Android插件的`Project`的实例中，可以看源码。闭包里的参数设置详细说明位于Reference的`AppExtension`中
  * `ompileSdkVersion`。编译版本。很奇怪的是`AppExtension`中说它是`String`类型的，但如果赋值为字符串，则又报错。
  * `defaultConfig`。接收`DefaultConfig`作为参数，其闭包里的参数说明见Reference的[DefaultConfig](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.DefaultConfig.html)
  * `buildTypes`。闭包里的参数设置请见[BuildType](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html)
* `dependencies`方法。位于`Project`中`void dependencies(Closure configureClosure);`，参数说明详见闭包的代理[DependencyHandler](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html)



:p，还是看源码比较直接。

-- -- --

<br>



# 待研究

* `aapt`命令中添加第三方依赖问题
* `aapt2`命令
* Android构建脚本配置。[官网-Configure your build](https://developer.android.com/studio/build/)写的比较详细，目前不准备写。
* Android构建最佳实践。[官网-Configure your build](https://developer.android.com/studio/build/)上写了许多，以后可以做些补充。
* Gradle和Android Plugin源码阅读和分析。非常值得研究，但需要花费很多时间，以后有时间加上。



-- -- --

<br>

# 参考

1. [官网-Configure your build](https://developer.android.com/studio/build/)
2. [Android APK 编译打包流程](https://segmentfault.com/a/1190000008071324)
3. [APK打包安装过程](https://segmentfault.com/a/1190000004916563)
4. [详细打包流程图](https://cdn-images-1.medium.com/max/1600/1*LbKGyabN6vr-9iR-rvnzPg.png)
5. [命令行编译Android](https://amorypepelu.github.io/2016/02/10/bash-Compile-Android/)
6. [How to make Android apps without IDE from command line](https://medium.com/@authmane512/how-to-build-an-apk-from-command-line-without-ide-7260e1e22676)
7. [Android AAPT](https://elinux.org/Android_aapt)
8. [AAPT2官方文档](https://developer.android.com/studio/command-line/aapt2)
9. [aapt2 资源 compile 过程](https://fucknmb.com/2017/10/31/aapt2%E8%B5%84%E6%BA%90compile%E8%BF%87%E7%A8%8B/)
10. [官网-Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/)
11. [Android Gradle Plugin 源码阅读与编译](https://fucknmb.com/2017/06/01/Android-Gradle-Plugin%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B8%8E%E7%BC%96%E8%AF%91/)