# 如何解决 Spring web flux DataBufferLimitException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-databufferlimitexception>

## 1.介绍

在本教程中，我们将探究为什么我们会在一个 [Spring Webflux](/web/20221225091828/https://www.baeldung.com/spring-webflux) 应用程序中看到`DataBufferLimitException` 。然后，我们将看一看解决这一问题的各种方法。

## 2.理解问题

让我们先了解问题，然后再寻求解决方案。

### 2.1.什么是`DataBufferLimitException?`

Spring WebFlux [限制编解码器中内存中数据的](https://web.archive.org/web/20221225091828/https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/web-reactive.html#webflux-codecs-limits)缓冲，以避免应用程序内存问题。**默认配置为 262，144 字节**。当这对我们的用例来说还不够时，我们将以`DataBufferLimitException`结束。

### 2.2.什么是`Codec`？

`spring-web`和`spring-core` 模块通过具有反应式流背压的非阻塞 I/O，为高级对象之间的字节内容序列化和反序列化提供支持。`[Codecs](https://web.archive.org/web/20221225091828/https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-codecs)`提供 Java 序列化的替代方案。一个[优势](https://web.archive.org/web/20221225091828/https://docs.spring.io/spring-integration/reference/html/codec.html#codec)是，通常，对象不需要实现`Serializable.  `

## 3.服务器端

让我们首先从服务器的角度来看一下`DataBufferLimitException`是如何运作的。

### 3.1.重现问题

让我们尝试向 Spring Webflux 服务器应用程序发送一个大小为 390 KB 的 JSON 有效负载来创建异常。我们将使用`curl `命令向我们的服务器发送一个`POST `请求:

```java
curl --location --request POST 'http://localhost:8080/1.0/process' \
  --header 'Content-Type: application/json' \
  --data-binary '@/tmp/390KB.json'
```

正如我们所见，`DataBufferLimitException`被抛出:

```java
org.springframework.core.io.buffer.DataBufferLimitException: Exceeded limit on max bytes to buffer : 262144
  at org.springframework.core.io.buffer.LimitedDataBufferList.raiseLimitException(LimitedDataBufferList.java:99) ~[spring-core-5.3.23.jar:5.3.23]
  Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
  *__checkpoint ⇢ HTTP POST "/1.0/process" [ExceptionHandlingWebHandler]
```

### 3.2.通过属性求解

最简单的解决方案是用**配置应用程序属性`spring.codec.max-in-memory-size`** 。让我们将以下内容添加到我们的`application.yaml `文件中:

```java
spring:
    codec:
        max-in-memory-size: 500KB
```

这样，我们现在应该能够在应用程序中缓冲大于 500 KB 的有效负载。

### 3.3.通过代码解决

或者，我们可以使用`WebFluxConfigurer`界面来配置相同的阈值。为此，我们将添加一个新的配置类，`WebFluxConfiguration:`

```java
@Configuration
public class WebFluxConfiguration implements WebFluxConfigurer {
    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        configurer.defaultCodecs().maxInMemorySize(500 * 1024);
    }
}
```

这种方法也将为我们提供相同的结果。

## 4.客户端

现在让我们换个角度来看看客户端的行为。

### 4.1.重现问题

我们将尝试使用 Webflux 的`WebClient. `来重现相同的行为。让我们创建一个处理程序，使用 390 KB 的有效负载来调用服务器:

```java
public Mono<Users> fetch() {
    return webClient
      .post()
      .uri("/1.0/process")
      .body(BodyInserters.fromPublisher(readRequestBody(), Users.class))
      .exchangeToMono(clientResponse -> clientResponse.bodyToMono(Users.class));
}
```

我们再次看到同样的异常被抛出，但这一次是由于`webClient`试图发送比允许的更大的有效载荷:

```java
org.springframework.core.io.buffer.DataBufferLimitException: Exceeded limit on max bytes to buffer : 262144
  at org.springframework.core.io.buffer.LimitedDataBufferList.raiseLimitException(LimitedDataBufferList.java:99) ~[spring-core-5.3.23.jar:5.3.23]
  Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
    *__checkpoint ⇢ Body from POST http://localhost:8080/1.0/process [DefaultClientResponse]
    *__checkpoint ⇢ Handler com.baeldung.[[email protected]](/web/20221225091828/https://www.baeldung.com/cdn-cgi/l/email-protection)428eedd9 [DispatcherHandler]
    *__checkpoint ⇢ HTTP POST "/1.0/trigger" [ExceptionHandlingWebHandler]
```

### 4.2.通过属性求解

同样，最简单的解决方案是**配置应用程序属性`spring.codec.max-in-memory-size`** 。让我们将以下内容添加到我们的`application.yaml `文件中:

```java
spring:
    codec:
        max-in-memory-size: 500KB
```

这样，我们现在应该能够从我们的应用程序发送大于 500 KB 的有效载荷。值得注意的是，这种配置适用于整个应用程序，也就是说适用于所有 web 客户端和服务器本身。

因此，如果我们只想为特定的 web 客户端配置这个限制，那么这将不是一个理想的解决方案。此外，这种方法还有一个[警告](https://web.archive.org/web/20221225091828/https://github.com/spring-projects/spring-boot/issues/27836)。用于创建`WebClients` 的构建器必须由 Spring 自动连接，如下所示:

```java
@Bean("webClient")
public WebClient getSelfWebClient(WebClient.Builder builder) {
    return builder.baseUrl(host).build();
}
```

### 4.3.通过代码解决

我们也有一种编程方式来配置 web 客户端以实现这一目标。让我们用以下配置创建一个`WebClient`:

```java
@Bean("progWebClient")
    public WebClient getProgSelfWebClient() {
        return WebClient
          .builder()
          .baseUrl(host)
          .exchangeStrategies(ExchangeStrategies
	  .builder()
	  .codecs(codecs -> codecs
            .defaultCodecs()
            .maxInMemorySize(500 * 1024))
	    .build())
          .build();
}
```

有了它，我们现在应该能够使用我们的 web 客户端成功地发送大于 500 KB 的有效载荷。

## 5.结论

在本文中，我们了解了什么是`DataBufferLimitException` ,并研究了如何在服务器端和客户端修复它们。我们研究了两种方法，首先是基于属性配置，其次是以编程方式。我们希望这个例外不会再给你带来麻烦。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221225091828/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules)