# 使用 Spring Cloud Gateway 重写 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-url-rewriting>

## 1.介绍

Spring Cloud Gateway 的一个常见用例是充当一个或多个服务的门面，从而为客户提供一种更简单的消费方式。

在本教程中，我们将展示通过在向后端发送请求之前重写 URL 来定制公开的 API 的不同方法。

## 2.Spring 云网关快速回顾

[Spring Cloud Gateway](/web/20220524065753/https://www.baeldung.com/spring-cloud-gateway) 项目建立在广受欢迎的 Spring Boot 2 号和[项目反应堆](/web/20220524065753/https://www.baeldung.com/reactor-core)之上，因此它继承了它的主要优点:

*   由于其反应性，资源使用率低
*   支持来自 Spring Cloud 生态系统的所有好东西(发现、配置等。)
*   使用标准弹簧模式易于扩展和/或定制

我们已经在以前的文章中介绍了它的主要特性，所以这里我们只列出主要的概念:

*   `Route`:一组匹配的入局请求在网关中经过的处理步骤
*   `Predicate`:一个 Java 8 [`Predicate`](https://web.archive.org/web/20220524065753/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html) 与一个`ServerWebExchange`进行比较。
*   `Filters` : `GatewayFilter`可以检查和/或更换`ServerWebExchange`的实例。网关支持全局过滤器和每个路由过滤器。

简而言之，传入请求的处理顺序如下:

*   网关使用与每条路由相关联的`Predicates`来查找哪条路由将处理请求
*   一旦找到路由，请求(一个`ServerWebExchange`实例)将通过每个配置好的过滤器，直到最终被发送到后端。
*   当后端发送回响应，或者出现错误(例如，超时或连接重置)时，过滤器在响应被发送回客户端之前再次获得处理响应的机会。

## 3.基于配置的 URL 重写

回到本文的主题，让我们看看如何定义一个路由，在将传入的 URL 发送到后端之前重写它。例如，假设给定一个形式为`/api/v1/customer/*`的传入请求，后端 URL 应该是`http://v1.customers/api/*`。在这里，我们使用“*”来表示“超过这一点的任何东西”。

为了创建一个基于配置的重写，我们只需要在应用程序的配置中添加一些属性。这里，为了清楚起见，我们将使用基于 YAML 的配置，但是该信息可以来自任何支持的`PropertySource`:

```java
spring:
  cloud:
    gateway:
      routes:
      - id: rewrite_v1
        uri: ${rewrite.backend.uri:http://example.com}
        predicates:
        - Path=/v1/customer/**
        filters:
        - RewritePath=/v1/customer/(?<segment>.*),/api/$\{segment} 
```

我们来剖析一下这个配置。首先，我们有路由的 id，这只是它的标识符。接下来，我们有由`uri`属性给出的后端 URI。**注意，只考虑主机名/端口，因为最终路径来自重写逻辑**。

`predicates`属性定义了激活该路线必须满足的条件。在我们的例子中，我们使用了`Path`谓词，它采用一个类似蚂蚁的路径表达式来匹配传入请求的路径。

最后，`filters`属性具有实际的重写逻辑。**`RewritePath`过滤器接受两个参数:一个正则表达式和一个替换字符串。**过滤器的实现通过简单地在请求的 URI 上执行`replaceAll()`方法来工作，使用提供的参数作为参数。

Spring 处理配置文件的方式需要注意的是，我们不能使用标准的`${group}`替换表达式，因为 Spring 会认为它是一个属性引用，并试图替换它的值。为了避免这种情况，我们需要在“$”和“{”字符之间添加一个反斜杠，过滤器实现将在使用它作为实际的替换表达式之前删除该反斜杠。

## 4.基于 DSL 的 URL 重写

虽然`RewritePath`非常强大且易于使用，但在重写规则具有一些动态方面的场景中，它显得有些不足。根据具体情况，仍然可以使用谓词为规则的每个分支编写多个规则。

**但是，如果情况并非如此，我们可以使用基于 DSL 的方法创建路由。**我们需要做的就是创建一个实现路由逻辑的`RouteLocator` bean。例如，让我们创建一个简单的路由，像以前一样，使用正则表达式重写传入的 URI。但是，这一次，替换字符串将在每次请求时动态生成:

```java
@Configuration
public class DynamicRewriteRoute {

    @Value("${rewrite.backend.uri}")
    private String backendUri;
    private static Random rnd = new Random();

    @Bean
    public RouteLocator dynamicZipCodeRoute(RouteLocatorBuilder builder) {
        return builder.routes()
          .route("dynamicRewrite", r ->
             r.path("/v2/zip/**")
              .filters(f -> f.filter((exchange, chain) -> {
                  ServerHttpRequest req = exchange.getRequest();
                  addOriginalRequestUrl(exchange, req.getURI());
                  String path = req.getURI().getRawPath();
                  String newPath = path.replaceAll(
                    "/v2/zip/(?<zipcode>.*)", 
                    "/api/zip/${zipcode}-" + String.format("%03d", rnd.nextInt(1000)));
                  ServerHttpRequest request = req.mutate().path(newPath).build();
                  exchange.getAttributes().put(GATEWAY_REQUEST_URL_ATTR, request.getURI());
                  return chain.filter(exchange.mutate().request(request).build());
              }))
              .uri(backendUri))
          .build();
    }
} 
```

这里，动态部分只是附加到替换字符串的一个随机数。现实世界中的应用程序可能有更复杂的逻辑，但是基本的机制是一样的，如图所示。

关于这段代码所经历的步骤有几点说明:首先，它调用来自`ServerWebExchangeUtils`类的`addOriginalRequestUrl(),`，将原始 URL 存储在交换的属性`GATEWAY_ORIGINAL_REQUEST_URL_ATTR`下。这个属性的值是一个列表，在进行任何修改之前，我们将把接收到的 URL 附加到这个列表中，并作为`X-Forwarded-For`头处理的一部分由网关内部使用。

其次，一旦我们应用了重写逻辑，我们必须将修改后的 URL 保存在`GATEWAY_REQUEST_URL_ATTR`交换的属性中。**文档中没有直接提到这一步，但是这一步可以确保我们的自定义过滤器与其他可用的过滤器配合良好。**

## 5.测试

为了测试我们的重写规则，我们将使用标准的 [`JUnit 5`](/web/20220524065753/https://www.baeldung.com/junit-5-test-annotation) 类，并稍加改动:我们将基于 Java SDK 的 [`com.sun.net.httpserver.HttpServer`](https://web.archive.org/web/20220524065753/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpServer.html) 类构建一个简单的服务器。服务器将在随机端口上启动，从而避免端口冲突。

**然而，这种方法的缺点是我们必须找出哪个端口实际上被分配给了服务器，并将其传递给 Spring，这样我们就可以用它来设置路由的`uri`属性**。幸运的是，Spring 为我们提供了一个优雅的解决方案:`[@DynamicPropertySource](/web/20220524065753/https://www.baeldung.com/spring-dynamicpropertysource).`在这里，我们将使用它来启动服务器，并用绑定端口的值注册一个属性:

```java
@DynamicPropertySource
static void registerBackendServer(DynamicPropertyRegistry registry) {
    registry.add("rewrite.backend.uri", () -> {
        HttpServer s = startTestServer();
        return "http://localhost:" + s.getAddress().getPort();
    });
} 
```

[测试处理程序](https://web.archive.org/web/20220524065753/https://github.com/eugenp/tutorials/blob/master/spring-cloud/spring-cloud-gateway/src/test/java/com/baeldung/springcloudgateway/rewrite/URLRewriteGatewayApplicationLiveTest.java)只是在响应体中回显接收到的 URI。这允许我们验证重写规则是否如预期的那样工作。例如，这是

```java
@Test
void testWhenApiCall_thenRewriteSuccess(@Autowired WebTestClient webClient) {
    webClient.get()
      .uri("http://localhost:" + localPort + "/v1/customer/customer1")
      .exchange()
      .expectBody()
      .consumeWith((result) -> {
          String body = new String(result.getResponseBody());
          assertEquals("/api/customer1", body);
      });
} 
```

## 6.结论

在这个快速教程中，我们展示了使用 Spring Cloud Gateway 库重写 URL 的不同方法。像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220524065753/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-gateway)