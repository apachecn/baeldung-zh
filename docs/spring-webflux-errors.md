# 处理 Spring WebFlux 中的错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-errors>

## 1。概述

在本教程中，**我们将通过一个实际的例子来看看在 Spring WebFlux 项目中处理错误的各种策略。**

我们还将指出使用一种策略优于另一种策略的地方，并在最后提供完整源代码的链接。

## 2。设置示例

Maven 的设置与我们的[上一篇文章](/web/20220628153743/https://www.baeldung.com/spring-webflux)相同，它提供了 Spring WebFlux 的介绍。

对于我们的示例，**我们将使用** **一个 RESTful 端点，它将用户名作为查询参数，并返回“Hello username”**作为结果。

首先，让我们创建一个路由器函数，它将`/hello`请求路由到传入处理程序中名为`handleRequest`的方法:

```java
@Bean
public RouterFunction<ServerResponse> routeRequest(Handler handler) {
    return RouterFunctions.route(RequestPredicates.GET("/hello")
      .and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), 
        handler::handleRequest);
    } 
```

接下来，我们将定义调用`sayHello()`方法的`handleRequest()`方法，并找到在`ServerResponse`主体中包含/返回其结果的方法:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return 
      //...
        sayHello(request)
      //...
} 
```

最后，`sayHello()`方法是一个简单的实用方法，它连接了“Hello”`String`和用户名:

```java
private Mono<String> sayHello(ServerRequest request) {
    //...
    return Mono.just("Hello, " + request.queryParam("name").get());
    //...
} 
```

只要用户名作为请求的一部分出现，例如，如果端点被称为“`/hello?username=Tonni`”，该端点将总是正确运行。

但是，**如果我们在没有指定用户名的情况下调用同一个端点，比如“`/hello`，就会抛出异常。**

下面，我们将看看在哪里以及如何重新组织我们的代码来处理 WebFlux 中的这个异常。

## 3。在功能级别处理错误

`Mono`和`Flux`API 内置了两个关键操作符，用于在功能级别处理错误。

让我们简单地探讨一下它们和它们的用法。

### 3.1.用`onErrorReturn`处理错误

**每当出现错误时，我们可以使用`onErrorReturn()`返回静态默认值**:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return sayHello(request)
      .onErrorReturn("Hello Stranger")
      .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s));
} 
```

这里，每当有问题的串联函数`sayHello()`抛出异常时，我们就返回一个静态的“Hello Stranger”。

### 3.2.用`onErrorResume`处理错误

我们有三种方法可以使用`onErrorResume`来处理错误:

*   计算动态回退值
*   使用回退方法执行替代路径
*   捕捉、包装并重新抛出错误，例如，作为自定义的业务异常

让我们看看如何计算一个值:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return sayHello(request)
      .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s))
      .onErrorResume(e -> Mono.just("Error " + e.getMessage())
        .flatMap(s -> ServerResponse.ok()
          .contentType(MediaType.TEXT_PLAIN)
          .bodyValue(s)));
} 
```

这里我们返回一个由动态获得的错误信息组成的字符串，每当`sayHello()`抛出一个异常时，这个错误信息就附加到字符串“error”上。

接下来，让我们**在错误发生时调用一个回退方法**:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return sayHello(request)
      .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s))
      .onErrorResume(e -> sayHelloFallback()
        .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s)));
} 
```

这里，每当`sayHello()`抛出异常时，我们就调用替代方法`sayHelloFallback()`。

使用`onErrorResume()`的最后一个选项是**捕捉、包装并重新抛出一个错误**，例如，作为一个`NameRequiredException`:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return ServerResponse.ok()
      .body(sayHello(request)
      .onErrorResume(e -> Mono.error(new NameRequiredException(
        HttpStatus.BAD_REQUEST, 
        "username is required", e))), String.class);
} 
```

在这里，每当`sayHello()`抛出异常时，我们都会抛出一个定制的异常，并显示消息“需要用户名”。

## 4。在全局级别处理错误

到目前为止，我们给出的所有例子都是在函数级处理错误。

然而，我们可以选择在全球范围内处理我们的 WebFlux 错误。为此，我们只需采取两个步骤:

*   自定义全局错误响应属性
*   实现全局错误处理程序

我们的处理程序抛出的异常将被自动转换成 HTTP 状态和 JSON 错误体。

为了定制这些，我们可以简单地通过**扩展`DefaultErrorAttributes`类**并覆盖它的`getErrorAttributes()`方法:

```java
public class GlobalErrorAttributes extends DefaultErrorAttributes{

    @Override
    public Map<String, Object> getErrorAttributes(ServerRequest request, 
      ErrorAttributeOptions options) {
        Map<String, Object> map = super.getErrorAttributes(
          request, options);
        map.put("status", HttpStatus.BAD_REQUEST);
        map.put("message", "username is required");
        return map;
    }

} 
```

这里我们希望状态:`BAD_REQUEST`和消息“需要用户名”在异常发生时作为错误属性的一部分返回。

接下来，让我们**实现全局错误处理程序。**

为此，Spring 提供了一个方便的`AbstractErrorWebExceptionHandler`类，让我们在处理全局错误时进行扩展和实现:

```java
@Component
@Order(-2)
public class GlobalErrorWebExceptionHandler extends 
    AbstractErrorWebExceptionHandler {

    // constructors

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(
      ErrorAttributes errorAttributes) {

        return RouterFunctions.route(
          RequestPredicates.all(), this::renderErrorResponse);
    }

    private Mono<ServerResponse> renderErrorResponse(
       ServerRequest request) {

       Map<String, Object> errorPropertiesMap = getErrorAttributes(request, 
         ErrorAttributeOptions.defaults());

       return ServerResponse.status(HttpStatus.BAD_REQUEST)
         .contentType(MediaType.APPLICATION_JSON)
         .body(BodyInserters.fromValue(errorPropertiesMap));
    }
} 
```

在这个例子中，我们将全局错误处理程序的顺序设置为-2。这是为了使**比在`@Order(-1)`注册的`DefaultErrorWebExceptionHandler`** 具有更高的优先级。

`errorAttributes`对象将是我们在 Web 异常处理程序的构造函数中传递的对象的精确副本。理想情况下，这应该是我们定制的错误属性类。

然后，我们清楚地声明，我们希望将所有错误处理请求路由到`renderErrorResponse()`方法。

最后，我们获取错误属性，并将它们插入到服务器响应体中。

然后，这会生成一个 JSON 响应，其中包含错误的详细信息、HTTP 状态和机器客户端的异常消息。对于浏览器客户端，它有一个“白标”错误处理程序，以 HTML 格式呈现相同的数据。当然，这是可以定制的。

## 5。结论

在本文中，我们研究了 Spring WebFlux 项目中处理错误的各种策略，并指出了使用一种策略优于另一种策略的地方。

正如承诺的那样，本文附带的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628153743/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-reactive)