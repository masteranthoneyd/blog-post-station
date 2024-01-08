# Maven 篇

# 前言

[***Maven***](https://maven.apache.org/) 作为 Java 项目构建工具已经非常成熟了, 当然有很多新项目也用了 [***Gradle***](https://gradle.org/), Maven 也从中吸取灵感开发了  [***Maven Daemon***](https://github.com/apache/maven-mvnd), 对于大型多模块项目的构建速度比传统 mvn 构建快很多, mvnd 也是基于 mvn, 所以简单安装后只需要将 mvn 命令替换成 mvnd 命令即可. 

下面子章节是一些前置知识.

## Maven 基本机制

Maven 的核心是围绕其[***生命流程***](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html), Maven 只负责执行**标准流程**, 构建能力由 Maven 插件提供.

![](https://image.cdn.yangbingdong.com/image/building-effective-java-enterprise-project/maven/96bffb74750a8fedf8dca61afcaba010-55b3f3.webp)

插件可以附着在 Maven 的某个 Phase 中, Maven 在执行到这个 Phase 时会调用该插件的 Goal 实现, 比如 enforce 插件的源码是这样的:

![](https://image.cdn.yangbingdong.com/image/building-effective-java-enterprise-project/maven/70570495341d56db9a5db53c1116d0f1-1daeed.jpg)

Maven 常用插件: ***[https://maven.apache.org/plugins/index.html](https://maven.apache.org/plugins/index.html)***

## 查看插件用法

在 Maven 构建过程中, 会用到很多插件, 对于插件的详细用法, 除了查看插件官网之外, 还可以通过 `help:describe` 来简单查看用法:

```
# 基本用法
mvn help:describe -Dplugin=${groupId}:${artifactId}

# 查看详细说明, 包括参数
mvn help:describe -Dplugin=${groupId}:${artifactId} -Ddetail

# 查看某个 goal 的说明
 mvn help:describe -Dplugin=${groupId}:${artifactId} -Ddetail -Dgoal=perform
```

* Maven 的一些常用插件

# 依赖管理

**对外聚合, 对内继承**. 对外提供 bom, 对内使用 parent.

当然, 对内也可以使用 dependencies bom, 外部引用此 bom 一定程度上能避免某些类库版本不兼容的情况. 参考 [***dubbo***](https://github.com/apache/dubbo) 工程中的 `dubbo-dependencies-bom` 模块.

通过 `dependencyManagement` 标签统一管理依赖版本:

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers</artifactId>
            <version>${testcontainers.version}</version>
        </dependency>
</dependencyManagement>
```

注意引入普通依赖以及 bom 依赖的区别, 如果是 bom 依赖, 需要指定标签:

* `type`: `pom`
* `scope`: `import`

**注意**: 通过 import 导入 bom **只会引入其中 `dependencyManagement` 的部分**, 是不会连同 build 中的信息也一起导入的, 注意自行不同 build 中的信息, 比如 plugin 以及 pluginManagement 标签里面的配置, 需要手动补充这些配置, 具体参考 `spring-boot-parent`.

> 当然也可以直接使用 spring-boot-starter-parent 作为父级 pom, 也有不少项目是这么做的, 方便省事, 通过 bom 的形式灵活度可控度会大一点.
> 也可以使用 apache 最为 parent, 相对来说侵入性会小一点, 包含了大部分打包会用到的插件管理.



插件可以通过 `pluginManagement` 管理(注意是抱在 `build` 标签中):

```
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>${maven-source-plugin.version}</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                        <phase>package</phase>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
<build>    
```

**注意**: 在 `pluginManagement` 中指定了额外配置, 子模块中也指定了额外配置, 最终是采取配置**合并**的策略, 如果配置的属性一样则以子模块的为准.



# 统一版本管理

Maven 从  3.5.0-beta-1 版本开始支持 [***Maven CI Friendly Versions***](https://maven.apache.org/maven-ci-friendly.html), 可以使用  `${revision}`, `${sha1}` 以及 `${changelist}` 作为 version 变量, 配合 [***flatten-maven-plugin***](https://www.mojohaus.org/flatten-maven-plugin/) 插件, 可以完美统一项目整体版本.

只需要在 parent pom 中定义一个 `revision` 的变量, 所有子模块都用这个变量即可, 编译时 flatten 会生成另外一个 pom 文件, 这个 pom 里面已经将 `revision` 替换成真实的版本值, 并且后续的构建都是基于找一份 pom 文件.

```
<groupId>${groupId}</groupId>
<artifactId>${artifactId}</artifactId>
<version>${revision}</version>

<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>flatten-maven-plugin</artifactId>
    <version>1.5.0</version>
    <configuration>
        <updatePomFile>true</updatePomFile>
        <flattenMode>resolveCiFriendliesOnly</flattenMode>
    </configuration>
    <executions>
        <execution>
            <id>flatten</id>
            <phase>process-resources</phase>
            <goals>
                <goal>flatten</goal>
            </goals>
        </execution>
        <execution>
            <id>flatten.clean</id>
            <phase>clean</phase>
            <goals>
                <goal>clean</goal>
            </goals>
        </execution>
    </executions>
</plugin>

```

注意: 使用这种方式管理版本, 会影响 Release pipeline, 详情看下面的 Release 管理.

# Release 管理

## 传统的 maven-release-plugin

> ***[https://maven.apache.org/maven-release/maven-release-plugin/index.html](https://maven.apache.org/maven-release/maven-release-plugin/index.html)***

这种方式简单粗暴, 这是一个比较重的插件, 可定制性比较低, 但用于一般项目的构建足以.

> 但请**注意**, 该插件与 flatten 插件是冲突的, 因为 release 插件运行的结果是直接将 revision 占位符替换成了真实的值, 这样会导致 flatten 插件毫无意义.

```
<project ...>
    ...
    <version>0.0.1-SNAPSHOT</version>

    <scm>
        <connection>scm:git:<<your-git-repo-url>></connection>
        <!-- developerConnection 需要具有读写权限, 而上面的 connection 是只读 -->
        <developerConnection>scm:git:<<your-git-repo-url>></developerConnection>
    </scm>

    <distributionManagement>
        <repository>
            <id>artifact-repository</id>
            <url><<your-artifact-repo-url>></url>
        </repository>
    </distributionManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>${maven-release-plugin.version}</version>
                <configuration>
                    <!-- 确保子模块版本保持一致 -->
                    <autoVersionSubmodules>true</autoVersionSubmodules>
                    <!-- 传递给 maven 的参数, -DskipTests表示跳过测试 -->
                    <arguments>-DskipTests</arguments>
                </configuration>
             </plugin>
        </plugins>
    </build>
    ...
</project>
```

```
mvn --batch-mode release:prepare release:perform -Darguments="-DskipTests"
```

* `--batch-mode`: 加上该参数可以跳过手动确认 tag 以及版本的流程
* `release:prepare`: 准备阶段, 比如将 SNAPSHOT 去掉, 然后提交到版本管理, 打上 tag, 生成 `release.properties`
* `release:perform`: 根据前面生成的 `release.properties` 指定构建, 比如 checkout release tag 进行打包并发布.
* `-Darguments="-DskipTests"`: 通过 `-Darguments` 传递 maven 构建时的参数, 这里的 `DskipTests`(跟上面 pom 里面的配置效果是一样的, 所以二选一即可) 代表跳过测试, 而不是像平常一样通过 `-Dmaven.test.skip=true`, 在 release 阶段使用这个是没用的.

## 更符合现代化持续构建的方案

根据以下两个链接的介绍:

* [***Maven Release Plugin: Dead and Buried***](https://axelfontaine.com/blog/dead-burried.html) 
* [***Maven Release Plugin together with cifriendly versions***](https://stackoverflow.com/questions/59641739/maven-release-plugin-together-with-cifriendly-versions)

Maven Release 对于大型项目的构建比较慢, 因为需要重复跑一些流程比如 compile 等, 另外它跟 [***Maven CI Friendly Versions***](https://maven.apache.org/maven-ci-friendly.html) 是冲突的, 当执行 `release:prepare` 后会将 revision 变量替换成真实的值.

所以提倡了一种更加符合**现代化**持续构建的方案, 铜火锅 [***Maven CI Friendly Versions***](https://maven.apache.org/maven-ci-friendly.html), [***flatten-maven-plugin***](https://www.mojohaus.org/flatten-maven-plugin/) 以及 [***maven-scm-plugin***](https://maven.apache.org/scm/maven-scm-plugin/), 加上现代的持续构建服务中一般都会维护版本号, 可以做到非常轻量级并且灵活配置的构建方法. 以下是一个 pom.xml 例子, flatten 的配置参考上面同一版本管理的章节:

```
<project ...>
    ...
    <version>${revision}</version>

    <properties>
        <!-- Sane default when no revision property is passed in from the commandline -->
        <revision>0.0.1-SNAPSHOT</revision>
    </properties>

    <scm>
        <connection>scm:git:<<your-git-repo-url>></connection>
        <!-- developerConnection 需要具有读写权限, 而上面的 connection 是只读 -->
        <developerConnection>scm:git:<<your-git-repo-url>></developerConnection>
    </scm>

    <distributionManagement>
        <repository>
            <id>artifact-repository</id>
            <url><<your-artifact-repo-url>></url>
        </repository>
    </distributionManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-scm-plugin</artifactId>
                <version>${maven-scm-plugin}</version>
                <configuration>
                    <tag>release-${project.version}</tag>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```

执行以下命令即可进行 release 操作啦:

```
mvn deploy scm:tag -Drevision=$BUILD_NUMBER
```

 其中 `BUILD_NUMBER` 是 CI 服务器提供的环境变量, 用于标识项目的当前内部版本号 .

如果想要更优雅得让开发知道当前项目版本号, 可以选择先更新 `pom.xml` 里面的 `revision` 的值并提交到 scm 中, 再进行 release 操作.

```
# 修改版本, 里面的版本号可动态替换
sed -i 's|<revision>0.0.1-SNAPSHOT</revision>|<revision>0.0.2-SNAPSHOT</revision>|' pom.xml

mvn -Dmessage="提交日志" scm:checkin scm:tag deploy
```

# Profile

> ***[https://maven.apache.org/guides/introduction/introduction-to-profiles.html](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)***

 Maven 的 profile 是一种用于在构建时修改 pom 的元素,  它们旨在用于不同的目标环境, 以提供等效但不同的参数. profile 中可包含  `properties`,  `dependencies` 以及  `plugins` 标签.

profile 可以声明在 `pom.xml`,  ` %USER_HOME%/.m2/settings.xml ` 或者 ` ${maven.home}/conf/settings.xml ` 中.

常用[***激活方式***](https://maven.apache.org/guides/introduction/introduction-to-profiles.html#Details_on_profile_activation):

* 在 Maven setting 中 通过 ` activeProfiles.activeByDefault` 标签激活, 比如 ` <activeProfile>profile-1</activeProfile> `

* 通过 `-P` 参数激活, `mvn package -Pprofile-1`

* 变量激活

  * ```
    <profiles>
      <profile>
        <activation>
          <jdk>1.4</jdk>
        </activation>
        ...
      </profile>
    </profiles>
    ```

  * ```
    <profiles>
      <profile>
        <activation>
          <property>
            <name>environment</name>
            <!-- value 是非必填的, 不填则存在 environment 这个变量就会触发 -->
            <value>test</value>
          </property>
        </activation>
        ...
      </profile>
    </profiles>
    ```

    





# 常用插件

## 编译插件 maven-compiler-plugin

> 内置插件
> ***[https://maven.apache.org/plugins/maven-compiler-plugin/index.html](https://maven.apache.org/plugins/maven-compiler-plugin/index.html)***

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <encoding>${encoding}</encoding>
        <parameters>true</parameters> <!-- 编译时保留参数信息, 便于反射式获取 -->
        <source>${maven.compiler.source}</source>
        <target>${maven.compiler.target}</target>
        <compilerArgs>--enable-preview</compilerArgs> <!-- 自定义编译参数, 此处启用预览功能 -->
    </configuration>
</plugin>
```

在 `properties` 中指定如下属性也能达到一样的效果:

```
<java.version>21</java.version>
<maven.compiler.source>${java.version}</maven.compiler.source>
<maven.compiler.target>${java.version}</maven.compiler.target>
```

但这个会影响全局, 包括用到这个属性的插件或者功能, 一般我会选择两者都配置上.

## 资源复制 maven-resources-plugin

> 内置插件
> ***[https://maven.apache.org/plugins/maven-resources-plugin/index.html](https://maven.apache.org/plugins/maven-resources-plugin/index.html)***

该插件用于复制一些资源到输出目录(默认情况下寻找 ` src/main/resources ` 里面的资源), 非 Maven 默认结构的资源也可以[***复制***](https://maven.apache.org/plugins/maven-resources-plugin/examples/copy-resources.html); 也可以给资源文件注入变量, 默认情况下会替换 `${}`.

Spring Boot 由于支持了在配置文件中使用一样的占位符 `${}`, 所以其修改了默认的行为, 使用 `@propertyName@` 这种形式:

```
<build>
    <resources>
        <resource>
            <directory>${basedir}/src/main/resources</directory>
            <!-- 替换变量 -->
            <filtering>true</filtering>
            <!-- 包含以下文件 -->
            <includes>
                <include>**/application*.yml</include>
                <include>**/application*.yaml</include>
                <include>**/application*.properties</include>
            </includes>
        </resource>
        <resource>
            <directory>${basedir}/src/main/resources</directory>
            <!-- 除了以下文件都包含进来, 因为上面对这些文件进行了包含并且替换变量 -->
            <excludes>
                <exclude>**/application*.yml</exclude>
                <exclude>**/application*.yaml</exclude>
                <exclude>**/application*.properties</exclude>
            </excludes>
        </resource>
    </resources>
</build>

<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <encoding>${encoding}</encoding>
                <propertiesEncoding>${project.build.sourceEncoding}</propertiesEncoding>
                <delimiters>
                    <!-- 使用@作为变量分隔符 -->
                    <delimiter>${resource.delimiter}</delimiter>
                </delimiters>
                <!-- 不使用默认的 ${} 形式, 需要跟上面的 delimiter 一起配置 -->
                <useDefaultDelimiters>false</useDefaultDelimiters>
            </configuration>
        </plugin>
</pluginManagement>
```

## 单元测试  maven-surefire-plugin 

> 内置插件
>
> ***[https://maven.apache.org/surefire/maven-surefire-plugin/plugin-info.html](https://maven.apache.org/surefire/maven-surefire-plugin/plugin-info.html)***

默认情况下, 该插件会自动特使符合一定命名规范的测试类:

- `**/Test*.java`
- `**/*Test.java`
- `**/*Tests.java`
- `**/*TestCase.java`

也可以手动 include 或者 exclude:

```
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>${maven-surefire-plugin.version}</version>
        <configuration>
          <includes>
            <include>Sample.java</include>
            <include>%regex[.*(Cat|Dog).*Test.*]</include>
          </includes>
          <excludes>
            <exclude>**/Abstract*.java</exclude>
            <exclude>**/TestSquare.java</exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```

## 集成测试 maven-failsafe-plugin

> ***[https://maven.apache.org/surefire/maven-failsafe-plugin/index.html](https://maven.apache.org/surefire/maven-failsafe-plugin/index.html)***

与 maven-surefire-plugin 不同, 该插件用于集成测试, 并且测试与构建是解耦的, 不会因为测试失败而导致构建失败, failsafe 这个名字就能体现这一点. 默认情况下, 会执行符合如下规则的测试类:

* ` **/IT*.java `
* ` **/*IT.java `
* ` **/*ITCase.java `

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <classesDirectory>${project.build.outputDirectory}</classesDirectory>
    </configuration>
</plugin>
```

## 约束检测 maven-enforcer-plugin

> ***[https://maven.apache.org/enforcer/maven-enforcer-plugin/](https://maven.apache.org/enforcer/maven-enforcer-plugin/)***

它提供了一些目标来执行规则, 以检查项目的环境约束是否符合要求. 这些约束可以包括 Maven 版本, JDK 版本, 操作系统等, 还有许多[***内置规则***](https://maven.apache.org/enforcer/enforcer-rules/index.html)和[***用户自定义规则***](https://maven.apache.org/enforcer/enforcer-api/writing-a-custom-rule.html)可供选择.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>${maven-enforcer-plugin.version}</version>
    <executions>
        <execution>
            <id>enforce-versions</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <bannedPlugins>
                        <level>WARN</level>
                        <excludes>
                            <exclude>org.apache.maven.plugins:maven-verifier-plugin</exclude>
                        </excludes>
                        <message>Please consider using the maven-invoker-plugin (http://maven.apache.org/plugins/maven-invoker-plugin/)</message>
                    </bannedPlugins>
                    <requireMavenVersion>
                        <version>3.9.0</version>
                    </requireMavenVersion>
                    <requireJavaVersion>
                        <version>21</version>
                    </requireJavaVersion>
                    <requireOS>
                        <family>unix</family>
                    </requireOS>
                </rules>
            </configuration>
        </execution>
        <execution>
            <id>enforce-banned-dependencies</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <bannedDependencies>
                        <exclude>org.apache.maven:badArtifact</exclude>
                        <searchTransitive>true</searchTransitive>
                    </bannedDependencies>
                </rules>
                <fail>true</fail>
            </configuration>
        </execution>
        <execution>
            <id>enforce-versions</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <banDuplicatePomDependencyVersions/>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 打包源码 maven-source-plugin

> ***[https://maven.apache.org/plugins/maven-source-plugin/](https://maven.apache.org/plugins/maven-source-plugin/)***

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>${maven-source-plugin.version}</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <goals>
                <goal>jar-no-fork</goal>
            </goals>
            <phase>package</phase>
        </execution>
    </executions>
</plugin>
```

## 生成文档 maven-javadoc-plugin

> ***[https://maven.apache.org/plugins/maven-javadoc-plugin/index.html](https://maven.apache.org/plugins/maven-javadoc-plugin/index.html)***

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>${maven-javadoc-plugin.version}</version>
    <executions>
        <execution>
            <id>attach-javadocs</id>
            <goals>
                <goal>jar</goal>
            </goals>
            <configuration>
                <source>${maven.compiler.source}</source>
                <detectJavaApiLink>false</detectJavaApiLink>
                <encoding>${file.encoding}</encoding>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 其他插件

#### 代码风格检测 maven-checkstyle-plugin

> ***[https://maven.apache.org/plugins/maven-checkstyle-plugin/](https://maven.apache.org/plugins/maven-checkstyle-plugin/)***
>
> [***CheckStyle 自定义编码规范***](https://blog.csdn.net/NOPR_W/article/details/100664526)

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>${maven-checkstyle-plugin.version}</version>
    <dependencies>
        <dependency>
            <groupId>com.puppycrawl.tools</groupId>
            <artifactId>checkstyle</artifactId>
            <version>${puppycrawl-tools-checkstyle.version}</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <id>checkstyle-validation</id>
            <phase>validate</phase>
            <inherited>true</inherited>
            <configuration>
                <skip>${disable.checks}</skip>
                <configLocation>checkstyle/checkstyle.xml</configLocation>
                <suppressionsLocation>checkstyle/checkstyle-suppressions.xml</suppressionsLocation>
                <encoding>UTF-8</encoding>
                <consoleOutput>true</consoleOutput>
                <propertyExpansion>
                    checkstyle.build.directory=${project.build.directory}
                </propertyExpansion>
                <includeTestSourceDirectory>${maven-checkstyle-plugin.includeTestSourceDirectory}
                </includeTestSourceDirectory>
                <failsOnError>${maven-checkstyle-plugin.failsOnError}</failsOnError>
                <failOnViolation>${maven-checkstyle-plugin.failOnViolation}</failOnViolation>
            </configuration>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```