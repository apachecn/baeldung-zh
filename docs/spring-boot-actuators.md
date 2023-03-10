# Spring Boot 执行器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-actuators>

## 1。概述

在这篇文章中，我们介绍 Spring Boot 致动器。**我们将首先介绍基础知识，然后详细讨论 Spring Boot 2.x 和 1.x 中的可用功能**

我们将学习如何利用反应式编程模型，在 Spring Boot 2.x 和 WebFlux 中使用、配置和扩展这个监控工具。然后我们将讨论如何使用 Boot 1.x 做同样的事情。

Spring Boot 执行器将于 2014 年 4 月上市，同时推出第一款 Spring Boot 产品。

随着 Spring Boot 2 号的[发布，致动器被重新设计，并增加了新的令人兴奋的端点。](/web/20221101065528/https://www.baeldung.com/new-spring-boot-2)

我们将本指南分为三个主要部分:

*   [什么是执行器？](#understanding-actuator)
*   [Spring Boot 2.x 致动器](#boot-2x-actuator)
*   [Spring Boot 1.x 致动器](#boot-1x-actuator)

## 延伸阅读:

## [配置 Spring Boot 网络应用](/web/20221101065528/https://www.baeldung.com/spring-boot-application-configuration)

Some of the more useful configs for a Spring Boot application.[Read more](/web/20221101065528/https://www.baeldung.com/spring-boot-application-configuration) →

## [用 Spring Boot 创建定制启动器](/web/20221101065528/https://www.baeldung.com/spring-boot-custom-starter)

A quick and practical guide to creating custom Spring Boot starters.[Read more](/web/20221101065528/https://www.baeldung.com/spring-boot-custom-starter) →

## [在 Spring Boot 测试](/web/20221101065528/https://www.baeldung.com/spring-boot-testing)

Learn about how the Spring Boot supports testing, to write unit tests efficiently.[Read more](/web/20221101065528/https://www.baeldung.com/spring-boot-testing) →

## 2。什么是执行器？

本质上，Actuator 为我们的应用带来了生产就绪的特性。

有了这种依赖，监控我们的应用、收集指标、了解流量或我们数据库的状态变得微不足道。

这个库的主要好处是，我们可以获得生产级工具，而不必自己实际实现这些特性。

Actuator 主要用于**公开关于正在运行的应用程序**的操作信息——健康、指标、信息、转储、环境等。它使用 HTTP 端点或 JMX bean 使我们能够与它进行交互。

一旦这种依赖关系出现在类路径上，我们就有了几个现成的端点。与大多数 Spring 模块一样，我们可以很容易地以多种方式配置或扩展它。

### 2.1。入门

要启用 Spring Boot 执行器，我们只需要将`spring-boot-actuator`依赖项添加到我们的包管理器中。

在 Maven 中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

请注意，无论引导版本是什么，这都是有效的，因为版本是在 Spring Boot 材料清单(BOM)中指定的。

## 3。Spring Boot 2.x 致动器

在 2.x 中，Actuator 保持了它的基本意图，但是简化了它的模型，扩展了它的功能，并加入了更好的缺省值。

首先，这个版本变得与技术无关。它还通过将其与应用程序模型合并来简化其安全模型。

在各种变化中，重要的是要记住它们中的一些正在破裂。这包括 HTTP 请求和响应以及 Java APIs。

最后，最新版本现在支持 CRUD 模型，而不是旧的读/写模型。

### 3.1。技术支持

在其第二个主要版本中，Actuator 现在是技术不可知的，而在 1.x 中，它被绑定到 MVC，因此被绑定到 Servlet API。

在 2.x 中，Actuator 将其模型定义为可插拔和可扩展的，而不依赖于 MVC。

因此，有了这个新模型，我们能够利用 MVC 和 WebFlux 作为底层 web 技术。

此外，通过实现正确的适配器，可以添加即将到来的技术。

最后，JMX 仍然支持公开端点，无需任何额外的代码。

### 3.2。重要变更

与以前的版本不同，**致动器的大多数端点被禁用。**

因此，默认情况下仅有的两个可用选项是`/health`和`/info`。

如果我们想启用它们，我们可以设置`management.endpoints.web.exposure.include=*`。或者，我们可以列出应该启用的端点。

**Actuator 现在与常规应用安全规则共享安全配置，因此安全模型得到了显著简化。**

因此，为了调整致动器安全规则，我们可以只为`/actuator/**`添加一个条目:

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .pathMatchers("/actuator/**").permitAll()
      .anyExchange().authenticated()
      .and().build();
}
```

我们可以在[全新致动器官方文件](https://web.archive.org/web/20221101065528/https://docs.spring.io/spring-boot/docs/2.0.x/actuator-api/html/)中找到更多细节。

另外，**默认情况下，所有的执行器端点现在都被放置在`/actuator`路径下`.`**

与前一版本相同，我们可以使用新属性 `management.endpoints.web.base-path`调整该路径。

### 3.3。预定义的端点

让我们看看一些可用的端点，其中大部分已经在 1.x 中可用了。

此外，**添加了一些端点，删除了一些端点，重组了一些端点**:

*   `/auditevents` 列出与安全审计相关的事件，如用户登录/注销。此外，我们可以在其他字段中按主体或类型进行过滤。
*   `/beans` 返回我们的`BeanFactory`中所有可用的 beans。不像`/auditevents`，它不支持过滤。
*   `/conditions`，以前被称为/ `autoconfig`，围绕自动配置构建一个条件报告。
*   `/configprops` 允许我们获取所有的`@ConfigurationProperties` 豆子。
*   `/env` 返回当前环境属性。此外，我们可以检索单个属性。
*   `/flyway` 详细介绍了我们的 Flyway 数据库迁移。
*   `/health` 总结了我们的应用程序的健康状态。
*   从我们的应用程序使用的 JVM 中构建并返回一个堆转储。
*   `/info` 返回一般信息。它可能是自定义数据、构建信息或关于最近提交的详细信息。
*   除了 Liquibase 之外，`/liquibase` 的行为与`/flyway` 相似。
*   `/logfile` 返回普通应用程序日志。
*   `/loggers` 使我们能够查询和修改应用程序的日志记录级别。
*   `/metrics` 详细说明我们应用的指标。这可能包括通用指标以及自定义指标。
*   `/prometheus` 返回与前一个类似的指标，但是被格式化以与 Prometheus 服务器一起工作。
*   `/scheduledtasks` 提供我们应用程序中每个预定任务的详细信息。
*   `/sessions` 列出 HTTP 会话，假设我们使用的是 Spring 会话。
*   `/shutdown` 执行应用程序的正常关闭。
*   `/threaddump` 转储底层 JVM 的线程信息。

### 3.4.执行器端点的超媒体

Spring Boot 添加了一个发现端点，它返回所有可用执行器端点的链接。这将有助于发现执行器端点及其对应的 URL。

**默认情况下，可以通过`/actuator `端点访问该发现端点。**

因此，如果我们向这个 URL 发送一个`GET `请求，它将返回各个端点的执行器链接:

```java
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    "features-arg0": {
      "href": "http://localhost:8080/actuator/features/{arg0}",
      "templated": true
    },
    "features": {
      "href": "http://localhost:8080/actuator/features",
      "templated": false
    },
    "beans": {
      "href": "http://localhost:8080/actuator/beans",
      "templated": false
    },
    "caches-cache": {
      "href": "http://localhost:8080/actuator/caches/{cache}",
      "templated": true
    },
    // truncated
}
```

如上所示，`/actuator `端点报告了`_links `字段下所有可用的执行器端点。

**此外，如果我们配置一个自定义管理基本路径，那么我们应该使用该基本路径作为发现 URL。**

例如，如果我们将`management.endpoints.web.base-path `设置为`/mgmt`，那么我们应该向`/mgmt `端点发送一个请求来查看链接列表。

非常有趣的是，当管理基路径被设置为`/`时，发现端点被禁用，以防止与其他映射冲突的可能性。

### 3.5。健康指标

就像在以前的版本中，我们可以很容易地添加自定义指标。与其他 API 相反，创建自定义健康端点的抽象保持不变。然而，**添加了一个新的接口`ReactiveHealthIndicator`，以实现反应性健康检查。**

让我们来看一个简单的自定义反应性运行状况检查:

```java
@Component
public class DownstreamServiceHealthIndicator implements ReactiveHealthIndicator {

    @Override
    public Mono<Health> health() {
        return checkDownstreamServiceHealth().onErrorResume(
          ex -> Mono.just(new Health.Builder().down(ex).build())
        );
    }

    private Mono<Health> checkDownstreamServiceHealth() {
        // we could use WebClient to check health reactively
        return Mono.just(new Health.Builder().up().build());
    }
}
```

健康指标的一个方便的特点是我们可以将它们作为层次结构的一部分进行汇总。

因此，按照前面的例子，我们可以将所有下游服务分组到一个`downstream-` `services`类别下。只要每个嵌套的`service`都是可达的，这个类别就是健康的。

查看我们关于[健康指标](/web/20221101065528/https://www.baeldung.com/spring-boot-health-indicators)的文章以获得更深入的了解。

### 3.6.健康团体

**从 Spring Boot 2.2 开始，我们可以将健康指标组织成[组](https://web.archive.org/web/20221101065528/https://github.com/spring-projects/spring-boot/blob/c3aa494ba32b8271ea19dd041327441b27ddc319/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthEndpointGroups.java#L30)，并对所有组成员应用相同的配置。**

例如，我们可以创建一个名为`custom `的健康组，将其添加到我们的`application.properties`中:

```java
management.endpoint.health.group.custom.include=diskSpace,ping
```

这样，`custom `组包含了`diskSpace `和`ping `健康指标。

现在，如果我们调用`/actuator/health `端点，它会在 JSON 响应中告诉我们新的健康组:

```java
{"status":"UP","groups":["custom"]}
```

对于健康组，我们可以看到一些健康指标的汇总结果。

在这种情况下，如果我们向`/actuator/health/custom`发送一个请求，那么:

```java
{"status":"UP"}
```

我们可以通过`application.properties`配置群组以显示更多细节:

```java
management.endpoint.health.group.custom.show-components=always
management.endpoint.health.group.custom.show-details=always
```

现在，如果我们向`/actuator/health/custom, `发送同样的请求，我们会看到更多的细节:

```java
{
  "status": "UP",
  "components": {
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 499963170816,
        "free": 91300069376,
        "threshold": 10485760
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

也可以只向授权用户显示这些详细信息:

```java
management.endpoint.health.group.custom.show-components=when_authorized
management.endpoint.health.group.custom.show-details=when_authorized
```

**我们还可以自定义状态映射。**

例如，它可以返回 207 状态代码，而不是 HTTP 200 OK 响应:

```java
management.endpoint.health.group.custom.status.http-mapping.up=207
```

这里，我们告诉 Spring Boot，如果`custom `组的状态是`UP.`，就返回一个 207 HTTP 状态码

### 3.7。Spring Boot 指标 2

**在 Spring Boot 2.0 中，内部度量标准被千分尺取代**，因此我们可以期待突破性的变化。如果我们的应用程序使用度量服务，如`GaugeService` 或`CounterService`，它们将不再可用。

相反，我们应该直接与[微米](/web/20221101065528/https://www.baeldung.com/micrometer)互动。在 Spring Boot 2.0 中，我们将获得一个自动配置的类型为`MeterRegistry` 的 bean。

此外，Micrometer 现在是 Actuator 依赖项的一部分，所以只要 Actuator 依赖项在类路径中，我们就可以开始了。

此外，我们将从`/metrics` 端点获得一个全新的响应:

```java
{
  "names": [
    "jvm.gc.pause",
    "jvm.buffer.memory.used",
    "jvm.memory.used",
    "jvm.buffer.count",
    // ...
  ]
}
```

如我们所见，没有像 1.x 中那样的实际指标。

要获得特定指标的实际值，我们现在可以导航到所需的指标，例如`/actuator/metrics/jvm.gc.pause`，并获得详细的响应:

```java
{
  "name": "jvm.gc.pause",
  "measurements": [
    {
      "statistic": "Count",
      "value": 3.0
    },
    {
      "statistic": "TotalTime",
      "value": 7.9E7
    },
    {
      "statistic": "Max",
      "value": 7.9E7
    }
  ],
  "availableTags": [
    {
      "tag": "cause",
      "values": [
        "Metadata GC Threshold",
        "Allocation Failure"
      ]
    },
    {
      "tag": "action",
      "values": [
        "end of minor GC",
        "end of major GC"
      ]
    }
  ]
}
```

现在，度量标准更加全面，不仅包括不同的值，还包括一些相关的元数据。

### 3.8。定制`/info`端点

`/info` 终点保持不变。**和以前一样，我们可以使用各自的 Maven 或 Gradle 依赖关系**来添加 git 细节:

```java
<dependency>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
</dependency>
```

同样，**我们也可以使用 Maven 或 Gradle 插件**来包含构建信息，包括名称、组和版本:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>build-info</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 3.9。创建自定义端点

正如我们之前指出的，我们可以创建自定义端点。然而，Spring Boot 2 已经重新设计了实现这一点的方式，以支持新的技术不可知的范式。

**让我们在应用程序中创建一个执行器端点来查询、启用和禁用特性标志**:

```java
@Component
@Endpoint(id = "features")
public class FeaturesEndpoint {

    private Map<String, Feature> features = new ConcurrentHashMap<>();

    @ReadOperation
    public Map<String, Feature> features() {
        return features;
    }

    @ReadOperation
    public Feature feature(@Selector String name) {
        return features.get(name);
    }

    @WriteOperation
    public void configureFeature(@Selector String name, Feature feature) {
        features.put(name, feature);
    }

    @DeleteOperation
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }

    public static class Feature {
        private Boolean enabled;

        // [...] getters and setters 
    }

}
```

为了获得端点，我们需要一个 bean。在我们的例子中，我们为此使用了`@Component`。还有，我们需要用`@Endpoint`来装饰这个 bean。

我们的端点路径由`@Endpoint.`的`id` 参数决定，在我们的例子中，它将请求路由到`/actuator/features`。

准备就绪后，我们可以开始使用以下内容定义操作:

*   `@ReadOperation`:它会映射到 HTTP `GET`。
*   `@WriteOperation`:它会映射到 HTTP `POST`。
*   `@DeleteOperation`:它会映射到 HTTP `DELETE`。

当我们用应用程序中的前一个端点运行应用程序时，Spring Boot 将注册它。

验证这一点的快速方法是检查日志:

```java
[...].WebFluxEndpointHandlerMapping: Mapped "{[/actuator/features/{name}],
  methods=[GET],
  produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
[...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features],
  methods=[GET],
  produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
[...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features/{name}],
  methods=[POST],
  consumes=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
[...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features/{name}],
  methods=[DELETE]}"[...]
```

在前面的日志中，我们可以看到 WebFlux 是如何公开我们的新端点的。如果我们切换到 MVC，它将简单地委托该技术，而不必更改任何代码。

此外，对于这种新方法，我们有一些重要的注意事项要记住:

*   不依赖于 MVC。
*   之前作为方法存在的所有元数据(`sensitive, enabled…)`不再存在。然而，我们可以使用`@Endpoint(id = “features”, enableByDefault = false)`来启用或禁用端点。
*   不像在 1.x 中，不再需要扩展给定的接口。
*   与旧的读/写模型相比，我们现在可以使用`@DeleteOperation`来定义`DELETE` 操作。

### 3.10。扩展现有端点

假设我们希望确保应用程序的生产实例永远不是`SNAPSHOT`版本。

我们决定通过更改返回该信息的执行器端点的 HTTP 状态代码来实现这一点，即`/info`。如果我们的应用碰巧是一个`SNAPSHOT`，我们将得到一个不同的`HTTP` 状态码。

**我们可以使用`@EndpointExtension`注释**，或其更具体的专门化`@EndpointWebExtension` 或`@EndpointJmxExtension`，轻松扩展预定义端点的行为:

```java
@Component
@EndpointWebExtension(endpoint = InfoEndpoint.class)
public class InfoWebEndpointExtension {

    private InfoEndpoint delegate;

    // standard constructor

    @ReadOperation
    public WebEndpointResponse<Map> info() {
        Map<String, Object> info = this.delegate.info();
        Integer status = getStatus(info);
        return new WebEndpointResponse<>(info, status);
    }

    private Integer getStatus(Map<String, Object> info) {
        // return 5xx if this is a snapshot
        return 200;
    }
}
```

### 3.11。启用所有端点

为了使用 HTTP 访问执行器端点，我们需要启用和公开它们。

默认情况下，除了`/shutdown`之外的所有端点都被启用。默认情况下，只有`/health`和`/info`端点是公开的。

我们需要添加以下配置来公开所有端点:

```java
management.endpoints.web.exposure.include=*
```

要显式启用特定端点(例如，`/shutdown), `,我们使用:

```java
management.endpoint.shutdown.enabled=true
```

为了显示除一个端点(例如`/loggers`)之外的所有已启用端点，我们使用:

```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=loggers
```

## 4。Spring Boot 1.x 致动器

在 1.x 中，Actuator 遵循读/写模型，这意味着我们可以从它那里读取数据，也可以向它写入数据。

例如，我们可以检索应用程序的指标或健康状况。或者，我们可以优雅地终止我们的应用程序或更改我们的日志配置。

为了让它工作，Actuator 需要 Spring MVC 通过 HTTP 公开它的端点。不支持其他技术。

### 4.1。端点

**在 1.x 中，Actuator 自带安全模型。它利用了 Spring 安全构造，但是需要独立于应用程序的其余部分进行配置。**

此外，大多数端点是敏感的，这意味着它们不是完全公开的，或者大多数信息将被省略，而少数端点则不是，例如`/info`。

以下是 Boot 提供的一些最常见的现成端点:

*   `/health` 显示应用程序健康信息(通过未认证的连接访问时是简单的`status`，通过认证时是完整的消息细节)；默认不敏感。
*   `/info` 显示任意应用信息；默认不敏感。
*   `/metrics` 显示当前应用程序的指标信息；默认是敏感的。
*   `/trace` 显示跟踪信息(默认为最后几个 HTTP 请求)。

我们可以在官方文档中找到[以上现有终点的完整列表。](https://web.archive.org/web/20221101065528/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)

### 4.2。配置现有端点

我们可以使用格式`endpoints.[endpoint name].[property to customize]`定制每个端点的属性。

有三种属性可用:

*   `id`:将通过 HTTP 访问此端点
*   `enabled`:如果为真，则可以访问；否则不会
*   `sensitive`:如果为真，那么需要授权通过 HTTP 显示关键信息

例如，添加以下属性将定制/ `beans`端点`:` 

```java
endpoints.beans.id=springbeans
endpoints.beans.sensitive=false
endpoints.beans.enabled=true
```

### 4.3。`/health`终点

**`/health`端点用于检查正在运行的应用程序的健康或状态。**

它通常由监控软件来执行，如果正在运行的实例由于其他原因(例如，数据库的连接问题、磁盘空间不足等)而停止运行或变得不健康，它会向我们发出警报。

默认情况下，未经授权的用户只能在通过 HTTP 访问时看到状态信息:

```java
{
    "status" : "UP"
} 
```

这些健康信息是从实现在我们的应用程序上下文中配置的`HealthIndicator`接口的所有 beans 中收集的。

由`HealthIndicator`返回的一些信息本质上是敏感的，但是我们可以配置`endpoints.health.sensitive=false`来公开更详细的信息，比如磁盘空间、消息传递代理连接、自定义检查等等。

请注意，这只适用于低于 1.5.0 的 Spring Boot 版本。对于 1.5.0 和更高版本，我们还应该通过设置`management.security.enabled=false`来禁用未授权访问的安全性。

我们还可以**实现我们自己的定制健康指示器**，它可以收集特定于应用程序的任何类型的定制健康数据，并通过`/health`端点自动公开这些数据:

```java
@Component("myHealthCheck")
public class HealthCheck implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down()
              .withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

    public int check() {
    	// Our logic to check health
    	return 0;
    }
} 
```

下面是输出的样子:

```java
{
    "status" : "DOWN",
    "myHealthCheck" : {
        "status" : "DOWN",
        "Error Code" : 1
     },
     "diskSpace" : {
         "status" : "UP",
         "free" : 209047318528,
         "threshold" : 10485760
     }
}
```

### 4.4。`/info`终点

我们还可以定制由`/info`端点显示的数据:

```java
info.app.name=Spring Sample Application
info.app.description=This is my first spring boot application
info.app.version=1.0.0
```

和示例输出:

```java
{
    "app" : {
        "version" : "1.0.0",
        "description" : "This is my first spring boot application",
        "name" : "Spring Sample Application"
    }
}
```

### 4.5。`/metrics`终点

**指标端点发布关于 OS 和 JVM 以及应用级指标的信息。**一旦启用，我们将获得诸如内存、堆、处理器、线程、加载的类、卸载的类、线程池以及一些 HTTP 指标等信息。

这是该端点开箱后的输出:

```java
{
    "mem" : 193024,
    "mem.free" : 87693,
    "processors" : 4,
    "instance.uptime" : 305027,
    "uptime" : 307077,
    "systemload.average" : 0.11,
    "heap.committed" : 193024,
    "heap.init" : 124928,
    "heap.used" : 105330,
    "heap" : 1764352,
    "threads.peak" : 22,
    "threads.daemon" : 19,
    "threads" : 22,
    "classes" : 5819,
    "classes.loaded" : 5819,
    "classes.unloaded" : 0,
    "gc.ps_scavenge.count" : 7,
    "gc.ps_scavenge.time" : 54,
    "gc.ps_marksweep.count" : 1,
    "gc.ps_marksweep.time" : 44,
    "httpsessions.max" : -1,
    "httpsessions.active" : 0,
    "counter.status.200.root" : 1,
    "gauge.response.root" : 37.0
} 
```

为了收集自定义指标，我们支持量表(单值数据快照)和计数器，即递增/递减指标。

让我们在`/metrics`端点中实现我们自己的定制指标。

我们将定制登录流来记录成功和失败的登录尝试:

```java
@Service
public class LoginServiceImpl {

    private final CounterService counterService;

    public LoginServiceImpl(CounterService counterService) {
        this.counterService = counterService;
    }

    public boolean login(String userName, char[] password) {
        boolean success;
        if (userName.equals("admin") && "secret".toCharArray().equals(password)) {
            counterService.increment("counter.login.success");
            success = true;
        }
        else {
            counterService.increment("counter.login.failure");
            success = false;
        }
        return success;
    }
}
```

输出可能是这样的:

```java
{
    ...
    "counter.login.success" : 105,
    "counter.login.failure" : 12,
    ...
} 
```

请注意，登录尝试和其他与安全相关的事件在 Actuator 中作为审计事件现成可用。

### 4.6。创建新的端点

除了使用 Spring Boot 提供的现有终端，我们还可以创建一个全新的终端。

首先，我们需要让新端点实现`Endpoint<T>`接口:

```java
@Component
public class CustomEndpoint implements Endpoint<List<String>> {

    @Override
    public String getId() {
        return "customEndpoint";
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public boolean isSensitive() {
        return true;
    }

    @Override
    public List<String> invoke() {
        // Custom logic to build the output
        List<String> messages = new ArrayList<String>();
        messages.add("This is message 1");
        messages.add("This is message 2");
        return messages;
    }
}
```

为了访问这个新端点，使用它的`id`来映射它。换句话说，我们可以练习击中`/customEndpoint`。

输出:

```java
[ "This is message 1", "This is message 2" ]
```

### 4.7。进一步定制

出于安全目的，我们可能会选择通过非标准端口公开执行器端点——`management.port` 属性可以很容易地用于配置。

此外，正如我们已经提到的，在 1.x 中，Actuator 基于 Spring 安全配置自己的安全模型，但独立于应用程序的其他部分。

因此，我们可以更改`management.address`属性来限制可以通过网络访问端点的位置:

```java
#port used to expose actuator
management.port=8081 

#CIDR allowed to hit actuator
management.address=127.0.0.1 

#Whether security should be enabled or disabled altogether
management.security.enabled=false
```

此外，除了`/info`之外的所有内置端点都是默认敏感的。

如果应用程序使用 Spring Security，我们可以通过在`application.properties`文件中定义默认的安全属性(用户名、密码和角色)来保护这些端点:

```java
security.user.name=admin
security.user.password=secret
management.security.role=SUPERUSER
```

## 5。结论

在这篇文章中，我们谈到了 Spring Boot 执行器。我们首先定义了 Actuator 的含义以及它为我们做了什么。

接下来，我们关注当前 Spring Boot 版本 2.x 的 Actuator，讨论如何使用、调整和扩展它。我们还讨论了在这个新版本中可以发现的重要安全变化。我们讨论了一些流行的端点以及它们是如何变化的。

然后我们讨论了早期 Spring Boot 1 版本中的致动器。

最后，我们演示了如何定制和扩展 Actuator。

和往常一样，本文中使用的代码可以在 GitHub 上找到，适用于 [Spring Boot 2.x](https://web.archive.org/web/20221101065528/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-security) 和 [Spring Boot 1.x](https://web.archive.org/web/20221101065528/https://github.com/eugenp/tutorials/tree/master/spring-4) 。