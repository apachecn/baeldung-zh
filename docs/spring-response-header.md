# 如何用 Spring 5 在响应上设置标题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-response-header>

## 1.概观

在这个快速教程中，**我们将通过不同的方式在服务响应**上设置一个头，要么是为非反应端点，要么是为使用 [Spring 的 5 WebFlux 框架](https://web.archive.org/web/20220602092821/https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html)的 API。

我们可以在[以前的帖子](/web/20220602092821/https://www.baeldung.com/spring-5)中找到关于这个框架的更多信息。

## 2.非反应性部件的接头

如果我们想在单个响应上设置标题，我们可以使用`HttpServletResponse`或`ResponseEntity`对象。

另一方面，如果我们的目标是为所有或多个响应添加一个过滤器，我们将需要配置一个`Filter`。

### 2.1.使用`HttpServletResponse`

我们只需将`HttpServletResponse`对象作为参数添加到我们的 REST 端点，然后使用`addHeader()`方法:

```java
@GetMapping("/http-servlet-response")
public String usingHttpServletResponse(HttpServletResponse response) {
    response.addHeader("Baeldung-Example-Header", "Value-HttpServletResponse");
    return "Response with header using HttpServletResponse";
}
```

如示例所示，我们不必返回响应对象。

### 2.2.使用`ResponseEntity`

在这种情况下，让我们使用由`ResponseEntity`类提供的`BodyBuilder`:

```java
@GetMapping("/response-entity-builder-with-http-headers")
public ResponseEntity<String> usingResponseEntityBuilderAndHttpHeaders() {
    HttpHeaders responseHeaders = new HttpHeaders();
    responseHeaders.set("Baeldung-Example-Header", 
      "Value-ResponseEntityBuilderWithHttpHeaders");

    return ResponseEntity.ok()
      .headers(responseHeaders)
      .body("Response with header using ResponseEntity");
}
```

**`HttpHeaders`类提供了许多方便的方法来设置最常见的头。**

我们可以在 Github repo 中看到更多说明这些观点的例子。

### 2.3.为所有回复添加标题

现在让我们假设我们想要为我们的许多端点设置一个特定的头。

当然，如果我们不得不在每个映射方法上复制以前的代码，这将是令人沮丧的。

更好的方法是通过在我们的服务中配置一个`Filter`:

```java
@WebFilter("/filter-response-header/*")
public class AddResponseHeaderFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
      FilterChain chain) throws IOException, ServletException {
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        httpServletResponse.setHeader(
          "Baeldung-Example-Filter-Header", "Value-Filter");
        chain.doFilter(request, response);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // ...
    }

    @Override
    public void destroy() {
        // ...
    }
}
```

**`@WebFilter`注释允许我们指出该`Filter `生效的`urlPatterns `。**

正如我们在本文中指出的[，为了让我们的`Filter`被 Spring 发现，我们需要向我们的 Spring 应用程序类添加`@ServletComponentScan `注释:](/web/20220602092821/https://www.baeldung.com/spring-servletcomponentscan)

```java
@ServletComponentScan
@SpringBootApplication
public class ResponseHeadersApplication {

    public static void main(String[] args) {
        SpringApplication.run(ResponseHeadersApplication.class, args);
    }
}
```

如果我们不需要`@WebFilter`提供的任何功能，我们可以通过在我们的`Filter`类中使用`@Component`注释来避免这最后一步。

## 3.反应终点的标题

同样，我们将看到如何使用`ServerHttpResponse`、`ResponseEntity `或`ServerResponse`(用于功能端点)类和接口来设置单个端点响应的头。

我们还将学习如何实现一个 Spring 5 `WebFilter`来为我们所有的响应添加一个标题。

### 3.1.使用`ServerHttpResponse`

这种方法与`HttpServletResponse`的对应方法非常相似:

```java
@GetMapping("/server-http-response")
public Mono<String> usingServerHttpResponse(ServerHttpResponse response) {
    response.getHeaders().add("Baeldung-Example-Header", "Value-ServerHttpResponse");
    return Mono.just("Response with header using ServerHttpResponse");
}
```

### 3.2.使用`ResponseEntity`

我们可以像对待非反应性端点一样使用`ResponseEntity`类:

```java
@GetMapping("/response-entity")
public Mono<ResponseEntity<String>> usingResponseEntityBuilder() {
    String responseHeaderKey = "Baeldung-Example-Header";
    String responseHeaderValue = "Value-ResponseEntityBuilder";
    String responseBody = "Response with header using ResponseEntity (builder)";

    return Mono.just(ResponseEntity.ok()
      .header(responseHeaderKey, responseHeaderValue)
      .body(responseBody));
}
```

### 3.3.使用`ServerResponse`

上两节中介绍的类和接口可以用在`@Controller `带注释的类中，但是不适合新的[Spring 5 Functional Web Framework](/web/20220602092821/https://www.baeldung.com/spring-5-functional-web)。

如果我们想让**在`HandlerFunction`上设置一个标题，那么我们需要得到`ServerResponse`** 接口:

```java
public Mono<ServerResponse> useHandler(final ServerRequest request) {
     return ServerResponse.ok()
        .header("Baeldung-Example-Header", "Value-Handler")
        .body(Mono.just("Response with header using Handler"),String.class);
}
```

### 3.4.为所有回复添加标题

最后， **Spring 5 提供了一个`WebFilter`接口来为服务获取的所有响应**设置一个头:

```java
@Component
public class AddResponseHeaderWebFilter implements WebFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        exchange.getResponse()
          .getHeaders()
          .add("Baeldung-Example-Filter-Header", "Value-Filter");
        return chain.filter(exchange);
    }
}
```

## 4.结论

总之，我们已经学习了许多在响应上设置头的不同方法，无论是我们想要在单个端点上设置它，还是我们想要配置我们所有的 rest APIs，即使我们正在迁移到反应堆栈，现在我们已经有了做所有这些事情的知识。

像往常一样，所有的例子都可以在我们的 Github 库中访问，包括[非反应式的](https://web.archive.org/web/20220602092821/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)和使用 [Spring 5 特定功能的](https://web.archive.org/web/20220602092821/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive)。