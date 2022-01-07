---
title: 项目发布-手动
date: 2018-12-15 21:09:18
tags:
- maven
- jcenter
- bintray
categories:
- 其他
---

# 前言

本文主要说明如何生成项目构件，并手动上传构件文件到Bintray平台，最后发布到JCenter仓库这一流程。

<!-- more -->

**发布流程**。参考上一篇文章，完整发布流程如下：

1. 准备工作。定义参数；准备本地待发布项目；配置Bintray平台账号、仓库、Package、版本。
2. 生成构件文件。本地生成待发布的构件文件(包括POM文件)。
3. 发布到Bintray。上传和发布本地构件文件到Bintray平台。
4. 发布到JCenter。发布Bintray平台上的项目到JCenter。

**源码地址**。本文涉及到的`nicelogger`项目Github地址：https://github.com/zhangliangnbu/nice-logger

---

# 准备工作

参考上一篇文章，如果已经做了，可以跳过。默认你已经有了一个本地项目，已经创建了Bintray平台账号和Maven仓库。

**定义参数**

- Bintray平台仓库名称。`android`。
- Bintray平台Package名称。`nicelogger`。
- POM文件`groupId`。`com.liang.android`。
- POM文件`artifactId`。`nicelogger`。
- POM文件`version`。取`0.0.1`。

**准备本地项目**。略。

**配置Bintray平台**。根据参数创建Package和版本。

<br>

---

# 生成构件

生成需要上传到Maven库的四个文件：

- `nicelogger-0.0.1.aar`
- `nicelogger-0.0.1-sources.jar`
- `nicelogger-0.0.1-javadoc.jar`
- `nicelogger-0.0.1.pom`

**生成`nicelogger-0.0.1.aar`文件**

在`nicelogger`的`build.gradle`最后中添加如下代码，用于重命名生成的aar文件：

```groovy
// 定义pom文件参数
def groupIdDefined = "com.liang.android"
def artifactIdDefined = "nicelogger"
def versionDefined = "0.0.1"

// 重命名生成的arr文件
android.libraryVariants.all { variant ->
    variant.outputs.all { output ->
        if (outputFile != null && outputFileName.endsWith('.aar')) {
            outputFileName = "${artifactIdDefined}-${versionDefined}.aar"
        }
    }
}
```

执行`./gradlew nicelogger:assembleRelease`生成arr文件，位于`library/build/outputs/aar/`中。



**生成`nicelogger-0.0.1-sources.jar`文件**

接着在`nicelogger`的`build.gradle`最后添加如下代码，建立生成所需文件的任务

```groovy
// 生成Java源码文件
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    baseName = "${artifactIdDefined}-${versionDefined}"
    classifier = 'sources'
}
```

然后执行`./gradlew nicelogger:sourcesJar`，就可以生成Java源码文件，位于`nicelogger/build/libs/`中。



**生成`nicelogger-0.0.1-javadoc.jar`文件**

接着在`nicelogger`的`build.gradle`最后添加如下代码，建立生成所需文件的任务

```groovy
// 生成Javadoc文件
task javadoc(type: Javadoc) {
    failOnError false
    source = android.sourceSets.main.java.sourceFiles
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    classpath += configurations.compile
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    baseName = "${artifactIdDefined}-${versionDefined}"
    classifier = 'javadoc'
    from javadoc.destinationDir
}
```

然后执行`./gradlew nicelogger:javadocJar`，就可以生成Javadoc文件，位于`nicelogger/build/libs/`中。



**生成`nicelogger-0.0.1.pom`文件**

有两种方式创建POM文件：

