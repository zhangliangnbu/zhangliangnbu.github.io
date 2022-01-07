---
title: Android构建07-进阶
date: 2019-05-15 14:51:42
tags:
- android
- build
- gradle
categories:
- Android开发
---

# 脚本和对象

构建时，脚本文件会转化为对象；写脚本时不懂的地方查对应对象API即可。

- `settings.gradle`会转化为`Settings`对象。
- `build.gradle`会转化为`Project`对象。
- 其他gradle文件，没有定义为class的，会转换为一个实现了`Script`接口的对象

<!-- more -->

# 任务

可以使用方法和关键字定义。

```groovy
// 关键字定义
task myTask
task myTask { configure closure } // 重点
task myTask(type: SomeType)
task myTask(type: SomeType) { configure closure } // 重点
 
// 方法定义
Task task(String name)
Task task(String name,Closure configureClosure)
Task task(String name,Action<? super Task> configureAction)
Task task(Map<String,?> args,String name)
Task task(Map<String,?> args,String name,Closure configureClosure)
```



# 常用属性

`Project`和`Settings`的常用属性

**扩展属性（Ext属性）**

```groovy
// 定义
[project.]ext.foo = "bar"
[project.]ext {
		foo = "bar"
} // 命名空间方式定义 ？？

// 使用
println project.ext.foo
println rootProject.ext.foo

// 范围：有穿透性？gradle 4.10.* 没有穿透性。即rootProject.foo != projet.foo
```



**文件`gradle.properites`属性**

可以直接被`Project`/`Settings`使用

```groovy
// gradle.properties中
test="test hw"

// build.gradle中
assert project.test == "test hw"
assert settings.test == "test hw"
```



# 应用脚本

```groovy
// 属性
def version = [test: 1.0]
def deps = [test : "com.example.test:test:${version.test}"]
ext.deps = deps

// 方法
def printHW() {
  println 'hello world !!!'
}
ext.printHW = this.&printHW

// build.gradle中使用
apply from: rootProject.getRootDir().getAbsolutePath() + '/deps.gradle'
println deps.test
printlnHW()
```



读取属性文件







# 修改Artifact文件目录和名称

在gradle 4.10和android gradle tools 3.3.*及以上

```groovy
// 通过 androd.applicationVariants.all {} 获取  ApplicationVariant
// ApplicationVariant获取生成物目录属性API，直接赋值即可修改
File dir = ApplicationVariant.getPackageApplicationProvider().get().outputDirectory

// 通过ApplicationVariant.getOutputs().each {} 获取VariantOutput
// VariantOutput获取生成物文件名称属性，直接赋值即可修改
String name = VariantOutput.outputFileName
```



完整代码，在module的`build.gradle`脚本文件中添加如下代码：

```groovy
android {
		...
    applicationVariants.all { variant ->
        refactorVariant(variant)
    }
}

import com.android.build.gradle.api.ApplicationVariant
// 更改生成物目录和名称
// gradle 4.10 and android-gradle-tools 3.3
def refactorVariant(ApplicationVariant variant) {
    // 获取目录
    println "old dir -> " + variant.getPackageApplicationProvider().get().outputDirectory.getAbsolutePath()
    // 修改目录
    variant.getPackageApplicationProvider().get().outputDirectory = new File("${project.projectDir.getPath()}/apk/${variant.getBuildType().name}")
    // 输出修改后的目录
    println "new dir -> " + variant.getPackageApplicationProvider().get().outputDirectory.getAbsolutePath()

    variant.getOutputs().each { output ->
        // 获取名称
        println "old name -> " + output.outputFileName
        if (output.outputFileName.endsWith('.apk')) {
            // 修改名称
            output.outputFileName = "GradleDemo_${variant.getBuildType().name}_${variant.getVersionName()}.apk"
            // 输出修改后的名称
            println "new name -> " + output.outputFileName
        }
    }
}
```





# 参考

- [深入理解Android之Gradle](https://blog.csdn.net/innost/article/details/48228651#t14)
- [Gradle的属性设置大全](http://www.huangbowen.net/blog/2013/09/12/setup-properties-in-gradle/)

