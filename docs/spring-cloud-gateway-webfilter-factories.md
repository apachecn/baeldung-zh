# Spring 云网关 WebFilter 工厂

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-webfilter-factories>

## 1.介绍

[春云网关](/web/20221117030334/https://www.baeldung.com/spring-cloud-gateway)是微服务中经常使用的智能代理服务。它透明地将请求集中在一个入口点，并将它们路由到适当的服务。它最有趣的特性之一是[滤镜](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#glossary) ( `WebFilter`或`GatewayFilter`)的概念。

`WebFilter,`与 [`Predicate`工厂](/web/20221117030334/https://www.baeldung.com/spring-cloud-gateway-routing-predicate-factories)一起，纳入完整的路由机制。 **Spring Cloud Gateway 提供了许多内置的`WebFilter`工厂，允许在到达代理服务之前与 HTTP 请求进行交互，在将结果传递给客户端之前与 HTTP 响应进行交互**。也可以实现[自定义过滤器](/web/20221117030334/https://www.baeldung.com/spring-cloud-custom-gateway-filters)。

在本教程中，我们将关注项目中包含的内置`WebFilter`工厂，以及如何在高级用例中使用它们。

## 2.`WebFilter`工厂

`WebFilter`(或`GatewayFilter`)工厂允许修改入站 HTTP 请求和出站 HTTP 响应。从这个意义上说，它提供了一组有趣的功能，可以在与下游服务交互之前和之后应用。

[![Spring Cloud Gateway WebFilter Factories Architecture](img/2e02144e182640ed83ea3337f5d17c78.png)](/web/20221117030334/https://www.baeldung.com/wp-content/uploads/2020/05/spring-cloud-gateway-webfilters-2.jpg)

处理程序映射管理客户端的请求。它检查它是否匹配某个配置的路由。然后，它将请求发送到 Web 处理程序，以执行该路由的特定过滤器链。虚线将前置过滤器逻辑和后置过滤器逻辑分开。传入过滤器在代理请求之前运行。输出过滤器在收到代理响应时开始动作。过滤器提供了修改中间过程的机制。

## 3.实施`WebFilter`工厂

让我们回顾一下 Spring Cloud Gateway 项目中最重要的`WebFilter`工厂。有**两种方式来实现它们，使用 YAML 或者[Java DSL](https://web.archive.org/web/20221117030334/https://docs.spring.io/spring-integration/docs/5.1.0.M1/reference/html/java-dsl.html)T4。我们将展示如何实现这两者的例子。**

### 3.1.HTTP 请求

内置的 **`WebFilter`工厂允许与 HTTP 请求**的头部和参数进行交互。我们可以[添加](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-addrequestheader-gatewayfilter-factory) `(AddRequestHeader),` [映射](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-maprequestheader-gatewayfilter-factory) `(MapRequestHeader)`，[设置或替换](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-setrequestheader-gatewayfilter-factory) `(SetRequestHeader),`，[删除](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-removerequestheader-gatewayfilter-factory) `(RemoveRequestHeader)`头值，并发送给被代理的服务。也可以保留原主机头(`PreserveHostHeader`)。

同样，我们可以通过[添加](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-addrequestparameter-gatewayfilter-factory) `(AddRequestParameter)`，[删除](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-removerequestparameter-gatewayfilter-factory) `(RemoveRequestParameter)`参数，由下游服务处理。让我们来看看怎么做:

```java
- id: add_request_header_route
  uri: https://httpbin.org
  predicates:
  - Path=/get/**
  filters:
  - AddRequestHeader=My-Header-Good,Good
  - AddRequestHeader=My-Header-Remove,Remove
  - AddRequestParameter=var, good
  - AddRequestParameter=var2, remove
  - MapRequestHeader=My-Header-Good, My-Header-Bad
  - MapRequestHeader=My-Header-Set, My-Header-Bad
  - SetRequestHeader=My-Header-Set, Set 
  - RemoveRequestHeader=My-Header-Remove
  - RemoveRequestParameter=var2
```

让我们检查一下是否一切正常。为此，我们将使用 curl 和公开可用的 httpbin.org:

```java
$ curl http://localhost:8080/get
{
  "args": {
    "var": "good"
  },
  "headers": {
    "Host": "localhost",
    "My-Header-Bad": "Good",
    "My-Header-Good": "Good",
    "My-Header-Set": "Set",
  },
  "origin": "127.0.0.1, 90.171.125.86",
  "url": "https://localhost:8080/get?var=good"
}
```

我们可以看到 curl 响应是配置请求过滤器的结果。他们把值`Good`加到`My-Header-Good`上，并把它的内容映射到`My-Header-Bad.`上，他们删除`My-Header-Remove`并给`My-Header-Set`设置一个新值。在`args` 和`url`部分，我们可以看到增加了一个新参数`var`。此外，最后一个过滤器移除了`var2`参数。

另外，**我们可以在到达被代理的服务**之前[修改请求体](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#modify-a-request-body-gatewayfilter-factory)。该过滤器只能使用 Java DSL 符号进行配置。下面的代码片段只是大写了响应正文的内容:

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
     return builder.routes()
       .route("modify_request_body", r -> r.path("/post/**")
         .filters(f -> f.modifyRequestBody(
           String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE, 
           (exchange, s) -> Mono.just(new Hello(s.toUpperCase()))))
         .uri("https://httpbin.org"))
       .build();
}
```

为了测试这个代码片段，让我们使用`-d` 选项执行 curl 来包含主体`“Content”`:

```java
$ curl -X POST "http://localhost:8080/post" -i -d "Content"
"data": "{\"message\":\"CONTENT\"}",
"json": {
    "message": "CONTENT"
}
```

我们可以看到，作为过滤的结果，正文的内容现在大写为`CONTENT`。

### 3.2.HTTP 响应

同样，**我们可以通过使用 [add](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-addresponseheader-gatewayfilter-factory) ( `AddResponseHeader`)、 [set](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-setresponseheader-gatewayfilter-factory) 或 replace ( `SetResponseHeader`)、 [remove](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#removeresponseheader-gatewayfilter-factory) ( `RemoveResponseHeader`)和 [rewrite](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-rewriteresponseheader-gatewayfilter-factory) ( `RewriteResponseHeader`)来修改响应头**。响应的另一个功能是[重复数据删除](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-deduperesponseheader-gatewayfilter-factory) ( `DedupeResponseHeader)`)以覆盖策略并避免重复。我们可以通过使用另一个内置工厂(`[RemoveLocationResponseHeader](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#rewritelocationresponseheader-gatewayfilter-factory)`)来消除关于版本、位置和主机的后端特定细节。

让我们来看一个完整的例子:

```java
- id: response_header_route
  uri: https://httpbin.org
  predicates:
  - Path=/header/post/**
  filters:
  - AddResponseHeader=My-Header-Good,Good
  - AddResponseHeader=My-Header-Set,Good
  - AddResponseHeader=My-Header-Rewrite, password=12345678
  - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
  - AddResponseHeader=My-Header-Remove,Remove
  - SetResponseHeader=My-Header-Set, Set
  - RemoveResponseHeader=My-Header-Remove
  - RewriteResponseHeader=My-Header-Rewrite, password=[^&]+, password=***
  - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
```

让我们使用 curl 来显示响应头:

```java
$ curl -X POST "http://localhost:8080/header/post" -s -o /dev/null -D -
HTTP/1.1 200 OK
My-Header-Good: Good
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
My-Header-Rewrite: password=***
My-Header-Set: Set
```

类似于 HTTP 请求，**我们可以[修改响应体](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#modify-a-response-body-gatewayfilter-factory)** 。对于本例，我们覆盖了 PUT 响应的主体:

```java
@Bean
public RouteLocator responseRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
      .route("modify_response_body", r -> r.path("/put/**")
        .filters(f -> f.modifyResponseBody(
          String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE, 
          (exchange, s) -> Mono.just(new Hello("New Body"))))
        .uri("https://httpbin.org"))
      .build();
}
```

让我们使用 PUT 端点来测试功能:

```java
$ curl -X PUT "http://localhost:8080/put" -i -d "CONTENT"
{"message":"New Body"}
```

### 3.3.小路

内置`WebFilter`工厂提供的特性之一是**与客户端**配置的路径的交互。可以通过[设置一个不同的路径](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-setpath-gatewayfilter-factory) ( `SetPath`)，[重写](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-rewritepath-gatewayfilter-factory) ( `RewritePath`)，添加一个[前缀](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-prefixpath-gatewayfilter-factory) ( `PrefixPath`)，以及[剥离](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-stripprefix-gatewayfilter-factory) ( `StripPrefix`)来只提取它的一部分。请记住，过滤器是根据它们在 YAML 文件中的位置按顺序执行的。让我们看看如何配置路由:

```java
- id: path_route
  uri: https://httpbin.org
  predicates:
  - Path=/new/post/**
  filters:
  - RewritePath=/new(?<segment>/?.*), $\{segment}
  - SetPath=/post
```

两个过滤器都在到达代理服务之前删除子路径`/new`。让我们执行 curl:

```java
$ curl -X POST "http://localhost:8080/new/post" -i
"X-Forwarded-Prefix": "/new"
"url": "https://localhost:8080/post"
```

我们也可以使用 [`StripPrefix`](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-stripprefix-gatewayfilter-factory) 工厂。使用`StripPrefix=1, `,我们可以在联系下游服务时去掉第一个子路径。

### 3.4.与 HTTP 状态相关

[`RedirectTo`](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-redirectto-gatewayfilter-factory) 带两个参数:状态和 URL。状态必须是一系列 300 重定向 HTTP 代码和有效的 URL。`[SetStatus](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-setstatus-gatewayfilter-factory) `接受一个参数状态，可以是 HTTP 代码或其字符串表示。让我们来看几个例子:

```java
- id: redirect_route
  uri: https://httpbin.org
  predicates:
  - Path=/fake/post/**
  filters:
  - RedirectTo=302, https://httpbin.org
- id: status_route
  uri: https://httpbin.org
  predicates:
  - Path=/delete/**
  filters:
  - SetStatus=401
```

**第一个过滤器作用于`/fake/post`路径，客户端被重定向到`https://httpbin.org`，HTTP 状态为 302** :

```java
$ curl -X POST "http://localhost:8080/fake/post" -i
HTTP/1.1 302 Found
Location: https://httpbin.org
```

**第二过滤器检测`/delete`路径，并且设置 HTTP 状态 401** :

```java
$ curl -X DELETE "http://localhost:8080/delete" -i
HTTP/1.1 401 Unauthorized
```

### 3.5.请求大小限制

最后，我们可以[限制请求(`RequestSize`)的大小限制](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-requestsize-gatewayfilter-factory)。**如果请求大小超出限制，网关拒绝访问服务**:

```java
- id: size_route
  uri: https://httpbin.org
  predicates:
  - Path=/anything
  filters:
  - name: RequestSize
    args:
       maxSize: 5000000
```

## 4.高级用例

Spring Cloud Gateway 提供了其他高级`WebFilter`工厂来支持微服务模式的基线功能。

### 4.1.断路器

春云网关有一个**内置`WebFilter`工厂用于[断路器](/web/20221117030334/https://www.baeldung.com/spring-cloud-circuit-breaker)能力**。工厂允许不同的回退策略和 Java DSL 路由配置。让我们看一个简单的例子:

```java
- id: circuitbreaker_route
  uri: https://httpbin.org
  predicates:
  - Path=/status/504
  filters:
  - name: CircuitBreaker
  args:
     name: myCircuitBreaker
     fallbackUri: forward:/anything
  - RewritePath=/status/504, /anything
```

对于断路器的配置，我们通过添加 [`spring-cloud-starter-circuitbreaker-reactor-resilience4j`](https://web.archive.org/web/20221117030334/https://search.maven.org/search?q=g:org.springframework.cloud%20a:spring-cloud-starter-circuitbreaker-reactor-resilience4j) 依赖关系来使用 [Resilience4J](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html) :

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

同样，我们可以使用 curl 测试功能:

```java
$ curl http://localhost:8080/status/504 
"url": "https://localhost:8080/anything"
```

### 4.2.重试

另一个高级特性**允许客户端在代理服务发生问题时重试访问**。它需要几个参数，例如应该重试的`retries`号、HTTP 状态码(`statuses`)和`methods`、每次重试后等待的`series`、`exceptions,`和`backoff`间隔。让我们来看看 YAML 的配置:

```java
- id: retry_test
  uri: https://httpbin.org
  predicates:
  - Path=/status/502
  filters:
  - name: Retry
    args:
       retries: 3
       statuses: BAD_GATEWAY
       methods: GET,POST
       backoff:
          firstBackoff: 10ms
          maxBackoff: 50ms
          factor: 2
          basedOnPreviousValue: false
```

当客户端到达`/status/502` (坏网关)时，过滤器重试三次，等待每次执行后配置的退避间隔。让我们看看它是如何工作的:

```java
$ curl http://localhost:8080/status/502
```

同时，我们需要检查服务器中的网关日志:

```java
Mapping [Exchange: GET http://localhost:8080/status/502] to Route{id='retry_test', ...}
Handler is being applied: {uri=https://httpbin.org/status/502, method=GET}
Received last HTTP packet
Handler is being applied: {uri=https://httpbin.org/status/502, method=GET}
Received last HTTP packet
Handler is being applied: {uri=https://httpbin.org/status/502, method=GET}
Received last HTTP packet
```

当网关接收到状态 502 时，过滤器对方法 GET 和 POST 用这种回退重试三次。

### 4.3.保存会话和安全标题

[`SecureHeader`](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-secureheaders-gatewayfilter-factory) 工厂将 **HTTP 安全报头添加到响应**中。同样， [`SaveSession`](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-savesession-gatewayfilter-factory) 与`Spring Session``Spring Security`连用时尤为重要:

```java
filters: 
- SaveSession
```

该过滤器**在进行前转呼叫**之前存储会话状态。

### 4.4.请求速率限制器

最后但同样重要的是， [`RequestRateLimiter`](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-requestratelimiter-gatewayfilter-factory) 工厂**确定请求是否可以继续**。如果没有，它返回一个`HTTP code status 429 – Too Many Requests`。**它使用不同的参数和分解器来指定速率限制器**。

[`RedisRateLimiter`](https://web.archive.org/web/20221117030334/https://cloud.spring.io/spring-cloud-gateway/reference/html/#the-redis-ratelimiter) 使用众所周知的 [`Redis`](/web/20221117030334/https://www.baeldung.com/spring-data-redis-tutorial) 数据库来检查桶可以保留的代币数量。它需要以下依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
 </dependency>
```

因此，它还需要配置`Spring Redis`:

```java
spring:
  redis:
    host: localhost
    port: 6379
```

该过滤器有几个属性。第一个参数`replenishRate,` 是每秒允许的请求数。第二个参数`burstCapacity,`是每秒钟的最大请求数。第三个参数`requestedTokens,`是请求花费多少令牌。让我们来看一个示例实现:

```java
- id: request_rate_limiter
  uri: https://httpbin.org
  predicates:
  - Path=/redis/get/**
  filters:
  - StripPrefix=1
  - name: RequestRateLimiter
    args:
       redis-rate-limiter.replenishRate: 10
       redis-rate-limiter.burstCapacity: 5
```

让我们用 curl 来测试过滤器。事先，记得启动一个`Redis` 实例，例如使用 [`Docker`](https://web.archive.org/web/20221117030334/https://hub.docker.com/_/redis/) :

```java
$ curl "http://localhost:8080/redis/get" -i
HTTP/1.1 200 OK
X-RateLimit-Remaining: 4
X-RateLimit-Requested-Tokens: 1
X-RateLimit-Burst-Capacity: 5
X-RateLimit-Replenish-Rate: 10
```

一旦剩余速率限制达到零，网关就发出 HTTP 代码 429。为了测试行为，我们可以使用单元测试。我们启动一个[嵌入式 Redis 服务器](/web/20221117030334/https://www.baeldung.com/spring-embedded-redis)，并行运行`RepeatedTests`。一旦铲斗达到极限，错误开始显示:

```java
00:57:48.263 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->200, reason->OK, remaining->[4]
00:57:48.394 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->200, reason->OK, remaining->[3]
00:57:48.530 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->200, reason->OK, remaining->[2]
00:57:48.667 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->200, reason->OK, remaining->[1]
00:57:48.826 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->200, reason->OK, remaining->[0]
00:57:48.851 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->429, reason->Too Many Requests, remaining->[0]
00:57:48.894 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->429, reason->Too Many Requests, remaining->[0]
00:57:49.135 [main] INFO  c.b.s.w.RedisWebFilterFactoriesLiveTest - Received: status->200, reason->OK, remaining->[4]
```

## 5.结论

在本教程中，我们介绍了 Spring Cloud Gateway 的`WebFilter`工厂。我们展示了如何在执行代理服务之前和之后与来自客户机的请求和响应进行交互。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221117030334/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway)