# Spring Boot 启动执行器端点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-actuator-startup>

## 1.介绍

Spring Boot 应用程序可以有复杂的组件图、启动阶段和资源初始化步骤。

在本文中，我们将看看**如何通过 [Spring Boot 执行器](/web/20220524021633/https://www.baeldung.com/spring-boot-actuators)端点**跟踪和监控这个启动信息。

## 2.应用程序启动跟踪

跟踪应用程序启动过程中的各个步骤可以提供有用的信息，帮助我们**了解应用程序启动的各个阶段所花费的时间**。这样的工具也可以**提高我们对上下文生命周期和应用程序启动序列的理解**。

[Spring 框架](/web/20220524021633/https://www.baeldung.com/spring-intro)提供了[记录应用启动](https://web.archive.org/web/20220524021633/https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-startup)和图形初始化的功能。此外，Spring Boot 执行器通过 HTTP 或 JMX 提供多种生产级监控和管理功能。

从 [Spring Boot 2.4](https://web.archive.org/web/20220524021633/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.4-Release-Notes#startup-endpoint) 开始，**应用程序启动跟踪指标现在可以通过`/actuator/startup`端点**获得。

## 3.设置

要启用 Spring Boot 执行器，让我们将 [`spring-boot-starter-actuator`](https://web.archive.org/web/20220524021633/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-actuator) 依赖项添加到 POM 中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.5.4</version>
</dependency>
```

我们还将添加 [`spring-boot-starter-web`](https://web.archive.org/web/20220524021633/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web) 依赖项，因为这是通过 HTTP 访问端点所必需的:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.4</version>
</dependency>
```

此外，我们还将通过在我们的`application.properties`文件中设置配置属性来通过 HTTP 公开所需的端点:

```java
management.endpoints.web.exposure.include=startup
```

最后，我们将分别使用 [`curl`](https://web.archive.org/web/20220524021633/https://man7.org/linux/man-pages/man1/curl.1.html) 和`[jq](/web/20220524021633/https://www.baeldung.com/linux/jq-command-json)`来查询执行器 HTTP 端点和解析 JSON 响应。

## 4.执行器端点

为了捕获启动事件，我们需要用`@ApplicationStartup`接口的实现来配置我们的应用程序。默认情况下，用于管理应用程序生命周期的`ApplicationContext`使用无操作实现。很明显，为了最小的开销，这不执行启动检测和跟踪。

因此，**与其他执行器端点不同，我们需要一些额外的设置**。

### 4.1.使用`BufferingApplicationStartup`

我们需要将应用程序的启动配置设置为`BufferingApplicationStartup.`的实例，这是 Spring Boot 提供的`ApplicationStartup`接口的内存实现。它**捕获 Spring 启动过程中的事件，并将它们存储在内部缓冲区**。

让我们首先为我们的应用程序创建一个简单的应用程序:

```java
@SpringBootApplication
public class StartupTrackingApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(StartupTrackingApplication.class);
        app.setApplicationStartup(new BufferingApplicationStartup(2048));
        app.run(args);
    }
}
```

这里，我们还为内部缓冲区指定了 2048 的容量。一旦缓冲区达到此事件容量，将不再记录更多数据。因此，根据应用程序的复杂性和启动过程中执行的各个步骤，我们使用合适的值来存储事件是很重要的。

至关重要的是，**执行器端点只有在该实现被配置后才可用**。

### 4.2.`startup`终点

现在，我们可以启动我们的应用程序并查询`startup`致动器端点。

让我们使用`curl`调用这个 POST 端点，并使用`jq`格式化 JSON 输出:

```java
> curl 'http://localhost:8080/actuator/startup' -X POST | jq
{
  "springBootVersion": "2.5.4",
  "timeline": {
    "startTime": "2021-10-17T21:08:00.931660Z",
    "events": [
      {
        "endTime": "2021-10-17T21:08:00.989076Z",
        "duration": "PT0.038859S",
        "startTime": "2021-10-17T21:08:00.950217Z",
        "startupStep": {
          "name": "spring.boot.application.starting",
          "id": 0,
          "tags": [
            {
              "key": "mainApplicationClass",
              "value": "com.baeldung.startup.StartupTrackingApplication"
            }
          ],
          "parentId": null
        }
      },
      {
        "endTime": "2021-10-17T21:08:01.454239Z",
        "duration": "PT0.344867S",
        "startTime": "2021-10-17T21:08:01.109372Z",
        "startupStep": {
          "name": "spring.boot.application.environment-prepared",
          "id": 1,
          "tags": [],
          "parentId": null
        }
      },
      ... other steps not shown
      {
        "endTime": "2021-10-17T21:08:12.199369Z",
        "duration": "PT0.00055S",
        "startTime": "2021-10-17T21:08:12.198819Z",
        "startupStep": {
          "name": "spring.boot.application.running",
          "id": 358,
          "tags": [],
          "parentId": null
        }
      }
    ]
  }
}
```

正如我们所看到的，详细的 JSON 响应包含一个被检测的启动事件列表。**它包含关于每个步骤的各种细节，如步骤名称、开始时间、结束时间，以及步骤计时细节**。关于响应结构的详细信息可在 [Spring Boot 致动器 Web API](https://web.archive.org/web/20220524021633/https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/#startup) 文档中获得。

此外，核心容器中定义的步骤的完整列表以及关于每个步骤的更多细节可在 [Spring 参考文档](https://web.archive.org/web/20220524021633/https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#application-startup-steps)中获得。

这里需要注意的一个重要细节是，端点的后续调用不提供详细的 JSON 响应。这是因为启动端点调用清除了内部缓冲区。因此，我们需要重启应用程序来调用同一个端点，并再次接收完整的响应。

**如果需要，我们应该保存有效负载以供进一步分析**。

### 4.3.过滤启动事件

正如我们所见，缓冲实现在内存中存储事件的容量是固定的。因此，可能不希望在缓冲区中存储大量事件。

我们可以过滤检测事件，只存储我们可能感兴趣的事件:

```java
BufferingApplicationStartup startup = new BufferingApplicationStartup(2048);
startup.addFilter(startupStep -> startupStep.getName().matches("spring.beans.instantiate"); 
```

在这里，我们使用了`addFilter`方法来只检测具有指定名称的步骤。

### 4.4.定制仪器

我们还可以扩展`BufferingApplicationStartup`来提供定制的启动跟踪行为，以满足我们特定的工具需求。

由于这种工具在测试环境中比在生产环境中更有价值，所以使用系统属性并在无操作和缓冲或定制实现之间切换是一个简单的练习。

## 5.分析启动时间

作为一个实际的例子，让我们尝试在启动过程中识别任何可能需要相对长时间来初始化的 bean 实例化。例如，这可能是应用程序启动期间的缓存加载、数据库连接池或其他一些昂贵的初始化。

我们可以像以前一样调用端点，但是这一次，我们将使用`jq`处理输出。

由于响应非常冗长，让我们过滤与名称`spring.beans.instantiate`匹配的步骤，并按持续时间对它们进行排序:

```java
> curl 'http://localhost:8080/actuator/startup' -X POST \
| jq '[.timeline.events
 | sort_by(.duration) | reverse[]
 | select(.startupStep.name | match("spring.beans.instantiate"))
 | {beanName: .startupStep.tags[0].value, duration: .duration}]'
```

上面的表达式处理响应 JSON 以提取计时信息:

*   按降序对`timeline.events`数组进行排序。
*   从排序后的数组中选择所有匹配名称`spring.beans.instantiate`的步骤。
*   用每个匹配步骤中的`beanName`和`duration`创建一个新的 JSON 对象。

因此，输出显示了在应用程序启动期间实例化的各种 beans 的简洁、有序和经过筛选的视图:

```java
[
  {
    "beanName": "resourceInitializer",
    "duration": "PT6.003171S"
  },
  {
    "beanName": "tomcatServletWebServerFactory",
    "duration": "PT0.143958S"
  },
  {
    "beanName": "requestMappingHandlerAdapter",
    "duration": "PT0.14302S"
  },
  ...
]
```

在这里，我们可以看到`resourceInitializer` bean 在启动过程中花费了大约 6 秒钟。这可能被认为是对整个应用程序启动时间造成了很大的影响。使用这种方法，**我们可以有效地发现问题，并集中精力进行进一步的调查和寻找可能的解决方案**。

需要注意的是 **`ApplicationStartup`仅在应用程序启动**时使用。换句话说，**它并没有取代 Java profilers 和应用程序规范的度量收集框架**。

## 6.结论

在本文中，我们研究了如何在 Spring Boot 应用程序中获取和分析详细的启动指标。

首先，我们看到了如何启用和配置 Spring Boot 执行器端点。然后，我们查看了从该端点获得的有用信息。

最后，我们通过一个例子来分析这些信息，以便更好地理解应用程序启动过程中的各个步骤。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524021633/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-actuator)