# 加快 Spring Boot 启动时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-startup-speed>

## 1.介绍

在本教程中，我们将涵盖不同的配置和设置，可以帮助减少 Spring Boot 启动时间。首先，我们将讨论 Spring 的具体配置。其次，我们将讨论 Java 虚拟机选项。最后，我们将介绍如何利用 GraalVM 和本机映像编译来进一步减少启动时间。

## 2.春季调整

在开始之前，让我们设置一个测试应用程序。我们将使用 Spring Boot 版本 2.5.4，依赖于 Spring Web、Spring Actuator 和 Spring Security。在`pom.xml,` 中，我们将添加带有配置的`spring-boot-maven-plugin` ,以将我们的应用程序打包到一个 jar 文件中:

```
<plugin> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-maven-plugin</artifactId> 
    <version>${spring-boot.version}</version> 
    <configuration> 
        <finalName>springStartupApp</finalName> 
        <mainClass>com.baeldung.springStart.SpringStartApplication</mainClass> 
    </configuration> 
    <executions> 
        <execution> 
            <goals> 
                <goal>repackage</goal> 
            </goals> 
        </execution> 
    </executions> 
</plugin>
```

我们用标准的`java -jar` 命令运行 jar 文件，并监控应用程序的启动时间:

```
c.b.springStart.SpringStartApplication   : Started SpringStartApplication in 3.403 seconds (JVM running for 3.961) 
```

正如我们所见，我们的应用程序大约在 3.4 秒后启动。我们将用这段时间作为未来调整的参考。

### 2.1.惰性初始化

