# Spring WebClient 过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webclient-filters>

## 1.概观

在本教程中，我们将探索 [`Spring WebFlux`](/web/20220628055059/https://www.baeldung.com/spring-5-functional-web) 中的`WebClient`过滤器，这是一个功能性的反应式 web 框架。

## 2.请求过滤器

过滤器可以拦截、检查和修改客户端请求(或响应)。过滤器非常适合为每个请求添加功能，因为逻辑停留在一个地方。用例包括监控、修改、记录和认证客户端请求，这里仅举几个例子。

一个请求有一个零个或多个过滤器的有序链。

在 Spring Reactive 中，过滤器是功能接口`ExchangeFilterFunction`的实例。过滤函数有两个参数:要修改的`ClientRequest`和下一个`ExchangeFilterFunction`。

通常，过滤器函数通过调用过滤器链中的下一个函数返回:

```java
ExchangeFilterFunction filterFunction = (clientRequest, nextFilter) -> {
    LOG.info("WebClient fitler executed");
    return nextFilter.exchange(clientRequest);
};
```

## 3.`WebClient`过滤

在实现请求过滤器之后，我们必须将它“附加”到`WebClient`实例。这只能在创建`WebClient`时完成。

那么，让我们看看如何创建一个`WebClient`。第一个选项是使用或不使用基本 URL 来调用`WebClient.create()`:

```java
WebClient webClient = WebClient.create();
```

不幸的是，这不允许添加过滤器。那么，第二种选择就是我们正在寻找的。

**通过使用`WebClient.builder()`，我们能够添加过滤器**:

```java
WebClient webClient = WebClient.builder()
  .filter(filterFunction)
  .build();
```

## 4.自定义过滤器

让我们从一个对客户机发送的 HTTP GET 请求进行计数的过滤器开始。

过滤器检查请求方法，并在 GET 请求的情况下增加“全局”计数器:

```java
ExchangeFilterFunction countingFunction = (clientRequest, nextFilter) -> {
    HttpMethod httpMethod = clientRequest.method();
    if (httpMethod == HttpMethod.GET) {
        getCounter.incrementAndGet();
    }
    return nextFilter.exchange(clientRequest);
};
```

我们将定义的第二个过滤器将一个版本号附加到请求 URL 路径上。我们利用`ClientRequest.from()`方法从当前对象创建一个新的请求对象，并设置修改后的 URL。

随后，我们继续使用新修改的请求对象执行过滤器链:

```java
ExchangeFilterFunction urlModifyingFilter = (clientRequest, nextFilter) -> {
    String oldUrl = clientRequest.url().toString();
    URI newUrl = URI.create(oldUrl + "/" + version);
    ClientRequest filteredRequest = ClientRequest.from(clientRequest)
      .url(newUrl)
      .build();
    return nextFilter.exchange(filteredRequest);
};
```

接下来，让我们定义一个过滤器来记录发送请求的方法及其 URL。这些细节在请求对象中可用。

然后我们要做的就是打印到一些输出流:

```java
ExchangeFilterFunction loggingFilter = (clientRequest, nextFilter) -> {
    printStream.print("Sending request " + clientRequest.method() + " " + clientRequest.url());
    return nextFilter.exchange(clientRequest);
};
```

## 5.标准过滤器

最后，**让我们来看看基本认证**——一个非常常见的请求过滤用例。

助手类`ExchangeFilterFunctions` 提供了`basicAuthentication()`过滤函数，负责将`authorization`头添加到请求中。

因此，我们不需要为它定义过滤器:

```java
WebClient webClient = WebClient.builder()
  .filter(ExchangeFilterFunctions.basicAuthentication(user, password))
  .build(); 
```

## 6.结论

在这篇短文中，我们探讨了在 Spring 中过滤 WebFlux 客户端。

与往常一样，代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20220628055059/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-reactive-client)