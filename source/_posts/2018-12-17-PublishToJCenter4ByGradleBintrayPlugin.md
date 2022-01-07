---
title: 项目发布-GradleBintrayPlugin
date: 2018-12-17 17:01:45
tags:
- maven
- jcenter
- bintray
categories:
- 其他
---

# 前言

本文介绍如何使用Bintray官方的Gradle插件[gradle-bintray-plugin](https://github.com/bintray/gradle-bintray-plugin#readme)发布项目到Bintray平台，并最终发布到JCenter。

主要参考官方的[gradle-bintray-plugin wiki](https://github.com/bintray/gradle-bintray-plugin#readme)和项目示例[bintray-examples](https://github.com/bintray/bintray-examples)。不过说实话官方wiki和示例写得都比较粗糙。

gradle-bintray-plugin插件在生成构件时有三种方式： Configurations, Publications and Copying files。每种方式的配置都有所不同，我们这里只说比较常见的Configurations和Publications两种方式，这也是wiki上比较推荐的方式。

**源码地址**。本文涉及到的`nicelogger`项目Github地址：https://github.com/zhangliangnbu/nice-logger

<!-- more -->

---

# 准备工作

参考上一篇文章，如果已经做了，可以跳过。默认你已经有了一个本地项目，已经创建了Bintray平台账号和Maven仓库。

**定义参数**

- Bintray平台仓库名称。`android`。
- Bintray平台Package名称。`nicelogger`。
- POM文件`groupId`。`com.liang.android`。
- POM文件`artifactId`。`nicelogger`。
- POM文件`version`。取`0.0.6`。

**准备本地项目**。有的话就不用创建。

**配置Bintray平台**。创建package，如果已经有了就不用创建了。

---

# 使用Publications方式发布项目

**插件仓库配置**

根据wiki上的说明，`Gradle >= 2.1`之后就可以不用单独配置插件仓库路径了，估计是已经包含在JCenter仓库里，我使用的是Gradle 4.6，也不用单独配置插件仓库路径。

**参数配置**

主要包括三个部分：Bintray平台参数配置、POM和构件文件参数配置、构件文件生成配置。

单独用一个文件`gradleBintrayPluginPublicationsUpload.gradle`维护这些配置，然后在module的`build.gradle`中引用。

项目`build.gradle`文件末尾添加：

```groovy
apply from: './gradleBintrayPluginPublicationsUpload.gradle'
```

`gradleBintrayPluginPublicationsUpload.gradle`配置如下：

```groovy
// 插件。无需另外单独配置插件仓库地址
apply plugin: 'maven-publish'
apply plugin: 'com.jfrog.bintray'

// 定义参数
def gitUrl = 'https://github.com/zhangliangnbu/nice-logger.git'   // Git仓库的url
def groupIdDefined = "com.liang.android"
def artifactIdDefined = "nicelogger"
def versionDefined = "0.0.5"

// bintray平台信息配置
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user") // local.properties里设置
    key = properties.getProperty("bintray.apikey") // local.properties里设置
    publications = ['MyPublication'] // 'MyPublication'与下面的publishing闭包里的名称对应
    publish = true // 上传后立即发布到Bintray平台
    pkg {
        repo = "android"  // 必填。bintray平台仓库名，必须已经创建过。
        name = "nicelogger"  // 必填。仓库里包package的名称，没有的话会自动创建。
        licenses = ["Apache-2.0"] // 首次创建package则必须，否则选填。
        vcsUrl = gitUrl // 首次创建package则必须，否则选填。
        version {
            name = "$versionDefined"
        }
    }
}


// 构件文件和POM信息配置
publishing {
    publications {
        MyPublication(MavenPublication) {
            artifact("$buildDir/outputs/aar/nicelogger-release.aar")
            artifact sourcesJar
            artifact javadocJar
            groupId "$groupIdDefined"
            artifactId "$artifactIdDefined"
            version "$versionDefined"
            pom.withXml {
                def dependenciesNode = asNode().appendNode('dependencies')
                // Iterate over the implementation dependencies (we don't want the test ones), adding a <dependency> node for each
                configurations.implementation.allDependencies.each {
                    // Ensure dependencies such as fileTree are not included in the pom.
                    if (it.name != 'unspecified') {
                        def dependencyNode = dependenciesNode.appendNode('dependency')
                        dependencyNode.appendNode('groupId', it.group)
                        dependencyNode.appendNode('artifactId', it.name)
                        dependencyNode.appendNode('version', it.version)
                    }
                }
            }
        }
    }
}

// 生成sourceJar和javaDocJar构件
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    failOnError false
    source = android.sourceSets.main.java.sourceFiles
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    classpath += configurations.compile
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives sourcesJar
    archives javadocJar
}
```

注释里已经有说明了，补充几点如下：

* bintray平台信息配置。`user`和`key`是自己的私有参数，写在本地`local.properties`文件中。
* 先要生成构件文件，之后才能上传。

**发布**

执行命令`./gradlew clean build bintrayUpload`，即可发布项目到Bintray平台，然后一键发布到JCenter。

----

# 使用Configurations方式发布项目

**插件仓库配置**

这种方式需要用到[android-maven-gradle-plugin](https://github.com/dcendents/android-maven-gradle-plugin)。在工程根目录`build.gradle`中添加仓库路径：

```groovy
buildscript {
    dependencies {
        // bintray plugin for configuration method
        classpath 'com.github.dcendents:android-maven-gradle-plugin:2.1'
    }
}
```



**参数配置**

主要包括三个部分：Bintray平台参数配置、POM和构件文件参数配置、构件文件生成配置。

单独用一个文件`gradleBintrayConfigurationsUpload.gradle`维护这些配置，然后在module的`build.gradle`中引用。

项目`build.gradle`文件末尾添加：

```groovy
apply from: './gradleBintrayConfigurationsUpload.gradle'
```

`gradleBintrayConfigurationsUpload.gradle`配置如下：

```groovy
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

// 定义参数
def gitUrl = 'https://github.com/zhangliangnbu/nice-logger.git'   // Git仓库的url
def groupIdDefined = "com.liang.android"
def artifactIdDefined = "nicelogger"
def versionDefined = "0.0.7"

// 待发布项目的groupId和version。android-maven-gradle-plugin插件需要这么配置。
group = "$groupIdDefined"
version = "$versionDefined"

// bintray平台信息配置
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user") // local.properties里设置
    key = properties.getProperty("bintray.apikey") // local.properties里设置
    configurations = ['archives']
    publish = true
    pkg {
        repo = "android"  // 必填。bintray平台仓库名，必须已经创建过。
        name = "nicelogger"  // 必填。仓库里包package的名称，没有的话会自动创建。
        licenses = ["Apache-2.0"] // 首次创建package则必须，否则选填。
        vcsUrl = gitUrl // 首次创建package则必须，否则选填。
        version {
            name = "$versionDefined"
        }
    }
}

// pom文件信息配置
install {
    repositories.mavenInstaller {
        pom.project {
            groupId "$groupIdDefined"
            artifactId "$artifactIdDefined"
            version "$versionDefined"
            packaging 'aar'

            licenses {
                license {
                    name 'The Apache Software License, Version 2.0'
                    url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                }
            }
        }
    }
}

// 生成sourceJar和javaDocJar
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    failOnError false
    source = android.sourceSets.main.java.sourceFiles
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    classpath += configurations.compile
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives sourcesJar
    archives javadocJar
}

// 执行 ./gradlew clean bintrayUpload
```

补充说明如下：

* [android-maven-gradle-plugin](https://github.com/dcendents/android-maven-gradle-plugin)的wiki上说需要在`settings.gradle`里配置`artifactId`,但我没有配置也发布成功了。
* 不需要单独执行构件生成任务，应该是在`bintrayUpload`中已经做了处理。

**发布**

执行命令`./gradlew clean bintrayUpload`，即可发布项目到Bintray平台，然后一键发布到JCenter。

---

# 小结

是不是感觉使用官方插件挺繁琐的？我的配置是非常精简的，选填的参数几乎没有填，看官方示例，配置真是繁琐。

到目前为止，已经介绍了三种发布项目到JCenter的方式，推荐使用BintrayRelease方式，配置简洁、易于维护。

---

# 参考

1. [gradle-bintray-plugin](https://github.com/bintray/gradle-bintray-plugin)
2. [gradle-bintray-plugin wiki](https://github.com/bintray/gradle-bintray-plugin#readme)
3. [bintray-examples](https://github.com/bintray/bintray-examples)
4. [android-maven-gradle-plugin](https://github.com/dcendents/android-maven-gradle-plugin)



---