# Gradle 6.0 的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-6-features>

## 1.概观

Gradle 6.0 版本带来了几个新的特性，这将有助于使我们的构建更加高效和健壮。这些功能包括改进的依赖性管理、模块元数据发布、任务配置避免和对 JDK 13 的支持。

在本教程中，我们将介绍 Gradle 6.0 中的新特性。我们的示例构建文件将使用 Gradle 的 Kotlin DSL。

## 2.依赖性管理改进

在最近几年的每一个版本中，Gradle 都对项目管理依赖关系的方式进行了不断的改进。这些依赖性的改进在 Gradle 6.0 中达到顶峰。让我们回顾一下现在稳定的依赖性管理改进。

### 2.1.API 和实现分离

`java-library`插件帮助我们创建一个可重用的 Java 库。该插件鼓励我们将属于我们库的公共 API 的依赖项与属于实现细节的依赖项分开。这种分离使得构建更加稳定，因为用户不会意外地引用不属于库的公共 API 的类型。

Gradle 3.4 中引入了`java-library`插件及其`api`和`implementation`配置。虽然这个插件对 Gradle 6.0 来说并不陌生，但它提供的增强的依赖关系管理功能是 Gradle 6.0 中实现的全面依赖关系管理的一部分。

### 2.2.丰富版本

我们的项目依赖图通常有同一个依赖的多个版本。当这种情况发生时，Gradle 需要选择项目最终将使用哪个版本的依赖项。

