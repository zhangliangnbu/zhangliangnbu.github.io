---
title: Gradle的Wrapper、命令行和环境配置
date: 2018-10-18 16:12:25
updated: 2018-10-18 17:52:48
tags:
- gradle
- build
categories:
- Android开发
---

# 前言

这三个知识点不难，但经常用到，如果不看官方文档，有时候并不知道怎么使用，出现问题也不知道原因，所以有必要做一个总结。由于每个知识点都不多，又有关联，所以放在同一节。

<!-- more -->


# Gradle Wrapper

Gradle Wrapper（以下简写为“Wrapper”）用于管理当前项目的Gradle版本，Gradle官方强烈推荐使用Wrapper构建项目。多人协作时，必须规定项目的Gradle版本，并以此版本的Gradle作为项目的构建工具，由于每个人在本地安装的Gradle版本可能并不一致（也没有必要一致），因此有必要在项目中统一管理Gradle版本。

Wrapper的文件结构如下（项目根目录中）：

```properties
├── build.gradle
├── settings.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
└── gradlew.bat
```

包括一个gradle文件和两个可执行的脚本文件gradlew（macOS等平台用）和gradlew.bat（Windows平台用）。

- `gradle-wrapper.jar`。用于下载所需版本的Gradle。
- `gradle-wrapper.properties`。配置Gradle的版本号、本地存储地址等。各属性说明请见[官方文档](https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:adding_wrapper)。
- `gradlew`, `gradlew.bat`。Wrapper的执行脚本，用于替代`gradle`命令来构建项目。

> 注意：`gradle-wrapper.properties`中有一个`distributionUrl`属性，用于定义Gradle版本地下载URL，如`distributionUrl=https\://services.gradle.org/distributions/gradle-4.0-all.zip`，版本号后面有个“-all”，有时你也可能看到“-bin”，是什么意思呢？"-all"表示会下载此版本Gralde的所有的资源，包括二进制运行时文件、示例代码和相关文档。“-bin”表示只下载二进制运行时文件。



Wrapper 构建项目时，其工作流程如下：

1. 检查规定的Gradle版本，如果没有则去服务器下载。
2. 下载的Gradle版本存储在Gradle的用户目录中。如macOS中默认存储所有的Gradle版本到`/Users/yourname/.gradle/wrapper/dists/`中。
3. 使用解压后的Gradle版本来构建项目。



