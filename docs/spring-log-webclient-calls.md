# 记录 Spring WebClient 调用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-log-webclient-calls>

## 1.概观

在本教程中，我们将展示如何定制 [Spring 的`WebClient`](/web/20221128043825/https://www.baeldung.com/spring-5-webclient)——一个反应式 HTTP 客户端——来记录请求和响应。

## 2.`WebClient`

`WebClient`是一个用于 HTTP 请求的反应式非阻塞接口，基于 [Spring WebFlux](/web/20221128043825/https://www.baeldung.com/spring-webflux) 。它有一个功能性的、流畅的 API，带有用于声明性组合的反应类型。

在后台，`WebClient`调用一个 HTTP 客户端。电抗器 Netty 是默认的，也支持 Jetty 的电抗`HttpClient`。此外，可以通过为`WebClient`设置一个`ClientConnector`来插入 HTTP 客户端的其他实现。

## 3.记录请求和响应

`WebClient`使用的默认`HttpClient` 是 Netty 实现，因此**在我们将`reactor.netty.http.client` 日志记录级别更改为 `DEBUG,`** 后，我们可以看到一些请求日志记录，但是如果我们需要定制的日志，我们可以通过 [`WebClient#filters`](/web/20221128043825/https://www.baeldung.com/spring-webclient-filters) 配置`e`我们的日志记录器:

```java
WebClient
  .builder()
  .filters(exchangeFilterFunctions -> {
      exchangeFilterFunctions.add(logRequest());
      exchangeFilterFunctions.add(logResponse());
  })
  .build()
```

在这个代码片段中，我们添加了两个单独的过滤器来记录请求和响应。

让我们通过使用`ExchangeFilterFunction#ofRequestProcessor`来实现`logRequest` :

```java
ExchangeFilterFunction logRequest() {
    return ExchangeFilterFunction.ofRequestProcessor(clientRequest -> {
        if (log.isDebugEnabled()) {
            StringBuilder sb = new StringBuilder("Request: \n");
            //append clientRequest method and url
            clientRequest
              .headers()
              .forEach((name, values) -> values.forEach(value -> /* append header key/value */));
            log.debug(sb.toString());
        }
        return Mono.just(clientRequest);
    });
}
```

`logResponse`和**是一样的，但是我们必须用`ExchangeFilterFunction#ofResponseProcessor`来代替。** 

现在，我们可以将`reactor.netty.http.client`日志级别更改为`INFO`或`ERROR`以获得更清晰的输出。

## 4.用正文记录请求和响应

HTTP 客户端具有记录请求和响应主体的特性。因此，**为了实现这个目标，我们将使用一个支持日志的 HTTP 客户端和我们的`WebClient.`**

我们可以通过手动设置 `WebClient.Builder#` `clientConnector – `来做到这一点，让我们看看 Jetty 和 Netty HTTP 客户端。

### 4.1.码头测井`HttpClient`

首先，让我们将 [`jetty-reactive-httpclient`](https://web.archive.org/web/20221128043825/https://search.maven.org/search?q=a:jetty-reactive-httpclient) 的 Maven 依赖项添加到 pom 中:

```java
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-reactive-httpclient</artifactId>
    <version>1.1.6</version>
</dependency>
```

然后我们将创建一个定制的 Jetty `HttpClient`:

```java
SslContextFactory.Client sslContextFactory = new SslContextFactory.Client();
HttpClient httpClient = new HttpClient(sslContextFactory) {
    @Override
    public Request newRequest(URI uri) {
        Request request = super.newRequest(uri);
        return enhance(request);
    }
};
```

这里，我们已经覆盖了`HttpClient#newRequest`，然后将`Request`包装在一个日志增强器中。

接下来，我们需要用请求注册事件，以便我们可以在请求的每个部分可用时进行记录:

```java
Request enhance(Request request) {
    StringBuilder group = new StringBuilder();
    request.onRequestBegin(theRequest -> {
        // append request url and method to group
    });
    request.onRequestHeaders(theRequest -> {
        for (HttpField header : theRequest.getHeaders()) {
            // append request headers to group
        }
    });
    request.onRequestContent((theRequest, content) -> {
        // append content to group
    });
    request.onRequestSuccess(theRequest -> {
        log.debug(group.toString());
        group.delete(0, group.length());
    });
    group.append("\n");
    request.onResponseBegin(theResponse -> {
        // append response status to group
    });
    request.onResponseHeaders(theResponse -> {
        for (HttpField header : theResponse.getHeaders()) {
            // append response headers to group
        }
    });
    request.onResponseContent((theResponse, content) -> {
        // append content to group
    });
    request.onResponseSuccess(theResponse -> {
        log.debug(group.toString());
    });
    return request;
}
```

最后，我们必须构建`WebClient`实例:

```java
WebClient
  .builder()
  .clientConnector(new JettyClientHttpConnector(httpClient))
  .build()
```

当然，正如我们之前所做的，我们需要将`RequestLogEnhancer` 的日志级别设置为`DEBUG`。

### 4.2.用 Netty `HttpClient`测井

首先，让我们创建一个 Netty `HttpClient`:

```java
HttpClient httpClient = HttpClient
  .create()
  .wiretap(true)
```

启用窃听后，每个请求和响应都将被详细记录。

接下来，我们必须将 Netty 的客户端包的日志级别`reactor.netty.http.client`设置为`DEBUG`:

```java
logging.level.reactor.netty.http.client=DEBUG
```

现在，让我们来构建`WebClient`:

```java
WebClient
  .builder()
  .clientConnector(new ReactorClientHttpConnector(httpClient))
  .build()
```

我们的`WebClient`将详细记录每个请求和响应，**但是 Netty 内置日志记录器的默认格式包含十六进制和文本表示的主体**，以及大量关于请求和响应事件的数据。

因此，如果我们只需要 Netty 的文本记录器，我们可以配置`HttpClient`:

```java
HttpClient httpClient = HttpClient
  .create()
  .wiretap("reactor.netty.http.client.HttpClient", 
    LogLevel.DEBUG, AdvancedByteBufFormat.TEXTUAL);
```

## 5.结论

在本教程中，我们在使用 Spring `WebClient`时使用了几种技术来记录请求和响应数据。

与往常一样，GitHub 上的[代码是可用的。](https://web.archive.org/web/20221128043825/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-client)