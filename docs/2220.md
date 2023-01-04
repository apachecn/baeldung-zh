# Java HttpClient 超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-httpclient-timeout>

## 1.概观

在本教程中，我们将展示如何使用从 Java 11 开始的新 Java HTTP 客户端和 Java。

万一需要刷新一下知识，可以从 [Java HTTP 客户端](/web/20220706004543/https://www.baeldung.com/java-9-http-client)的教程开始。

另一方面，要了解如何使用旧库设置超时，请参见`[HttpUrlConnection.](/web/20220706004543/https://www.baeldung.com/java-http-request#configuring-timeouts)`

## 2.配置超时

首先，我们需要设置一个 HttpClient 来发出 HTTP 请求:

```
private static HttpClient getHttpClientWithTimeout(int seconds) {
    return HttpClient.newBuilder()
      .connectTimeout(Duration.ofSeconds(seconds))
      .build();
}
```

上面，我们创建了一个方法，该方法返回一个用参数定义的超时配置的`HttpClient`。简而言之，**我们使用[构建器设计模式](/web/20220706004543/https://www.baeldung.com/creational-design-patterns#builder)实例化一个`HttpClient`，并使用`connectTimeout`方法**配置超时。此外，使用静态方法`ofSeconds`，我们创建了一个`Duration`对象的实例，它以秒为单位定义了我们的超时。

之后，我们检查`HttpClient`超时是否配置正确:

```
httpClient.connectTimeout().map(Duration::toSeconds)
  .ifPresent(sec -> System.out.println("Timeout in seconds: " + sec));
```

因此，我们使用`connectTimeout`方法来获得超时。结果，它返回一个我们映射到秒的`Duration,`的`Optional`。

## 3.处理超时

此外，我们需要创建一个`HttpRequest`对象，我们的客户机将使用它来发出 HTTP 请求:

```
HttpRequest httpRequest = HttpRequest.newBuilder()
  .uri(URI.create("http://10.255.255.1")).GET().build();
```

为了模拟超时，我们呼叫一个不可路由的 IP 地址。换句话说，所有的 TCP 数据包都会丢弃，并在前面配置的预定义持续时间后强制超时。

现在，让我们更深入地了解一下如何处理超时。

### 3.1.处理同步调用超时

例如，要进行同步调用，请使用`send`方法:

```
HttpConnectTimeoutException thrown = assertThrows(
  HttpConnectTimeoutException.class,
  () -> httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString()),
  "Expected send() to throw HttpConnectTimeoutException, but it didn't");
assertTrue(thrown.getMessage().contains("timed out"));
```

**同步调用强制捕捉`HttpConnectTimeoutException` 延伸**的`IOException`。因此，在上面的测试中，我们预计`HttpConnectTimeoutException`会显示一条错误消息。

### 3.2.处理异步调用超时

类似地，使用`sendAsync`方法进行异步调用:

```
CompletableFuture<String> completableFuture = httpClient.sendAsync(httpRequest, HttpResponse.BodyHandlers.ofString())
  .thenApply(HttpResponse::body)
  .exceptionally(Throwable::getMessage);
String response = completableFuture.get(5, TimeUnit.SECONDS);
assertTrue(response.contains("timed out"));
```

上面对`sendAsync`的调用返回一个`CompletableFuture<HttpResponse>`。因此，我们需要定义如何在功能上处理响应。具体来说，当没有错误发生时，我们从响应中获得主体。否则，我们会从 throwable 中得到错误消息。最后，我们通过等待最多 5 秒钟从`CompletableFuture`中得到结果。同样，这个请求在 3 秒钟后抛出了一个`HttpConnectTimeoutException`,正如我们所预料的那样。

## 4.在请求级别配置超时

上面，我们为`sync`和`async`调用重用了同一个客户端实例。但是，我们可能希望对每个请求的超时进行不同的处理。同样，我们可以为单个请求设置超时:

```
HttpRequest httpRequest = HttpRequest.newBuilder()
  .uri(URI.create("http://10.255.255.1"))
  .timeout(Duration.ofSeconds(1))
  .GET()
  .build();
```

类似地，我们使用`timeout`方法为这个请求设置超时。这里，我们为这个请求配置了 1 秒的超时。

**客户端和请求之间的最短持续时间设置了请求的超时。**

## 5.结论

在本文中，我们使用新的 Java HTTP 客户机成功地配置了超时，并在超时溢出时优雅地处理了请求。

和往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220706004543/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)