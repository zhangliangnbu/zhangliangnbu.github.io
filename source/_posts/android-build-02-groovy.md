---
title: Android构建02-Groovy基础
date: 2018-08-27 17:52:09
updated: 2018-10-17 14:52:48
tags:
- android
- build
- gradle
- groovy
categories:
- Android开发
---
# 前言
Groovy是JVM语言，与Java语法类似，如果你熟悉kotlin的话，你会发现它与kotlin更类似些。它可以作为Java平台的脚本语言使用。详细介绍请见[Groovy官网](http://groovy-lang.org/index.html)和[维基百科](https://zh.wikipedia.org/wiki/Groovy)。

>提示：“Groovy基础”这一部分建议不要花费太多时间，能看懂语法，尤其是闭包的语法，以及知道如何查阅API即可。

<!-- more -->

# 语法

对于语法的学习，我这里就不详细说了，请大家按照下面步骤去学习：
1. 浏览一遍官方文档的[语言规范](http://www.groovy-lang.org/documentation.html)，里面有示例，很容易看懂。要知道Groovy语言规范大致讲了哪些内容，以后遇到不懂的语法，可以回来快速查阅。
2. 另外，也可以看下国人写的入门博客：[任玉刚 Gradle从入门到实战 - Groovy基础](https://blog.csdn.net/singwhatiwanna/article/details/76084580)、[使用Groovy开发之新特性](https://www.jianshu.com/p/ba55dc163dfd)。

**重点看的部分**
1. 闭包[Closures](http://www.groovy-lang.org/closures.html)。定义、使用、代理策略(`owner`, `delegate` and `this`)
2. 语义[Semantics](http://www.groovy-lang.org/semantics.html)。重点看`Promotion and coercion`、`Optionality `和`Typing `部分，尤其是和闭包相关的部分。

-- -- --

<br>



# API文档使用说明

Groovy API包括两部分，一部分是Groovy化的JDK API(`groovy-jdk`)，另一部分是新增的纯Groovy API(`gapi`)。

文档入口：
1. [官网Document](http://groovy-lang.org/documentation.html)->API documentation
![Groovy API入口](https://upload-images.jianshu.io/upload_images/2658578-cae297c94add4e77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2. 直接进入[API文档](http://docs.groovy-lang.org/)
![Groovy Api Document](https://upload-images.jianshu.io/upload_images/2658578-0da24a5176f7dff8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**[Groovy化的JDK API](http://docs.groovy-lang.org/docs/latest/html/groovy-jdk/)**

>This document describes the methods added to the JDK to make it more groovy.

Groovy在JDK中增加了许多方法，如在`List`增加了`public List each(Closure closure)`、`public List dropWhile(Closure condition)`等方法，使其Groovy化。

查找API示例：查看`List`中`public List each(Closure closure)`方法详情。
1. 通过包(`Package`)查。`java.util` -> `Interfaces` -> `List` -> `each(Closure closure)`

2. 通过索引(`Index`)查。`Index` -> 页面搜索`each(groovy.lang.Closure)`(`Mac`版`Chrome`浏览器快捷键`Command+F`) -> 找到`Method in interface java.util.List`一项

最终结果如下：
![List的each方法详情](https://upload-images.jianshu.io/upload_images/2658578-23a786dc872d9aa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
描述说是迭代一个`List`，并将每个子项作为参数传递到闭包中进行处理。



**[新增的Groovy类 API](http://docs.groovy-lang.org/docs/latest/html/gapi/)**

>Groovy - An agile dynamic language for the Java Platform
(GroovyDoc for Groovy and Java classes)

上面说的是`Groovy and Java classes`文档，包括`Groovy`类和原生`Java`类(可通过文档中的类的链接查看)。尤其要注意，这个文档中的`JDK API`点击后都会链接到原始的Java类，而不是Groovy化的Java类。
查看某个API详情的方法与上面的相同，略。

-- -- --

<br>



# 示例-Map语法
创建Gradle项目，并添加名为groovySyntax的任务
```bash
$ mkdir groovy-syntax
$ cd groovy-syntax
$ vi build.gradle
```
`build.gradle`文件
```groovy
task(groovySyntax).doLast {
    println 'start groovy syntax task'
    doMap()
}

def doMap() {
    def emptyMap = [:]
    def test = ["id":1, "name":"zhangliang", "isMale":true]
    test["id"] = 2
    test.id = 900
    println test.id
    println test.isMale
    println test

    test.each { key ,value ->
        println "key=$key, value=$value"
    }

    test.each { entry ->
        println "entry->$entry,${entry.key}, ${entry.value}"
    }
}
```
执行`gradle groovySyntax`，输出如下
```bash
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :groovySyntax
start groovy syntax task
900
true
{id=900, name=zhangliang, isMale=true}
key=id, value=900
key=name, value=zhangliang
key=isMale, value=true
entry->id=900,id, 900
entry->name=zhangliang,name, zhangliang
entry->isMale=true,isMale, true

BUILD SUCCESSFUL in 2s
1 actionable task: 1 executed
```
上面的代码和输出结果不难理解，分析如下：
1. 方法没有嵌套时，括号可以省略。`println 'start groovy syntax task'`和`test.each{...}`写全就是`println('start groovy syntax task)'`和`test.each({...})`
2. `Map`的定义参见[语言规范-Map](http://www.groovy-lang.org/syntax.html#_maps)。
3. `Map`的方法`each(Closure closure)`的API文档说明如下
>Allows a Map to be iterated through using a closure. If the closure takes one parameter then it will be passed the Map.Entry otherwise if the closure takes two parameters then it will be passed the key and the value.
可以用一个闭包来迭代`Map`，闭包的参数如果是一个就作为`Map.Entry`，如果两个参数就作为`key`和`value`。

-- -- --

<br>



# 示例-修改Android项目输出的APK名称

Android开发中，打包完后修改apk的名称是一个比较常见的做法。之前开发的Android项目中，在`app/build.gradle`中加入了几行代码，每次打包后自动修改apk的名称，相关代码如下：
`app/build.gradle`

```

...

// 利用git commit count 作为构建号
static def gitBuildCode() {
    def cmd = 'git rev-list HEAD --first-parent --count'
    cmd.execute().text.trim().toInteger()
}

static def appName() {
    return "youchat"
}

// 修改输出文件的名称
android.applicationVariants.all { variant ->
    variant.outputs.all {
        if (outputFileName != null && outputFileName.endsWith('.apk')) {
            def endStr = outputFileName
            if (outputFileName.contains("debug")) {
                endStr = "debug.apk"
            } else if (outputFileName.contains("preRelease")) {
                endStr = "preRelease.apk"
            } else if (outputFileName.contains("release")) {
                endStr = "release.apk"
            }
            outputFileName = "${appName()}_${android.defaultConfig.versionName}_${gitBuildCode()}_${endStr}"
        }
    }
}

...

```
打包完后，可以得到类似下面的文件
`app/build/outputs/apk/preRelease/youchat_7.5.0_999_preRelease.apk`
上面的代码逻辑比较简单，简单分析如下：
1. 定义了两个方法。`gitBuildCode ()`方法没有写`return`关键字，因为在`Groovy`中，方法总会返回值的，如果没有写`return`，就返回最后一行产生的值。参见[语言规范-Object orientation-Method](http://www.groovy-lang.org/objectorientation.html#_methods)
>Methods in Groovy always return some value. If no return statement is provided, the value evaluated in the last line executed will be returned
2. `android.applicationVariants.all(Closure var1)`和`variant.outputs.all(Closure var1)`是Gradle DSL语法，可以查看它的[javadoc](https://docs.gradle.org/current/javadoc/)
>Executes the given closure against all objects in this collection, and any objects subsequently added to this collection. The object is passed to the closure as the closure delegate. Alternatively, it is also passed as a parameter.
迭代容器中所有的对象，并传递给闭包进行处理。
3. `variant.outputs.all(Closure var1)`闭包中的代码逻辑是很简单的，只说下最后一行`outputFileName`的赋值语句，它使用了[字符串插值](http://groovy-lang.org/syntax.html#all-strings)语法`${}`，与kotlin里字符串模板是类似的。

-- -- --

<br>



# 参考
1. [Groovy 官网](http://www.groovy-lang.org/)
2. [Apache Groovy Documentation](http://docs.groovy-lang.org/)
3. [任玉刚 Gradle从入门到实战 - Groovy基础](https://blog.csdn.net/singwhatiwanna/article/details/76084580)
4. [Gradle DSL Javadoc](https://docs.gradle.org/current/javadoc/)
5. [使用Groovy开发之新特性](https://www.jianshu.com/p/ba55dc163dfd)
6. [Gradle构建](https://blog.csdn.net/innost/article/details/48228651)

