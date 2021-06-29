---
title: Android组件化实践
date: 2019-04-11 09:02:21
tags:
- android
- 组件化
categories:
- Android开发
---

# 导引

主要参考以下两种组件化方案

 [知乎 Android 客户端组件化实践](<https://zhuanlan.zhihu.com/p/45374964>)

- 多工程多仓库。主工程通过 aar 依赖各个组件。
- 组件划分：主工程、业务组件（完整业务）、基础组件（基础业务）、基础SDK（业务无关）
- 组件解耦：公用代码处理、初始化（任务依赖）、通信（路由、接口、EventBus、组件 API 模块）。
- 组件半自动拆分。
- 联合编译完整包。动态引入组件。
- 包含子业务线的组件的处理。子业务模块独立配置、编译和运行，组合发布。

[网易友品 Android 客户端组件化演进](<https://zhuanlan.zhihu.com/p/59245063>)

- 通信：路由、服务和全局通知
- 路由。公共路由表、注解路由地址
- 接口。
- 拆分流程：建Module -> 移动业务代码到Module -> 根据编译报错修改 -> 接口解耦 -> 配置独立编译功能。
- 组件库的独立发布和维护；本地开发调试模式，开发时使用本地Module依赖，发布时使用远程ARR依赖。

---

# 实践

组件划分。主工程；业务组件（Home、Login、Wisdomsite、Attendance、PersonCenter）及其相对应的API组件；基础业务层（Common、Design等）；基础SDK（Tools、Net等）。

重构。从最底层开始抽离。

路由。使用Arouter。

业务组件间调用。使用相对应的API组件。

本地module与AAR依赖切换：当依赖无需改变时使用AAR依赖，当依赖需要更改时使用源码依赖。



### 通信-接口组件实现

- 创建业务组件A和对应的接口组件A-API，且组件A依赖组件A-API
- 在组件A-API里定义A需要对外暴露的接口，并用单例维护所有的接口变量
- 在组件A里实现组件A-API里的接口，并赋给A-API里的单例
- 其他组件B需要使用组件A的方法，则依赖组件A-API，并调用对应接口即可。

之前项目里有一个wisdomsite业务组件，以及对应的wisdomsite-api组件，现在其他业务组件需要跳转进入wisdomsite组件里，相关代码如下：

组件wisdomsite相关代码

```kotlin
// build.gradle
dependencies {
  	// 依赖wisdomsite-api组件
    implementation deps.business.wisdomsite_api
}

// 初始化器，在组件初始化时调用init方法
object WisdomsiteInitializer {
    internal lateinit var app: Application
    @JvmStatic
    fun init(app: Application) {
        this.app = app
      	// 初始化A-API组件里的接口
        initProvider()
      	// 其他...
    }

    private fun initProvider() {
      	// 实现A-API组件里的接口
        WisdomsiteProvider.wi = WisdomsiteInterfaceImpl()
    }

  	// 实现A-API组件里的接口
    class WisdomsiteInterfaceImpl : WisdomsiteInterface {
        override fun toWisdomsiteHome(userId: Long) {
            ARouter.getInstance()
                .build(WisdomsiteRoute.PATH_WISDOM_SITE_MAIN)
                .withLong("_userId", projectId)
                .navigation()
        }
    }
}
```

组件wisdomsite-api相关代码

```kotlin
// 组件wisdomsite需要对外暴露的接口
interface WisdomsiteInterface {
    // 主页
    fun toWisdomsiteHome(projectId: Long)
}

// 用一个单例维护接口
object WisdomsiteProvider {
    var wi: WisdomsiteInterface? = null
}
```

其他组件，需要进入wisdomsite组件

```kotlin
// build.gradle
dependencies {
  	// 依赖wisdomsite-api组件
    implementation deps.business.wisdomsite_api
}

// 跳转代码
WisdomsiteProvider.wi?.toWisdomsiteHome(123456789L);
```



### 动态依赖

新建名为`deps.gradle`的Gradle文件，在其中添加组件与AAR的对应关系；并在每个Module里引入该文件。

```groovy
// 其中key为各module的名称，value为上传至Maven仓库里AAR全称
// com.xxx.xxx为公司和部门名称，yyy为项目名称
// 项目基础模块
def xxx = [
    net         : 'com.xxx.xxx:android-net:1.0.1',
    utils       : 'com.xxx.xxx:android-utils:1.1.1',
    common  	: 'com.xxx.xxx:android-yyy-common:1.0.5',
    design  	: 'com.xxx.xxx:android-yyy-design:1.0.1',
]

// 项目业务模块
def xxxBusiness = [
    login     	: 'com.xxx.xxx:android-login:1.0.0',
    login_api	: 'com.xxx.xxx:android-login-api:1.0.0',
    news    	: 'com.xxx.xxx:android-wisdomsite:1.0.0',
    news-api	: 'com.xxx.xxx:android-wisdomsite-api:1.0.0',
]

ext.deps = [
    'xxx'        : xxx,
    'xxxBusiness': xxxBusiness,
]
```



`Settings.gradle`文件配置如下

```groovy
include ':app'

/** -----------组件化-动态构建----------- */
apply from: getRootDir().getAbsolutePath() + '/deps.gradle'

// 构建中使用源码的组件配置  默认依赖的组件都使用AAR
def useSource(String prj, String artifact) {
    include "$prj"
    ext.useSourceModuleConfigs.put("$prj", "$artifact")
}
// 构建中使用源码的组件配置  默认依赖的组件都使用AAR
ext.useSourceModuleConfigs = [:]
// 业务层
useSource(':wisdomsite_api', deps.xxxBusiness.wisdomsite_api)
useSource(':wisdomsite', deps.xxxBusiness.wisdomsite)
useSource(':login_api', deps.xxxBusiness.login_api)
useSource(':login', deps.xxxBusiness.login)
// 基础组件
useSource(':net', deps.xxx.net)
useSource(':common', deps.xxx.common)
useSource(':design', deps.xxx.design)
useSource(':utils', deps.xxx.utils)
println "use source modules -> ${ext.useSourceModuleConfigs.keySet()}"

// 这一步是使用源码替换AAR依赖
// 注释上面的某一行就可以使用其对应组件的AAR依赖
gradle.allprojects { project ->
    if (project == project.rootProject) {
        return
    }

    project.configurations.all {
        resolutionStrategy.dependencySubstitution {
            settings.ext.useSourceModuleConfigs.each { prj, artifact ->
                substitute module(artifact) with it.project(prj)
            }
        }
    }
}
```



### 其他注意事项

各组件需要单独维护，能生成AAR文件并上传至Maven仓库。

各模块之间的资源名不能相同，需要配置`resourcePrefix`。



### 简单开发规范

以wisdomsite组件为例

```bash
## 组件结构
-- data 本地和远程数据层
-- feature 业务层
-- helper 公共业务相关辅助类，如果仅仅是单个业务使用则写在那个业务的包里
-- utils 公共业务无关工具类
-- widgets 公共控件，如果仅仅是单个业务使用则写在那个业务的包里

## 业务模块划分
- main、video、machine、labor、environment、safety、quality

## 切图规范
- 只需要放@3x一张图片到drawable-xxhdpi文件即可
- 切图名称：module名称_ic/bg_业务模块名_功能名，如：wisdomsite_ic_home_empty

## 资源规范-string
- 命名：module名称_业务模块名_功能名，如：wisdomsite_labor_title

## 资源规范-color
- colors.xml中定义业务无关的颜色值，colors_feature.xml中使用前者的颜色定义业务相关的颜色值
- colors_feature.xml中的命名：module名称_业务模块名_功能名，如：wisdomsite_machine_error

## 空值展示
- *默认空值*定义在 `@string/wisdomsite_empty_text_filler`
- 在布局中，如果TextView的值可能为空，则默认展示*默认空值*
- 在代码中，给TextView赋值时必须要考虑到空值问题，Kotlin代码使用`EmptyTextFiller.kt`中的扩展函数进行空值处理。
```

---



# 参考

- [Android模块化实践](<https://juejin.im/post/5b44a0d76fb9a04f932fe147#heading-25>)
- [Android组件化方案](<https://blog.csdn.net/guiying712/article/details/55213884>)
- [Android 组件化最佳实践](<https://juejin.im/post/5b5f17976fb9a04fa775658d>)
- [从零开始的Android新项目11 - 组件化实践（1）](<https://zhuanlan.zhihu.com/p/23147164>)
- [AppJoint](https://juejin.im/post/5bb9c0d55188255c7566e1e2)
- [ARouter](https://github.com/alibaba/ARouter)
- [使用ARouter实现组件化](<https://juejin.im/post/58f79130a0bb9f006abb066b>)