![wrapper workflow](https://docs.gradle.org/current/userguide/img/wrapper-workflow.png)



**添加Wrapper**

使用命令行添加Wrapper有两种方式：

1. 使用`gradle init`创建新项目，则会初始化一个带有Wrapper的Gradle项目。
2. 使用`gradle wrapper`在旧的项目中添加Wrapper。

> `wrapper`是Gradle的内建任务

一般的IDE创建项目时都会自动生产Wrapper文件，如Android Studio。



**使用Wrapper执行任务**

用Wrapper脚本替换掉`gradle`来执行任务即可。以macOS平台为例，在项目根目录下执行`./gradlew [task name] `即可，如列出当前项目的所有任务，项目根目录下执行：

```bash
$ ./gradlew tasks
```



**更新Wrapper**

有两种方式更新Wrapper

1. 命令行方法。`.gradlew wrapper --gradle-version [要更新的版本号]`。
2. 修改`gradle/wrapper/gradle-wrapper.properties`中的`distributionUrl`属性。





<br>

# Gradle命令行

与Gradle交互有两种方式，一是[命令行界面](https://docs.gradle.org/current/userguide/command_line_interface.html)，二是IDE的图形界面，如Android Studio。IDE方式各有不同且未必包括所有的命令行指令，但命令行方式却是不变的，因此需要学会基本的命令行指令，以不变应万变。

> 注意：在项目中使用命令行方式时，推荐使用`./gradlew` or `gradlew.bat` 来代替 `gradle`。示例中为了方便，统一使用`gradle`

命令行指令可以自定义，参见[declaring_and_using_command_line_options](https://docs.gradle.org/current/userguide/custom_tasks.html#sec:declaring_and_using_command_line_options)



**指令形式**

```shell
gradle [taskName...] [--option-name...]
```

说明

* 任务名（taskName）有多个时，使用空格分开，如`gradle task1 task2`

* 在多项目工程中，执行某个项目的任务时，可以用“:”将项目名添加到任务名之前，如

  ```shell
  $ gradle projectName:taskName
  $ gradle :projectName:taskName
  ```

* 可选项（option-name）如果接收参数，建议使用`=`拼接，如

  ```shell
  $ gradle task1 --console=plain
  ```

* 可选项规定一个行为时，可使用`--no-`作为其全称（long-form）前缀，来指定它的对立行为。如

  ```shell
  --build-cache
  --no-build-cache
  ```

* 可选项的全称名称常有简写形式 ，如

  ```shell
  --help
  -h
  ```



**[常见内建（built-in）任务指令](https://docs.gradle.org/current/userguide/command_line_interface.html#common_tasks)**

* `gradle build`。生成所有的输出，并执行所有的检查。
* `gradle run`。生成应用程序并执行某些脚本或二进制文件
* `gradle check`。执行所有检测类任务如tests、linting等
* `gradle clean`。删除build文件目录。
* `gradle projects`。查看项目结构。
* `gradle tasks`。查看任务列表。查看某个任务详细信息，可用`gradle help --task someTask`
* `gradle dependencies`。查看依赖列表。



部分常见命令行选项说明如下：

**调试类**

* `-?`, `-h`, `--help`。查看帮助信息。
* `-v`,` --version`。查看版本信息。
* `-s`,` --stacktrace`。执行任务时，打印栈信息。如`gradle build --s`

其他选项说明见官方文档[Debugging options](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:command_line_debugging)



**日志类**

- `-q`, `--quiet`。只打印errors类信息。
- `-i`, `--info`。打印详细的信息。

其他选项说明见官方文档[Logging options](https://docs.gradle.org/current/userguide/command_line_interface.html#setting_log_level)，更多关于日志类描述请见[Logging](https://docs.gradle.org/current/userguide/logging.html#logging)。



**性能类**

用于优化构建的性能

- `--configure-on-demand`,` --no-configure-on-demand`。是否开启按需配置模式。
- `--build-cache`, `--no-build-cache`。是否使用缓存。

其他选项说明见官方文档[Performance Options](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:command_line_performance)，更多关于性能优化方面的描述请见 [improving performance of Gradle builds here](https://guides.gradle.org/performance/).



**环境配置类**

用于配置构建时的环境

- `-c`, `--settings-file`。如果不用默认的`settings.gradle`时，可指定Setting文件。
- `-b`, `--build-file`。指定构建文件。
- `-Dorg.gradle.jvmargs`。设置JVM参数。

其他选项说明见官方文档[Environment options](https://docs.gradle.org/current/userguide/command_line_interface.html#environment_options)，更多关于环境配置方面的描述请见[build environment](https://docs.gradle.org/current/userguide/build_environment.html#build_environment)



<br>

# 环境配置

[配置构建环境](https://docs.gradle.org/current/userguide/build_environment.html)，主要配置Gradle构建参数和对应的JVM参数，如代理策略等。其目的是为了多人协作时，保持在一致的环境下进行项目开发。

配置环境有几种途径，优先级从高往低，列出如下：

1. 命令行。
2. `GRADLE_USER_HOME`目录中的`gradle.properties`文件。
3. 项目根目录中的`gradle.properties`文件。
4. 环境变量。运行Gradle环境的变量，如JAVA_HOME等。



配置环境其实是设置各种属性参数，以上四种方式对于某些属性都可以配置。其中命令行和`gradle.properties`文件方式支持所有属性的配置。属性有不同的级别，按照优先级从高到底列出如下：

1. 系统属性（System properties）。
2. Gradle属性。



**系统属性**

用于设置Gradle的JVM环境。

1. 命令行方式。通过给属性添加`-D`前缀来设置。如`gradle -Dgradle.user.home=foo`
2. `gradle.properties`文件方式。根目录中，为属性添加`systemProp.`前缀，如`systemProp.gradle.wrapperUser=myuser`



几个系统属性如下：

* `gradle.wrapperUser=(myuser)`。设置Wrapper下载Gradle版本的代理服务器的用户名。
* `gradle.wrapperPassword=(mypassword)`。设置Wrapper下载Gradle版本的代理服务器的密码。
* `gradle.user.home=(path to directory)`。设置Gradle用户目录地址。



**Gradle属性**

Gradle属性都有对应的命令行方式，`gradle.properties`文件属性和对应的命令行如下：

| `gradle.properties`文件属性                                  | 命令行指令                                                   |
| ------------------------------------------------------------ | :----------------------------------------------------------- |
| `org.gradle.caching=(true,false)`                            | `--build-cache`,  `--no-build-cache`                         |
| `org.gradle.caching.debug=(true,false)`                      | ?                                                            |
| `org.gradle.configureondemand=(true,false)`                  | `--configure-on-demand`, `--no-configure-on-demand`          |
| `org.gradle.console=(auto,plain,rich,verbose)`               | `--console=(auto,plain,rich,verbose)`                        |
| `org.gradle.daemon=(true,false)`                             | `--daemon`, `--no-daemon`                                    |
| `org.gradle.daemon.idletimeout=(# of idle millis)`           | `-Dorg.gradle.daemon.idletimeout=(number of milliseconds)`   |
| `org.gradle.debug=(true,false)`                              | `-Dorg.gradle.debug=（true,false)`                           |
| `org.gradle.java.home=(path to JDK home)`                    | `-Dorg.gradle.java.home`                                     |
| `org.gradle.jvmargs=(JVM arguments)`                         | `-Dorg.gradle.jvmargs`                                       |
| `org.gradle.logging.level=(quiet,warn,lifecycle,info,debug)` | `-Dorg.gradle.logging.level=(quiet,warn,lifecycle,info,debug) `或者 `-q`、`-w`、`-i`、`-d` |
| `org.gradle.parallel=(true,false)`                           | `--parallel`, `--no-parallel`                                |
| `org.gradle.warning.mode=(all,none,summary)`                 | `-Dorg.gradle.warning.mode=(all,none,summary)` 或者`--warning-mode=(all,none,summary)` |
| `org.gradle.workers.max=(max # of worker processes)`         | `--max-workers`                                              |



详细说明请见[Gradle properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties)

另外官网中有项目属性和通过项目属性配置任务的说明，这里不做描述，请见官网[Project properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:project_properties)



**示例-设置JVM参数**

对于每个Java虚拟机进程，Gradle默认配置堆空间最大为1024MB（`-Xmx1024m`）。-Xmx1024m根据项目大小，可以修改这一参数。如果使用了Gradle Daemon（`org.gradle.daemon=true`，默认开启），则配置参数如下：

```properties
org.gradle.jvmargs=-Xmx2g -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

Java虚拟机参数设置参考Java HotSpot VM Options：[Java 8及以上](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)，[Java 7参考](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)。上面的参数说明如下：

* `-Xmx[size]`。设置虚拟机最大内存空间。必须是1024的倍数，且不能小于2MB，单位为k/K、m/M、g/G，默认值随系统配置而变。与`XX:MaxHeapSize`等价。示例80MB：

  ```properties
  -Xmx83886080
  -Xmx81920k
  -Xmx80m
  ```

* `-XX:MaxPermSize=[size]`。设置持久代最大空间，持久代用于存储常量等，如果报`java.lang.OutOfMemoryError: PermGen space`的异常，则需要增加这一空间。

  > Java虚拟机内存划分请见Oracle官方文档[Generations](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html)

* `-XX:-HeapDumpOnOutOfMemoryError`。当抛出` java.lang.OutOfMemoryError `异常时，将堆内容存储到文件中，用于调试。

* `-Dfile.encoding=UTF-8`。虚拟机编码格式。

  > 在参考文档中并没有找到``-Dfile.encoding`设置

**示例-设置代理**

国内下载依赖和Google资源时有时候速度很慢，可以通过设置HTTP和HTTPS代理服务解决。在`gradle.properties`中设置如下：

```properties
# http代理
systemProp.http.proxyHost=www.somehost.org
systemProp.http.proxyPort=8080
systemProp.http.proxyUser=userid
systemProp.http.proxyPassword=password
systemProp.http.nonProxyHosts=*.nonproxyrepos.com|localhost

# https代理
systemProp.https.proxyHost=www.somehost.org
systemProp.https.proxyPort=8080
systemProp.https.proxyUser=userid
systemProp.https.proxyPassword=password
systemProp.https.nonProxyHosts=*.nonproxyrepos.com|localhost
```

如果没有user和password，可以不用写。

其他的代理参数可以参考Java文档[Networking Properties](https://docs.oracle.com/javase/7/docs/technotes/guides/net/properties.html)



<br>

# 参考

1. [Gradle官网-Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)
2. [Gradle官网-Command-Line Interface](https://docs.gradle.org/current/userguide/command_line_interface.html)
3. [Gradle官网-Build Environment](https://docs.gradle.org/current/userguide/build_environment.html)