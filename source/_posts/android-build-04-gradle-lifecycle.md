---
title: Android构建04-Gradle构建流程
date: 2018-08-27 20:24:50
updated: 2018-10-17 14:52:48
tags:
- android
- build
- gradle
categories:
- Android开发
---





# 构建的生命周期

Gradle项目的构建分为三个阶段：初始化、配置、执行。
参考官方手册 [Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html)。



## 初始化（Initialization）

在这个阶段中，Gradle决定哪些项目加入到构建中（因为Gradle支持多项目构建），并为这些项目分别创建一个`Project`实例。

Gradle支持单项目和多项目构建，如果是单项目构建，则初始化当前项目即可，如果是多项目构建，则需要决定哪些项目需要加入到构建中并初始化。那么：
1. Gradle是如何判断当前构建是单项目还是多项目构建呢？
2. 如果是多项目构建，Gradle又是如何决定将哪些项目加入到构建中呢？

**多项目还是单项目构建？**
这里的关键是一个名为`setting.gradle`的文件，Gradle在初始化阶段会首先去查找`setting.gradle`文件，查找的规则如下：

1. 查找当前构建目录下的`setting.gradle`文件。
2. 如果没有找到，则去与当前目录有相同嵌套级别的`master`目录查找。
3. 如果没有找到，则去父目录查找。
4. 如果没有找到，则进行单项目构建。
5. 如果找到了，Gradle去检查当前项目在`settings.gradle`中是否有定义。如果没有，则进行单项目构建，否则进行多项目构建。

多项目工程在根目录必须存在`setting.gradle`文件，单项目工程则可以不需要这个文件。

**多项目构建的项目集如何决定？**
多项目构建时，项目集由`setting.gradle`创建。项目集可以用一个树形结构表示，每个节点表示一个项目，并且有对应的项目路径。在大多数情况下，项目路径与文件系统中项目的位置一致，当然这也是可以配置的。
在`setting.gradle`中，有很多方法可以定义项目树，常见的是层次化和扁平化的布局。

层次化布局
```groovy
// setting.gradle
include 'project1', 'project2:child', 'project3:child1'
```
`include`方法接收多个项目路径作为参数。
每个项目路径对应文件系统中的一个目录，通常，项目路径和文件目录一一对应，比如`project2:child`对应项目根目录下的`project2/child`目录。
如果项目树的叶子节点和它的父节点都需要包含在构建中，则只需指定叶子节点即可。比如，`project2:child`，将创建两个项目实例`project2`和`project2:child`。

扁平化布局
```groovy
// setting.gradle
includeFlat 'project3', 'project4'
```
`includeFlat`方法将目录名作为参数。这些目录是根项目目录的兄弟目录，并作为多项目树中根项目的子项目。



