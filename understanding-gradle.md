# 什么是 Gradle

简单来说, Gradle 是一个构建工具, 类比 Maven 这些, 是一个相对现代化以及更加先进的构建工具, Spring Boot 如今也是基于 Gradle 进行构建的, 以及众多的 Kotlin 项目亦是如此.

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/6446a7098f8400ec7aeb5125be728e9a-dedd6c.png)

# Learning Basics

## Gradle Wrapper

***[Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)*** 是为了在 Gradle 快速迭代的背景下, 项目只用某个特定版本的 Gradle 来进行项目构建的工具, 它通过生成一个配置文件记录版本以及一个是用于特定平台的 gradlew 命令放到项目中, 使得项目无论在哪个环境都始终用同一个版本的 Gradle.

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/d1e62132f3376bdd09a173e5837ab069-272cab.png)

在项目根目录下, 通过下面命令生成 gradle wrapper, 之后使用生成的 gradlew 或者 gradlew.bat 操作 gradle wrapper:

```
gradle wrapper --gradle-version=8.8|latest \
--distribution-type=bin|all \
--gradle-distribution-url=https://mirrors.tencent.com/gradle/gradle-8.8-all.zip
```

## Command Line

***[Command-Line Interface Reference](https://docs.gradle.org/current/userguide/command_line_interface.html)***

一些有用的命令参数:

```
# 列出所有任务
gradle tasks

# 日志级别
-q, --quiet | -w, --warn | -i, --info | -d, --debug
```

## Build Lifecycle

***[Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html)***

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/60183b3970bda131a4faad40c899a1f9-04dfe6.png)

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/00527f7491a740d931c896588765d1b4-fb1335.png)

## Gradle-managed Directories

***[Gradle-managed Directories](https://docs.gradle.org/current/userguide/directory_layout.html)***

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/ca63586345da7973f6a95d8cb2842a63-4aac0e.png)

## Multi-Project Builds

***[Multi-Project Build Basics](https://docs.gradle.org/current/userguide/intro_multi_project_builds.html)***

The Gradle community has two standards for multi-project build structures:

1. **[Multi-Project Builds using buildSrc](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html#sec:using_buildsrc)** - where `buildSrc` is a subproject-like directory at the Gradle project root containing all the build logic.
2. **[Composite Builds](https://docs.gradle.org/current/userguide/composite_builds.html#composite_builds)** - a build that includes other builds where `build-logic` is a build directory at the Gradle project root containing reusable build logic.

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/48a722b5fdc5c1a20d50bc72be044cfb-53b840.png)

> Note: The downside of using buildSrc is that any change to it will invalidate every task in your project and require a rerun.

# Settings File

***[Writing Settings Files](https://docs.gradle.org/current/userguide/writing_settings_files.html)***

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/95af67aba7f4713d011cf5d442d6eced-bb5931.png)

*  Define the location of plugins .
*  Apply settings plugins.
*  Define the root project name.
*  Define dependency resolution strategies. 
*  Add subprojects to the build. 

# Build Script

***[Writing Build Scripts](https://docs.gradle.org/current/userguide/writing_build_scripts.html)***

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/5605e47b9daad7887287e441da7787c0-921c15.png)

*  Apply plugins to the build. 
*  Define the locations where dependencies can be found. 
  * 推荐在 **settings 文件** 中统一配置
*  Add dependencies. 
*  Set properties. 
*  Register and configure tasks. 
  * 推荐在 **convention plugin** 中管理

# Plugins

***[Understanding Plugins](https://docs.gradle.org/current/userguide/custom_plugins.html)***

3类插件:

* ***[Core Plugins](https://docs.gradle.org/current/userguide/plugin_reference.html)*** - plugins that come from Gradle.
* **Community Plugins** - plugins that come from [Gradle Plugin Portal](https://plugins.gradle.org/) or a public repository.
* **Local or Custom Plugins** - plugins that you develop yoursel

> ***[Convention Plugin](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html#sec:convention_plugins)***
>
> ***[Understanding Implementation Options for Plugins](https://docs.gradle.org/current/userguide/implementing_gradle_plugins.html)***

# Tasks

* ***[Understanding Tasks](https://docs.gradle.org/current/userguide/more_about_tasks.html)***
* ***[ Actionable and Lifecycle tasks ](https://docs.gradle.org/current/userguide/organizing_tasks.html)***

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/b67ece412c99223aea12aa894dada3c5-d18767.png)

# Dependency Management

***[Dependency Management Terminology](https://docs.gradle.org/current/userguide/dependency_management_terminology.html)***

![](https://image.cdn.yangbingdong.com/image/understanding-gradle/b022d4980dadcce8b214361de20fa65f-bc4d3d.png)

* 中心化配置 Repository: ***[Centralizing repositories declaration](https://docs.gradle.org/current/userguide/declaring_repositories.html#sub:centralized-repository-declaration)***
* 仓库认证: ***[HTTP(S) authentication schemes configuration](https://docs.gradle.org/current/userguide/declaring_repositories.html#sec:authentication_schemes)***
* 版本声明
  * ***[Version Catalog](https://docs.gradle.org/current/userguide/dependency_management_basics.html#version_catalog)***
  * ***[Dependency constraint](https://docs.gradle.org/current/userguide/dependency_management_terminology.html#sub:terminology_dependency_constraint)***
  * ***[Using bills of materials (BOMs)](https://docs.gradle.org/current/userguide/migrating_from_maven.html#migmvn:using_boms)***
  * ***[Lock Version](https://docs.gradle.org/current/userguide/dependency_locking.html)***
* 冲突解决: ***[Understanding dependency resolution](https://docs.gradle.org/current/userguide/dependency_resolution.html)***
  * ***[Upgrading versions](https://docs.gradle.org/current/userguide/dependency_resolution.html)***
  * ***[Downgrading versions](https://docs.gradle.org/current/userguide/dependency_downgrade_and_exclude.html)***

# Migrating Builds From Apache Maven

***[Migrating Builds From Apache Maven](https://docs.gradle.org/current/userguide/migrating_from_maven.html)***

# Highlight

**统一依赖源**: 使用 `dependencyResolutionManagement`, 比如:

```
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
    }
    includeBuild("../my-build-logic")
}

dependencyResolutionManagement {
    repositories {
        mavenCentral()
    }
    includeBuild("../my-other-project")
}
```



# 附录

* ***[How Gradle Works Part 1 - Startup](https://blog.gradle.org/how-gradle-works-1)***
* ***[How Gradle Works Part 2 - Inside The Daemon](https://blog.gradle.org/how-gradle-works-2)***
* ***[How Gradle Works Part 3 - Build Script](https://blog.gradle.org/how-gradle-works-3)***
* ***[understanding-gradle](https://github.com/jjohannes/understanding-gradle)***