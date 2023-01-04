# Spring 5 WebClient

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-5-webclient>

## 1.概观

在本教程中，我们将研究`WebClient`，它是 Spring 5 中引入的一个反应式 web 客户端。

我们还将看看设计用于测试的`WebTestClient,` a `WebClient`。

## 延伸阅读:

## [Spring WebClient 过滤器](/web/20221101163207/https://www.baeldung.com/spring-webclient-filters)

Learn about WebClient filters in Spring WebFlux[Read more](/web/20221101163207/https://www.baeldung.com/spring-webclient-filters) →

## [带参数的 Spring WebClient 请求](/web/20221101163207/https://www.baeldung.com/webflux-webclient-parameters)

Learn how to reactively consume REST API endpoints with WebClient from Spring Webflux.[Read more](/web/20221101163207/https://www.baeldung.com/webflux-webclient-parameters) →

## 2.什么是`WebClient`？

简单地说， `WebClient`是一个表示执行 web 请求的主入口点的接口。

它是作为 Spring Web 反应模块的一部分创建的，将在这些场景中取代经典的`RestTemplate` 。此外，新的客户机是一个被动的、非阻塞的解决方案，它基于 HTTP/1.1 协议工作。

值得注意的是，尽管它实际上是一个非阻塞的客户机，并且属于`spring-webflux` 库，但是该解决方案提供了对同步和异步操作的支持，这使得它也适用于运行在 Servlet 栈上的应用程序。

这可以通过阻止获取结果的操作来实现。当然，如果我们正在处理一个反应式堆栈，这种做法是不推荐的。

最后，该接口有一个单独的实现，即我们将要使用的`DefaultWebClient`类。

## 3.属国

因为我们使用的是 Spring Boot 应用程序，所以我们需要的是 [`spring-boot-starter-webflux`](https://web.archive.org/web/20221101163207/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-webflux%22%20AND%20g%3A%22org.springframework.boot%22) 依赖项来获得 Spring Framework 的反应式 Web 支持。

### 3.1。用 Maven 构建

让我们将以下依赖项添加到`pom.xml`文件中:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

### 3.2。用 Gradle 建造

对于 Gradle，我们需要将以下条目添加到`build.gradle`文件中:

```
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-webflux'
}
```

## 4.与`WebClient`一起工作

为了与客户正常合作，我们需要知道如何:

*   创建实例
*   请求
*   处理响应

### 4.1。创建一个`WebClient`实例

有三个选项可供选择。第一个是使用默认设置创建一个 *WebClient* 对象:

```
WebClient client = WebClient.create(); 
```

第二个选项是用给定的基本 URI 启动一个 *WebClient* 实例:

```
WebClient client = WebClient.create("http://localhost:8080"); 
```

第三个选项(也是最高级的一个)是使用 *DefaultWebClientBuilder* 类构建一个客户端，它允许完全定制:

```
WebClient client = WebClient.builder()
  .baseUrl("http://localhost:8080")
  .defaultCookie("cookieKey", "cookieValue")
  .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE) 
  .defaultUriVariables(Collections.singletonMap("url", "http://localhost:8080"))
  .build();
```

### 4.2。创建超时的`WebClient`实例

通常，默认的 30 秒 HTTP 超时对于我们的需求来说太慢了，为了定制这个行为，我们可以创建一个`HttpClient`实例并配置我们的`WebClient` 来使用它。

我们可以:

*   **通过`ChannelOption.CONNECT_TIMEOUT_MILLIS`选项**设置连接超时
*   **使用`ReadTimeoutHandler`和`WriteTimeoutHandler`分别设置读写超时**
*   **使用`responseTimeout` 指令**配置响应超时

正如我们所说，所有这些都必须在我们将要配置的`HttpClient`实例中指定:

```
HttpClient httpClient = HttpClient.create()
  .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
  .responseTimeout(Duration.ofMillis(5000))
  .doOnConnected(conn -> 
    conn.addHandlerLast(new ReadTimeoutHandler(5000, TimeUnit.MILLISECONDS))
      .addHandlerLast(new WriteTimeoutHandler(5000, TimeUnit.MILLISECONDS)));

WebClient client = WebClient.builder()
  .clientConnector(new ReactorClientHttpConnector(httpClient))
  .build();
```

注意**虽然我们也可以在客户端请求上调用`timeout` ，但这是一个信号超时，而不是 HTTP 连接、读/写或响应超时；单声道/通量发行商暂停了。**

### 4.3。准备请求–定义方法

首先，我们需要通过调用*方法(HttpMethod method)* 来指定请求的 HTTP 方法:

```
UriSpec<RequestBodySpec> uriSpec = client.method(HttpMethod.POST);
```

或者调用其快捷方式如`get`、 *post* 、 *delete* :

```
UriSpec<RequestBodySpec> uriSpec = client.post();
```

注意:虽然看起来我们重复使用了请求规范变量(`WebClient.UriSpec`、`WebClient.RequestBodySpec`、`WebClient.RequestHeadersSpec`、`WebClient.`、`ResponseSpec`)，但这只是为了简单起见，来介绍不同的方法。这些指令不应该在不同的请求中重用，它们检索引用，因此后面的操作会修改我们在前面步骤中所做的定义。

### 4.4。准备请求–定义 URL

下一步是提供一个 URL。同样，我们有不同的方法。

我们可以将它作为*字符串:*传递给 *uri* API

```
RequestBodySpec bodySpec = uriSpec.uri("/resource");
```

使用`UriBuilder Function`:

```
RequestBodySpec bodySpec = uriSpec.uri(
  uriBuilder -> uriBuilder.pathSegment("/resource").build());
```

或者作为一个`java.net.URL`实例:

```
RequestBodySpec bodySpec = uriSpec.uri(URI.create("/resource"));
```

请记住，如果我们为`WebClient`定义了一个默认的基本 URL，那么最后一个方法将覆盖这个值。

### 4.5。准备请求–定义主体

然后，如果需要的话，我们可以设置请求体、内容类型、长度、cookies 或标题。

例如，如果我们想要设置一个请求体，有几种可用的方法。可能最常见和最直接的选择是使用`bodyValue` 方法:

```
RequestHeadersSpec<?> headersSpec = bodySpec.bodyValue("data");
```

或者通过向`body` 方法提供一个`Publisher` (以及将要发布的元素的类型):

```
RequestHeadersSpec<?> headersSpec = bodySpec.body(
  Mono.just(new Foo("name")), Foo.class);
```

或者，我们可以利用`BodyInserters`实用程序类。例如，让我们看看如何使用一个简单的对象填充请求体，就像我们对`bodyValue` 方法`:`所做的那样

```
RequestHeadersSpec<?> headersSpec = bodySpec.body(
  BodyInserters.fromValue("data"));
```

类似地，如果我们使用一个反应器实例，我们可以使用`BodyInserters#fromPublisher`方法:

```
RequestHeadersSpec headersSpec = bodySpec.body(
  BodyInserters.fromPublisher(Mono.just("data")),
  String.class);
```

这个类还提供了其他直观的函数来涵盖更高级的场景。例如，如果我们必须发送多部分请求:

```
LinkedMultiValueMap map = new LinkedMultiValueMap();
map.add("key1", "value1");
map.add("key2", "value2");
RequestHeadersSpec<?> headersSpec = bodySpec.body(
  BodyInserters.fromMultipartData(map));
```

所有这些方法都创建了一个`BodyInserter`实例，然后我们可以将它作为请求的`body` 来呈现。

***body inserter*是一个接口，负责用给定的输出消息和插入期间使用的上下文填充*ReactiveHttpOutputMessage*主体。**

一个`Publisher`是一个负责提供无限数量的有序元素的电抗组件。它也是一个接口，最流行的实现是`Mono` 和 `Flux.`

### 4.6。准备请求–定义标题

在我们设置了主体之后，我们可以设置头、cookies 和可接受的媒体类型。**实例化客户端时，值将添加到已经设置的值中。**

此外，还额外支持最常用的头文件，如`“If-None-Match”, “If-Modified-Since”, “Accept”,`和 `“Accept-Charset”.`

以下是如何使用这些值的示例:

```
ResponseSpec responseSpec = headersSpec.header(
    HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
  .accept(MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML)
  .acceptCharset(StandardCharsets.UTF_8)
  .ifNoneMatch("*")
  .ifModifiedSince(ZonedDateTime.now())
  .retrieve();
```

### 4.7。得到回应

最后一个阶段是发送请求和接收响应。我们可以通过使用`exchangeToMono/exchangeToFlux`或`retrieve`方法来实现这一点。

`exchangeToMono` 和`exchangeToFlux`方法允许访问`ClientResponse`及其状态和标题:

```
Mono<String> response = headersSpec.exchangeToMono(response -> {
  if (response.statusCode().equals(HttpStatus.OK)) {
      return response.bodyToMono(String.class);
  } else if (response.statusCode().is4xxClientError()) {
      return Mono.just("Error response");
  } else {
      return response.createException()
        .flatMap(Mono::error);
  }
});
```

而`retrieve`方法是直接获取身体的最短路径:

```
Mono<String> response = headersSpec.retrieve()
  .bodyToMono(String.class);
```

需要注意的是`ResponseSpec.` `bodyToMono`方法，如果状态码为 *4xx* (客户端错误)或 *5xx* (服务器错误)，该方法将抛出一个 *WebClientException* 。

## 5.与`WebTestClient`一起工作

*WebTestClient* 是测试 WebFlux 服务器端点的主要入口点。它有一个与 *WebClient* 非常相似的 API，它将大部分工作委托给一个内部 *WebClient* 实例，主要关注于提供一个测试环境。 *DefaultWebTestClient* 类是一个单接口实现。

用于测试的客户端可以绑定到真实的服务器，或者使用特定的控制器或功能。

### 5.1.绑定到服务器

要完成对正在运行的服务器的实际请求的端到端集成测试，我们可以使用 *bindToServer* 方法:

```
WebTestClient testClient = WebTestClient
  .bindToServer()
  .baseUrl("http://localhost:8080")
  .build(); 
```

### 5.2.绑定到路由器

我们可以通过将特定的 *RouterFunction* 传递给 *bindToRouterFunction* 方法来测试它:

```
RouterFunction function = RouterFunctions.route(
  RequestPredicates.GET("/resource"),
  request -> ServerResponse.ok().build()
);

WebTestClient
  .bindToRouterFunction(function)
  .build().get().uri("/resource")
  .exchange()
  .expectStatus().isOk()
  .expectBody().isEmpty(); 
```

### 5.3.绑定到 Web 处理程序

使用`bindToWebHandler`方法可以实现相同的行为，该方法采用一个`WebHandler`实例:

```
WebHandler handler = exchange -> Mono.empty();
WebTestClient.bindToWebHandler(handler).build();
```

### 5.4.绑定到应用程序上下文

当我们使用`bindToApplicationContext`方法时，会出现一个更有趣的情况。它接受一个`ApplicationContext` 并分析控制器 beans 和`@EnableWebFlux`配置的上下文。

如果我们注入一个`ApplicationContext`的实例，一个简单的代码片段可能如下所示:

```
@Autowired
private ApplicationContext context;

WebTestClient testClient = WebTestClient.bindToApplicationContext(context)
  .build(); 
```

### 5.5.绑定到控制器

一个更短的方法是提供一组我们想通过`bindToController`方法测试的控制器。假设我们已经有了一个*控制器*类，并将它注入到一个需要的类中，我们可以写:

```
@Autowired
private Controller controller;

WebTestClient testClient = WebTestClient.bindToController(controller).build(); 
```

### 5.6.提出请求

在构建了一个 *WebTestClient* 对象之后，链中的所有后续操作都将类似于*webtest client*，直到*交换*方法(获得响应的一种方式)提供了 *WebTestClient。ResponseSpec* 接口使用有用的方法，如 *expectStatus* 、 *expectBody* 和 *expectHeader* :

```
WebTestClient
  .bindToServer()
    .baseUrl("http://localhost:8080")
    .build()
    .post()
    .uri("/resource")
  .exchange()
    .expectStatus().isCreated()
    .expectHeader().valueEquals("Content-Type", "application/json")
    .expectBody().jsonPath("field").isEqualTo("value"); 
```

## 6.结论

在本文中，我们探索了一种新的增强的 Spring 机制，用于在客户端发出请求。

我们还通过配置客户机、准备请求和处理响应来了解它提供的好处。

文章中提到的所有代码片段都可以在我们的 GitHub 资源库中找到。