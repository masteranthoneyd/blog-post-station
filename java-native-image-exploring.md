

![](https://image.cdn.yangbingdong.com/image/java-native-image-exploring/5f1d2ca90862bab4fa5afe565f7dd070-696f8e.png)

# 前言

近年来一直会有一种声音出现: "Java 已死", 随着一些云原生语言的发展, Java 作为历史悠久的语言沉淀下来的经验毋庸置疑, 曾经实现 "Write Once, Run Anywhere" 的 Java 虚拟机却成为了云原生应用的负担, 在 GO, Rust 等语言写出来的应用不仅启动快, 占用资源也小, 这样下去 Java 真的会被逐渐边缘化吗? 我并不觉得, Java 能常年霸占[***编程语言排行版***](https://www.tiobe.com/tiobe-index/) Top N 可不是盖的, 对于构建复杂大型业务应用的能力应该是没有几个语言能媲美的, 正因为其具有面向对象的特性, 拟人化的思维, 加上它庞大的用户群和极其成熟的软件生态, 这在朝夕之间难以撼动, 但危机感依旧存在.

就在 2018 年 4 月，Oracle Labs 新公开了一项黑科技：***[Graal VM](https://www.graalvm.org/)***，从它的口号 "Run Programs Faster Anywhere" 就能感觉到一颗蓬勃的野心, 并且它提供了一种革命性的技术 [***Java Native Image***](https://www.graalvm.org/latest/reference-manual/native-image/) 以适应当今云原生时代.

# What is GraalVM Native Image

> 官方介绍: [***GraalVM Overview***](https://www.graalvm.org/latest/docs/introduction/)
> 维基百科: [***GraalVM - Wiki***](https://en.wikipedia.org/wiki/GraalVM)

![](https://image.cdn.yangbingdong.com/image/java-native-image-exploring/d5d8d8de13ad1ebd0903e7c93922c102-8224c5.png)

 GraalVM 是一种**高性能的通用虚拟机**(意味着可以运行多种语言), 最早关于 GraalVM 工作可以追溯到 2012 年, 然而，最初的版本并没有立即成为公共可用的产品, GraalVM 项目于2018年逐渐成熟，GraalVM 19.0 是首个包含 **GraalVM Native Image** 的公开版本，这是在2019年1月发布的, GraalVM Native Image 提供了一种将 Java 程序编译成本地机器代码的方法, 也就是 AOT( Ahead-of-Time Compilation)，以便在启动和运行时获得更好的性能和资源利用率, 这使得 Java 应用程序能够更好地**适应云原生**和容器化环境.

**AOT的优点**:

- 在程序运行前编译，可以**避免在运行时的编译性能消耗和内存消耗**
- 可以**在程序运行初期就达到最高性能，程序启动速度快**
- 运行产物只有机器码，**打包体积小**

**AOT的缺点**:

- 由于是静态提前编译，不能根据硬件情况或程序运行情况择优选择机器指令序列，**理论峰值性能不如JIT**
- **没有动态能力**,  Java的动态特性, 包括反射, JNI, 代理都需要通过配置文件在构建前实现声明好 
- 同一份产物**不能跨平台运行**(不能交叉编译)

关于 GraalVM Native Image 更详细的介绍请看官网或者这篇文章: [***Revolutionizing Java with GraalVM Native Image***](https://www.infoq.com/articles/native-java-graalvm/)

# Getting Start

> 官方 Demo:  ***[graalvm-demos](https://github.com/graalvm/graalvm-demos)*** 

## Prerequisites

Native Image 的构建需要平台本地的一些类库支持, 取决于你在什么平台编译, 比如对于 Ubuntu 需要额外安装:
```
apt install build-essential libz-dev zlib1g-dev
```

对于 Windows 需要安装 Visual Studio, 有关不同平台配置和依赖更详细的说明, 请参考: [***Native Image Prerequisites*** ](https://www.graalvm.org/latest/reference-manual/native-image/#prerequisites)



## Build Tool: native-image 

> [***Guides - Build(graalvm.org)***](https://www.graalvm.org/latest/guides/?topic=build)

`native-image` 是 GraalVM SDK 用于镜像本地可运行二进制文件的工具.

相关文档:

* [***Build a Native Executable from a JAR File***](https://www.graalvm.org/latest/reference-manual/native-image/guides/build-native-executable-from-jar/)
* [***Build Java Modules into a Native Executable***](https://www.graalvm.org/latest/reference-manual/native-image/guides/build-java-modules-into-native-executable/)

通过 `native-image --help` 可以看出该工具支持 3 种构建方式: class/jar/module:

```
Usage: native-image [options] class [imagename] [options]
           (to build an image for a class)
   or  native-image [options] -jar jarfile [imagename] [options]
           (to build an image for a jar file)
   or  native-image [options] -m <module>[/<mainclass>] [options]
       native-image [options] --module <module>[/<mainclass>] [options]
           (to build an image for a module)
```

> Windows 中的命令可能是 `native-image.exe` 或者 `native-image.cmd`, 取决于你的安装方式

其他集成工具比如 Maven 插件原理上也是在项目构建时拼接参数传递给 native-image 工具.

### 参数解释

* `-cp`/`-classpath`/`--class-path`: 指定 `classpath`

* `-o`: 指定二进制镜像名

* `--static`: 构建静态链接的可执行文件, 适合用于什么依赖都没有的 `scratch` 镜像, 但还需要 `--lic` 参数支持

* `--libc`: 选择支持的库用于构建可执行文件, 支持的选项有 [`glibc`, `musl`, `bionic`]

* `--static-nolibc`: 构建 Mostly-Static 的可执行文件 

* `-O`: 优化二进制镜像的大小, 性能, 构建时间等, 参考: ***[Optimizations and Performance](https://www.graalvm.org/dev/reference-manual/native-image/optimizations-and-performance/)***
  * `--emit build-report`: 生成优化报告

* `--no-fallback`: build stand-alone image or report failure

* `--verbose`: 显示纤细的信息

* `-H:+ReportExceptionStackTraces`: 构建原生应用时输出详细错误信息

* `--pgo-instrument`: 构建用于生成 PGO 文件的 native executable, 默认文件名为 `default.iprof`, 可通过 `-XX:ProfilesDumpFile=xxx.iprof` 执行生成 PGO 文件的名字.

* `--pgo`: 构建通过 PGO 优化的 native executable, 后面加参数可指定 PGO 文件: `--pgo=xxx.iprof`

* `--shared`: 构建 shared library, 可用于 C 语言中, 详情见: ***[Build a Native Shared Library](https://www.graalvm.org/latest/reference-manual/native-image/guides/build-native-shared-library/)***

### Using native-image in Docker

如果不想在本地安装依赖, 尤其是 Windows 平台还是挺麻烦的, 这时候可以使用 Docker 来跑 native-image 命令. 原理也很简单, 制作一个包含 GraalVM JDK 21 以及 Native Image 构建依赖库的镜像即可. 下面是 Dockerfile 参考:

```
FROM yangbingdong/ubuntu24-sdkman:latest

SHELL ["bash", "--login", "-c"]
ADD GRAALVM_VERSION .
RUN apt-get update -y && apt-get install build-essential libz-dev zlib1g-dev -y
RUN sdk install java $(cat GRAALVM_VERSION) && sdk use java $(cat GRAALVM_VERSION)
ENV JAVA_HOME=/root/.sdkman/candidates/java/current
ENV PATH=$JAVA_HOME/bin:$PATH
RUN which native-image

ENTRYPOINT ["/bin/bash","--login", "-c"]
```

基于该镜像制作一个用于跑项目编译后 jar 包的 Native Image 镜像:

```
FROM yangbingdong/ubuntu24-graalvm21:latest as builder

WORKDIR /tmp/build
ADD . /tmp/build
RUN native-image -jar Hello.jar -o hello

FROM frolvlad/alpine-glibc:glibc-2.34
COPY --from=builder /tmp/build/hello /
ENTRYPOINT ["/hello"]
```

## 通过 Maven 插件构建

> [***Maven plugin for GraalVM Native Image building***](https://graalvm.github.io/native-build-tools/latest/maven-plugin.html)

### 基本使用

> 参考项目 ***[graalvm-demos/fortune-demo](https://github.com/graalvm/graalvm-demos/blob/master/fortune-demo/README.md)***

流程:

* 编写代码, 配置 pom native-maven-plugin
* 普通编译或者打包
  * `mvn clean compile`
* 运行 native agent 采集 native metadata
  * `mvn -Pnative -Dagent exec:exec@java-agent`
* 打包 native image
  * `mvn -Pnative -Dagent package`

插件配置:

```
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <!-- 启用拓展, 对 Junit 提供支持 -->
    <extensions>true</extensions>
    <executions>
        <execution>
            <id>build-native</id>
            <goals>
                <goal>compile-no-fork</goal>
            </goals>
            <phase>package</phase>
        </execution>
        <execution>
            <id>test-native</id>
            <goals>
                <goal>test</goal>
            </goals>
            <phase>test</phase>
        </execution>
    </executions>
    <configuration>
        <fallback>false</fallback>

        <!-- 编译后可执行文件的名字 -->
        <imageName>${imageName}</imageName>

        <!-- 启用 tracing agent 帮助生成 native image metadata, 默认是关闭的, 代替的, 也可以通过命令行添加 -Dagent=true 开启 -->
        <agent>
            <enabled>true</enabled>
            <defaultMode>Standard</defaultMode>
        </agent>
    </configuration>
</plugin>
```

### 插件参数

native-maven-plugin 会自动采集 `META-INF/native-image/` 路径下的[***配置文件***](https://www.graalvm.org/latest/reference-manual/native-image/overview/BuildConfiguration/), 这些配置也可以通过插件的 `<configuration>` 节点配置.

* `<mainClass>`: 如果执行时抛出 `no main manifest attribute, in target/<name>.jar` 错误信息, 那么需要手动配置这个 mainClass 到 native-maven-plugin 中. 一般情况下, native-maven-plugin 会自动感知 pom 里面的打包插件寻找里面的 mainClass, 插件优先级:

  * `<maven-shade-plugin> <transformers> <transformer> <mainClass>`
  * `<maven-assembly-plugin> <archive> <manifest> <mainClass>`
  * `<maven-jar-plugin> <archive> <manifest> <mainClass>`

* `<imageName>`: 生成二进制问价的名字, 不配置默认取项目的 artifact ID.

* `<dryRun>`: 只输出 native-image 命令, 不再往下执行.

* `<buildArgs>`: 传递额外的构建参数, 参数列表参考: ***[Native Image Build Options](https://www.graalvm.org/latest/reference-manual/native-image/overview/BuildOptions/)***

  * `<buildArgs><arg>--argument</arg></buildArgs>`

  * 该标签可以在父工程跟子工程之间进行组合: 

    * ```
      <!-- 父工程 -->
      <plugin>
        <groupId>org.graalvm.buildtools</groupId>
        <artifactId>native-maven-plugin</artifactId>
        <version>${current_plugin_version}</version>
        <configuration>
          <imageName>${project.artifactId}</imageName>
          <mainClass>${exec.mainClass}</mainClass>
          <buildArgs>
            <buildArg>--no-fallback</buildArg>
          </buildArgs>
        </configuration>
      </plugin>
      
      <!-- 子工程 -->
      <plugin>
        <groupId>org.graalvm.buildtools</groupId>
        <artifactId>native-maven-plugin</artifactId>
        <configuration>
          <buildArgs combine.children="append">
            <buildArg>--verbose</buildArg>
          </buildArgs>
        </configuration>
      </plugin>
      ```

* `<skipNativeBuild>`: 是否跳过 native 构建

* `<skipNativeTests>`: 是否跳过 native 测试

* `<debug>`: 是否生成 debug 信息

* `<verbose>`: 是否输出 verbose 信息

* `<sharedLibrary>`: 是否最为 shared library

* `<useArgFile>`: 是否使用 argument file

* `<quickBuild>`: 是否启用快速构建模式, 该模式适合开发阶段快速构建 native 镜像, 这类型镜像性能可能变弱因为舍弃了一部分代码的优化换取了构建时间, 以下是来自[***官网的原话***](https://www.graalvm.org/latest/reference-manual/native-image/overview/BuildOutput/#qbm-use-quick-build-mode-for-faster-builds):

  * > Consider using the quick build mode (`-Ob`) to speed up your builds during development. More precisely, this mode reduces the number of optimizations performed by the Graal compiler and thus reduces the overall time of the [compilation stage](https://www.graalvm.org/latest/reference-manual/native-image/overview/BuildOutput/#stage-compiling). The quick build mode is not only useful for development, it can also cause the generated executable file to be smaller in size. Note, however, that the overall peak throughput of the executable may be lower due to the reduced number of optimizations.

* `<excludeConfig>`: 排除的配置

  * ```
    <excludeConfig>
        <entry>
            <jarPath>dummy/path/to/file.jar</jarPath>
            <resourcePattern>*</resourcePattern>
        </entry>
    </excludeConfig>
    ```

* `<environment>`: 配置 native 构建时的环境变量

  * ```
    <environment>
        <variable>value</variable>
    </environment>
    ```

* `<systemPropertyVariables>`: 指定构建时的 system properties

  * ```
    <systemPropertyVariables>
        <propertyName>value</propertyName>
    </systemPropertyVariables>
    ```

* `<jvmArgs>`: 指定构建时的 JVM 参数

  * ```
    <jvmArgs>
        <arg>argument1</arg>
        <arg>argument2</arg>
    </jvmArgs>
    ```

* `<configurationFileDirectories>`: 自定义配置文件检索目录

  * ```
    <configurationFileDirectories>
        <dir>path/to/dir</dir>
    </configurationFileDirectories>
    ```

* `<classpath>`: 自定义 classpath

  * ```
    <classpath>
        <param>path/to/file.jar</param>
        <param>path/to/classes</param>
    </classpath>
    ```

* `<classesDirectory>`: 指定打包 JAR 的自定义路径，或只包含应用程序类的自定义目录，但希望插件仍能自动为依赖项添加 classpath 条目

  * ```
    <classesDirectory>
        path/to/dir
    </classesDirectory>
    ```

* `<agent>`: native agent tool 配置

  * ```
    <agent>
      <enabled>true</enabled>
    </agent>
    ```



### 动态特性支持

native image 不支持像 Java 反射, 动态代理等运行时才确定的因素, 因此需要通过在构建时告诉 native build tool 这些动态的代码或者配置.

#### native image agent 支持

native-image-agent 生成的配置文件默认放在了 `target/native/agent-output` 目录下, 建议将这些配置文件放到你的源码中.

启用 agent 支持:

* 通过命令行参数: `-Dagent=true`

* pom

  * ```
    <configuration>
      <agent>
        <enabled>true</enabled>
      </agent>
    </configuration>
    ```

以下是配置示例:

```
<configuration>
    <agent>
        <enabled>true</enabled>
        <defaultMode>Standard</defaultMode>
        <modes>
            <direct>config-output-dir=${project.build.directory}/native/agent-output</direct>
            <conditional>
                <userCodeFilterPath>user-code-filter.json</userCodeFilterPath>
                <extraFilterPath>extra-filter.json</extraFilterPath>
                <parallel>true</parallel>
            </conditional>
        </modes>
        <options>
            <callerFilterFiles>
                <filterFile>caller-filter-file1.json</filterFile>
                <filterFile>caller-filter-file2.json</filterFile>
            </callerFilterFiles>
            <accessFilterFiles>
                <filterFile>access-filter-file1.json</filterFile>
                <filterFile>access-filter-file2.json</filterFile>
            </accessFilterFiles>
            <builtinCallerFilter>true</builtinCallerFilter>
            <builtinHeuristicFilter>true</builtinHeuristicFilter>
            <enableExperimentalPredefinedClasses>true</enableExperimentalPredefinedClasses>
            <enableExperimentalUnsafeAllocationTracing>
                true
            </enableExperimentalUnsafeAllocationTracing>
            <trackReflectionMetadata>true</trackReflectionMetadata>
        </options>
        <metadataCopy>
            <disabledStages>
                <stage>main</stage>
            </disabledStages>
            <merge>true</merge>
            <outputDirectory>/tmp/test-output-dir</outputDirectory>
        </metadataCopy>
    </agent>
</configuration>
```

* ***[Caller-based Filters](https://www.graalvm.org/latest/reference-manual/native-image/metadata/AutomaticMetadataCollection/#caller-based-filters)*** 以及 ***[Access Filters](https://www.graalvm.org/latest/reference-manual/native-image/metadata/AutomaticMetadataCollection/#access-filters)*** 请看链接

* 参数:

  * `enabled`: 是否启用 agent
  * `defaultMode`, 有三种模式: 
    * `standard`: 这种模式下只能使用 `options` 标签配置 agent
    * `direct`: 这种模式下用户需要提供全量的 agent 配置, pom 中的剩下关于 agent 的配置将被忽略
    * `conditional`: 可以提供额外的 filter, 更多请看[***这里***](https://www.graalvm.org/latest/reference-manual/native-image/metadata/ExperimentalAgentOptions/)
  * `modes`: 对 direct 以及 conditional 模式起效, 参考上面的示例
  * `options`: agent 参数选项, 通用参数可以在[***这里***](https://www.graalvm.org/latest/reference-manual/native-image/metadata/AutomaticMetadataCollection/)找到.
  * `metadataCopy`: 操纵 agent 生成的 metadata 文件
    * `<outputDirectory>`: 复制 native 配置文件的目标文件夹
    * `<merge>`: 如果目标文件夹存在了配置文件, 是否进行合并, 否则覆盖
    * `<disabledStages>`: 在 main 阶段 或者 test 阶段禁用该复制操作

  > 按官方描述, metadataCopy 操作应该是在 agent 完成之后, 但本地测试后并没有进行复制操作, 需要额外在 mvn 构建命令中追加 `native:metadata-copy` 才行.

#### Reachability Metadata Support

添加如下配置:

```
<metadataRepository>
    <enabled>true</enabled>
    <version>0.3.6</version>
</metadataRepository>
```

## 基于 Buildpack 构建

Requirements:

* [***pack cli***](https://buildpacks.io/docs/install-pack/)

# 特性说明

##  Difference between Mostly Static and Fully Static 

通过上面给出的 Dockerfile 构建出来的二进制文件并不能直接跑在没有任何依赖的 `scratch` 镜像中, 至少需要有 `glibc` 库的支持, 这是因为这还不是一个 Fully Static Image, 默认情况下, 构建出来的是 dynamically linked binaries, 关于如何构建 Fully Static Image, 请参考: 

* ***[Build a Static or Mostly-Static Native Executable](https://www.graalvm.org/latest/reference-manual/native-image/guides/build-static-executables/)***
* ***[https://github.com/graalvm/graalvm-demos/tree/master/tiny-java-containers](https://github.com/graalvm/graalvm-demos/tree/master/tiny-java-containers)***


![](https://image.cdn.yangbingdong.com/image/java-native-image-exploring/8b5ce0bf2292e3782b8f329e64150f32-c1919f.png)

## Polyglot 多语言支持

GraalVM 支持在 Java 代码中使用其他的编程语言比如 Python/ Js 等, 并一同打包成 Native Image, 详情见文档: ***[Embedding Languages](https://www.graalvm.org/latest/reference-manual/embed-languages/)***

下面是一个[***官方例子***](https://www.graalvm.org/latest/reference-manual/native-image/guides/build-polyglot-native-executable/), 调用 Js 的 `JSON.stringify` 将输入的 json 格式化:

```
import java.io.*;
import java.util.stream.*;
import org.graalvm.polyglot.*;

public class PrettyPrintJSON {
  public static void main(String[] args) throws java.io.IOException {
    BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
    String input = reader.lines()
    .collect(Collectors.joining(System.lineSeparator()));
    try (Context context = Context.create("js")) {
      Value parse = context.eval("js", "JSON.parse");
      Value stringify = context.eval("js", "JSON.stringify");
      Value result = stringify.execute(parse.execute(input), null, 2);
      System.out.println(result.asString());
    }
  }
}
```

> 多语言支持下, 构建出来的 Native Image 会比正常的大, 因为包含了 Truffle framework.

# 优化

## UPX Compress 进行二进制文件压缩

构建出来的二进制镜像可以被 ***[UPX](https://github.com/upx/upx)*** 进一步压缩, 下面是一个来自官方的 ***[Demo](https://github.com/graalvm/graalvm-demos/blob/master/tiny-java-containers/helloworld/Dockerfile)***:

```
# Build in a container with Oracle GraalVM Native Image and MUSL
FROM container-registry.oracle.com/graalvm/native-image:23-muslib AS nativebuild
WORKDIR /build
# Install UPX 
ARG UPX_VERSION=4.2.2
ARG UPX_ARCHIVE=upx-${UPX_VERSION}-amd64_linux.tar.xz
RUN microdnf -y install wget xz && \
    wget -q https://github.com/upx/upx/releases/download/v${UPX_VERSION}/${UPX_ARCHIVE} && \
    tar -xJf ${UPX_ARCHIVE} && \
    rm -rf ${UPX_ARCHIVE} && \
    mv upx-${UPX_VERSION}-amd64_linux/upx . && \
    rm -rf upx-${UPX_VERSION}-amd64_linux

# Compile the Hello class to Java bytecode
COPY Hello.java Hello.java
RUN javac Hello.java
# Build a native executable with native-image
RUN native-image -Os --static --libc=musl Hello -o hello
RUN ls -lh hello

# Compress the executable with UPX
RUN ./upx --lzma --best -o hello.upx hello
RUN ls -lh hello.upx

# Copy the compressed executable into a scratch container
FROM scratch
COPY --from=nativebuild /build/hello.upx /hello.upx
ENTRYPOINT ["/hello.upx"]
```

**注意:** upx 虽然可以压缩执行文件大小, 但在运行时会**损耗额外的性能解压**, 越大的文件解压所需要的时间越久.

## PGO (Profile-Guided Optimization)

***[Profile-Guided Optimization](https://www.graalvm.org/latest/reference-manual/native-image/optimizations-and-performance/PGO/)***

需要生成两次 Native Image:

* 通过 `--pgo-instrument` 参数生成插桩的可执行文件, 通过运行该可执行文件生成 `default.iprof` 文件
* 通过 `--pgo` 应用 `default.iprof` 文件生成优化后的可执行文件.

# Remark

* Windows 下建议使用 Docker 构建 Native Image, 避免出现奇奇怪怪的问题, Quarkus 以及 Spring Boot 都有支持.
* Windows 下是用 Docker Desktop 管理 Docker 的, 尽量多分配点资源, 否则容易出现编译慢甚至卡死的现象, 辛苦等半天结果编译失败了
* GraalVM不支持交叉编译, 不过可以利用Windows提供的Linux子系统来编译源码。



GraalVM 生成本地可执行文件的过程。具体步骤如下：

1. 初始化：初始化必要的信息，如使用的编译器等。
2. 分析：对代码进行分析，统计类、字段、方法等的数量。
3. 构建宇宙：建立存在的整体模型。
4. 解析方法：对代码中的方法进行解析。
5. 内联方法：对方法进行内联。
6. 编译方法：将方法编译成机器码。
7. 生成图像：生成最终的可执行文件。



# Refs

* ***[Lab: GraalVM Native Image, Spring and Containerisation](https://luna.oracle.com/lab/fdfd090d-e52c-4481-a8de-dccecdca7d68)***



