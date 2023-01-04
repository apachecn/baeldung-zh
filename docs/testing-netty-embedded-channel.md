# 使用嵌入式通道测试 Netty

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/testing-netty-embedded-channel>

## 1.介绍

在本文中，我们将看到如何使用`EmbeddedChannel `来测试我们的入站和出站通道处理程序的功能。

Netty 是一个非常通用的框架，用于编写高性能的异步应用程序。如果没有合适的工具，对这样的应用程序进行单元测试可能会很棘手。

谢天谢地，框架为我们提供了 **`EmbeddedChannel `类——这方便了`ChannelHandlers`** 的测试。

## 2.设置

`EmbeddedChannel`是 Netty 框架的一部分，所以唯一需要的依赖是 Netty 本身的依赖。

依赖关系可以在 [Maven Central](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.netty%22%20AND%20a%3A%22netty-all%22) 上找到:

```java
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.24.Final</version>
</dependency>
```

## 3. **`EmbeddedChannel`概述**

**`EmbeddedChannel`类只是`AbstractChannel `** 的另一个实现——它传输数据**而不需要真正的网络连接**。

这很有用，因为我们可以通过在入站通道上写入数据来模拟传入消息，还可以检查出站通道上生成的响应。这样我们可以单独测试每个`ChannelHandler `或者在整个通道管道中。

为了测试一个或多个`ChannelHandlers`、，我们首先必须使用一个构造函数创建一个`EmbeddedChannel `实例。

**初始化`EmbeddedChannel `最常见的方式是将`ChannelHandlers `的列表传递给它的构造函数:**

```java
EmbeddedChannel channel = new EmbeddedChannel(
  new HttpMessageHandler(), new CalculatorOperationHandler());
```

如果我们想对处理程序插入管道的顺序有更多的控制，我们可以用默认的构造函数创建一个`EmbeddedChannel`并直接添加处理程序:

```java
channel.pipeline()
  .addFirst(new HttpMessageHandler())
  .addLast(new CalculatorOperationHandler());
```

同样，**当我们创建一个`EmbeddedChannel,`时，它会有一个由`DefaultChannelConfig`类给定的默认配置。**

当我们想要使用自定义配置时，比如降低默认的连接超时值，我们可以通过使用`config()`方法来访问`ChannelConfig `对象:

```java
DefaultChannelConfig channelConfig = (DefaultChannelConfig) channel
  .config();
channelConfig.setConnectTimeoutMillis(500);
```

`EmbeddedChannel `包括我们可以用来读写数据到我们的`ChannelPipeline`的方法。最常用的方法有:

*   `readInbound()`
*   `readOutbound()`
*   `writeInbound(Object… msgs)`
*   `writeOutbound(Object… msgs)`

**read 方法检索并删除入站/出站队列中的第一个元素。**当我们需要在不删除任何元素的情况下访问整个消息队列时，我们可以使用`outboundMessages() `方法:

```java
Object lastOutboundMessage = channel.readOutbound();
Queue<Object> allOutboundMessages = channel.outboundMessages();
```

**当消息成功添加到`Channel:`** 的入站/出站管道时，写方法返回`true `

```java
channel.writeInbound(httpRequest)
```

这个想法是，我们在入站管道上编写消息，这样 out `ChannelHandlers`将处理它们，并且我们期望结果可以从出站管道中读取。

## 4.测试`ChannelHandlers`

让我们看一个简单的例子，在这个例子中，我们想测试一个由两个`ChannelHandlers `组成的管道，这两个管道接收一个 HTTP 请求，并期待一个包含计算结果的 HTTP 响应:

```java
EmbeddedChannel channel = new EmbeddedChannel(
  new HttpMessageHandler(), new CalculatorOperationHandler());
```

第一个，`HttpMessageHandler `将从 HTTP 请求中提取数据，并将其传递给管道中的第二个`ChannelHandler `，`CalculatorOperationHandler`，对数据进行处理。

现在，让我们编写 HTTP 请求，看看入站管道是否处理它:

```java
FullHttpRequest httpRequest = new DefaultFullHttpRequest(
  HttpVersion.HTTP_1_1, HttpMethod.GET, "/calculate?a=10&b;=5");
httpRequest.headers().add("Operator", "Add");

assertThat(channel.writeInbound(httpRequest)).isTrue();
long inboundChannelResponse = channel.readInbound();
assertThat(inboundChannelResponse).isEqualTo(15);
```

我们可以看到，我们已经使用`writeInbound() `方法在入站管道上发送了 HTTP 请求，并使用`readInbound()`读取了结果；`inboundChannelResponse `是我们发送的数据经过入站管道中所有`ChannelHandlers `处理后产生的消息。

现在，让我们检查我们的 Netty 服务器是否用正确的 HTTP 响应消息进行响应。为此，我们将检查出站管道上是否存在消息:

```java
assertThat(channel.outboundMessages().size()).isEqualTo(1);
```

在本例中，出站消息是一个 HTTP 响应，因此让我们检查内容是否正确。我们通过读取出站管道中的最后一条消息来做到这一点:

```java
FullHttpResponse httpResponse = channel.readOutbound();
String httpResponseContent = httpResponse.content()
  .toString(Charset.defaultCharset());
assertThat(httpResponseContent).isEqualTo("15");
```

## 4.测试异常处理

另一个常见的测试场景是异常处理。

我们可以通过实现`exceptionCaught() `方法来处理`ChannelInboundHandlers `中的异常，但是有些情况下我们不想处理异常，而是将它传递给管道中的下一个`ChannelHandler`。

我们可以使用来自`EmbeddedChannel`类的`checkException() `方法来检查是否在管道上收到了任何`Throwable `对象，并重新抛出它。

这样我们可以捕捉到`Exception `并检查`ChannelHandler`是否应该抛出它:

```java
assertThatThrownBy(() -> {
    channel.pipeline().fireChannelRead(wrongHttpRequest);
    channel.checkException();
}).isInstanceOf(UnsupportedOperationException.class)
  .hasMessage("HTTP method not supported");
```

我们可以在上面的例子中看到，我们已经发送了一个 HTTP 请求，我们期望它触发一个`Exception`。通过使用`checkException() `方法，我们可以重新抛出管道中存在的最后一个异常，因此我们可以断言从中需要什么。

## 5.结论

`EmbeddedChannel `是 Netty 框架提供的一个很棒的特性，可以帮助我们测试 out `ChannelHandler `管道的正确性。它可以用来单独测试每个`ChannelHandler `，更重要的是测试整个管道。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/libraries-server)