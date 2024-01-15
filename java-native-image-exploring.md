![](https://image.cdn.yangbingdong.com/image/java-native-image-exploring/5f1d2ca90862bab4fa5afe565f7dd070-696f8e.png)

# 前言

近年来一直会有一种声音出现: "Java 已死", 随着一些云原生语言的发展, Java 作为历史悠久的语言沉淀下来的经验毋庸置疑, 曾经实现 "Write Once, Run Anywhere" 的 Java 虚拟机却成为了云原生应用的负担, 在 GO, Rust 等语言写出来的应用不仅启动快, 占用资源也小, 这样下去 Java 真的会被逐渐边缘化吗? 我并不觉得, Java 能常年霸占[***编程语言排行版***](https://www.tiobe.com/tiobe-index/) Top N 可不是盖的, 对于构建复杂大型业务应用的能力应该是没有几个语言能媲美的, 正因为其具有面向对象的特性, 拟人化的思维, 加上它庞大的用户群和极其成熟的软件生态, 这在朝夕之间难以撼动, 但危机感依旧存在.

就在 2018 年 4 月，Oracle Labs 新公开了一项黑科技：***[Graal VM](https://www.graalvm.org/)***，从它的口号 "Run Programs Faster Anywhere" 就能感觉到一颗蓬勃的野心, 并且它提供了一种革命性的技术 [***Java Native Image***](https://www.graalvm.org/latest/reference-manual/native-image/) 以适应当今云原生时代.

# GraalVM Native Image

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

# Remark

* Windows 下建议使用 Docker 构建 Native Image, 避免出现奇奇怪怪的问题, Quarkus 以及 Spring Boot 都有支持.
* Windows 下是用 Docker Desktop 管理 Docker 的, 尽量多分配点资源, 否则容易出现编译慢甚至卡死的现象, 辛苦等半天结果编译失败了 T.T



GraalVM 生成本地可执行文件的过程。具体步骤如下：

1. 初始化：初始化必要的信息，如使用的编译器等。
2. 分析：对代码进行分析，统计类、字段、方法等的数量。
3. 构建宇宙：建立存在的整体模型。
4. 解析方法：对代码中的方法进行解析。
5. 内联方法：对方法进行内联。
6. 编译方法：将方法编译成机器码。
7. 生成图像：生成最终的可执行文件。