一，参考[POM Reference](http://maven.apache.org/pom.html)手动创建POM文件，填入相关参数。

二，使用[Gradle Maven Plugin](https://docs.gradle.org/current/userguide/maven_plugin.html)生成POM，接着在`nicelogger`的`build.gradle`最后中添加如下代码

```groovy
// 生成pom文件
apply plugin: 'maven'

task createPom {
    doLast {
        pom {
            project {
                groupId "${groupIdDefined}"
                artifactId "${artifactIdDefined}"
                version "${versionDefined}"
                packaging "aar"
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
            }
        }.writeTo("$libsDir/${artifactIdDefined}-${versionDefined}.pom")
    }
}
```

后执行`./gradlew nicelogger:createPom`，就可以生成所需的POM文件，位于`nicelogger/build/libs/`中，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.liang.android</groupId>
  <artifactId>nicelogger</artifactId>
  <version>0.0.1</version>
  <packaging>aar</packaging>
  <licenses>
    <license>
      <name>The Apache Software License, Version 2.0</name>
      <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
    </license>
  </licenses>
</project>
```

生成四个文件后，可以把它们放在一个文件夹里，方面之后的上传操作。

---

# 发布到Bintray

**上传文件**

* 进入上传文件页。在Bintray官网上，进入`{usrname}/android/nicelogger/0.0.1`版本页，点击“Upload Files”进入上传文件页。
* 上传文件。点击“Click to Add Files” 上传四个文件， “Target Repository Path”填`com/liang/android/nicelogger/0.0.1`，一般上传了POM文件后，就会有提示。然后点击“Save Changes”即可。

> Target Repository Path为{groupId}/{artifactId}/{version}，其中“.”都需要替换为“/”。

**发布到Bintray仓库**

上传文件完成后，会提示如何处理文件：“Discard”还是“Publish”。选择“Publish”，就会发布文件到Bintray平台的Maven仓库，这样我们就能解析，只不过要带上用户自己的仓库地址而已。

> 发布到Bintray仓库后，会自动生成`maven-metadata.xml`文件。

用户Bintray仓库地址。发布后的项目在`https://dl.bintray.com/{username}/{repository_name}`下可查看，我刚才发布的项目在`https://dl.bintray.com/zhangliang/android/com/liang/android/nicelogger/`。

**Bintray仓库依赖解析**

Gradle项目使用`nicelogger`依赖方式如下：

一，添加仓库地址。

在需要此依赖的项目`build.gradle`配置仓库地址：

```groovy
repositories {
    maven {
        url "https://dl.bintray.com/zhangliang/android"
    }
}
```

或工程目录`build.gradle`下添加配置：

```groovy
allprojects {
    repositories {
        maven {
            url "https://dl.bintray.com/zhangliang/android"
        }
    }
}
```

二，添加依赖。在项目`build.gradle`配置依赖

```groovy
dependencies {
    implementation 'com.liang.android:nicelogger:0.0.1'
}
```



---

# 发布到JCenter

进入`nicelogger`的package页面，点击“Add to JCenter”，进入下一页，点击“Send”即可。

发布到JCenter仓库需要审核，主要审核文件格式是否符合规范和以及名称是否唯一等，没有问题的话，一般一个工作日以内都会通过。通过后，可以直接用jcenter作为仓库地址，无需配置用户自己的Bintray仓库地址。

JCenter仓库地址。`http://jcenter.bintray.com/{地址化的groupId}`，`地址化的groupId`即用“/”代替“.”。

具体详情请见用户指南[发布到JCenter](https://www.jfrog.com/confluence/display/BT/Central+Repositories#CentralRepositories-JCenter)。

**可能遇到的问题**

一，点击“Add to JCenter”报错：

> Please fix the following before submitting a JCenter inclusion request: the POM is invalid or was uploaded to the wrong coordinates

我之前是因为文件名不一致才出现这个错误提示。必须为`{artifactId}-{version}xxx`等。

终于写完了一个完整的流程，O(∩_∩)O。

----

# 参考

1. [Bintray官网](https://bintray.com)
2. [JCenter仓库](https://jcenter.bintray.com/)
3. [JFrog用户指南](https://www.jfrog.com/confluence/display/BT/Introduction)

---