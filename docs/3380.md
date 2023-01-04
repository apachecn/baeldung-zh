# 在 Spring 5 Webflux WebClient 中设置超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-timeout>

## 1.概观

Spring 5 增加了一个全新的框架——[Spring web flux](/web/20221208143917/https://www.baeldung.com/spring-webflux)，它支持我们的 web 应用中的反应式编程。为了执行 HTTP 请求，我们可以使用`WebClient` 接口，它提供了一个基于[反应器项目](/web/20221208143917/https://www.baeldung.com/reactor-core)的功能 API。

在本教程中，我们将关注`WebClient` 的**超时设置。我们将讨论不同的方法，如何正确地设置不同的超时，既包括整个应用程序中的全局超时，也包括特定于请求的超时。**

## 2.`WebClient` 和 HTTP 客户端

在我们继续之前，让我们快速回顾一下。Spring WebFlux 包含了自己的客户端 [`WebClient`类](/web/20221208143917/https://www.baeldung.com/spring-5-webclient)，以一种被动的方式执行 HTTP 请求。`WebClient` 还需要一个 HTTP 客户端库才能正常工作。Spring [为其中一些提供了内置支持](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client)，但是默认使用[反应器 Netty](https://web.archive.org/web/20221208143917/https://github.com/reactor/reactor-netty) 。

大多数配置，包括超时，都可以使用这些客户端来完成。

## 3.通过 HTTP 客户端配置超时

正如我们之前提到的，在我们的应用程序中设置不同的`WebClient`超时的最简单的方法是使用底层 HTTP 客户端对**进行全局设置。**这也是最有效的方法。

由于 Netty 是 Spring WebFlux 的默认客户端库，我们将使用[Reactor Netty`HttpClient`类](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/reference/index.html#http-client)来涵盖我们的示例。

### 3.1.响应超时

响应超时是我们在发送请求后等待**接收响应的时间。**我们可以用 [`responseTimeout()`](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/http/client/HttpClient.html#responseTimeout-java.time.Duration-) 的方法为客户端进行配置:

```
HttpClient client = HttpClient.create()
  .responseTimeout(Duration.ofSeconds(1)); 
```

在本例中，我们将超时时间配置为 1 秒。默认情况下，Netty 不设置响应超时。

之后，我们可以将`HttpClient`提供给弹簧`WebClient`:

```
WebClient webClient = WebClient.builder()
  .clientConnector(new ReactorClientHttpConnector(httpClient))
  .build();
```

在此之后， `WebClient` **继承底层`HttpClient `为所有发送的请求提供的所有配置**。

### 3.2.连接超时

连接超时是**客户端和服务器之间必须建立连接的时间段`. `** 我们可以使用不同的通道选项键和 [option()](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/transport/Transport.html#option-io.netty.channel.ChannelOption-O-) 方法来进行配置:

```
HttpClient client = HttpClient.create()
  .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);

// create WebClient...
```

提供的值以毫秒为单位，因此我们将超时时间配置为 10 秒。默认情况下，Netty 将该值设置为 30 秒。

此外，我们可以配置 keep-alive 选项，它将在连接空闲时发送 TCP 检查探测:

```
HttpClient client = HttpClient.create()
  .option(ChannelOption.SO_KEEPALIVE, true)
  .option(EpollChannelOption.TCP_KEEPIDLE, 300)
  .option(EpollChannelOption.TCP_KEEPINTVL, 60)
  .option(EpollChannelOption.TCP_KEEPCNT, 8);

// create WebClient...
```

因此，我们启用了保持活动检查，在空闲 5 分钟后以 60 秒的间隔进行探测。我们还将连接中断前的最大探测数量设置为 8。

**当连接在给定时间内没有建立或断开时，抛出 [`ConnectTimeoutException`](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/channel/ConnectTimeoutException.html) 。**

### 3.3.读写超时

当**在特定时间段内没有读取数据时，发生读取超时，**，而当**写操作不能在特定时间完成时，发生写超时。**`HttpClient`允许配置额外的处理程序来配置那些超时:

```
HttpClient client = HttpClient.create()
  .doOnConnected(conn -> conn
    .addHandler(new ReadTimeoutHandler(10, TimeUnit.SECONDS))
    .addHandler(new WriteTimeoutHandler(10)));

// create WebClient...
```

在这种情况下，我们通过 [`doOnConnected()`](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/transport/ClientTransport.html#doOnConnect-java.util.function.Consumer-) 方法配置了一个连接的回调，其中我们创建了额外的处理程序。为了配置超时，我们添加了`[ReadTimeOutHandler](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/handler/timeout/ReadTimeoutHandler.html) `和 [`WriteTimeOutHandle` r](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/handler/timeout/WriteTimeoutHandler.html) 实例。我们把它们都设置为 10 秒。

这些处理程序的构造函数接受两种不同的参数。对于第一个，我们提供了一个带有`TimeUnit` 规范的数字，而第二个将给定的数字转换成秒。

底层 Netty 库相应地提供 **[`ReadTimeoutException`](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/handler/timeout/ReadTimeoutException.html) 和 [`WriteTimeoutException`](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/handler/timeout/WriteTimeoutException.html) 类来处理错误**。

### 3.4.SSL/TLS 超时

握手超时是**暂停操作**之前系统尝试建立 SSL 连接的持续时间。我们可以通过`[secure()](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/http/client/HttpClient.html#secure--)`方法设置 SSL 配置:

```
HttpClient.create()
  .secure(spec -> spec.sslContext(SslContextBuilder.forClient())
    .defaultConfiguration(SslProvider.DefaultConfigurationType.TCP)
    .handshakeTimeout(Duration.ofSeconds(30))
    .closeNotifyFlushTimeout(Duration.ofSeconds(10))
    .closeNotifyReadTimeout(Duration.ofSeconds(10)));

// create WebClient...
```

如上所述，我们将握手超时设置为 30 秒(默认:10 秒)，而`close_notify`刷新(默认:3 秒)和读取(默认:0 秒)超时设置为 10 秒。所有方法都由 [`SslProvider.Builder`](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/tcp/SslProvider.Builder.html) 接口交付。

**[`SslHandshakeTimeoutException`](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/handler/ssl/SslHandshakeTimeoutException.html "class in io.netty.handler.ssl") 用于因配置超时而导致**握手失败的情况。

### 3.5.代理超时

`HttpClient`还支持代理功能。如果到对等体的**连接建立尝试没有在代理超时内完成，则连接尝试失败**。我们在 [`proxy()`](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/transport/ClientTransport.html#proxy-java.util.function.Consumer-) 配置期间设置此超时:

```
HttpClient.create()
  .proxy(spec -> spec.type(ProxyProvider.Proxy.HTTP)
    .host("proxy")
    .port(8080)
    .connectTimeoutMillis(30000));

// create WebClient...
```

当默认值为 10 时，我们使用 [`connectTimeoutMillis()`](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/transport/ProxyProvider.Builder.html#connectTimeoutMillis-long-) 将超时设置为 30 秒。

Netty 库还实现了自己的**[`ProxyConnectException`](https://web.archive.org/web/20221208143917/https://netty.io/4.1/api/io/netty/handler/proxy/ProxyConnectException.html)，以防**失败。

## 4.请求级超时

在上一节中，我们使用`HttpClient`全局配置了不同的超时。然而，我们也可以**独立于全局设置**来设置特定于响应请求的超时。

### 4.1.响应超时–使用`HttpClientRequest`

如前所述，我们可以在请求级别配置**响应超时:**

```
webClient.get()
  .uri("https://baeldung.com/path")
  .httpRequest(httpRequest -> {
    HttpClientRequest reactorRequest = httpRequest.getNativeRequest();
    reactorRequest.responseTimeout(Duration.ofSeconds(2));
  });
```

在上面的案例中，我们使用了`WebClient's` [`httpRequest()`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.RequestHeadersSpec.html#httpRequest-java.util.function.Consumer-) 方法从底层的 Netty 库中访问本地的`[HttpClientReques](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/netty/release/api/reactor/netty/http/client/HttpClientRequest.html)t` 。接下来，我们用它将超时值设置为 2 秒。

这种响应超时设置**会覆盖`HttpClient`级别**的任何响应超时。我们还可以将这个值设置为`null`来删除任何之前配置的值。

### 4.2.反应超时–使用反应堆堆芯

Reactor Netty 使用 Reactor Core 作为其反应流实现。要配置另一个超时，我们可以使用由`Mono`和 `**Flux**` 出版商提供的 **`[timeout()](https://web.archive.org/web/20221208143917/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#timeout-java.time.Duration-)` 操作符:**

```
webClient.get()
  .uri("https://baeldung.com/path")
  .retrieve()
  .bodyToFlux(JsonNode.class)
  .timeout(Duration.ofSeconds(5));
```

在这种情况下，如果在给定的 5 秒钟内没有物品到达，则会出现 **`TimeoutException` 。**

请记住，最好使用 Reactor Netty 中提供的更具体的超时配置选项，因为它们为特定的目的和用例提供了更多的控制。

`timeout()`方法适用于整个操作，**从建立到远程对等点的连接到接收**响应。它**不会覆盖**任何与`HttpClient`相关的设置。

## 5.异常处理

我们刚刚了解了不同的超时配置。现在是时候快速讨论一下异常处理了。每种类型的超时都提供了一个专用的异常，所以我们可以很容易地**使用活动流和`onError`块** `:`处理它们

```
webClient.get()
  .uri("https://baeldung.com/path")
  .retrieve()
  .bodyToFlux(JsonNode.class)
  .timeout(Duration.ofSeconds(5))
  .onErrorMap(ReadTimeoutException.class, ex -> new HttpTimeoutException("ReadTimeout"))
  .onErrorReturn(SslHandshakeTimeoutException.class, new TextNode("SslHandshakeTimeout"))
  .doOnError(WriteTimeoutException.class, ex -> log.error("WriteTimeout"))
  ...
```

我们可以重用之前描述的任何异常，并使用 Reactor 编写我们自己的处理方法。

而且，我们还可以**根据 HTTP 状态**添加一些逻辑:

```
webClient.get()
  .uri("https://baeldung.com/path")
  .onStatus(HttpStatus::is4xxClientError, resp -> {
    log.error("ClientError {}", resp.statusCode());
    return Mono.error(new RuntimeException("ClientError"));
  })
  .retrieve()
  .bodyToFlux(JsonNode.class)
  ...
```

## 6.结论

在本教程中，我们使用 Netty 示例在我们的`WebClient`上学习了**如何在 Spring WebFlux** 中配置超时。

我们快速讨论了不同的超时和在`HttpClient`级别正确设置它们的方法，以及如何将它们应用到我们的全局设置中。然后，我们处理单个请求，在特定于请求的级别上配置响应超时。最后，我们展示了处理发生的异常的不同方法。

文章中提到的所有代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-5-webflux)