# 获取 Spring Cloud Sleuth 中的当前跟踪 ID

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-sleuth-get-trace-id>

## 1。概述

在这篇文章中，我们将看看 [Spring Cloud Sleuth](/web/20220524071029/https://www.baeldung.com/spring-cloud-sleuth-single-application) ，看看我们如何使用它在 Spring Boot 进行追踪。**它为我们的日志添加了有用的额外信息，并通过添加唯一标识符使调试操作变得更容易。在 Sleuth 术语中，这些操作被称为跟踪。它们可以由几个步骤组成，称为跨度。**

例如，跟踪可以是从我们的应用程序中查询数据的 GET 请求。当我们的应用程序处理请求时，它可以被分成更小的步骤:用户授权、执行数据库查询、转换响应。这些步骤中的每一步都是属于同一轨迹的唯一跨度。

在某些情况下，我们可能想要获得当前跟踪或跨度的 ID。例如，当发生事故时，我们可以将这些发送给开发团队。然后他们可以用它来调试和修复问题。

## 2。应用程序设置

**让我们首先创建一个 Spring Boot 项目并添加 [spring-cloud-starter-sleuth 依赖项](https://web.archive.org/web/20220524071029/https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-sleuth/3.1.0) :**

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
    <version>3.1.0</version>
</dependency>
```

这个 starter 依赖项与 Spring Boot 集成得很好，并提供了开始使用 Spring Cloud Sleuth 的必要配置。

然而，我们可以多走一步。**让我们在 application.properties 文件中设置应用程序的名称，这样我们就可以在日志中看到这个名称以及跟踪和跨度 id:**

```java
spring.application.name=Baeldung Sleuth Tutorial
```

现在我们需要一个应用程序的入口点。让我们创建一个具有单一 GET 端点的 REST 控制器:

```java
@RestController
public class SleuthTraceIdController {

    @GetMapping("/traceid")
    public String getSleuthTraceId() {
        return "Hello from Sleuth";
    }
} 
```

让我们访问位于[http://localhost:8080/traceid](https://web.archive.org/web/20220524071029/http://localhost:8080/traceid)的 API 端点。我们应该在响应中看到“来自 Sleuth 的问候”。

## 3。记录日志

让我们给`getSleuthTraceId`方法添加一个日志语句。首先，我们需要一个`Logger`给我们的类。然后我们可以记录消息:

```java
private static final Logger logger = LoggerFactory.getLogger(SleuthTraceIdController.class);

@GetMapping("/traceid")
public String getSleuthTraceId() {
    logger.info("Hello with Sleuth");
    return "Hello from Sleuth";
}
```

让我们再次调用我们的 API 端点 并检查日志。我们应该会找到类似这样的东西:

```java
INFO [Baeldung Sleuth Tutorial,e48f140a63bb9fbb,e48f140a63bb9fbb] 9208 --- [nio-8080-exec-1] c.b.s.traceid.SleuthTraceIdController : Hello with Sleuth
```

**请注意，应用程序名称在开头的括号内。这些括号是 Sleuth 添加的。它们代表应用程序名称、跟踪 ID 和 span ID。**

## 4。电流轨迹和跨度

我们可以使用上面的例子来调试我们的应用程序中的问题，但是要确定是什么原因导致了这个问题，以及应该跟踪哪个问题，这可能是一个挑战。这就是为什么我们将以编程方式获取当前跟踪，然后我们可以使用它进行任何进一步的调查。

在我们的实现中，我们将简化这个用例，我们只将跟踪 id 记录到控制台。

**首先，我们需要获得一个`Tracer`对象的实例。让我们把它注入我们的控制器，得到当前的跨度:**

```java
@Autowired
private Tracer tracer;

@GetMapping("/traceid")
public String getSleuthTraceId() {
    logger.info("Hello with Sleuth");
    Span span = tracer.currentSpan();
    return "Hello from Sleuth";
}
```

**注意，如果此时没有活动的 span，`currentSpan`方法可以返回 null。**因此我们必须执行一个额外的检查，看看我们是否可以继续使用这个`Span`对象而不获取`NullPointerException`。让我们实现这个检查并记录当前的跟踪和范围 id:

```java
Span span = tracer.currentSpan();
if (span != null) {
    logger.info("Trace ID {}", span.context().traceIdString());
    logger.info("Span ID {}", span.context().spanIdString());
}
```

让我们运行应用程序，并在访问 API 端点时寻找这些消息。它们应该包含与 Sleuth 添加的括号相同的 id。

## 5。十进制数形式的跟踪和跨度 ID

还有另一种方法可以用`spanId`方法而不是`spanIdString`来获得 span ID。**两者的区别在于，后者返回值的十六进制表示，而前者返回十进制数。**让我们比较一下它们的运行情况，并记录下十进制值:

```java
Span span = tracer.currentSpan();
if (span != null) {
    logger.info("Span ID hex {}", span.context().spanIdString());
    logger.info("Span ID decimal {}", span.context().spanId());
}
```

这两个值代表相同的数字，输出应该类似于:

```java
INFO [Baeldung Sleuth Tutorial,0de46b6fcbc8da83,0de46b6fcbc8da83] 8648 --- [nio-8080-exec-3] c.b.s.traceid.SleuthTraceIdController    : Span ID hex 0de46b6fcbc8da83
INFO [Baeldung Sleuth Tutorial,0de46b6fcbc8da83,0de46b6fcbc8da83] 8648 --- [nio-8080-exec-3] c.b.s.traceid.SleuthTraceIdController    : Span ID decimal 1001043145087572611
```

同样，这也适用于跟踪 id。我们可以使用`traceId`方法来代替`traceIdString,` 。`traceIdString`返回十六进制值，而`traceId`返回十进制值:

```java
logger.info("Trace ID hex {}", span.context().traceIdString());
logger.info("Trace ID decimal {}", span.context().traceId());
```

输出与上一个非常相似。它首先包含十六进制的跟踪 ID，然后包含十进制的跟踪 ID:

```java
INFO [Baeldung Sleuth Tutorial,34ec0b8ac9d65e91,34ec0b8ac9d65e91] 7384 --- [nio-8080-exec-1] c.b.s.traceid.SleuthTraceIdController    : Trace ID hex 34ec0b8ac9d65e91
INFO [Baeldung Sleuth Tutorial,34ec0b8ac9d65e91,34ec0b8ac9d65e91] 7384 --- [nio-8080-exec-1] c.b.s.traceid.SleuthTraceIdController    : Trace ID decimal 3813435675195629201
```

## 6。结论

在本文中，我们讨论了 Spring Cloud Sleuth 如何帮助调试和跟踪 Spring Boot 的事件。首先，我们**使用`Tracer`对象来引用当前跨度和`TraceContext`。之后，我们能够获得当前跟踪和跨度的 ID。此外，我们看到了不同的方法如何在不同的数字系统中返回 ID。**

和往常一样，GitHub 上的[提供了这些例子的源代码。](https://web.archive.org/web/20220524071029/https://github.com/eugenp/tutorials/tree/master/spring-sleuth)