# 在 WebFlux WebClient 中测试状态代码时如何获取响应体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webclient-get-response-body>

## 1.概观

使用 HTTP 响应中的状态代码来确定应用程序下一步应该如何处理给定的响应通常很有帮助。

在本教程中，我们将看看如何使用 WebFlux 的`WebClient.`来访问状态代码和从 REST 请求返回的响应体

[`WebClient`](/web/20220627165341/https://www.baeldung.com/spring-5-webclient) 是在 Spring 5 中引入的，可以在调用 RESTful 服务时用于异步 I/O。

## 2.用例

当对其他服务进行 RESTful 调用时，应用程序通常使用返回的[状态代码](https://web.archive.org/web/20220627165341/https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)来触发不同的功能。典型的用例包括**优雅的错误处理、触发请求重试以及确定用户错误。**

因此，在进行 REST 调用时，仅仅获得响应代码通常是不够的。有时我们也需要响应体。

在下面的例子中，让我们看看如何解析来自 REST 客户端`WebClient`的响应体。我们将把我们的行为与返回的状态代码联系起来，并将利用`WebClient`提供的两种状态代码提取方法: `onStatus` 和`ExchangeFilterFunction.`

## 3.使用`onStatus`

[`onStatus`](https://web.archive.org/web/20220627165341/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.ResponseSpec.html#onStatus-java.util.function.Predicate-java.util.function.Function-) 是一个内置的机制，可以用来处理`WebClient`响应。**这允许我们基于特定的响应**(比如 400、500、503 等)应用细粒度的功能。)或状态类别(如 4XX 和 5XX 等)。):

```java
WebClient
  .builder()
  .build()
  .post()
  .uri("/some-resource")
  .retrieve()
  .onStatus(
    HttpStatus.INTERNAL_SERVER_ERROR::equals,
    response -> response.bodyToMono(String.class).map(Exception::new))
```

`onStatus`方法需要两个参数。第一个是接受状态代码的谓词。第二个参数的执行基于第一个参数的输出。第二个是将响应映射到`Mono`或`Exception.`的函数

在这种情况下，如果我们看到一个`INTERNAL_SERVER_ERROR`(即 500)，我们将使用`bodyToMono,`获取主体，然后将其映射到一个新的`Exception`。

**我们可以链接`onStatus`调用**，以便能够为不同的状态条件提供功能:

```java
Mono<String> response = WebClient
  .builder()
  .build()
  .post()
  .uri("some-resource")
  .retrieve()
  .onStatus( 
    HttpStatus.INTERNAL_SERVER_ERROR::equals,
    response -> response.bodyToMono(String.class).map(CustomServerErrorException::new)) 
  .onStatus(
    HttpStatus.BAD_REQUEST::equals,
    response -> response.bodyToMono(String.class).map(CustomBadRequestException::new))
  ... 
  .bodyToMono(String.class);

// do something with response
```

现在`onStatus`调用映射到我们的自定义异常。我们为两种错误状态中的每一种定义了异常类型。`onStatus `方法允许我们使用我们选择的任何类型。

## 4.使用 ExchangeFilterFunction

一个`[ExchangeFilterFunction](https://web.archive.org/web/20220627165341/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/ExchangeFilterFunction.html)` 是另一种处理特定状态代码和获得响应体的方式`.` 与`onStatus`不同，交换过滤器是灵活的，并适用于基于任何布尔表达式的过滤器功能。

我们可以从一个`ExchangeFilterFunction` 到**的灵活性中获益，它覆盖了与`onStatus`函数**相同的类别。

首先，我们将定义一个方法来处理基于给定状态码的返回逻辑:

```java
private static Mono<ClientResponse> exchangeFilterResponseProcessor(ClientResponse response) {
    HttpStatus status = response.statusCode();
    if (HttpStatus.INTERNAL_SERVER_ERROR.equals(status)) {
        return response.bodyToMono(String.class)
          .flatMap(body -> Mono.error(new CustomServerErrorException(body)));
    }
    if (HttpStatus.BAD_REQUEST.equals(status)) {
        return response.bodyToMono(String.class)
          .flatMap(body -> Mono.error(new CustomBadRequestException(body)));
    }
    return Mono.just(response);
}
```

接下来，我们将定义过滤器，并使用对处理程序的方法引用:

```java
ExchangeFilterFunction errorResponseFilter = ExchangeFilterFunction
  .ofResponseProcessor(WebClientStatusCodeHandler::exchangeFilterResponseProcessor);
```

类似于`onStatus`调用，我们在出错时映射到我们的`Exception`类型。然而，使用`Mono.error `会将这个`Exception`包装在一个`ReactiveException.`中，当[处理错误](/web/20220627165341/https://www.baeldung.com/spring-webflux-errors)时，应该记住这个嵌套。

现在，让我们**将其应用到一个`WebClient`的实例中，以达到与`onStatus`** 链式调用相同的效果:

```java
Mono<String> response = WebClient
  .builder()
  .filter(errorResponseFilter)
  .build()
  .post()
  .uri("some-resource")
  .retrieve()
  .bodyToMono(String.class);

// do something with response
```

## 5.结论

在本文中，我们介绍了几种基于`HTTP`状态头获取响应体的方法。基于状态代码，`onStatus`方法允许我们插入特定的功能。此外，我们可以使用`filter`方法插入一个通用方法来对所有响应进行后处理。

一如既往，本教程中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220627165341/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-reactive-client)