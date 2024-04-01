# Gradle 安装

可以装 SDKMAN 的系统可以直接使用该工具安装:

```
sdk install gradle 8.7
```

Windows 用户需要手动下载配置环境变量:

* 下载地址: ***[https://gradle.org/releases/](https://gradle.org/releases/)***
* 变量
  * 新增 Gradle Home 路径变量: `GRADLE_HOME`, 值为 Gradle 目录路径, 比如 `D:\dev\tool\gradle\gradle-8.7`
  * Path 变量追加 `%GRADLE_HOME%\bin`
  * 新增 Gradle 本地仓库路径变量: `GRADLE_USER_HOME`, 比如 `D:\dev\tool\gradle\.gradle`

**注意**: **gradle 和 maven 是不能共用仓库目录的**, 因为两者的存放依赖的路径规则不一样, maven 是 `...\org\demo\v1\...`, 而 gradle 则是 `...\org.demo.v1...`, 但 gradle 中可以配置 `mavenLocal()` 让 gradle 从本地 maven 仓库中下载依赖.



配置镜像, 在 `$GRADLE_USER_HOME` 或者 `$GRADLE_HOME/init.d` 目录下新增  `init.gradle` 文件, 内容为:

```
settingsEvaluated { settings ->
    println "aliyun pluginManagement"
    settings.pluginManagement {
        repositories {
            maven { url "https://maven.aliyun.com/repository/gradle-plugin" }
            maven { url "https://maven.aliyun.com/repository/spring-plugin" }
            gradlePluginPortal()
        }
    }
}

allprojects{
    repositories {
        def ALIYUN_REPOSITORY_URL = 'https://maven.aliyun.com/repository/central/'
        def ALIYUN_JCENTER_URL = 'https://maven.aliyun.com/repository/public/'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo.maven.org/maven2') || url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('http://repo.maven.org/maven2') || url.startsWith('http://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com/') || url.startsWith('http://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                    remove repo
                }
            }
        }
        maven { url ALIYUN_REPOSITORY_URL }
        maven { url ALIYUN_JCENTER_URL }
        maven { url 'https://maven.aliyun.com/repository/spring' }
        maven { url 'https://maven.aliyun.com/repository/google/' }
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin/' }
        maven { url 'https://maven.aliyun.com/repository/jcenter/' }
    }


    buildscript{
        repositories {
            def ALIYUN_REPOSITORY_URL = 'https://maven.aliyun.com/repository/central/'
            def ALIYUN_JCENTER_URL = 'https://maven.aliyun.com/repository/public/'
            all { ArtifactRepository repo ->
                if(repo instanceof MavenArtifactRepository){
                    def url = repo.url.toString()
                    if (url.startsWith('https://repo.maven.org/maven2') || url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('http://repo.maven.org/maven2') || url.startsWith('http://repo1.maven.org/maven2')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                        remove repo
                    }
                    if (url.startsWith('https://jcenter.bintray.com/') || url.startsWith('http://jcenter.bintray.com/')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                        remove repo
                    }
                }
            }
            maven { url ALIYUN_REPOSITORY_URL }
            maven { url ALIYUN_JCENTER_URL }
            maven { url 'https://maven.aliyun.com/repository/spring' }
            maven { url 'https://maven.aliyun.com/repository/google/' }
            maven { url 'https://maven.aliyun.com/repository/gradle-plugin/' }
            maven { url 'https://maven.aliyun.com/repository/jcenter/' }
        }
    }
}
```

