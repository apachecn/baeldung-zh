# Spring 中服务器发送的事件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-server-sent-events>

## 1.概观

在本教程中，我们将看到如何用 Spring 实现基于服务器发送事件的 API。

简单地说，Server-Sent-Events(简称 SSE)是一种 HTTP 标准，它允许 web 应用处理单向事件流，并在服务器发出数据时接收更新。

Spring 4.2 版本已经支持它，但是从 Spring 5 开始，[我们现在有了一个更习惯和方便的方法来处理它](/web/20220625081657/https://www.baeldung.com/spring-webflux)。

## 2.带 Spring 5 Webflux 的 SSE

为了实现这一点，**我们可以利用一些实现，比如由`Reactor`库提供的`Flux`类，或者潜在的`ServerSentEvent`实体**，这让我们可以控制事件元数据。

### 2.1.使用`Flux`流事件

`Flux`是一个事件流的反应式表示——它根据指定的请求或响应媒体类型进行不同的处理。

要创建 SSE 流端点，我们必须遵循 W3C 规范的[，并将其 MIME 类型指定为`text/event-stream`:](https://web.archive.org/web/20220625081657/https://www.w3.org/TR/eventsource/)

```java
@GetMapping(path = "/stream-flux", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamFlux() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(sequence -> "Flux - " + LocalTime.now().toString());
}
```

**`interval`方法创建一个`Flux`，它递增地发出`long`值。然后，我们将这些值映射到我们想要的输出。**

让我们启动我们的应用程序，然后通过浏览端点来尝试一下。

我们将看到浏览器如何对服务器一秒一秒推送的事件做出反应。关于`Flux`和 `Reactor Core`的更多信息，我们可以查看[这篇文章](/web/20220625081657/https://www.baeldung.com/reactor-core)。

### 2.2.利用`ServerSentEvent`元素

我们现在将把输出`String`包装到一个`ServerSentSevent`对象中，并检查这样做的好处:

```java
@GetMapping("/stream-sse")
public Flux<ServerSentEvent<String>> streamEvents() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(sequence -> ServerSentEvent.<String> builder()
        .id(String.valueOf(sequence))
          .event("periodic-event")
          .data("SSE - " + LocalTime.now().toString())
          .build());
}
```

正如我们所理解的，**使用`ServerSentEvent`实体**有几个好处:

1.  我们可以处理事件元数据，这是我们在真实案例场景中需要的
2.  我们可以忽略"`text/event-stream`"媒体类型声明

在本例中，我们指定了一个`id`，一个`event name`，以及最重要的，事件的实际`data`。

此外，我们可以添加一个`comments`属性和一个`retry`值，它将指定在尝试发送事件时使用的重新连接时间。

### 2.3.使用 WebClient 消费服务器发送的事件

现在让我们用一个 [`WebClient`](/web/20220625081657/https://www.baeldung.com/spring-5-webclient) 来消费我们的事件流。：

```java
public void consumeServerSentEvent() {
    WebClient client = WebClient.create("http://localhost:8080/sse-server");
    ParameterizedTypeReference<ServerSentEvent<String>> type
     = new ParameterizedTypeReference<ServerSentEvent<String>>() {};

    Flux<ServerSentEvent<String>> eventStream = client.get()
      .uri("/stream-sse")
      .retrieve()
      .bodyToFlux(type);

    eventStream.subscribe(
      content -> logger.info("Time: {} - event: name[{}], id [{}], content[{}] ",
        LocalTime.now(), content.event(), content.id(), content.data()),
      error -> logger.error("Error receiving SSE: {}", error),
      () -> logger.info("Completed!!!"));
}
```

**`subscribe` 方法允许我们指出当我们成功接收到一个事件时，当一个错误发生时，以及当流式传输完成时，我们将如何继续。**

在我们的例子中，我们使用了`retrieve`方法，这是一种简单直接的获取响应体的方法。

如果我们收到 4xx 或 5xx 响应，这个方法会自动抛出一个`WebClientResponseException`，除非我们处理添加一个`onStatus`语句的场景。

另一方面，我们也可以使用`exchange`方法，它提供了对`ClientResponse`的访问，也不会对失败的响应发出错误信号。

我们必须考虑到，如果我们不需要事件元数据，我们可以绕过`ServerSentEvent`包装器。

## 3.Spring MVC 中的 SSE 流

正如我们所说，自从 Spring 4.2 引入了`SseEmitter`类，SSE 规范就受到了支持。

**简单来说，我们将定义一个`ExecutorService`，一个线程，在那里`SseEmitter`将完成推送数据的工作，并返回发射器实例，以这种方式保持连接打开:**

```java
@GetMapping("/stream-sse-mvc")
public SseEmitter streamSseMvc() {
    SseEmitter emitter = new SseEmitter();
    ExecutorService sseMvcExecutor = Executors.newSingleThreadExecutor();
    sseMvcExecutor.execute(() -> {
        try {
            for (int i = 0; true; i++) {
                SseEventBuilder event = SseEmitter.event()
                  .data("SSE MVC - " + LocalTime.now().toString())
                  .id(String.valueOf(i))
                  .name("sse event - mvc");
                emitter.send(event);
                Thread.sleep(1000);
            }
        } catch (Exception ex) {
            emitter.completeWithError(ex);
        }
    });
    return emitter;
}
```

**始终确保为您的用例场景选择正确的`ExecutorService`。**

通过阅读[这篇有趣的教程](/web/20220625081657/https://www.baeldung.com/spring-mvc-sse-streams)，我们可以在 Spring MVC 中了解更多关于 SSE 的知识，并看看其他的例子。

## 4.了解服务器发送的事件

现在我们知道了如何实现 SSE 端点，让我们通过理解一些底层概念来尝试更深入一点。

SSE 是大多数浏览器采用的一种规范，允许在任何时候单向流式传输事件。

“事件”只是一个 UTF-8 编码的文本数据流，遵循规范定义的格式。

这种格式由一系列由换行符分隔的键值元素(id、重试、数据和事件，表示名称)组成。

也支持注释。

该规范不以任何方式限制数据有效载荷格式；我们可以使用简单的`String`或更复杂的 JSON 或 XML 结构。

我们必须考虑的最后一点是使用 SSE 流和`[WebSockets](/web/20220625081657/https://www.baeldung.com/spring-5-reactive-websockets)`的区别。

**而`WebSockets`在服务器和客户端之间提供全双工(双向)通信，而 SSE 使用单向通信。**

另外，`WebSockets`不是 HTTP 协议，与 SSE 相反，它不提供错误处理标准。

## 5.结论

总之，在本文中，我们已经了解了 SSE 流的主要概念，这无疑是一个很好的资源，可以让我们创建下一代系统。

当我们使用这个协议时，我们现在处于一个很好的位置来理解在引擎盖下发生了什么。

此外，我们用一些简单的例子来补充这个理论，这些例子可以在我们的 Github 库中找到。