Spring 框架支持惰性初始化。 **[惰性初始化](/web/20220707143855/https://www.baeldung.com/spring-boot-lazy-initialization)意味着 Spring 不会在启动时创建所有的 beans。此外，Spring 不会注入任何依赖项，直到需要这个 bean。**从 Spring Boot 版本 2.2 开始。可以使用`application.properties`启用延迟初始化:

```
spring.main.lazy-initialization=true
```

在构建一个新的 jar 文件并像前面的例子一样启动它之后，新的启动时间稍微好一点:

```
 c.b.springStart.SpringStartApplication   : Started SpringStartApplication in 2.95 seconds (JVM running for 3.497) 
```

根据我们代码库的大小，延迟初始化可以显著减少启动时间。缩减取决于我们的应用程序的依赖图。

此外，在开发期间使用 DevTools 热重启功能时，惰性初始化也有好处。通过延迟初始化增加重启次数将使 JVM 能够更好地优化代码。

然而，惰性初始化有一些缺点。最大的缺点是应用程序对第一个请求的响应速度较慢。因为 Spring 需要时间来初始化所需的 beans，所以另一个缺点是我们可能会在启动时错过一些错误。这可能会在运行时导致`ClassNotFoundException `。

### 2.2.排除不必要的自动配置

Spring Boot 总是偏爱传统而不是结构。 **Spring 可能会初始化我们的应用程序不需要的 beans。**我们可以使用启动日志检查所有自动配置的 beans。将`application.properties`中的 `org.springframework.boot.autoconfigure` 的记录级别设置为调试:

```
logging.level.org.springframework.boot.autoconfigure=DEBUG
```

在日志中，我们将看到专门用于自动配置的新行，从以下内容开始:

```
============================
CONDITIONS EVALUATION REPORT
============================ 
```

使用此报告，我们可以排除应用程序的部分配置。为了排除部分配置，我们使用`@EnableAutoConfiguration` 注释:

```
@EnableAutoConfiguration(exclude = {JacksonAutoConfiguration.class, JvmMetricsAutoConfiguration.class, 
  LogbackMetricsAutoConfiguration.class, MetricsAutoConfiguration.class})
```

如果我们排除了 Jackson JSON 库和一些我们不使用的指标配置，我们可以节省一些启动时间:

```
c.b.springStart.SpringStartApplication   : Started SpringStartApplication in 3.183 seconds (JVM running for 3.732) 
```

### 2.3.其他小调整

Spring Boot 附带了一个嵌入式 servlet 容器。默认情况下，我们得到 Tomcat。虽然 Tomcat 在大多数情况下已经足够好了，但是其他 servlet 容器的性能可能会更好。[在](/web/20220707143855/https://www.baeldung.com/spring-boot-servlet-containers)的测试中，JBoss 的 Undertow 比 Tomcat 或 Jetty 表现更好。它需要的内存更少，平均响应时间更短。要切换到回流，我们需要改变`pom.xml`:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

在类路径扫描中可以进行以下小的改进。Spring 类路径扫描是快速动作。当我们有一个大的代码库时，我们可以通过创建一个静态索引来改善启动时间。**我们需要给`spring-context-indexer to generate the index.`** Spring 添加一个依赖项，它不需要任何额外的配置。在编译期间，Spring 将在`META-INF\spring.components`中创建一个额外的文件。Spring 会在启动时自动使用它:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-indexer</artifactId>
    <version>${spring.version}</version>
    <optional>true</optional>
</dependency>
```

由于我们只有一个 Spring 组件，这个调整在我们的测试中没有产生显著的结果。

**接下来，`application.properties`(或者说。yml)文件**。最常见的是在类路径根或 jar 文件所在的文件夹中。我们可以通过使用`spring.config.location` 参数设置显式路径来避免搜索多个位置，并节省几毫秒的搜索时间:

```
java -jar .\target\springStartupApp.jar --spring.config.location=classpath:/application.properties
```

最后，Spring Boot 提供了一些 MBeans 来使用 JMX 监控我们的应用程序。**完全关闭 JMX，避免创建这些 beans 的成本:**

```
spring.jmx.enabled=false
```

## 3.JVM 调整

### 3.1.`V`验证旗帜

这个标志设置字节码验证器模式。字节码验证提供了类的格式是否正确，是否符合 JVM 规范的约束。我们在启动时在 JVM 上设置了这个标志。

此标志有几个选项:

*   **`-Xverify`是默认值，启用对所有非引导加载程序类的验证。**
*   **`-Xverify:all`启用所有类别的验证。**这种设置将对初创公司产生显著的负面绩效影响。
*   **`-Xverify:none`(或`-Xnoverify`)。**该选项完全禁用验证器，并将显著减少启动时间。

我们可以在启动时传递这个标志:

```
java -jar -noverify .\target\springStartupApp.jar 
```

我们将收到来自 JVM 的警告，该选项已被否决。此外，启动时间也会减少:

```
 c.b.springStart.SpringStartApplication   : Started SpringStartApplication in 3.193 seconds (JVM running for 3.686) 
```

这个标志带来了一个重要的折衷。我们的应用程序可能会在运行时因错误而中断，而我们可以更早地捕捉到错误。这也是这个选项在 Java 13 中被标记为不推荐使用的原因之一。因此，它将在未来的版本中被删除。

### 3.2.`TieredCompilation`标志

Java 7 引入了[分层编译](/web/20220707143855/https://www.baeldung.com/jvm-tiered-compilation)。HotSpot 编译器将对代码使用不同级别的编译。

众所周知，Java 代码首先被解释成字节码。接下来，字节码被编译成机器码。这种转换发生在方法层。C1 编译器在一定数量的调用后编译一个方法。在更多的运行之后，C2 编译器编译它，进一步提高性能。

**使用`-XX:-TieredCompilation` 标志，我们可以禁用中间编译层。**这意味着我们的方法将被 C2 编译器解释或编译，以获得最大的优化。这不会导致启动速度下降。我们需要的是禁用 C2 编译。我们可以通过`-XX:TieredStopAtLevel=1` 选项做到这一点。结合`-noverify`标志，这可以减少启动时间。不幸的是，这将在后面的阶段降低 JIT 编译器的速度。

仅 TieredCompilation 标志就带来了坚实的改进:

```
 c.b.springStart.SpringStartApplication   : Started SpringStartApplication in 2.754 seconds (JVM running for 3.172) 
```

另外，同时运行本节中的两个标志可以进一步减少启动时间:

```
 java -jar -XX:TieredStopAtLevel=1 -noverify .\target\springStartupApp.jar
c.b.springStart.SpringStartApplication : Started SpringStartApplication in 2.537 seconds (JVM running for 2.912) 
```

## 4.春天本土

本机映像是使用提前编译程序编译并打包到可执行文件中的 Java 代码。它不需要 Java 来运行。因为没有 JVM 开销，所以生成的程序速度更快，对内存的依赖性更小。GraalVM 项目引入了本地映像和所需的构建工具。

**[Spring Native](/web/20220707143855/https://www.baeldung.com/spring-native-intro) 是一个实验性模块，支持使用 GraalVM 原生映像编译器对 Spring 应用程序进行原生编译。**提前编译器在构建期间执行多项任务，以减少启动时间(静态分析、删除未使用的代码、创建固定的类路径等)。).本机映像仍有一些限制:

*   它不支持所有的 Java 特性
*   反射需要特殊的配置
*   惰性类加载不可用
*   Windows 兼容性是一个问题。

要将一个应用程序编译成本地映像，我们需要将`spring-aot`和`spring-aot-maven-plugin`依赖项添加到`pom.xml.` 中，Maven 将在`target`文件夹中的`package`命令上创建本地映像。

## 5.结论

在本文中，我们探索了改善 Spring Boot 应用程序启动时间的不同方法。首先，我们介绍了有助于减少启动时间的各种 Spring 相关特性。接下来，我们展示了特定于 JVM 的选项。最后，我们介绍了 Spring 本机和本机映像创建。和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143855/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)