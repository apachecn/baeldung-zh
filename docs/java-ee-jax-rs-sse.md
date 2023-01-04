# JAX-RS 中服务器发送的事件(SSE)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-jax-rs-sse>

## 1。概述

服务器发送事件(SSE)是一种基于 HTTP 的规范，它提供了一种建立从服务器到客户端的长时间运行的单通道连接的方法。

客户端通过使用`Accept`报头中的媒体类型`text/event-stream`来发起 SSE 连接。

稍后，它会自动更新，而无需请求服务器。

我们可以在官方规格上查看更多关于规格的细节。

在本教程中，我们将介绍 SSE 的新 JAX-RS 2.1 实现。

因此，我们将看看如何用 JAX-RS 服务器 API 发布事件。此外，我们还将探索如何通过 JAX-RS 客户端 API 或者仅仅是像`curl`工具这样的 HTTP 客户端来使用它们。

## 2。了解 SSE 事件

SSE 事件是由以下字段组成的文本块:

*   `Event:`事件的类型。服务器可以发送许多不同类型的消息，而客户端可能只监听特定类型的消息，或者可以对每个事件类型进行不同的处理
*   `Data:`服务器发送的消息。对于同一事件，我们可以有许多数据线
*   `Id:`事件的 id，用于在连接重试后发送`Last-Event-ID`报头。这很有用，因为它可以防止服务器发送已经发送的事件
*   `Retry:`当电流丢失时，客户端建立新连接的时间，以毫秒为单位。最后收到的 Id 将通过`Last-Event-ID `报头自动发送
*   `:`':这是一个注释，被客户端忽略

此外，两个连续的事件由一个双换行符'`\n\n`'分隔。

此外，同一事件中的数据可以写入多行，如下例所示:

```java
event: stock
id: 1
: price change
retry: 4000
data: {"dateTime":"2018-07-14T18:06:00.285","id":1,
data: "name":"GOOG","price":75.7119}

event: stock
id: 2
: price change
retry: 4000
data: {"dateTime":"2018-07-14T18:06:00.285","id":2,"name":"IBM","price":83.4611}
```

在`JAX RS`中，SSE 事件由`SseEvent`接口`,`抽象，或者更准确地说，由两个子接口`OutboundSseEvent`和`InboundSseEvent.` 抽象

**在服务器 API 上使用`OutboundSseEvent`并设计一个发送的事件，而`InboundSseEvent `由客户端 API 使用并抽象一个接收的事件**。

## 3。发布 SSE 事件

现在我们已经讨论了什么是 SSE 事件，让我们看看如何构建它并将其发送到 HTTP 客户端。

### 3.1。项目设置

我们已经有了关于建立一个基于 JAX RS 的 Maven 项目的教程。请随意查看那里，了解如何设置依赖关系以及如何开始使用 JAX RS。

### 3.2。SSE 资源方法

SSE 资源方法是 JAX RS 方法，它:

*   可以产生一个`text/event-stream`媒体类型
*   有一个注入的`SseEventSink`参数，事件被发送到这里
*   也可能有一个注入的`Sse`参数，该参数用作创建事件生成器的入口点

```java
@GET
@Path("prices")
@Produces("text/event-stream")
public void getStockPrices(@Context SseEventSink sseEventSink, @Context Sse sse) {
    //...
}
```

因此，客户端应该使用以下 HTTP 头发出第一个 HTTP 请求:

```java
Accept: text/event-stream 
```

### 3.3。SSE 实例

SSE 实例是一个上下文 bean，JAX RS 运行时将使其可用于注入。

我们可以把它作为一个工厂来创造:

*   `OutboundSseEvent.Builder – `允许我们创建事件
*   `SseBroadcaster – `允许我们向多个用户广播事件

让我们看看它是如何工作的:

```java
@Context
public void setSse(Sse sse) {
    this.sse = sse;
    this.eventBuilder = sse.newEventBuilder();
    this.sseBroadcaster = sse.newBroadcaster();
}
```

现在，让我们关注事件构建器。`OutboundSseEvent.Builder`负责创建`OutboundSseEvent`:

```java
OutboundSseEvent sseEvent = this.eventBuilder
  .name("stock")
  .id(String.valueOf(lastEventId))
  .mediaType(MediaType.APPLICATION_JSON_TYPE)
  .data(Stock.class, stock)
  .reconnectDelay(4000)
  .comment("price change")
  .build();
```

正如我们所看到的，**构建器有方法为上面显示的**的所有事件字段设置值。此外，`mediaType() `方法用于将数据字段 Java 对象序列化为合适的文本格式。

默认情况下，数据字段的媒体类型为`text/plain`。因此，在处理`String`数据类型时，不需要显式指定它。

否则，如果我们想要处理一个自定义对象，我们需要指定媒体类型或者提供一个自定义的`MessageBodyWriter.`**。JAX RS 运行时为大多数已知的媒体类型**提供了`MessageBodyWriters`。

Sse 实例还有两个生成器快捷方式，用于创建仅包含数据字段或类型和数据字段的事件:

```java
OutboundSseEvent sseEvent = sse.newEvent("cool Event");
OutboundSseEvent sseEvent = sse.newEvent("typed event", "data Event");
```

### 3.4。发送简单事件

现在我们知道了如何构建事件，并且了解了 SSE 资源是如何工作的。我们来发一个简单的事件。

`SseEventSink`接口抽象出一个 HTTP 连接。JAX-RS 运行时只能通过在 SSE 资源方法中注入来使其可用。

发送事件就像调用`SseEventSink.` `send(). `一样简单

在下一个示例中，将发送大量股票更新，并最终关闭事件流:

```java
@GET
@Path("prices")
@Produces("text/event-stream")
public void getStockPrices(@Context SseEventSink sseEventSink /*..*/) {
    int lastEventId = //..;
    while (running) {
        Stock stock = stockService.getNextTransaction(lastEventId);
        if (stock != null) {
            OutboundSseEvent sseEvent = this.eventBuilder
              .name("stock")
              .id(String.valueOf(lastEventId))
              .mediaType(MediaType.APPLICATION_JSON_TYPE)
              .data(Stock.class, stock)
              .reconnectDelay(3000)
              .comment("price change")
              .build();
            sseEventSink.send(sseEvent);
            lastEventId++;
        }
     //..
    }
    sseEventSink.close();
}
```

发送完所有事件后，服务器通过显式调用`close()`方法关闭连接，或者最好通过使用`try-with-resource,`，因为`SseEventSink`扩展了`AutoClosable`接口:

```java
try (SseEventSink sink = sseEventSink) {
    OutboundSseEvent sseEvent = //..
    sink.send(sseEvent);
}
```

在我们的示例应用程序中，如果我们访问:

```java
http://localhost:9080/sse-jaxrs-server/sse.html
```

### 3.5。广播事件

广播是将事件同时发送到多个客户端的过程。这是由`SseBroadcaster` API 完成的，分三个简单的步骤完成:

首先，我们从注入的 Sse 上下文中创建`SseBroadcaster`对象，如前所示:

```java
SseBroadcaster sseBroadcaster = sse.newBroadcaster();
```

然后，客户端应该订阅以便能够接收 Sse 事件。这通常在 SSE 资源方法中完成，其中注入了一个`SseEventSink`上下文实例:

```java
@GET
@Path("subscribe")
@Produces(MediaType.SERVER_SENT_EVENTS)
public void listen(@Context SseEventSink sseEventSink) {
    this.sseBroadcaster.register(sseEventSink);
}
```

最后，**我们可以通过调用`broadcast()`方法**来触发事件发布:

```java
@GET
@Path("publish")
public void broadcast() {
    OutboundSseEvent sseEvent = //...;
    this.sseBroadcaster.broadcast(sseEvent);
}
```

这将向每个注册的`SseEventSink.`发送相同的事件

为了展示广播，我们可以访问以下 URL:

```java
http://localhost:9080/sse-jaxrs-server/sse-broadcast.html
```

然后我们可以通过调用 broadcast()资源方法来触发广播:

```java
curl -X GET http://localhost:9080/sse-jaxrs-server/sse/stock/publish
```

## 4。消费上交所事件

为了使用服务器发送的 SSE 事件，我们可以使用任何 HTTP 客户端，但是在本教程中，我们将使用 JAX RS 客户端 API。

### 4.1。用于 SSE 的 JAX 客户端 API

为了开始使用 SSE 的客户端 API，我们需要提供 JAX RS 客户端实现的依赖关系。

这里，我们将使用 Apache CXF 客户端实现:

```java
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-rs-client</artifactId>
    <version>${cxf-version}</version>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-rs-sse</artifactId>
    <version>${cxf-version}</version>
</dependency>
```

`SseEventSource`是这个 API 的核心，它是从`WebTarget.` 构建的

我们首先监听由`InboundSseEvent`接口抽象的传入事件:

```java
Client client = ClientBuilder.newClient();
WebTarget target = client.target(url);
try (SseEventSource source = SseEventSource.target(target).build()) {
    source.register((inboundSseEvent) -> System.out.println(inboundSseEvent));
    source.open();
}
```

**一旦连接建立，注册的事件消费者将为每个接收到的**调用`**InboundSseEvent**.`

然后我们可以使用`readData()`方法读取原始数据`String:`

```java
String data = inboundSseEvent.readData();
```

或者，我们可以使用重载版本，通过合适的媒体类型获得反序列化的 Java 对象:

```java
Stock stock = inboundSseEvent.readData(Stock.class, MediaType.Application_Json);
```

这里，我们只提供了一个简单的事件消费者，它在控制台中打印传入的事件。

## 5。结论

在本教程中，我们重点介绍了如何在 JAX RS 2.1 中使用服务器发送的事件。我们提供了一个示例，展示了如何向单个客户端发送事件，以及如何向多个客户端广播事件。

最后，我们使用 JAX-RS 客户端 API 消费这些事件。

像往常一样，这个教程的代码可以在 Github 上找到[。](https://web.archive.org/web/20221205114008/https://github.com/eugenp/tutorials/tree/master/apache-cxf-modules/sse-jaxrs)