**Settings语法**
这里主要说一下`include`方法的语法，其余的语法请见对应的[DSL Reference](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html)。`setting.gradle`在构建时会创建代理对象`Settings`，其`include`方法对应着 `Settings.include(java.lang.String[])`，查看其[DSL Reference](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html#org.gradle.api.initialization.Settings:include(java.lang.String[])):

1. `include`方法接收项目路径数组作为参数，并将对应的项目添加到构建中。
2. 参数中支持的项目路径分隔符为“:”，而不是“/”。
3. 路径的最后一个节点是项目的名称。
4. 项目路径是根项目目录下的相对路径，可以使用[`ProjectDescriptor.setProjectDir(java.io.File)`](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/ProjectDescriptor.html#setProjectDir-java.io.File-))更改。

示例
```groovy
// 引入两个项目, 'foo' and 'foo:bar'
// 对应的文件目录是 $rootDir/a 和 $rootDir/a/b
// directories are inferred by replacing ':' with '/'
include 'foo:bar'

// include one project whose project dir does not match the logical project path
// 引入‘baz’项目，对应的文件目录是foo/baz
include 'baz'
project(':baz').projectDir = file('foo/baz')
```



## 配置（Configuration）

在这个阶段，Gradle会配置加入到构建中的`Project`实例，执行所有项目对应的构建脚本。

>作为名词的‘配置（Configuration）'是指，Gradle执行项目的构建脚本这一过程。
>作为动词的'配置（ Configure）'是指，Gradle执行项目的构建脚本这一动作。



**默认配置模式**

默认情况下，Gradle会配置`settings.gradle`里的所有项目，不论这些项目与最终执行的任务是否有关系。这样做是因为Gradle允许一个项目在配置和执行阶段访问任何其他项目，如Gradle里可以进行[跨项目配置(Cross project configuration)](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:cross_project_configuration)，一个项目的配置可能会依赖其他项目，所以在执行任务之前，需要配置所有的项目。

项目配置按照广度（ breadth-wise）顺序来执行，如父项目先于子项目被配置。



**按需配置模式**

由于多项目配置中，可能存在大量无需配置的项目，如果需要配置所有项目后才执行任务则会浪费大量的时间。从Gradle1.4开始，有一个孵化中的特性，叫做[按需配置（configuration on demand）](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:cross_project_configuration)模式。按需配置项目时，Gradle只配置与最终任务相关联的项目，以缩短构建时间。这个模式以后可能会成为默认模式，甚至成为唯一的模式。截至我写这篇文章（Gradle发布4.10）时，按需配置模式还只是一个实验中的特性，不能保证所有的构建都能正确运行，尤其是存在耦合的多项目构建。

> 项目的耦合和解耦请见文档的[Decoupled Projects](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:decoupled_projects)。一个简单的耦合项目就是在子项目中使用`allprojects` and `subprojects`等方法，在根项目中使用这两个方法则不会影响按需配置模式。

按需配置模式下，项目配置遵循规则如下：

* 根项目总会被配置。
* 构建的当前项目也会被配置。
* 项目的依赖会被配置。如果项目A将项目B作为依赖，则构建A时，A和B都会被配置。
* 任务依赖对应的项目会被配置。如’someTask.dependsOn(":someOtherProject:someOtherTask")‘，someOtherProject会被配置。
* 通过命令行构建任务时会配置相应的项目。如构建 'projectA:projectB:someTask'时会配置projectB。



进行构建时，可以在命令行加入`--configure-on-demand`来指定用按需配置模式来进行构建，如`gradle hello --configure-on-demand`将以按需配置模式执行‘’hello“任务。



## 执行（Execution）

在这个阶段，首先，Gradle确定要执行的任务集，任务集是由输入到命令行的任务名称参数和当前目录决定的。然后，Gradle会去执行任务集中的每个任务。
任务（`Task`）是由一系列的活动（`Action`）组成的，当任务执行的时候，活动会依次执行。可以通过`doFirst`和`doLast`方法将活动添加到任务中。

问题：

1. 多项目构建中，Gradle如何确定任务是哪个项目的？尤其是任务名称相同的情况下。
2. 如果确定多个任务执行的顺序？

**任务的位置**
以`$ gradle hello`为例，在执行阶段，Gradle会从构建的当前项目目录开始，依据项目树往下查找名称为`hello`的任务并执行。因此Gradle不会执行当前项目的父项目和兄弟节点项目的任务。

**任务的顺序**
如果没有额外配置，Gradle将以字母数字顺序执行任务。比如 “:consumer:action” 将先于 “:producer:action”执行。

> If nothing else is defined, Gradle executes the task in alphanumeric order

-- -- --

<br>



# 示例-单项目构建
```bash
# 创建项目目录
$ mkdir single-project-build-lifecycle-demo
$ cd single-project-build-lifecycle-demo

# 创建settings.gradle文件
$ vi settings.gradle
```
`settings.gradle`文件
```
rootProject.name = 'single-project-build-lifecycle-demo'
println 'settings.gradle -> this is executed during the initialization phase.'
```

```bash
# 创建build.gradle文件
$ vi build.gradle
```
` build.gradle`文件

```groovy
println 'build.gradle -> global println -> This is executed during the configuration phase.'

task configured {
    println 'build.gradle -> task configured -> This is also executed during the configuration phase.'
}

task test {
    doLast {
        println 'build.gradle -> task test -> doLast -> This is executed during the execution phase.'
    }
}

task testBoth {
    doFirst {
      println 'build.gradle -> task testBoth -> doFirst -> This is executed first during the execution phase.'
    }
    doLast {
      println 'build.gradle -> task testBoth -> doLast -> This is executed last during the execution phase.'
    }
    println 'build.gradle -> task testBoth -> This is executed during the configuration phase as well.'
}
```

开始构建，并执行testBoth任务 `$ gradle testBoth`，输出如下：

```bash
settings.gradle -> this is executed during the initialization phase.

> Configure project :
build.gradle -> global println -> This is executed during the configuration phase.
build.gradle -> task configured -> This is also executed during the configuration phase.
build.gradle -> task testBoth -> This is executed during the configuration phase as well.

> Task :testBoth
build.gradle -> task testBoth -> doFirst -> This is executed first during the execution phase.
build.gradle -> task testBoth -> doLast -> This is executed last during the execution phase.

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```
构建过程如下：
1. 初始化阶段。
    执行`settings.gradle`脚本，所以首先输出'settings.gradle -> this is executed during the initialization phase.'。

2. 配置阶段。
    由于`settings.gradle`没有定义项目，根项目即当前项目有`build.gradle`，因此只是执行这个文件。顺序执行：打印语句、三个task方法。
    这里的三个task方法对应`Project`实例中`Task task(String name, Closure configureClosure)`，查看其[Reference文档](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:task(java.lang.String,%20groovy.lang.Closure))，说是这个方法会产生一个任务对象并添加到当前`Project`实例中，并且在返回这个任务的时候会执行闭包以配置这个任务。也就说在配置我们这个项目时，三个`task`方法的闭包都会被执行。
    以testBoth任务对应的`task`方法为例，它的闭包里包含`doFirst`、`doLast`和`println`三个方法。其中`doFirst`和`doLast`是`Task`的方法，查看`doFirst`的[Reference文档](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html#org.gradle.api.Task:doFirst(groovy.lang.Closure))，说是添加闭包到任务的action列表中，当任务执行的时候，会执行这个闭包。在配置阶段，并不会去执行任务，也就是说不会去执行`doFirst`和`doLast`闭包里的内容。
    因此，在配置阶段，Gradle会依次执行三个任务方法里的闭包，而不会执行闭包里添加action的闭包即`doFirst`和`doLast`的闭包，因此会依次输出
    'build.gradle -> task configured -> ...' 和‘build.gradle -> task testBoth -...’

3. 执行阶段。
    由于在命令行只构建testBoth任务，因此这个阶段只执行testBoth任务中的`Action`列表，依次执行`doFirst`和`doLast`的闭包。

-- -- --

<br>



# 示例-常见的多项目构建
参考官方手册[Authoring Multi-Project Builds](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:project_jar_dependencies)
模拟一个常见的Java项目，目录结构如下

```
multi-project-build-lifecycle-demo/
  settings.gradle
  build.gradle
  api/
    src/main/java/
      org/gradle/sample/
        api/
          Person.java
        apiImpl/
          PersonImpl.java
  services/personService/
    src/
      main/java/
        org/gradle/sample/services/
          PersonService.java
      test/java/
        org/gradle/sample/services/
          PersonServiceTest.java
  shared/
    src/main/java/
      org/gradle/sample/shared/
        Helper.java
```
根目录`settings.gradle`
```groovy
include 'api', 'shared', 'services:personService'
```
根目录`build.gradle`
```groovy
subprojects {
    apply plugin: 'java'
    group = 'org.gradle.sample'
    version = '1.0'
    repositories {
        mavenCentral()
    }
    dependencies {
        testCompile "junit:junit:4.12"
    }
}

project(':shared') {
    task hello {
        doLast {
            println "shared task hello -> doLast"
        }
        println "shared task hello"
    }
}

project(':api') {
    dependencies {
        compile project(':shared')
    }
    task hello {
        doLast {
            println "api task hello -> doLast"
        }
        println "api task hello"
    }
}

project(':services:personService') {
    dependencies {
        compile project(':shared'), project(':api')
    }
    task hello {
        doLast {
            println "personService task hello -> doLast"
        }
        println "personService task hello"
    }
}
```

执行命令`gradle -i :services:personService:hello`，输出如下
> `-i` 是gradle的log等级的一种，输出较为详细的信息


```bash
Initialized native services in: /Users/zhangliang/.gradle/native
The client will now receive all logging from the daemon (pid: 27188). The daemon log file: /Users/zhangliang/.gradle/daemon/4.8.1/daemon-27188.out.log
Starting 4th build in daemon [uptime: 2 mins 55.761 secs, performance: 98%, no major garbage collections]
Using 8 worker leases.
Starting Build
Settings evaluated using settings file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/settings.gradle'.
Projects loaded. Root project using build file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/build.gradle'.
Included projects: [root project 'multi-project-build-lifecycle-demo', project ':api', project ':services', project ':shared', project ':services:personService']

> Configure project :
Evaluating root project 'multi-project-build-lifecycle-demo' using build file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/build.gradle'.
shared task hello
api task hello
personService task hello

> Configure project :api
Evaluating project ':api' using build file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/api/build.gradle'.

> Configure project :services
Evaluating project ':services' using build file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/services/build.gradle'.

> Configure project :shared
Evaluating project ':shared' using build file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/shared/build.gradle'.

> Configure project :services:personService
Evaluating project ':services:personService' using build file '/Users/zhangliang/Projects/tools/learn-gradle/multi-project-build-lifecycle-demo/services/personService/build.gradle'.
All projects evaluated.
Selected primary task ':services:personService:hello' from project :services:personService
Tasks to be executed: [task ':services:personService:hello']
:services:personService:hello (Thread[Task worker for ':',5,main]) started.

> Task :services:personService:hello
Task ':services:personService:hello' is not up-to-date because:
  Task has not declared any outputs despite executing actions.
personService task hello -> doLast
:services:personService:hello (Thread[Task worker for ':',5,main]) completed. Took 0.001 secs.

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```
从输出可以看出：
1. Gradle首先查找`settings.gradle`文件进行初始化，确定根项目目录并创建所有`include`的项目实例。其中包括`services `。
2. 配置`settings.gradle`中`include`的所有的项目。项目配置的顺序，首先是根项目，其次是子项目，多个子项目看来也是按照字母顺序依次进行的。
>当前配置排序是
>[root project 'multi-project-build-lifecycle-demo', project ':api', project ':services', project ':shared', project ':services:personService']
>当我将‘shared’项目重命名为‘ashared’后，配置排序变为
>[root project 'multi-project-build-lifecycle-demo', project ':api', project ':ashared', project ':services', project ':services:personService']


3. 最后执行任务`:services:personService:hello`。

-- -- --

<br>



# 示例-按需配置的多项目构建

参考文档[Build script of water (parent) project](https://docs.gradle.org/current/userguide/multi_project_builds.html#example_build_script_of_water_parent_project)

创建一个多项目工程，目录结构如下：

```bash
.
├── bluewhale/
├── build.gradle
├── krill/
└── settings.gradle
```

`settings.gradle`文件

```groovy
rootProject.name = 'water'
include 'bluewhale', 'krill'
```

`build.gradle`文件

```groovy
allprojects {
    task hello {
        doLast { task ->
            println "I'm $task.project.name"
        }
    }
}
```

根目录下执行`gradle -i :bluewhale:hello`，输出如下（省略部分无关信息）：

```bash
Initialized native services in: /Users/zhangliang/.gradle/native
Included projects: [root project 'water', project ':bluewhale', project ':krill']

> Configure project :
> Configure project :bluewhale
> Configure project :krill
All projects evaluated.

> Task :bluewhale:hello
I'm bluewhale
```

从输出可以看出，所有的项目都进行了配置。

根目录下执行`gradle -i :bluewhale:hello --configure-on-demand`，输出如下（省略部分无关信息）：

```bash
Initialized native services in: /Users/zhangliang/.gradle/native
Included projects: [root project 'water', project ':bluewhale', project ':krill']
Configuration on demand is an incubating feature.

> Configure project :
> Configure project :bluewhale
All projects evaluated.

> Task :bluewhale:hello
I'm bluewhale
```

上面的输出中出现了一句话"Configuration on demand is an incubating feature"，表示当前的“按需配置模式”还只是实验中的功能。从输出中可以看出，没有配置与任务无关的“krill”项目。

-- -- --

<br>



# 参考
1. [Gradle官网](https://gradle.org/)