# Spring Boot 2 有什么新功能？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/new-spring-boot-2>

## 1。概述

Spring Boot 给春天的生态系统带来了一种自以为是的方法。2014 年年中首次发布。Spring Boot 经历了许多发展和改进。它的 2.0 版本今天准备在 2018 年初发布。

这个受欢迎的图书馆试图在不同的领域帮助我们:

*   依赖性管理。通过启动器和各种包管理器集成
*   自动配置。尝试最小化 Spring 应用程序需要的配置量，并且支持约定胜于配置
*   生产就绪功能。例如`Actuator`、更好的日志记录、监控、指标或各种 PAAS 集成
*   增强的开发体验。使用多个测试工具或更好的反馈回路

在本文中，我们将探讨 Spring Boot 2.0 中的一些变化和特性。我们还将描述这些变化如何帮助我们变得更有效率。

## 2。依赖性

### 2.1。Java 基线

**Spring Boot 2.x 将不再支持 Java 7 及更低版本**，Java 8 是最低要求。

它也是第一个支持 Java 9 的版本。没有计划在 1.x 分支上支持 Java 9。这意味着**如果你想使用最新的 Java 版本并利用这个框架，Spring Boot 2.x 是你唯一的选择**。

### 2.2。材料清单

随着 Spring Boot 的每一个新版本，Java 生态系统的各种依赖关系的版本都会升级。这是在开机[中定义的`bill of materials`又名`BOM`。](/web/20220625221415/https://www.baeldung.com/spring-maven-bom)

在 2.x 中也不例外。列出它们是没有意义的，但是我们可以看一下 [`spring-boot-dependencies.pom`](https://web.archive.org/web/20220625221415/https://github.com/spring-projects/spring-boot/blob/2.0.x/spring-boot-project/spring-boot-dependencies/pom.xml) 来了解在任何给定的时间点都在使用什么版本。

关于最低要求版本的几个要点:

*   Tomcat 支持的最低版本是 8.5
*   Hibernate 支持的最低版本是 5.2
*   Gradle 支持的最低版本是 3.4

### 2.3。Gradle 插件

Gradle 插件经历了重大的改进和一些突破性的改变。

**制造胖罐子，`bootRepackage` 格拉德的任务被替换为`bootJar` 和`bootWar`** 分别制造罐子和战争。

如果我们想用 Gradle 插件运行我们的应用程序，在 1.x 中，我们可以执行 2.x 中的`gradle bootRun.`**`bootRun` 扩展 Gradle 的`JavaExec.`** 这意味着我们可以更容易地配置它，应用我们通常在经典`JavaExec` 任务中使用的相同配置。

有时我们发现自己想利用 Spring Boot·邦。但有时我们不想构建一个完整的启动应用程序或重新打包它。

在这方面，有趣的是知道 **Spring Boot 2.x 将不再默认应用依赖管理插件**。

如果我们想要 Spring Boot 依赖管理，我们应该添加:

```java
apply plugin: 'io.spring.dependency-management'
```

在上述场景中，这为我们提供了更大的灵活性和更少的配置。

## 3。自动配置

### 3.1。安全性

在 2.x 中，安全配置得到了极大的简化。默认情况下，一切都是安全的，包括静态资源和执行器端点。

一旦用户明确配置了安全性，Spring Boot 默认值将不再起作用。然后，用户可以在一个地方配置所有访问规则。

这将防止我们在订购问题上纠结。这些问题通常发生在以定制方式配置执行器和应用安全规则时。

让我们来看一个混合了执行器和应用程序规则的简单安全片段:

```java
http.authorizeRequests()
  .requestMatchers(EndpointRequest.to("health"))
    .permitAll() // Actuator rules per endpoint
  .requestMatchers(EndpointRequest.toAnyEndpoint())
    .hasRole("admin") // Actuator general rules
  .requestMatchers(PathRequest.toStaticResources().atCommonLocations()) 
    .permitAll() // Static resource security 
  .antMatchers("/**") 
    .hasRole("user") // Application security rules 
  // ...
```

### 3.2。反应支持

**新 Spring Boot 协议为不同的反应模块带来了一系列新的启动器。一些例子是 WebFlux，以及 MongoDB、Cassandra 或 Redis 的反应式对应物。**

也有针对 WebFlux 的测试工具。特别是，我们可以利用`@WebFluxTest.`这类似于 1.4.0 中最初作为各种测试`slices`的一部分引入的旧`@WebMvcTest`。

## 4。生产就绪特性

Spring Boot 带来了一些有用的工具，使我们能够创建生产就绪的应用程序。在其他方面，我们可以利用 Spring Boot 致动器。

Actuator 包含各种工具，用于简化监控、跟踪和一般应用程序自检。关于致动器的更多细节可以在我们的[上一篇文章](/web/20220625221415/https://www.baeldung.com/spring-boot-actuators)中找到。

**与它的第 2 版`actuator` 已经经过了重大改进。**这个迭代关注于简化定制。它还支持其他网络技术，包括新的反应模块。

### 4.1。技术支持

在 Spring Boot 1.x 中，只有 Spring-MVC 被支持用于执行器端点。然而，在 2.x 中，它变得独立和可插拔。 Spring boot 现在提供了对 WebFlux、Jersey 和 Spring-MVC 的开箱即用支持。

和以前一样，JMX 仍然是一个选项，可以通过配置启用或禁用。

### 4.2。创建自定义端点

**新型执行器基础设施不受技术限制。正因为如此，开发模型从零开始重新设计。**

新的模式也带来了更大的灵活性和表现力。

让我们看看如何为执行器创建一个`Fruits` 端点:

```java
@Endpoint(id = "fruits")
public class FruitsEndpoint {

    @ReadOperation
    public Map<String, Fruit> fruits() { ... }

    @WriteOperation
    public void addFruits(@Selector String name, Fruit fruit) { ... }
}
```

一旦我们在我们的`ApplicationContext,` 中注册了`FruitsEndpoint`,就可以使用我们选择的技术将它公开为 web 端点。根据我们的配置，我们还可以通过 JMX 公开它。

将我们的端点转换为 web 端点，这将导致:

*   `GET`在`/application/fruits`返回水果
*   `POST`在`/applications/fruits/{a-fruit}`上处理应包含在有效载荷中的水果

还有更多的可能性。我们可以检索更精细的数据。此外，我们可以定义每个底层技术的具体实现(例如，JMX 对 Web)。出于本文的目的，我们将把它作为一个简单的介绍，而不涉及太多的细节。

### 4.3。致动器的安全性

在 Spring Boot，1.x Actuator 定义了自己的安全模型。这个安全模型不同于我们的应用程序所使用的模型。

当用户试图改进安全性时，这是许多棘手问题的根源。

在 2.x 中，安全配置应该与应用程序的其他部分使用相同的配置。

默认情况下，大多数执行器端点被禁用。这与 Spring 安全性是否在类路径中无关。除了`status` 和`info,` 之外，所有其他端点都需要由用户启用。

### 4.4。其他重要变化

*   大多数配置属性现在位于`management.xxx` 下，例如:`management.endpoints.jmx`
*   一些端点具有新的格式。例如:`env, flyway` 或`liquibase`
*   预定义的端点路径不再可配置

## 5。增强的开发体验

### 5.1。更好的反馈

1.3 中引入的弹簧靴`devtools` 。

它负责解决典型的开发问题。例如，视图技术的缓存。它还执行自动重启和浏览器实时重载。此外，它使我们能够远程调试应用程序。

**在 2.x 中，当我们的应用程序在`devtools`前重启时，一份“增量”报告将被打印出来**。该报告将指出发生了什么变化，以及它可能对我们的应用程序产生的影响。

假设我们定义了一个 JDBC 数据源，覆盖了 Spring Boot 配置的数据源。

D `evtools` 将指示自动配置的那个不再被创建。它还会指出原因，一旦执行重启，类型 `javax.sql.DataSource.` D `evtools` 在`@ConditionalOnMissingBean` 中的不利匹配将打印此报告。

### 5.2。重大变更

由于 JDK 9 的问题，devtools 不再支持通过 HTTP 进行远程调试。

## 6。总结

在本文中，我们讨论了 Spring Boot 新协议将带来的一些变化。

我们讨论了依赖性以及 Java 8 如何成为支持的最低版本。

接下来，我们讨论了自动配置。除其他外，我们还关注安全性。我们还谈到了 actuator 以及它所获得的许多改进。

最后，我们讨论了在所提供的开发工具中发生的一些小的调整。