Gradle 6.0 允许我们将[丰富的版本信息](https://web.archive.org/web/20220630131547/https://docs.gradle.org/6.0.1/userguide/rich_versions.html)添加到依赖项中。**丰富的版本信息帮助 Gradle 在解决依赖冲突时做出最佳选择。**

例如，考虑一个依赖于番石榴的项目。进一步假设这个项目使用的是 Guava 版本 28.1-jre，即使我们知道它只使用自版本 10.0 以来就稳定的 Guava APIs。

我们可以使用`require`声明来告诉 Gradle，这个项目可以使用 10.0 以后的任何版本的 Guava，我们使用`prefer `声明来告诉 Gradle，如果没有其他约束阻止它这样做，它应该使用 28.1-jre。`because`声明添加了一个注释来解释这个丰富的版本信息:

```java
implementation("com.google.guava:guava") {
    version {
        require("10.0")
        prefer("28.1-jre")
        because("Uses APIs introduced in 10.0\. Tested with 28.1-jre")
    }
}
```

这如何帮助我们的构建更加稳定？假设这个项目还依赖于一个必须使用 16.0 版本的 Guava 的依赖项`foo`。`foo`项目的构建文件会将依赖关系声明为:

```java
dependencies {
    implementation("com.google.guava:guava:16.0")
}
```

由于`foo`项目依赖于 Guava 16.0，而我们的项目同时依赖于 Guava 版本 28.1-jre 和`foo`，我们就产生了冲突。 **Gradle 的默认行为是挑选最新版本。然而，在这种情况下，选择最新版本是错误的选择**，因为`foo`必须使用 16.0 版本。

在 Gradle 6.0 之前，用户必须自己解决冲突。因为 Gradle 6.0 允许我们告诉 Gradle，我们的项目可能会使用低至 10.0 的番石榴版本，所以 Gradle 会正确解决这个冲突，选择 16.0 版本。

除了`require`和`prefer`声明，我们还可以使用`strictly`和`reject`声明。`strictly`声明描述了我们的项目必须使用的依赖版本范围。`reject`声明描述了与我们的项目不兼容的依赖版本。

如果我们的项目依赖于一个我们知道将在 Guava 29 中删除的 API，那么我们使用`strictly`声明来阻止 Gradle 使用高于 28 的 Guava 版本。同样，如果我们知道在 Guava 27.0 中有一个 bug 给我们的项目带来了问题，我们使用`reject`来排除它:

```java
implementation("com.google.guava:guava") {
    version {
        strictly("[10.0, 28[")
        prefer("28.1-jre")
        reject("27.0")
        because("""
            Uses APIs introduced in 10.0 but removed in 29\. Tested with 28.1-jre.
            Known issues with 27.0
        """)
    }
}
```

### 2.3.平台

`java-platform`插件允许我们跨项目重用一组依赖约束。平台作者声明了一组紧密耦合的依赖项，其版本由平台控制。

**依赖平台的项目不需要为平台控制的任何依赖项指定版本。** Maven 用户会发现这类似于 Maven 母 POM 的`dependencyManagement`特性。

平台在多项目构建中特别有用。多项目构建中的每个项目可能使用相同的外部依赖项，我们不希望这些依赖项的版本不同步。

让我们创建一个新平台，以确保我们的多项目构建跨项目使用相同版本的 Apache HTTP Client。首先，我们创建一个使用`java-platform`插件的项目`httpclient-platform,`:

```java
plugins {
    `java-platform`
}
```

接下来，我们**声明这个平台中包含的依赖关系**的约束。在本例中，我们将选择希望在项目中使用的 Apache HTTP 组件的版本:

```java
dependencies {
    constraints {
        api("org.apache.httpcomponents:fluent-hc:4.5.10")
        api("org.apache.httpcomponents:httpclient:4.5.10")
    }
}
```

最后，让我们添加一个使用 Apache HTTP 客户端 Fluent API 的`person-rest-client`项目。这里，我们使用`platform`方法在我们的`httpclient-platform`项目上添加一个依赖项。我们还将添加对`org.apache.httpcomponents:fluent-hc`的依赖。这种依赖性不包括版本，因为`httpclient-platform`决定了要使用的版本:

```java
plugins {
    `java-library`
}

dependencies {
    api(platform(project(":httpclient-platform")))
    implementation("org.apache.httpcomponents:fluent-hc")
}
```

**`java-platform`插件有助于避免运行时由于构建中未对齐的依赖而导致的不受欢迎的意外。**

### 2.4.测试夹具

在 Gradle 6.0 之前，希望跨项目共享测试夹具的构建作者将这些夹具提取到另一个库项目中。现在，**构建作者可以使用`java-test-fixtures`插件发布他们项目中的测试夹具。**

让我们构建一个库，它定义了一个抽象，并发布了验证该抽象所期望的契约的测试装置。

在这个例子中，我们的抽象是一个 Fibonacci 序列生成器，测试设备是一个 [JUnit 5 测试混合](https://web.archive.org/web/20220630131547/https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-interfaces-and-default-methods)。斐波那契序列生成器的实现者可以使用测试混合来验证他们已经正确地实现了序列生成器。

首先，让我们为我们的抽象和测试设备创建一个新项目`fibonacci-spi`。这个项目需要`java-library`和`java-test-fixtures`插件:

```java
plugins {
    `java-library`
    `java-test-fixtures`
}
```

接下来，让我们将 JUnit 5 依赖项添加到我们的测试设备中。正如`java-library`插件定义了`api`和`implementation`配置一样，`java-test-fixtures`插件定义了`testFixturesApi`和`testFixturesImplementation`配置:

```java
dependencies {
    testFixturesApi("org.junit.jupiter:junit-jupiter-api:5.8.1")
    testFixturesImplementation("org.junit.jupiter:junit-jupiter-engine:5.8.1")
}
```

有了依赖项，让我们将 JUnit 5 测试混合添加到由`java-test-fixtures`插件创建的`src/testFixtures/java`源集中。这个测试混合验证了我们的`FibonacciSequenceGenerator`抽象的契约:

```java
public interface FibonacciSequenceGeneratorFixture {

    FibonacciSequenceGenerator provide();

    @Test
    default void whenSequenceIndexIsNegative_thenThrows() {
        FibonacciSequenceGenerator generator = provide();
        assertThrows(IllegalArgumentException.class, () -> generator.generate(-1));
    }

    @Test
    default void whenGivenIndex_thenGeneratesFibonacciNumber() {
        FibonacciSequenceGenerator generator = provide();
        int[] sequence = { 0, 1, 1, 2, 3, 5, 8 };
        for (int i = 0; i < sequence.length; i++) {
            assertEquals(sequence[i], generator.generate(i));
        }
    }
}
```

这就是我们需要做的所有事情，以便**与其他项目**共享这个测试夹具。

现在，让我们创建一个新项目`fibonacci-recursive`，它将重用这个测试夹具。这个项目将在我们的`dependencies`块中使用`testFixtures`方法声明对来自我们的`fibonacci-spi`项目的测试夹具的依赖:

```java
dependencies {
    api(project(":fibonacci-spi"))

    testImplementation(testFixtures(project(":fibonacci-spi")))
    testImplementation("org.junit.jupiter:junit-jupiter-api:5.8.1")
    testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine:5.8.1")
}
```

最后，我们现在可以使用在`fibonacci-spi`项目中定义的测试混合来为我们的递归 fibonacci 序列生成器创建一个新的测试:

```java
class RecursiveFibonacciUnitTest implements FibonacciSequenceGeneratorFixture {
    @Override
    public FibonacciSequenceGenerator provide() {
        return new RecursiveFibonacci();
    }
}
```

Gradle 6.0 `java-test-fixtures`插件**给了构建作者更多的灵活性来跨项目共享他们的测试夹具**。

## 3.分级模块元数据发布

传统上，Gradle 项目将构建工件发布到 Ivy 或 Maven 存储库中。这包括分别生成 ivy.xml 或 pom.xml 元数据文件。

ivy.xml 和 pom.xml 模型不能存储我们在本文中讨论过的丰富的依赖信息。这意味着当我们将我们的库发布到 Maven 或 Ivy 库时,,下游项目不会从这些丰富的依赖信息中受益。

Gradle 6.0 通过引入 [Gradle 模块元数据规范](https://web.archive.org/web/20220630131547/https://github.com/gradle/gradle/blob/master/subprojects/docs/src/docs/design/gradle-module-metadata-latest-specification.md)解决了这个问题。**Gradle 模块元数据规范是一种 JSON 格式，支持存储 Gradle 6.0 中引入的所有增强模块依赖元数据。**

除了传统的 ivy.xml 和 pom.xml 元数据文件之外，项目还可以构建这个元数据文件并将其发布到 Ivy 和 Maven 存储库。这种**向后兼容性允许 Gradle 6.0 项目在不破坏遗留工具的情况下利用这个模块元数据(如果它存在的话)**。

为了发布 Gradle 模块元数据文件，项目必须使用新的 [Maven 发布插件](https://web.archive.org/web/20220630131547/https://docs.gradle.org/current/userguide/publishing_maven.html)或 [Ivy 发布插件](https://web.archive.org/web/20220630131547/https://docs.gradle.org/current/userguide/publishing_ivy.html)。从 Gradle 6.0 开始，这些插件默认发布 Gradle 模块元数据文件。这些插件取代了[的传统出版系统。](https://web.archive.org/web/20220630131547/https://docs.gradle.org/current/userguide/artifact_management.html)

### 3.1.将梯度模块元数据发布到 Maven

让我们配置一个 build 来向 Maven 发布 Gradle 模块元数据。首先，我们将`maven-publish`包含在我们的构建文件中:

```java
plugins {
    `java-library`
    `maven-publish`
}
```

接下来，我们配置一个发布。一个出版物可以包含任意数量的工件。让我们添加与`java`配置相关联的工件:

```java
publishing {
    publications {
        register("mavenJava", MavenPublication::class) {
            from(components["java"])
        }
    }
}
```

`maven-publish`插件增加了`publishToMavenLocal`任务。让我们使用此任务来测试我们的 Gradle 模块元数据发布:

```java
./gradlew publishToMavenLocal
```

接下来，让我们列出这个工件在本地 Maven 存储库中的目录:

```java
ls ~/.m2/repository/com/baeldung/gradle-6/1.0.0/
gradle-6-1.0.0.jar	gradle-6-1.0.0.module	gradle-6-1.0.0.pom
```

正如我们在控制台输出中看到的，除了 Maven POM 之外，Gradle 还生成了模块元数据文件。

## 4.配置避免 API

从版本 5.1 开始，Gradle 鼓励插件开发者使用新的、正在酝酿的配置回避 API。这些 API 有助于构建尽可能避免相对缓慢的任务配置步骤。Gradle 称这种性能改进为[任务配置避免](https://web.archive.org/web/20220630131547/https://docs.gradle.org/current/userguide/task_configuration_avoidance.html)。Gradle 6.0 把这个孵化 API 提升到稳定。

虽然配置避免特性主要影响插件作者，但是在构建中创建任何自定义`Configuration`、`Task`或`Property`的构建作者也会受到影响。插件作者和构建作者一样，现在**可以使用新的[惰性配置 API](https://web.archive.org/web/20220630131547/https://docs.gradle.org/current/userguide/lazy_configuration.html)用`Provider`类型包装对象，这样 Gradle 将避免“实现”这些对象，直到需要它们的时候**。

让我们使用惰性 API 添加一个定制任务。首先，我们使用`TaskContainer.registering`扩展方法注册任务。因为`registering`返回一个`TaskProvider`，所以`Task`实例的创建被推迟，直到 Gradle 或构建作者调用`TaskProvider.get()`。最后，我们提供了一个闭包，它将在 Gradle 创建之后配置我们的`Task`:

```java
val copyExtraLibs by tasks.registering(Copy::class) {
    from(extralibs)
    into(extraLibsDir)
}
```

Gradle 的任务配置避免[迁移指南](https://web.archive.org/web/20220630131547/https://docs.gradle.org/current/userguide/task_configuration_avoidance.html#sec:task_configuration_avoidance_migration_guidelines)帮助插件作者和构建作者迁移到新的 API。构建作者最常见的迁移包括:

*   `tasks.register`而不是`tasks.create`
*   `tasks.named`而不是`tasks.getByName`
*   `configurations.register`而不是`configurations.create`
*   `project.layout.buildDirectory.dir(“foo”)`而不是`File(project.buildDir, “foo”)`

## 5.JDK 13 支持

Gradle 6.0 引入了对 JDK 13 构建项目的支持。我们可以通过熟悉的`sourceCompatibility`和`targetCompatibility`设置配置我们的 Java 构建来使用 Java 13:

```java
sourceCompatibility = JavaVersion.VERSION_13
targetCompatibility = JavaVersion.VERSION_13
```

JDK 13 的一些最令人兴奋的语言功能，如原始字符串文字，仍处于预览状态。让我们配置 Java 构建中的任务，以启用这些预览特性:

```java
tasks.compileJava {
    options.compilerArgs.add("--enable-preview")
}
tasks.test {
    jvmArgs.add("--enable-preview")
}
tasks.javadoc {
    val javadocOptions = options as CoreJavadocOptions
    javadocOptions.addStringOption("source", "13")
    javadocOptions.addBooleanOption("-enable-preview", true)
}
```

## 6.结论

在本文中，我们讨论了 Gradle 6.0 中的一些新特性。

我们讨论了增强的依赖性管理、发布 Gradle 模块元数据、任务配置避免，以及早期采用者如何配置他们的构建以使用 Java 13 preview 语言特性。

和往常一样，这篇文章的代码在 GitHub 上[。](https://web.archive.org/web/20220630131547/https://github.com/eugenp/tutorials/tree/master/gradle-6)