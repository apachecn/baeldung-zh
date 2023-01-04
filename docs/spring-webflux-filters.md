# Spring WebFlux 过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-filters>

## 1.概观

过滤器的使用在 web 应用程序中非常普遍，因为它们为我们提供了一种在不改变端点的情况下修改请求或响应的方法。

在这个快速教程中，我们将描述用 WebFlux 框架实现它们的可能方法。

由于我们不会深入讨论 WebFlux 框架本身的细节，您可能想要查看[这篇文章](/web/20220625073600/https://www.baeldung.com/spring-5-functional-web)以了解更多细节。

## 2.Maven 依赖性

首先，让我们声明 WebFlux Maven 依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

## 3.端点

我们必须首先创建一些端点。每种方法一个:基于注释的和基于函数的。

让我们从基于注释的控制器开始:

```java
@GetMapping(path = "/users/{name}")
public Mono<String> getName(@PathVariable String name) {
    return Mono.just(name);
}
```

对于功能端点，我们必须首先创建一个处理程序:

```java
@Component
public class PlayerHandler {
    public Mono<ServerResponse> getName(ServerRequest request) {
        Mono<String> name = Mono.just(request.pathVariable("name"));
        return ok().body(name, String.class);
    }
}
```

以及路由器配置映射:

```java
@Bean
public RouterFunction<ServerResponse> route(PlayerHandler playerHandler) {
    return RouterFunctions
      .route(GET("/players/{name}"), playerHandler::getName)
      .filter(new ExampleHandlerFilterFunction());
}
```

## 4.WebFlux 过滤器的类型

WebFlux 框架提供两种类型的过滤器: [`WebFilter` s](https://web.archive.org/web/20220625073600/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/server/WebFilter.html) 和`[HandlerFilterFunctions](https://web.archive.org/web/20220625073600/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/server/HandlerFilterFunction.html)`。

它们之间的主要区别在于， **`WebFilter`实现适用于所有端点**，而 **`HandlerFilterFunction`实现仅适用于基于`Router`的端点。**

### 4.1.`WebFilter`

我们将实现一个`WebFilter`来向响应添加一个新的头。因此，所有响应都应该具有这种行为:

```java
@Component
public class ExampleWebFilter implements WebFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange serverWebExchange, 
      WebFilterChain webFilterChain) {

        serverWebExchange.getResponse()
          .getHeaders().add("web-filter", "web-filter-test");
        return webFilterChain.filter(serverWebExchange);
    }
}
```

### 4.2.`HandlerFilterFunction`

对于这一个，我们实现了一个逻辑，当“name”参数等于“test”时，将 HTTP 状态设置为`FORBIDDEN`。

```java
public class ExampleHandlerFilterFunction 
  implements HandlerFilterFunction<ServerResponse, ServerResponse> {

    @Override
    public Mono<ServerResponse> filter(ServerRequest serverRequest,
      HandlerFunction<ServerResponse> handlerFunction) {
        if (serverRequest.pathVariable("name").equalsIgnoreCase("test")) {
            return ServerResponse.status(FORBIDDEN).build();
        }
        return handlerFunction.handle(serverRequest);
    }
}
```

## 5.测试

在 WebFlux 框架中，有一个简单的方法来测试我们的过滤器:`WebTestClient`。它允许我们测试对端点的 HTTP 调用。

以下是基于注释的端点的示例:

```java
@Test
public void whenUserNameIsBaeldung_thenWebFilterIsApplied() {
    EntityExchangeResult<String> result = webTestClient.get()
      .uri("/users/baeldung")
      .exchange()
      .expectStatus().isOk()
      .expectBody(String.class)
      .returnResult();

    assertEquals(result.getResponseBody(), "baeldung");
    assertEquals(
      result.getResponseHeaders().getFirst("web-filter"), 
      "web-filter-test");
}

@Test
public void whenUserNameIsTest_thenHandlerFilterFunctionIsNotApplied() {
    webTestClient.get().uri("/users/test")
      .exchange()
      .expectStatus().isOk();
}
```

对于功能端点:

```java
@Test
public void whenPlayerNameIsBaeldung_thenWebFilterIsApplied() {
    EntityExchangeResult<String> result = webTestClient.get()
      .uri("/players/baeldung")
      .exchange()
      .expectStatus().isOk()
      .expectBody(String.class)
      .returnResult();

    assertEquals(result.getResponseBody(), "baeldung");
    assertEquals(
      result.getResponseHeaders().getFirst("web-filter"),
      "web-filter-test");
} 

@Test 
public void whenPlayerNameIsTest_thenHandlerFilterFunctionIsApplied() {
    webTestClient.get().uri("/players/test")
      .exchange()
      .expectStatus().isForbidden(); 
}
```

## 6.结论

我们已经在本教程中讨论了这两种类型的 WebFlux 过滤器，并看了一些代码示例。

关于 WebFlux 框架的更多信息，请看一下[文档](https://web.archive.org/web/20220625073600/https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html)。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625073600/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-reactive)