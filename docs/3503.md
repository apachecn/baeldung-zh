# 如何用 Spring WebFlux 返回 404

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-404>

## 1.概观

有了 Spring Boot 2 和新的非阻塞服务器 Netty，我们不再有 Servlet 上下文 API，所以让我们讨论如何使用新的堆栈来表达不同种类的 HTTP 状态代码。

## 2.语义响应状态

遵循标准的 RESTful 实践，我们自然需要利用完整的 HTTP 状态代码来恰当地表达 API 的语义。

### 2.1.默认退货状态

当然，当一切顺利时，默认的响应状态是 **200 (OK)** :

```
@GetMapping(
  value = "/ok",
  produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public Flux<String> ok() {
    return Flux.just("ok");
} 
```

### 2.2.使用注释

我们可以通过向方法添加`@ResponseStatus`注释来改变默认的返回状态:

```
@GetMapping(
  value = "/no-content",
  produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
@ResponseStatus(HttpStatus.NO_CONTENT)
public Flux<String> noContent() {
    return Flux.empty();
}
```

### 2.3.以编程方式更改状态

在某些情况下，根据我们的服务器的行为，我们可以决定以编程的方式改变返回的状态，而不是默认使用的带前缀的返回状态或带注释的返回状态。

我们可以在我们的方法中直接实现注入`ServerHttpResponse `:

```
@GetMapping(
  value = "/accepted",
  produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public Flux<String> accepted(ServerHttpResponse response) {
    response.setStatusCode(HttpStatus.ACCEPTED);
    return Flux.just("accepted");
}
```

现在我们可以选择在实现中返回哪个 HTTP 状态代码。

### 2.4.引发异常

每当我们抛出一个异常，默认的 HTTP 返回状态被忽略，Spring 试图找到一个异常处理程序来处理它:

```
@GetMapping(
  value = "/bad-request"
)
public Mono<String> badRequest() {
    return Mono.error(new IllegalArgumentException());
}
@ResponseStatus(
  value = HttpStatus.BAD_REQUEST,
  reason = "Illegal arguments")
@ExceptionHandler(IllegalArgumentException.class)
public void illegalArgumentHandler() {
    // 
}
```

要了解如何做到这一点的更多信息，一定要查看 Baeldung 上的[错误处理文章。](/web/20220626210451/https://www.baeldung.com/exception-handling-for-rest-with-spring)

### 2.5.用`ResponseEntity`

现在让我们快速看一下另一个有趣的选择——`ResponseEntity`类。

这允许我们选择想要返回的 HTTP 状态，并使用非常有用的 fluent API 进一步定制我们的响应:

```
@GetMapping(
  value = "/unauthorized"
)
public ResponseEntity<Mono<String>> unathorized() {
    return ResponseEntity
      .status(HttpStatus.UNAUTHORIZED)
      .header("X-Reason", "user-invalid")
      .body(Mono.just("unauthorized"));
}
```

### 2.6.具有泛函端点

有了 Spring 5，我们可以用函数的方式定义端点，所以我们也可以用编程的方式改变默认的 HTTP 状态:

```
@Bean
public RouterFunction<ServerResponse> notFound() {
    return RouterFunctions
      .route(GET("/statuses/not-found"),
         request -> ServerResponse.notFound().build());
}
```

## 3.结论

当实现一个 HTTP API 时，该框架提供了许多选项来智能地处理我们向客户端公开的状态代码。

这篇文章应该是一个很好的起点，可以探索这些问题，并理解如何使用干净、RESTful 的语义推出有表现力的、友好的 API。

当然，本教程中使用的完整代码示例可以从 Github 上的[处获得。](https://web.archive.org/web/20220626210451/https://github.com/eugenp/tutorials/tree/master/spring-5-webflux)