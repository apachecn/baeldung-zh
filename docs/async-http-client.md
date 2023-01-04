# 在 Java 中使用异步 HTTP-客户端的异步 HTTP

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/async-http-client>

## 1.概观

[AsyncHttpClient](https://web.archive.org/web/20220926195912/https://github.com/AsyncHttpClient/async-http-client) (AHC)是构建在 [Netty](/web/20220926195912/https://www.baeldung.com/netty) 之上的库，目的是方便地异步执行 HTTP 请求和处理响应。

在本文中，我们将介绍如何配置和使用 HTTP 客户端，如何使用 AHC 执行请求和处理响应。

## 2.设置

该库的最新版本可以在 [Maven 资源库](https://web.archive.org/web/20220926195912/https://mvnrepository.com/artifact/org.asynchttpclient/async-http-client)中找到。我们应该小心使用组 id 为`org.asynchttpclient`的依赖项，而不是组 id 为`com.ning:`的依赖项

```java
<dependency>
    <groupId>org.asynchttpclient</groupId>
    <artifactId>async-http-client</artifactId>
    <version>2.2.0</version>
</dependency>
```

## 3.HTTP 客户端配置

获取 HTTP 客户端最直接的方法是使用`Dsl`类。静态的`asyncHttpClient()`方法返回一个`AsyncHttpClient`对象:

```java
AsyncHttpClient client = Dsl.asyncHttpClient();
```

如果我们需要 HTTP 客户端的定制配置，我们可以使用构建器`DefaultAsyncHttpClientConfig.Builder`构建`AsyncHttpClient`对象:

```java
DefaultAsyncHttpClientConfig.Builder clientBuilder = Dsl.config()
```

这提供了配置超时、代理服务器、HTTP 证书等的可能性:

```java
DefaultAsyncHttpClientConfig.Builder clientBuilder = Dsl.config()
  .setConnectTimeout(500)
  .setProxyServer(new ProxyServer(...));
AsyncHttpClient client = Dsl.asyncHttpClient(clientBuilder);
```

一旦我们配置并获得了 HTTP 客户端的实例，我们就可以在整个应用程序中重用它。我们不需要为每个请求创建一个实例，因为它会在内部创建新的线程和连接池，这会导致性能问题。

此外，需要注意的是，一旦我们使用完客户端，我们应该调用`close()`方法来防止任何内存泄漏或挂起资源。

## 4.创建 HTTP 请求

有两种方法可以使用 AHC 定义 HTTP 请求:

*   束缚
*   解开

就性能而言，这两种请求类型没有太大区别。它们只代表我们可以用来定义请求的两个独立的 API。一个绑定的请求被绑定到创建它的 HTTP 客户端，并且如果没有另外指定，默认情况下将使用该特定客户端的配置。

例如，当创建绑定请求时，从 HTTP 客户端配置中读取`disableUrlEncoding`标志，而对于非绑定请求，默认情况下，该标志设置为 false。这很有用，因为通过使用作为 VM 参数传递的系统属性，无需重新编译整个应用程序就可以更改客户端配置:

```java
java -jar -Dorg.asynchttpclient.disableUrlEncodingForBoundRequests=true
```

完整的属性列表可以在`ahc-default.properties`文件中找到。

### 4.1.绑定请求

为了创建一个绑定请求，我们使用来自类`AsyncHttpClient`的以前缀`“prepare”`开始的帮助器方法。同样，我们可以使用`prepareRequest()`方法来接收一个已经创建的`Request`对象。

例如，`prepareGet()`方法将创建一个 HTTP GET 请求:

```java
BoundRequestBuilder getRequest = client.prepareGet("http://www.baeldung.com");
```

### 4.2.未绑定请求

可以使用`RequestBuilder`类创建一个未绑定的请求:

```java
Request getRequest = new RequestBuilder(HttpConstants.Methods.GET)
  .setUrl("http://www.baeldung.com")
  .build();
```

或者通过使用`Dsl`助手类，它实际上使用`RequestBuilder`来配置请求的 HTTP 方法和 URL:

```java
Request getRequest = Dsl.get("http://www.baeldung.com").build()
```

## 5.执行 HTTP 请求

库的名字给了我们一个关于如何执行请求的提示。AHC 支持同步和异步请求。

执行请求取决于其类型。当使用一个**绑定请求时，我们使用来自`BoundRequestBuilder`** 类的`execute()`方法，当我们有一个**未绑定请求时，我们将使用来自`AsyncHttpClient`接口**的 executeRequest()方法的实现之一来执行它。

### 5.1.同步地

该库被设计成异步的，但是当需要时，我们可以通过阻塞`Future`对象来模拟同步调用。`execute()`和`executeRequest()`方法都返回一个`ListenableFuture<Response>`对象。该类扩展了 Java `Future`接口，从而继承了`get()`方法，该方法可用于阻塞当前线程，直到 HTTP 请求完成并返回响应:

```java
Future<Response> responseFuture = boundGetRequest.execute();
responseFuture.get();
```

```java
Future<Response> responseFuture = client.executeRequest(unboundRequest);
responseFuture.get();
```

当试图调试我们的部分代码时，使用同步调用是有用的，但是不建议在生产环境中使用，在生产环境中异步执行会带来更好的性能和吞吐量。

### 5.2.异步

当我们谈论异步执行时，我们也谈论用于处理结果的侦听器。AHC 库提供了 3 种类型的侦听器，可用于异步 HTTP 调用:

*   `AsyncHandler`
*   `AsyncCompletionHandler`
*   `ListenableFuture`听众

`AsyncHandler`监听器提供了在 HTTP 调用完成之前控制和处理它的可能性。使用它可以处理一系列与 HTTP 调用相关的事件:

```java
request.execute(new AsyncHandler<Object>() {
    @Override
    public State onStatusReceived(HttpResponseStatus responseStatus)
      throws Exception {
        return null;
    }

    @Override
    public State onHeadersReceived(HttpHeaders headers)
      throws Exception {
        return null;
    }

    @Override
    public State onBodyPartReceived(HttpResponseBodyPart bodyPart)
      throws Exception {
        return null;
    }

    @Override
    public void onThrowable(Throwable t) {

    }

    @Override
    public Object onCompleted() throws Exception {
        return null;
    }
});
```

enum 让我们控制 HTTP 请求的处理。通过返回 **`State.ABORT`，我们可以在特定时刻停止处理**，通过使用`State.CONTINUE`，我们让处理结束。

值得一提的是 **`AsyncHandler`不是线程安全的，在执行并发请求时不应该被重用。**

`AsyncCompletionHandler`继承了`AsyncHandler`接口的所有方法，并添加了用于处理调用完成的`onCompleted(Response)` 帮助器方法。所有其他监听器方法都被覆盖以返回`State.` CONTINUE，从而使代码更具可读性:

```java
request.execute(new AsyncCompletionHandler<Object>() {
    @Override
    public Object onCompleted(Response response) throws Exception {
        return response;
    }
});
```

`ListenableFuture`接口让我们添加将在 HTTP 调用完成时运行的监听器。

此外，我们还可以通过使用另一个线程池来执行监听器中的代码:

```java
ListenableFuture<Response> listenableFuture = client
  .executeRequest(unboundRequest);
listenableFuture.addListener(() -> {
    Response response = listenableFuture.get();
    LOG.debug(response.getStatusCode());
}, Executors.newCachedThreadPool());
```

此外，添加监听器的选项，`ListenableFuture`接口让我们将`Future`响应转换为`CompletableFuture`。

## 7.结论

AHC 是一个非常强大的库，有很多有趣的特性。它提供了一种非常简单的方式来配置 HTTP 客户端，并提供了执行同步和异步请求的能力。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220926195912/https://github.com/eugenp/tutorials/tree/master/libraries-http)