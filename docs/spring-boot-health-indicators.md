# Spring Boot 的健康指标

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-health-indicators>

## 1.概观

Spring Boot 提供了几种不同的方法来检查正在运行的应用程序及其组件的状态和健康状况。在这些方法中，`[HealthContributor](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/api/org/springframework/boot/actuate/health/HealthContributor.html) `和`[HealthIndicator](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/HealthIndicator.html) `API 是其中两个值得注意的方法。

在本教程中，我们将熟悉这些 API，了解它们是如何工作的，并看看我们如何向它们提供自定义信息。

## 2.属国

健康信息贡献者是 [Spring Boot 致动器](/web/20221129013058/https://www.baeldung.com/spring-boot-actuators)模块的一部分，因此我们需要适当的 [Maven 依赖关系](https://web.archive.org/web/20221129013058/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-actuator):

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## 3.内置的

开箱即用， **Spring Boot 注册了许多`HealthIndicator`来报告特定应用程序方面**的健康状况。

其中一些指标几乎总是被注册，如`[DiskSpaceHealthIndicator](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.html) `或 [`PingHealthIndicator`](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/PingHealthIndicator.html) 。前者报告磁盘的当前状态，后者充当应用程序的 ping 端点。

**另一方面，Spring Boot 有条件地登记一些指标**。也就是说，如果类路径上有一些依赖项或者满足了其他一些条件，Spring Boot 也可能注册一些其他的`HealthIndicator`。例如，如果我们使用关系数据库，那么 Spring Boot 注册 [`DataSourceHealthIndicator`](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.html) 。同样，如果我们碰巧使用 Cassandra 作为我们的数据存储，它会注册`[CassandraHealthIndicator](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.html) `。

**为了检查 Spring Boot 应用程序的健康状态，我们可以调用`/actuator/health `端点**。该端点将报告所有注册的`HealthIndicator`的汇总结果。

**此外，要查看某个特定指标的健康报告，我们可以调用`/actuator/health/{name} `端点**。例如，调用`/actuator/health/diskSpace `端点将从`DiskSpaceHealthIndicator`返回一个状态报告:

```java
{
  "status": "UP",
  "details": {
    "total": 499963170816,
    "free": 134414831616,
    "threshold": 10485760,
    "exists": true
  }
}
```

## 4.习俗

除了内置的，我们可以注册自定义的`HealthIndicator`来报告组件或子系统的健康状况。为此，**我们所要做的就是将`HealthIndicator` 接口的一个实现注册为一个 Spring bean** 。

例如，以下实现随机报告一个故障:

```java
@Component
public class RandomHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        double chance = ThreadLocalRandom.current().nextDouble();
        Health.Builder status = Health.up();
        if (chance > 0.9) {
            status = Health.down();
        }
        return status.build();
    }
} 
```

根据该指标的运行状况报告，应用程序应该只有 90%的时间处于运行状态。这里我们使用`[Health](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Health.html) `构建器来报告健康信息。

**然而，在反应式应用程序中，我们应该注册一个类型为 [`ReactiveHealthIndicator`](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/ReactiveHealthIndicator.html)** 的 bean。反应式`health() `方法返回一个`[Mono<Health>](/web/20221129013058/https://www.baeldung.com/reactor-core#2-mono) `，而不是一个简单的`Health`。除此之外，这两种 web 应用程序类型的其他细节是相同的。

### 4.1.指示器名称

为了查看这个特定指标的报告，我们可以调用`/actuator/health/random `端点。例如，API 响应可能是这样的:

```java
{"status": "UP"}
```

`/actuator/health/random ` URL 中的`random `是该指示器的标识符。**特定`HealthIndicator `** **实现的标识符等同于不带`HealthIndicator `** **后缀的 bean 名称。**因为 bean 名称是`randomHealthIdenticator`，所以前缀`random `将是标识符。

使用这种算法，如果我们将 bean 名称更改为，比方说，`rand`:

```java
@Component("rand")
public class RandomHealthIndicator implements HealthIndicator {
    // omitted
}
```

那么指示符标识符将是`rand `而不是`random`。

### 4.2.禁用指示器

**要禁用某个指示器，我们可以将`“` `management.health.<indicator_identifier>.enabled” `配置属性设置为`false`** 。例如，如果我们将以下内容添加到我们的`application.properties`:

```java
management.health.random.enabled=false
```

然后 Spring Boot 将禁用`RandomHealthIndicator`。为了激活这个配置属性，我们还应该在指示器上添加`[@ConditionalOnEnabledHealthIndicator](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/autoconfigure/health/ConditionalOnEnabledHealthIndicator.html) `注释:

```java
@Component
@ConditionalOnEnabledHealthIndicator("random")
public class RandomHealthIndicator implements HealthIndicator { 
    // omitted
}
```

现在如果我们调用`/actuator/health/random`，Spring Boot 将返回一个 404 Not Found HTTP 响应:

```java
@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(properties = "management.health.random.enabled=false")
class DisabledRandomHealthIndicatorIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void givenADisabledIndicator_whenSendingRequest_thenReturns404() throws Exception {
        mockMvc.perform(get("/actuator/health/random"))
          .andExpect(status().isNotFound());
    }
}
```

请注意，禁用内置或自定义指示器彼此相似。因此，我们也可以将相同的配置应用于内置指示器。

### 4.3.其他详细信息

除了报告状态，我们还可以使用 [`withDetail(key, value)`](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Health.Builder.html#withDetail-java.lang.String-java.lang.Object-) 附加额外的键值细节:

```java
public Health health() {
    double chance = ThreadLocalRandom.current().nextDouble();
    Health.Builder status = Health.up();
    if (chance > 0.9) {
        status = Health.down();
    }

    return status
      .withDetail("chance", chance)
      .withDetail("strategy", "thread-local")
      .build();
}
```

这里我们向状态报告添加了两条信息。同样，我们可以通过向`[withDetails(map)](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Health.Builder.html#withDetail-java.lang.String-java.lang.Object-) `方法传递一个`Map<String, Object> `来实现同样的事情:

```java
Map<String, Object> details = new HashMap<>();
details.put("chance", chance);
details.put("strategy", "thread-local");

return status.withDetails(details).build();
```

现在，如果我们调用`/actuator/health/random`，我们可能会看到这样的内容:

```java
{
  "status": "DOWN",
  "details": {
    "chance": 0.9883560157173152,
    "strategy": "thread-local"
  }
}
```

我们也可以用自动化测试来验证这种行为:

```java
mockMvc.perform(get("/actuator/health/random"))
  .andExpect(jsonPath("$.status").exists())
  .andExpect(jsonPath("$.details.strategy").value("thread-local"))
  .andExpect(jsonPath("$.details.chance").exists());
```

有时，在与系统组件(如数据库或磁盘)通信时会出现异常。我们可以使用`[withException(ex)](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Health.Builder.html#withException-java.lang.Throwable-) `方法报告这样的异常:

```java
if (chance > 0.9) {
    status.withException(new RuntimeException("Bad luck"));
}
```

我们还可以将异常传递给我们之前看到的`[down(ex)](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Health.Builder.html#down-java.lang.Throwable-) `方法:

```java
if (chance > 0.9) {
    status = Health.down(new RuntimeException("Bad Luck"));
}
```

现在，运行状况报告将包含堆栈跟踪:

```java
{
  "status": "DOWN",
  "details": {
    "error": "java.lang.RuntimeException: Bad Luck",
    "chance": 0.9603739107139401,
    "strategy": "thread-local"
  }
}
```

### 4.4.细节曝光

**`management.endpoint.health.show-details `配置属性控制每个健康端点可以公开的细节级别。**

例如，如果我们将这个属性设置为`always, `，那么 Spring Boot 将总是返回健康报告中的`details `字段，就像上面的例子一样。

另一方面，**如果我们将这个属性设置为`never`，那么 Spring Boot 将总是从输出**中省略`details`。还有一个`when_authorized `值，它只为授权用户显示额外的`details`。当且仅当满足以下条件时，用户才被授权:

*   她通过认证了
*   她拥有在`management.endpoint.health.roles `配置属性中指定的角色

### 4.5.健康状态

默认情况下，Spring Boot 定义了四个不同的健康值`[Status](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Status.html)`:

*   组件或子系统按预期工作
*   `DOWN ` —组件不工作
*   `OUT_OF_SERVICE ` —组件暂时停止服务
*   `UNKNOWN ` —组件状态未知

这些状态被声明为`[public static final](https://web.archive.org/web/20221129013058/https://github.com/spring-projects/spring-boot/blob/310ef6e9995fab302f6af8b284d0a59ca0f212e9/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java#L43) `实例，而不是 Java 枚举。因此，我们可以定义自己的健康状况。为此，我们可以使用`[status(name)](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/Health.Builder.html#status-java.lang.String-) `方法:

```java
Health.Builder warning = Health.status("WARNING");
```

**健康状态影响健康端点的 HTTP 状态代码**。默认情况下，Spring Boot 映射`DOWN`和`OUT_OF_SERVICE `状态来抛出一个 503 状态代码。另一方面，`UP `和任何其他未映射的状态将被转换为 200 OK 状态代码。

为了定制这个映射，**我们可以将`management.endpoint.health.status.http-mapping.<status> `配置属性设置为所需的 HTTP 状态代码:**

```java
management.endpoint.health.status.http-mapping.down=500
management.endpoint.health.status.http-mapping.out_of_service=503
management.endpoint.health.status.http-mapping.warning=500
```

现在，Spring Boot 将把`DOWN `状态映射到 500，`OUT_OF_SERVICE `映射到 503，`WARNING `映射到 500 HTTP 状态代码:

```java
mockMvc.perform(get("/actuator/health/warning"))
  .andExpect(jsonPath("$.status").value("WARNING"))
  .andExpect(status().isInternalServerError());
```

类似地，**我们可以注册一个类型为`[HttpCodeStatusMapper](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/api/org/springframework/boot/actuate/health/HttpCodeStatusMapper.html) `的 bean 来定制 HTTP 状态代码映射**:

```java
@Component
public class CustomStatusCodeMapper implements HttpCodeStatusMapper {

    @Override
    public int getStatusCode(Status status) {
        if (status == Status.DOWN) {
            return 500;
        }

        if (status == Status.OUT_OF_SERVICE) {
            return 503;
        }

        if (status == Status.UNKNOWN) {
            return 500;
        }

        return 200;
    }
} 
```

`getStatusCode(status) `方法将健康状态作为输入，并将 HTTP 状态代码作为输出返回。此外，还可以映射定制的`Status `实例:

```java
if (status.getCode().equals("WARNING")) {
    return 500;
}
```

默认情况下，Spring Boot 用默认映射注册了这个接口的一个简单实现。**`[SimpleHttpCodeStatusMapper](https://web.archive.org/web/20221129013058/https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/api/org/springframework/boot/actuate/health/SimpleHttpCodeStatusMapper.html) `也能够从配置文件中读取映射，正如我们前面看到的。**

## 5.健康信息与指标

重要的应用程序通常包含一些不同的组件。例如，考虑一个使用 Cassandra 作为数据库、Apache Kafka 作为发布-订阅平台、Hazelcast 作为内存数据网格的 Spring Boot 应用程序。

**我们应该使用`HealthIndicator` s 来查看应用程序是否可以与这些组件通信**。如果通信链路出现故障，或者组件本身出现故障或运行缓慢，那么我们应该意识到组件不健康。换句话说，这些指标应该用于报告不同组件或子系统的健康状况。

相反，我们应该避免使用`HealthIndicator`来测量值、计数事件或测量持续时间。这就是为什么我们有度量标准。简而言之，**指标是报告 CPU 使用率、平均负载、堆大小、HTTP 响应分布等的更好工具。**

## 6.结论

在本教程中，我们看到了如何为执行器健康端点提供更多的健康信息。此外，我们深入讨论了健康 API 中的不同组件，比如`Health`、`Status`，以及 HTTP 状态映射的状态。

最后，我们简单讨论了健康信息和指标之间的区别，并了解了何时使用它们。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221129013058/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-actuator)