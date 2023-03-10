# Spring Native 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-native-intro>

## 1.概观

在本文中，我们将了解本机映像以及如何从 Spring Boot 应用程序和 GraalVM 的本机映像构建器创建本机映像。我们参考了 Spring Boot 协议 3(当前版本是 3.0.0-RC2)，但是我们将在适当的时候解决与 Spring Boot 协议 2 的差异。

## 2.原生图像

本地映像是一种将 Java 代码构建成独立可执行文件的技术。该可执行文件包括应用程序类、其依赖关系中的类、运行时库类以及来自 JDK 的静态链接本机代码。JVM 被打包到本机映像中，因此在目标系统上不需要任何 Java 运行时环境，但是构建工件是平台相关的。因此，我们需要为每个受支持的目标系统构建一个版本，当我们使用像 Docker 这样的容器技术时，这将变得更容易，我们可以将容器构建为可以部署到任何 Docker 运行时的目标系统。

### 2.1.GraalVM 和本机映像构建器

`General Recursive Applicative and Algorithmic Language Virtual Machine` (Graal VM)是为 Java 和其他 JVM 语言编写的高性能 JDK 发行版，同时支持 JavaScript、Ruby、Python 和其他几种语言。它提供了一个`Native Image`构建器——一个从 Java 应用程序构建本机代码并将其与 VM 一起打包成独立的可执行文件的工具。它得到了 Spring Boot [Maven](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-boot/docs/3.0.0-RC2/maven-plugin/reference/htmlsingle/) 和 [Gradle](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-boot/docs/3.0.0-RC2/gradle-plugin/reference/htmlsingle/) 插件的官方支持，只有[几个例外](https://web.archive.org/web/20221126235145/https://github.com/spring-projects/spring-boot/wiki/Known-GraalVM-Native-Image-Limitations)(最糟糕的是，Mockito 目前不支持原生测试)。

### 2.2.特殊特点

在构建本机映像时，我们会遇到两个典型的特性。

`Ahead-Of-Time (AOT) Compilation`是将高级 Java 代码编译成本机可执行代码的过程。通常，这是由 JVM 的实时编译器(JIT)在运行时完成的，它允许在执行应用程序时进行观察和优化。在 AOT 汇编的情况下，这种优势就丧失了。

通常，在 AOT 编译之前，可选地有一个称为`AOT processing`的单独步骤，即从代码中收集元数据并将它们提供给 AOT 编译器。分成这两个步骤是有意义的，因为 AOT 处理可以是框架特定的，而 AOT 编译器更通用。下图给出了一个概述:

[![Overview: Native Build Steps](img/288a51cccac0e5c0ddea16d96fca3bc8.png)](/web/20221126235145/https://www.baeldung.com/wp-content/uploads/2021/06/build.png)

Java 平台的另一个特点是它在目标系统上的可扩展性，只需将 jar 放到类路径中。由于启动时的反射和注释扫描，我们在应用程序中获得了扩展的行为。

不幸的是，这减慢了启动时间，并且没有带来任何好处，特别是对于云原生应用程序，其中甚至服务器运行时和 Java 基类都被打包到 JAR 中。因此，我们放弃了这个特性，然后可以使用`Closed World Optimization`构建应用程序。

这两个特性都减少了运行时需要执行的工作量。

### 2.3.优势

**原生映像提供各种优势，如即时启动和减少内存消耗**。它们可以打包到一个轻量级容器映像中，以便更快、更有效地部署，并且它们提供的攻击面更小。

### 2.4.限制

由于封闭世界的优化，我们在编写应用程序代码和使用框架时必须意识到一些限制。简言之:

*   为了更快的启动和更好的峰值性能，类初始化器可以在构建时执行。但我们必须意识到，这可能会破坏代码中的一些假设，例如，当加载一个在构建时必须可用的文件时。
*   反射和动态代理在运行时开销很大，因此在封闭世界的假设下在构建时进行优化。当在构建时执行时，我们可以在类初始化器中无限制地使用它。任何其他用法都必须向 AOT 编译器声明，本机映像构建器试图通过执行静态代码分析来实现这一点。如果失败，我们必须提供该信息，例如通过[配置文件](https://web.archive.org/web/20221126235145/https://www.graalvm.org/22.1/reference-manual/native-image/BuildConfiguration/)。
*   这同样适用于所有基于反射的技术，如 JNI 和序列化。
*   此外，本机映像构建器提供了自己的本机接口，该接口比 JNI 简单得多，并且开销更低。
*   对于原生映像构建，字节码在运行时不再可用，因此不可能使用针对 JVMTI 的工具进行调试和监控。然后，我们必须使用本地调试器和监控工具。

**关于 Spring Boot，我们必须意识到像概要文件、条件 beans 和`.enable`属性这样的特性在运行时不再被[完全支持。](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-boot/docs/3.0.0-RC2/reference/htmlsingle/#native-image.introducing-graalvm-native-images.understanding-aot-processing)**如果我们使用概要文件，它们必须在构建时被指定。

## 3.基本设置

在构建本地映像之前，我们必须安装工具。

### 3.1.GraalVM 和本机映像

首先，我们按照[安装说明](https://web.archive.org/web/20221126235145/https://graalvm.github.io/native-build-tools/latest/graalvm-setup.html)安装当前版本的 GraalVM 和`native-image`构建器。(Spring Boot 要求 22.3 版本)我们应该确保安装目录通过`GRAALVM_HOME`环境变量可用，并且`“<GRAALVM_HOME>/bin”`被添加到`PATH` 变量中。

### 3.2 本机编译器

在构建期间，本机映像构建器调用特定于平台的本机编译器。因此，我们需要这个本地编译器，遵循我们平台的[“先决条件”指令](https://web.archive.org/web/20221126235145/https://www.graalvm.org/22.3/reference-manual/native-image/)。这将使构建依赖于平台。我们必须意识到，只能在特定于平台的命令行中运行构建。例如，在 Windows 上使用 Git Bash 运行构建是行不通的。我们需要使用 Windows 命令行来代替。

### 3.3 码头工人

作为先决条件，我们将确保安装 [Docker，稍后运行本机映像](/web/20221126235145/https://www.baeldung.com/dockerizing-spring-boot-application)需要它。Spring Boot Maven 和 Gradle 插件使用 [Paketo Tiny Builder](https://web.archive.org/web/20221126235145/https://github.com/paketo-buildpacks/tiny-builder) 构建一个容器。

## 4.使用 Spring Boot 配置和构建项目

使用 Spring Boot 的本地构建特性非常简单。我们创建我们的项目，例如，通过使用 [Spring Initializr](https://web.archive.org/web/20221126235145/https://start.spring.io/) 并添加应用程序代码。然后，要用 GraalVM 的原生映像构建器构建原生映像，我们需要用 GraalVM 本身提供的 Maven 或 Gradle 插件来扩展我们的构建。

### 4.1.专家

[Spring Boot Maven 插件](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-boot/docs/3.0.0-RC2/maven-plugin/reference/htmlsingle/)的目标是 AOT 处理(即，不是 AOT 编译本身，而是为 AOT 编译器收集元数据，例如，在代码中注册反射的用法)和构建可以用 Docker 运行的 OCI 映像。我们可以直接调用这些目标:

```java
mvn spring-boot:process-aot
mvn spring-boot:process-test-aot
mvn spring-boot:build-image
```

我们不需要这样做，因为 Spring Boot 的父 POM 定义了一个`native`概要文件，将这些目标绑定到构建中。我们需要使用这个激活的配置文件进行构建:

```java
mvn clean package -Pnative
```

如果我们还想执行本地测试，我们可以激活第二个概要文件:

```java
mvn clean package -Pnative,nativeTest
```

如果我们想要构建一个原生映像，我们必须添加相应的目标`native-maven-plugin`。因此，我们也可以定义一个`native`概要文件。因为这个插件是由父 POM 管理的，所以我们可以保留版本号:

```java
<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>build-native</id>
                            <goals>
                                <goal>compile-no-fork</goal>
                            </goals>
                            <phase>package</phase>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

目前，在本地测试执行中不支持 Mockito。因此，我们可以排除模拟测试，或者简单地跳过本地测试，将它添加到我们的 POM 中:

```java
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.graalvm.buildtools</groupId>
                <artifactId>native-maven-plugin</artifactId>
                <configuration>
                    <skipNativeTests>true</skipNativeTests>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

### 4.2.使用没有母体 POM 的 Spring Boot

如果我们不能从 Spring Boot 的父 POM 继承，而是把它作为一个 [`import`范围的依赖](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-boot/docs/3.0.0-RC2/maven-plugin/reference/htmlsingle/#using.import)，我们必须自己配置插件和配置文件。然后，我们必须将它添加到我们的 POM 中:

```java
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.graalvm.buildtools</groupId>
                <artifactId>native-maven-plugin</artifactId>
                <version>${native-build-tools-plugin.version}</version>
                <extensions>true</extensions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <image>
                            <builder>paketobuildpacks/builder:tiny</builder>
                            <env>
                                <BP_NATIVE_IMAGE>true</BP_NATIVE_IMAGE>
                            </env>
                        </image>
                    </configuration>
                    <executions>
                        <execution>
                            <id>process-aot</id>
                            <goals>
                                <goal>process-aot</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <configuration>
                        <classesDirectory>${project.build.outputDirectory}</classesDirectory>
                        <metadataRepository>
                            <enabled>true</enabled>
                        </metadataRepository>
                        <requiredVersion>22.3</requiredVersion>
                    </configuration>
                    <executions>
                        <execution>
                            <id>add-reachability-metadata</id>
                            <goals>
                                <goal>add-reachability-metadata</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>nativeTest</id>
        <dependencies>
            <dependency>
                <groupId>org.junit.platform</groupId>
                <artifactId>junit-platform-launcher</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>process-test-aot</id>
                            <goals>
                                <goal>process-test-aot</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <configuration>
                        <classesDirectory>${project.build.outputDirectory}</classesDirectory>
                        <metadataRepository>
                            <enabled>true</enabled>
                        </metadataRepository>
                        <requiredVersion>22.3</requiredVersion>
                    </configuration>
                    <executions>
                        <execution>
                            <id>native-test</id>
                            <goals>
                                <goal>test</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
<properties>
    <native-build-tools-plugin.version>0.9.17</native-build-tools-plugin.version>
</properties> 
```

### 4.3\. Gradle

[Spring Boot Gradle 插件](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-boot/docs/3.0.0-RC2/gradle-plugin/reference/htmlsingle/)为 AOT 处理(即，不是 AOT 编译本身，而是为 AOT 编译器收集元数据，例如，在代码中注册反射的使用)和构建可以用 Docker:

```java
gradle processAot
gradle processTestAot
gradle bootBuildImage
```

如果我们想要构建一个原生映像，我们必须为 GraalVM 原生映像构建添加 [Gradle 插件:](https://web.archive.org/web/20221126235145/https://graalvm.github.io/native-build-tools/latest/gradle-plugin.html)

```java
plugins {
    // ...
    id 'org.graalvm.buildtools.native' version '0.9.17'
}
```

然后，我们可以运行测试并通过调用

```java
gradle nativeTest
gradle nativeCompile
```

**目前，在本地测试执行中不支持 Mockito。**所以我们可以通过如下配置`graalvmNative`扩展来排除模拟测试或者跳过本地测试:

```java
graalvmNative {
    testSupport = false
}
```

## 5.扩展本机映像构建配置

正如已经提到的，我们必须注册反射、类路径扫描、动态代理等的每次使用。，用于 AOT 编译器。**因为 Spring 的内置原生支持是一个非常年轻的特性，目前并不是所有的 Spring 模块都有内置支持，所以我们目前需要自己添加这个。**这可以通过手动创建构建配置来完成，但使用 Spring Boot 提供的接口更容易，这样 Maven 和 Gradle 插件都可以在 AOT 处理期间使用我们的代码来生成构建配置。

指定附加本地配置的一种可能性是`Native Hints`。**在[目前可用的文档](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-framework/docs/6.0.0-RC4/reference/html/core.html#aot)中，这个主题仍然缺失，所以我们只能在[文档来源](https://web.archive.org/web/20221126235145/https://github.com/spring-projects/spring-framework/blob/main/framework-docs/src/docs/asciidoc/core/core-aot.adoc)中找到它(最后一段，“运行时提示”)。**那么，让我们来看两个目前缺失的内置支持的例子，以及如何将它添加到我们的应用程序中以使其工作。

### 5.1.样本:杰克逊的`PropertyNamingStrategy`

在 MVC web 应用程序中，REST 控制器方法的每个返回值都由 Jackson 序列化，并将每个属性自动命名为一个 JSON 元素。我们可以通过在应用程序属性文件中配置 Jackson 的 PropertyNamingStrategy 来全面影响名称映射:

```java
spring.jacksonproperty-naming-strategy=SNAKE_CASE
```

`SNAKE_CASE`是`PropertyNamingStrategies`类型的静态成员的名称。不幸的是，这个成员是通过反射解决的。所以 AOT 编译器需要知道这一点，否则，我们会得到一个错误消息:

```java
Caused by: java.lang.IllegalArgumentException: Constant named 'SNAKE_CASE' not found
  at org.springframework.util.Assert.notNull(Assert.java:219) ~[na:na]
  at org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration
        $Jackson2ObjectMapperBuilderCustomizerConfiguration
        $StandardJackson2ObjectMapperBuilderCustomizer.configurePropertyNamingStrategyField(JacksonAutoConfiguration.java:287) ~[spring-features.exe:na]
```

为此，我们可以用一种简单的方式实现并注册`RuntimeHintsRegistrar`,如下所示:

```java
@Configuration
@ImportRuntimeHints(JacksonRuntimeHints.PropertyNamingStrategyRegistrar.class)
public class JacksonRuntimeHints {

    static class PropertyNamingStrategyRegistrar implements RuntimeHintsRegistrar {

        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            try {
                hints
                  .reflection()
                  .registerField(PropertyNamingStrategies.class.getDeclaredField("SNAKE_CASE"));
            } catch (NoSuchFieldException e) {
                // ...
            }
        }
    }

}
```

从版本 3.0.0-RC2 开始，Spring Boot 内部解决此问题的[拉请求](https://web.archive.org/web/20221126235145/https://github.com/spring-projects/spring-boot/pull/33080)已经合并。

### 5.2.示例:GraphQL 模式文件

如果我们想[实现一个 GraphQL API](/web/20221126235145/https://www.baeldung.com/spring-graphql) ，我们需要创建一个模式文件，并将其放在`“classpath:/graphql/*.graphqls”`下，在这里它会被 Springs GraphQL 自动配置自动检测到。这是通过类路径扫描以及集成的 GraphiQL 测试客户端的欢迎页面来完成的。因此，为了在本机可执行文件中正确工作，AOT 编译器需要了解这一点。我们可以用同样的方式注册:

```java
@ImportRuntimeHints(GraphQlRuntimeHints.GraphQlResourcesRegistrar.class)
@Configuration
public class GraphQlRuntimeHints {

    static class GraphQlResourcesRegistrar implements RuntimeHintsRegistrar {

        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            hints.resources()
              .registerPattern("graphql/**/")
              .registerPattern("graphiql/index.html");
        }
    }

}
```

Spring GraphQL 团队已经在做这个了，所以我们可能会在未来的版本中内置这个。

## 6.写作测试

为了测试`RuntimeHintsRegistrar`实现，我们甚至不需要运行 Spring Boot 测试，我们可以创建一个简单的 JUnit 测试，如下所示:

```java
@Test
void shouldRegisterSnakeCasePropertyNamingStrategy() {
    // arrange
    final var hints = new RuntimeHints();
    final var expectSnakeCaseHint = RuntimeHintsPredicates
      .reflection()
      .onField(PropertyNamingStrategies.class, "SNAKE_CASE");
    // act
    new JacksonRuntimeHints.PropertyNamingStrategyRegistrar()
      .registerHints(hints, getClass().getClassLoader());
    // assert
    assertThat(expectSnakeCaseHint).accepts(hints);
}
```

如果我们想用集成测试来测试它，我们可以检查 Jackson `ObjectMapper`以获得正确的配置:

```java
@SpringBootTest
class JacksonAutoConfigurationIntegrationTest {

    @Autowired
    ObjectMapper mapper;

    @Test
    void shouldUseSnakeCasePropertyNamingStrategy() {
        assertThat(mapper.getPropertyNamingStrategy())
          .isSameAs(PropertyNamingStrategies.SNAKE_CASE);
    }

} 
```

为了用本地模式测试它，我们必须运行一个本地测试:

```java
# Maven
mvn clean package -Pnative,nativeTest
# Gradle
gradle nativeTest
```

如果我们需要为 Spring Boot 测试提供特定于测试的 AOT 支持，我们可以使用 [`AotTestExecutionListener`接口](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/aot/AotTestExecutionListener.html)实现一个 [`TestRuntimeHintsRegistrar`](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/aot/TestRuntimeHintsRegistrar.html) 或者一个`[TestExecutionListener](/web/20221126235145/https://www.baeldung.com/spring-testexecutionlistener)` 。我们可以在[官方文档](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-aot)中找到细节。

## 7.Spring Boot 2

Spring 6 和 Spring Boot 3 在原生映像构建方面迈出了一大步。但是有了之前的主要版本，这也是可以的。我们只需要知道还没有内置的支持，也就是说，有一个补充的 [Spring Native initiative](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/) 处理这个主题。因此，我们必须在我们的项目中手动包含和配置它。对于 AOT 处理，有一个单独的 Maven 和 Gradle 插件，它没有合并到 Spring Boot 插件中。当然，集成库没有像现在一样提供本地支持(将来会更好)。

### 7.1.Spring 本机依赖项

首先，我们必须为 Spring Native 添加 Maven 依赖性:

```java
<dependency>
    <groupId>org.springframework.experimental</groupId>
    <artifactId>spring-native</artifactId>
    <version>0.12.1</version>
</dependency>
```

然而，**对于 Gradle 项目，Spring Native 是由 Spring AOT 插件**自动添加的。

我们要注意的是**每个 Spring 原生版本只支持一个特定的 Spring Boot 版本**——比如 Spring 原生 0.12.1 只支持 Spring Boot 2.7.1。因此，我们应该确保在`pom.xml`中使用兼容的 Spring Boot Maven 依赖项。

### 7.2.构建包

为了构建 OCI 映像，我们需要显式地[配置一个构建包](/web/20221126235145/https://www.baeldung.com/spring-boot-docker-images#buildpacks)。

对于 Maven，我们将要求 [`spring-boot-maven-plugin`](https://web.archive.org/web/20221126235145/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-maven-plugin) 使用 [Paketo Java 构建包](https://web.archive.org/web/20221126235145/https://paketo.io/docs/buildpacks/language-family-buildpacks/java/)进行本机映像配置:

```java
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <image>
                        <builder>paketobuildpacks/builder:tiny</builder>
                        <env>
                            <BP_NATIVE_IMAGE>true</BP_NATIVE_IMAGE>
                        </env>
                    </image>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build> 
```

这里，**我们将使用各种可用构建器中的`tiny`构建器，比如`base`和`full`来构建一个本地映像**。此外，我们通过向`BP_NATIVE_IMAGE`环境变量提供`true`值来启用构建包。

类似地，当使用 Gradle 时，我们可以将`tiny`构建器和`BP_NATIVE_IMAGE`环境变量一起添加到`build.gradle`文件中:

```java
bootBuildImage {
    builder = "paketobuildpacks/builder:tiny"
    environment = [
        "BP_NATIVE_IMAGE" : "true"
    ]
}
```

### 7.3.Spring AOT 插件

接下来，我们需要添加 [Spring AOT](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/#spring-aot) 插件，该插件执行[提前转换](/web/20221126235145/https://www.baeldung.com/ahead-of-time-compilation)，有助于改善本机映像的占用空间和兼容性。

所以，让我们把最新的 [`spring-aot-maven-plugin`](https://web.archive.org/web/20221126235145/https://repo.spring.io/artifactory/release/org/springframework/experimental/spring-aot-maven-plugin/) Maven 依赖添加到我们的`pom.xml`:

```java
<plugin>
    <groupId>org.springframework.experimental</groupId>
    <artifactId>spring-aot-maven-plugin</artifactId>
    <version>0.12.1</version>
    <executions>
        <execution>
            <id>generate</id>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
</plugin> 
```

同样，对于一个 Gradle 项目，我们可以在`build.gradle`文件中添加最新的 [`org.springframework.experimental.aot`](https://web.archive.org/web/20221126235145/https://repo.spring.io/artifactory/release/org/springframework/experimental/aot/org.springframework.experimental.aot.gradle.plugin/) 依赖项:

```java
plugins {
    id 'org.springframework.experimental.aot' version '0.10.0'
}
```

此外，正如我们前面提到的，这将自动向 Gradle 项目添加 Spring 原生依赖项。

Spring AOT 插件为[提供了几个选项来确定源代码生成](https://web.archive.org/web/20221126235145/https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/#spring-aot-configuration)。例如，像`removeYamlSupport`和`removeJmxSupport`这样的选项分别移除 [Spring Boot Yaml](/web/20221126235145/https://www.baeldung.com/spring-yaml) 和 Spring Boot [JMX](/web/20221126235145/https://www.baeldung.com/java-management-extensions) 支持。

### 7.4.构建并运行映像

就是这样！我们准备使用 Maven 命令构建我们的 Spring Boot 项目的本机映像:

```java
$ mvn spring-boot:build-image
```

### 7.5.本机映像构建

接下来，我们将添加一个名为`native`的概要文件，它支持一些插件，如 [`native-maven-plugin`](https://web.archive.org/web/20221126235145/https://search.maven.org/search?q=g:org.graalvm.buildtools%20a:native-maven-plugin) 和`spring-boot-maven-plugin`:

```java
<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <version>0.9.17</version>
                    <executions>
                        <execution>
                            <id>build-native</id>
                            <goals>
                                <goal>build</goal>
                            </goals>
                            <phase>package</phase>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <classifier>exec</classifier>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

这个概要文件将在打包阶段从构建中调用`native-image`编译器。

但是，在使用 Gradle 时，我们会将最新的 [`org.graalvm.buildtools.native`](https://web.archive.org/web/20221126235145/https://search.maven.org/search?q=g:org.graalvm.buildtools.native%20a:org.graalvm.buildtools.native.gradle.plugin) 插件添加到`build.gradle`文件中:

```java
plugins {
    id 'org.graalvm.buildtools.native' version '0.9.17'
}
```

就是这样！我们准备通过在 Maven `package`命令中提供本机概要文件来构建我们的本机映像:

```java
mvn clean package -Pnative
```

## 8.结论

在本教程中，我们探索了使用 Spring Boot 和 GraalVM 的原生构建工具构建原生映像。我们了解了 Spring 的内置原生支持。

像往常一样，所有的代码实现都可以在 GitHub ( [Spring Boot 2 样本](https://web.archive.org/web/20221126235145/https://github.com/eugenp/tutorials/tree/master/spring-native))上获得`