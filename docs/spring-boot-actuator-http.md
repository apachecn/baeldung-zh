# 用 Spring Boot 执行器 HTTP 跟踪记录 HTTP 请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-actuator-http>

## 1。简介

当我们使用微服务或 web 服务时，了解用户如何与我们的服务交互是非常有用的。这可以通过跟踪访问我们服务的所有请求并收集这些信息以供以后分析来实现。

有一些可用的系统可以帮助我们做到这一点，并且可以很容易地与 Spring 集成，如 [Zipkin](/web/20221208143830/https://www.baeldung.com/tracing-services-with-zipkin) 。然而， **Spring Boot 执行器在**中内置了这种功能，可以通过跟踪所有 HTTP 请求的`httpTrace`端点来使用。在本教程中，我们将展示如何使用它以及如何定制它以更好地满足我们的需求。

## 2。`HttpTrace`端点设置

出于本教程的考虑，我们将使用一个专家 Spring Boot 项目。

我们需要做的第一件事是将[Spring Boot 致动器依赖](https://web.archive.org/web/20221208143830/https://search.maven.org/search?q=a:spring-boot-starter-actuator%20AND%20g:org.springframework.boot)添加到我们的项目中:

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

之后，我们必须在应用程序中启用`httpTrace`端点。

为此，我们只需要**修改我们的`application.properties`文件来包含`httpTrace`端点**:

```java
management.endpoints.web.exposure.include=httptrace
```

如果我们需要更多的端点，我们可以用逗号将它们连接起来，或者使用通配符`*`将它们全部包含在内。

现在，我们的`httpTrace`端点应该出现在应用程序的执行器端点列表中:

```java
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    "httptrace": {
      "href": "http://localhost:8080/actuator/httptrace",
      "templated": false
    }
  }
}
```

注意，我们可以通过转到 web 服务的`/actuator`端点来列出所有启用的执行器端点。

## 3。分析痕迹

现在让我们分析一下`httpTrace`致动器端点返回的轨迹。

让我们向服务发出一些请求，调用`/actuator/httptrace` 端点并获取一个返回的跟踪:

```java
{
  "traces": [
    {
      "timestamp": "2019-08-05T19:28:36.353Z",
      "principal": null,
      "session": null,
      "request": {
        "method": "GET",
        "uri": "http://localhost:8080/echo?msg=test",
        "headers": {
          "accept-language": [
            "en-GB,en-US;q=0.9,en;q=0.8"
          ],
          "upgrade-insecure-requests": [
            "1"
          ],
          "host": [
            "localhost:8080"
          ],
          "connection": [
            "keep-alive"
          ],
          "accept-encoding": [
            "gzip, deflate, br"
          ],
          "accept": [
            "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8"
          ],
          "user-agent": [
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36 OPR/62.0.3331.66"
          ]
        },
        "remoteAddress": null
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Length": [
            "12"
          ],
          "Date": [
            "Mon, 05 Aug 2019 19:28:36 GMT"
          ],
          "Content-Type": [
            "text/html;charset=UTF-8"
          ]
        }
      },
      "timeTaken": 82
    }
  ]
}
```

如我们所见，响应分为几个节点:

*   `timestamp`:收到请求的时间
*   `principal`:提出请求的认证用户(如果适用)
*   `session`:与请求相关联的任何会话
*   `request`:关于请求的信息，例如方法、完整 URI 或标题
*   `response`:关于响应的信息，例如状态或标题
*   `timeTaken`:处理请求所用的时间

如果我们觉得这个回答过于冗长，我们可以根据自己的需要进行调整。我们可以通过在我们的`application.properties`的`management.trace.http.include`属性中指定**来告诉 Spring 我们想要返回哪些字段:**

```java
management.trace.http.include=RESPONSE_HEADERS
```

在这种情况下，我们指定只需要响应头。因此，我们可以看到以前包含的字段，如请求头或花费的时间，现在在响应中不存在了:

```java
{
  "traces": [
    {
      "timestamp": "2019-08-05T20:23:01.397Z",
      "principal": null,
      "session": null,
      "request": {
        "method": "GET",
        "uri": "http://localhost:8080/echo?msg=test",
        "headers": {},
        "remoteAddress": null
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Length": [
            "12"
          ],
          "Date": [
            "Mon, 05 Aug 2019 20:23:01 GMT"
          ],
          "Content-Type": [
            "text/html;charset=UTF-8"
          ]
        }
      },
      "timeTaken": null
    }
  ]
}
```

所有可能包含的值都可以在[源代码](https://web.archive.org/web/20221208143830/https://github.com/spring-projects/spring-boot/blob/8bc780762a44c7d57e6ddf15abb7071f499857ce/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/trace/http/Include.java)中找到，默认的也一样。

## 4。定制`HttpTraceRepository`T2

**默认情况下，`httpTrace`端点只返回最后 100 个请求，并将它们存储在内存**中。好消息是，我们也可以通过创建自己的`HttpTraceRepository`来定制它。

现在让我们创建我们的存储库。`HttpTraceRepository` 接口非常简单，我们只需要实现两个方法:`findAll() `来检索所有的踪迹；和`add() `向存储库添加跟踪。

为简单起见，我们的存储库也将在内存中存储跟踪，我们将只存储最后一个命中我们服务的 GET 请求:

```java
@Repository
public class CustomTraceRepository implements HttpTraceRepository {

    AtomicReference<HttpTrace> lastTrace = new AtomicReference<>();

    @Override
    public List<HttpTrace> findAll() {
        return Collections.singletonList(lastTrace.get());
    }

    @Override
    public void add(HttpTrace trace) {
        if ("GET".equals(trace.getRequest().getMethod())) {
            lastTrace.set(trace);
        }
    }

}
```

尽管这个简单的例子看起来不是很有用，但我们可以看到它的强大之处，以及我们如何在任何地方存储我们的日志。

## 5。过滤要追踪的路径

我们要讨论的最后一件事是如何过滤我们想要跟踪的路径，这样我们就可以忽略一些我们不感兴趣的请求。

如果我们在向我们的服务发出一些请求后稍微使用一下`httpTrace`端点，我们可以看到我们还获得了执行器请求的跟踪:

```java
{
  "traces": [
    {
      "timestamp": "2019-07-28T13:56:36.998Z",
      "principal": null,
      "session": null,
      "request": {
        "method": "GET",
        "uri": "http://localhost:8080/actuator/",
         // ...
}
```

我们可能不会发现这些痕迹对我们有用，我们宁愿排除它们。在这种情况下，我们只需要**创建我们自己的`HttpTraceFilter`** ，并在`shouldNotFilter`方法中指定我们希望**忽略的路径:**

```java
@Component
public class TraceRequestFilter extends HttpTraceFilter {

  public TraceRequestFilter(HttpTraceRepository repository, HttpExchangeTracer tracer) {
      super(repository, tracer);
  }

  @Override
  protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
      return request.getServletPath().contains("actuator");
  }
}
```

注意，`HttpTraceFilter`只是一个普通的 Spring 过滤器，但是具有一些特定于跟踪的功能。

## 6。结论

在本教程中，我们介绍了`httpTrace` Spring Boot 致动器端点，并展示了其主要特点。我们还深入探讨了如何改变一些默认行为，以更好地满足我们的特定需求。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-runtime)