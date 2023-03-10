# 在 Spring Cloud Gateway 中处理响应体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-response-body>

## 1.介绍

在本教程中，我们将了解如何使用 Spring Cloud Gateway 在将响应发送回客户端之前检查和/或修改响应体。

## 2.Spring 云网关快速回顾

Spring Cloud Gateway，简称 SCG，是 Spring Cloud 家族的一个子项目，它提供了一个构建在反应式 web 堆栈之上的 API 网关。我们已经在[的早期教程](/web/20220813070922/https://www.baeldung.com/?s=%22spring+cloud+gateway%22)中介绍了它的基本用法，所以我们在这里就不再赘述了。

相反，这一次我们将关注围绕 API 网关设计解决方案时不时出现的特定使用场景:如何在将后端响应负载发送回客户端之前对其进行处理？

这里列出了一些我们可能会用到这种能力的情况:

*   保持与现有客户端的兼容性，同时允许后端发展
*   掩盖某些领域遵守 PCI 或 GDPR 等法规的责任

更实际地说，满足这些需求意味着我们需要实现一个过滤器来处理后端响应。由于过滤器是 SCG 中的一个核心概念，所以我们支持响应处理所需要做的就是实现一个应用所需转换的自定义过滤器。

此外，一旦我们创建了过滤器组件，我们就可以将它应用于任何声明的路由。

## 3.实现数据清理过滤器

为了更好地说明响应体操作是如何工作的，让我们创建一个简单的过滤器来屏蔽基于 JSON 的响应中的值。例如，假设一个 JSON 有一个名为“ssn”的字段:

```java
{
  "name" : "John Doe",
  "ssn" : "123-45-9999",
  "account" : "9999888877770000"
}
```

我们希望用一个固定值替换它们的值，从而防止数据泄漏:

```java
{
  "name" : "John Doe",
  "ssn" : "****",
  "account" : "9999888877770000"
}
```

### 3.1.实施`GatewayFilterFactory`

一个`GatewayFilterFactory`顾名思义就是给定时间的滤镜的工厂。在启动时，Spring 寻找任何实现这个接口的[`@Component`](/web/20220813070922/https://www.baeldung.com/spring-component-annotation)-注释类。然后，它构建一个可用过滤器的注册表，我们可以在声明路由时使用:

```java
spring:
  cloud:
    gateway:
      routes:
      - id: rewrite_with_scrub
        uri: ${rewrite.backend.uri:http://example.com}
        predicates:
        - Path=/v1/customer/**
        filters:
        - RewritePath=/v1/customer/(?<segment>.*),/api/$\{segment}
        - ScrubResponse=ssn,*** 
```

**注意，当使用这种基于配置的方法来定义路线时，根据 SCG 预期的命名惯例** : `FilterNameGatewayFilterFactory`来命名我们的工厂是很重要的。考虑到这一点，我们将把我们的工厂命名为`ScrubResponseGatewayFilterFactory.`

SCG 已经有了几个实用程序类，我们可以用它们来实现这个工厂。这里，我们将使用一个开箱即用的过滤器常用的类:`AbstractGatewayFilterFactory<T>`，一个模板化的基类，其中 T 代表与我们的过滤器实例相关联的配置类。在我们的例子中，我们只需要两个配置属性:

*   `fields`:用于匹配字段名的正则表达式
*   `replacement`:将替换原始值的字符串

**我们必须实现的关键方法是`apply()`** 。SCG 为使用我们的过滤器的每个路由定义调用这个方法。例如，在上面的配置中，`apply()`将只被调用一次，因为只有一个路由定义。

在我们的例子中，实现很简单:

```java
@Override
public GatewayFilter apply(Config config) {
    return modifyResponseBodyFilterFactory
       .apply(c -> c.setRewriteFunction(JsonNode.class, JsonNode.class, new Scrubber(config)));
} 
```

这种情况下非常简单，因为我们使用了另一个内置过滤器`ModifyResponseBodyGatewayFilterFactory`，我们将所有与主体解析和类型转换相关的繁重工作委托给它。我们使用构造函数注入来获得这个工厂的一个实例，在`apply(),` 中，我们将创建一个`GatewayFilter`实例的任务委托给它。

**这里的关键点是使用`apply()`方法变体，它不接受配置对象，而是为配置**期待一个`Consumer`。同样重要的是，这个配置是一个`ModifyResponseBodyGatewayFilterFactory`配置。这个配置对象提供了我们在代码中调用的`setRewriteFunction()`方法。

### 3.2.使用`setRewriteFunction()`

现在，让我们更深入地了解一下`setRewriteFunction().`

这个方法有三个参数:两个类(in 和 out)和一个可以从传入类型转换为传出类型的函数。在我们的例子中，我们没有转换类型，所以输入和输出都使用同一个类:`JsonNode`。这个类来自 [Jackson](/web/20220813070922/https://www.baeldung.com/jackson) 库，位于类层次结构的最顶端，用于表示 JSON 中不同的节点类型，比如对象节点、数组节点等等。使用`JsonNode`作为输入/输出类型允许我们处理任何有效的 JSON 有效负载，这是我们在本例中想要的。

对于 transformer 类，我们传递了我们的`Scrubber`的一个实例，它在其`apply()`方法中实现了所需的`RewriteFunction`接口:

```java
public static class Scrubber implements RewriteFunction<JsonNode,JsonNode> {
    // ... fields and constructor omitted
    @Override
    public Publisher<JsonNode> apply(ServerWebExchange t, JsonNode u) {
        return Mono.just(scrubRecursively(u));
    }
    // ... scrub implementation omitted
} 
```

传递给`apply()`的第一个参数是当前的`ServerWebExchange`，它让我们可以访问到目前为止的请求处理上下文。我们不会在这里使用它，但很高兴知道我们有这个能力。下一个参数是接收的主体，已经转换为通知的类内。

预期收益是一个 [`Publisher`](/web/20220813070922/https://www.baeldung.com/reactor-core) 的通知器类的实例。所以，只要我们不做任何类型的阻塞 I/O 操作，我们就可以在重写函数内部做一些复杂的工作。

### 3.3.`Scrubber`实施

所以，现在我们知道了重写函数的契约，让我们最终实现我们的擦洗器逻辑。**这里，我们将假设有效载荷相对较小，因此我们不必担心存储接收到的对象**的内存需求。

它的实现只是递归遍历所有节点，查找与配置的模式匹配的属性，并替换掩码的相应值:

```java
public static class Scrubber implements RewriteFunction<JsonNode,JsonNode> {
    // ... fields and constructor omitted
    private JsonNode scrubRecursively(JsonNode u) {
        if ( !u.isContainerNode()) {
            return u;
        }

        if (u.isObject()) {
            ObjectNode node = (ObjectNode)u;
            node.fields().forEachRemaining((f) -> {
                if ( fields.matcher(f.getKey()).matches() && f.getValue().isTextual()) {
                    f.setValue(TextNode.valueOf(replacement));
                }
                else {
                    f.setValue(scrubRecursively(f.getValue()));
                }
            });
        }
        else if (u.isArray()) {
            ArrayNode array = (ArrayNode)u;
            for ( int i = 0 ; i < array.size() ; i++ ) {
                array.set(i, scrubRecursively(array.get(i)));
            }
        }

        return u;
    }
} 
```

## 4.测试

我们在示例代码中包含了两个测试:一个简单的单元测试和一个集成测试。第一个只是常规的 [JUnit](/web/20220813070922/https://www.baeldung.com/junit-5) 测试，用于对洗涤器进行健全性检查。**集成测试更有趣，因为它展示了 SCG 开发环境中的有用技术。**

首先，存在提供一个可以发送消息的真实后端的问题。一种可能性是使用外部工具，如 Postman 或等效工具，这给典型的 CI/CD 场景带来了一些问题。相反，我们将使用 JDK 鲜为人知的`HttpServer`类，它实现了一个简单的 HTTP 服务器。

```java
@Bean
public HttpServer mockServer() throws IOException {
    HttpServer server = HttpServer.create(new InetSocketAddress(0),0);
    server.createContext("/customer", (exchange) -> {
        exchange.getResponseHeaders().set("Content-Type", "application/json");

        byte[] response = JSON_WITH_FIELDS_TO_SCRUB.getBytes("UTF-8");
        exchange.sendResponseHeaders(200,response.length);
        exchange.getResponseBody().write(response);
    });

    server.setExecutor(null);
    server.start();
    return server;
} 
```

该服务器将在`/customer`处理请求，并返回我们测试中使用的固定 JSON 响应。请注意，返回的服务器已经启动，并将在随机端口侦听传入的请求。我们还指示服务器创建一个新的缺省值`Executor`来管理用于处理请求的线程

其次，我们以编程方式创建一个包含我们的过滤器的路由`@Bean`。这相当于使用配置属性构建路由，但允许我们完全控制测试路由的所有方面:

```java
@Bean
public RouteLocator scrubSsnRoute(
  RouteLocatorBuilder builder, 
  ScrubResponseGatewayFilterFactory scrubFilterFactory, 
  SetPathGatewayFilterFactory pathFilterFactory, 
  HttpServer server) {
    int mockServerPort = server.getAddress().getPort();
    ScrubResponseGatewayFilterFactory.Config config = new ScrubResponseGatewayFilterFactory.Config();
    config.setFields("ssn");
    config.setReplacement("*");

    SetPathGatewayFilterFactory.Config pathConfig = new SetPathGatewayFilterFactory.Config();
    pathConfig.setTemplate("/customer");

    return builder.routes()
      .route("scrub_ssn",
         r -> r.path("/scrub")
           .filters( 
              f -> f
                .filter(scrubFilterFactory.apply(config))
                .filter(pathFilterFactory.apply(pathConfig)))
           .uri("http://localhost:" + mockServerPort ))
      .build();
} 
```

最后，有了那些 beans 现在是`@TestConfiguration`的一部分，我们可以把它们和 [`WebTestClient`](/web/20220813070922/https://www.baeldung.com/spring-5-webclient#workingwebtestclient) 一起注入到实际的测试中。实际测试使用这个`WebTestClient`来驱动旋转的 SCG 和后端:

```java
@Test
public void givenRequestToScrubRoute_thenResponseScrubbed() {
    client.get()
      .uri("/scrub")
      .accept(MediaType.APPLICATION_JSON)
      .exchange()
      .expectStatus()
        .is2xxSuccessful()
      .expectHeader()
        .contentType(MediaType.APPLICATION_JSON)
      .expectBody()
        .json(JSON_WITH_SCRUBBED_FIELDS);
} 
```

## 5.结论

在本文中，我们展示了如何访问后端服务的响应体，并使用 Spring Cloud Gateway 库对其进行修改。像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220813070922/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway)