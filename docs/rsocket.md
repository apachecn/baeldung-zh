# 火箭介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rsocket>

## 1.介绍

在本教程中，我们将首先了解一下 [RSocket](https://web.archive.org/web/20221205180251/http://rsocket.io/) 以及它是如何实现客户机-服务器通信的。

## 2.什么是`RSocket`？

RSocket 是一种二进制点对点通信协议，旨在用于分布式应用。从这个意义上说，它提供了 HTTP 等其他协议的替代方案。

对 RSocket 和其他协议进行全面的比较超出了本文的范围。相反，我们将关注 RSocket 的一个关键特性:它的交互模型。

RSocket 提供了四种交互模型。记住这一点，我们将通过一个例子来探究每一个问题。

## 3.Maven 依赖性

对于我们的示例，RSocket 只需要两个直接依赖项:

```java
<dependency>
    <groupId>io.rsocket</groupId>
    <artifactId>rsocket-core</artifactId>
    <version>0.11.13</version>
</dependency>
<dependency>
    <groupId>io.rsocket</groupId>
    <artifactId>rsocket-transport-netty</artifactId>
    <version>0.11.13</version>
</dependency>
```

在 Maven Central 上可以获得 [rsocket-core](https://web.archive.org/web/20221205180251/https://search.maven.org/search?q=rsocket-core) 和 [rsocket-transport-netty](https://web.archive.org/web/20221205180251/https://search.maven.org/search?q=rsocket-transport-netty) 依赖项。

**一个重要的注意事项是 RSocket 库频繁使用[反应流](https://web.archive.org/web/20221205180251/https://projectreactor.io/)** 。本文通篇都在使用`[Flux](https://web.archive.org/web/20221205180251/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)`和`[Mono](https://web.archive.org/web/20221205180251/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)`类，因此对它们有一个基本的了解会有所帮助。

## 4.服务器设置

首先，让我们创建`Server`类:

```java
public class Server {
    private final Disposable server;

    public Server() {
        this.server = RSocketFactory.receive()
          .acceptor((setupPayload, reactiveSocket) -> Mono.just(new RSocketImpl()))
          .transport(TcpServerTransport.create("localhost", TCP_PORT))
          .start()
          .subscribe();
    }

    public void dispose() {
        this.server.dispose();
    }

    private class RSocketImpl extends AbstractRSocket {}
}
```

**这里我们使用`RSocketFactory`来设置和监听一个 TCP 套接字。**我们传递我们的客户`RSocketImpl`来处理来自客户的请求。我们将一边进行一边向`RSocketImpl`添加方法。

接下来，要启动服务器，我们只需要实例化它:

```java
Server server = new Server();
```

一个服务器实例可以处理多个连接。因此，仅仅一个服务器实例就可以支持我们所有的例子。

当我们完成时，`dispose`方法将停止服务器并释放 TCP 端口。

## 4.交互模型

### 4.1.请求/回应

RSocket 提供了一个请求/响应模型——每个请求接收一个响应。

对于这个模型，我们将创建一个简单的服务，将消息返回给客户端。

让我们首先向我们的`AbstractRSocket, ` `RSocketImpl`扩展添加一个方法:

```java
@Override
public Mono<Payload> requestResponse(Payload payload) {
    try {
        return Mono.just(payload); // reflect the payload back to the sender
    } catch (Exception x) {
        return Mono.error(x);
    }
}
```

**`requestResponse`方法为每个请求**返回一个结果，正如我们从`Mono<Payload>`响应类型中看到的。

**`Payload`是包含消息内容和元数据**的类。它被所有的交互模型所使用。有效负载的内容是二进制的，但是有一些方便的方法支持基于`String`的内容。

接下来，我们可以创建我们的客户端类:

```java
public class ReqResClient {

    private final RSocket socket;

    public ReqResClient() {
        this.socket = RSocketFactory.connect()
          .transport(TcpClientTransport.create("localhost", TCP_PORT))
          .start()
          .block();
    }

    public String callBlocking(String string) {
        return socket
          .requestResponse(DefaultPayload.create(string))
          .map(Payload::getDataUtf8)
          .block();
    }

    public void dispose() {
        this.socket.dispose();
    }
}
```

客户端使用`RSocketFactory.connect()`方法启动与服务器的套接字连接。**我们使用套接字上的`requestResponse`方法向服务器**发送有效载荷。

我们的有效负载包含传入客户端的`String`。**当`Mono`** `<Payload>`响应到达时，我们可以使用`getDataUtf8() `方法来访问响应的`String`内容。

最后，我们可以运行集成测试来查看请求/响应的运行情况。我们将向服务器发送一个`String`,并验证是否返回了相同的`String`:

```java
@Test
public void whenSendingAString_thenRevceiveTheSameString() {
    ReqResClient client = new ReqResClient();
    String string = "Hello RSocket";

    assertEquals(string, client.callBlocking(string));

    client.dispose();
}
```

### 4.2.火了就忘了

使用“一劳永逸”模式，**客户端将不会收到来自服务器**的响应。

在本例中，客户端将以 50 毫秒的间隔向服务器发送模拟测量值。服务器将公布测量结果。

让我们在服务器的`RSocketImpl`类中添加一个一次性处理程序:

```java
@Override
public Mono<Void> fireAndForget(Payload payload) {
    try {
        dataPublisher.publish(payload); // forward the payload
        return Mono.empty();
    } catch (Exception x) {
        return Mono.error(x);
    }
}
```

这个处理程序看起来非常类似于请求/响应处理程序。但是， **`fireAndForget`返回的是`Mono<Void>`** 而不是`Mono<Payload>`。

`dataPublisher`是 [`org.reactivestreams.Publisher`](https://web.archive.org/web/20221205180251/http://www.reactive-streams.org/reactive-streams-1.0.2-javadoc/org/reactivestreams/Publisher.html?is-external=true) 的一个实例。因此，它使有效载荷对订户可用。我们将在请求/流示例中利用这一点。

接下来，我们将创建一次性客户端:

```java
public class FireNForgetClient {
    private final RSocket socket;
    private final List<Float> data;

    public FireNForgetClient() {
        this.socket = RSocketFactory.connect()
          .transport(TcpClientTransport.create("localhost", TCP_PORT))
          .start()
          .block();
    }

    /** Send binary velocity (float) every 50ms */
    public void sendData() {
        data = Collections.unmodifiableList(generateData());
        Flux.interval(Duration.ofMillis(50))
          .take(data.size())
          .map(this::createFloatPayload)
          .flatMap(socket::fireAndForget)
          .blockLast();
    }

    // ... 
}
```

插座设置与之前完全相同。

`sendData()`方法使用一个`Flux`流来发送多个消息。对于每条消息，我们调用`socket::fireAndForget` 。

**我们需要订阅每个消息**的`Mono<Void>`响应。如果我们忘记订阅，那么`socket::fireAndForget`将不会执行。

`flatMap`操作员确保`Void`响应被传递给订阅者，而`blockLast`操作员充当订阅者。

我们将等到下一部分来运行一次性测试。在这一点上，我们将创建一个请求/流客户机来接收由“一劳永逸”客户机推送的数据。

### 4.3.请求/流

在请求/流模型中，**单个请求可能会收到多个响应**。为了看到这一点，我们可以建立一个“一劳永逸”的例子。为此，让我们请求一个流来检索我们在上一节中发送的度量。

和以前一样，让我们从向服务器上的`RSocketImpl`添加一个新的监听器开始:

```java
@Override
public Flux<Payload> requestStream(Payload payload) {
    return Flux.from(dataPublisher);
}
```

**`requestStream`处理器返回一个`Flux<Payload>`流**。正如我们在上一节中回忆的那样，`fireAndForget`处理程序将传入的数据发布给了`dataPublisher.`，现在，我们将使用同一个`dataPublisher`作为事件源来创建一个`Flux`流。通过这样做，测量数据将异步地从我们的“一劳永逸”的客户机流向我们的请求/流客户机。

接下来让我们创建请求/流客户端:

```java
public class ReqStreamClient {

    private final RSocket socket;

    public ReqStreamClient() {
        this.socket = RSocketFactory.connect()
          .transport(TcpClientTransport.create("localhost", TCP_PORT))
          .start()
          .block();
    }

    public Flux<Float> getDataStream() {
        return socket
          .requestStream(DefaultPayload.create(DATA_STREAM_NAME))
          .map(Payload::getData)
          .map(buf -> buf.getFloat())
          .onErrorReturn(null);
    }

    public void dispose() {
        this.socket.dispose();
    }
}
```

我们以与之前的客户端相同的方式连接到服务器。

在`getDataStream()` **中，我们使用`socket.requestStream()`从服务器**接收一个流量<有效载荷>流。从这个流中，我们从二进制数据中提取出`Float`值。最后，流被返回给调用者，允许调用者订阅它并处理结果。

现在我们来测试一下。**我们将验证从“一劳永逸”到请求/流的往返过程。**

我们可以断言每个值的接收顺序与发送顺序相同。然后，我们可以断言，我们收到的值与发送的值数量相同:

```java
@Test
public void whenSendingStream_thenReceiveTheSameStream() {
    FireNForgetClient fnfClient = new FireNForgetClient(); 
    ReqStreamClient streamClient = new ReqStreamClient();

    List<Float> data = fnfClient.getData();
    List<Float> dataReceived = new ArrayList<>();

    Disposable subscription = streamClient.getDataStream()
      .index()
      .subscribe(
        tuple -> {
            assertEquals("Wrong value", data.get(tuple.getT1().intValue()), tuple.getT2());
            dataReceived.add(tuple.getT2());
        },
        err -> LOG.error(err.getMessage())
      );

    fnfClient.sendData();

    // ... dispose client & subscription

    assertEquals("Wrong data count received", data.size(), dataReceived.size());
}
```

### 4.4.频道

**通道模型提供双向通信**。在这个模型中，消息流在两个方向上异步流动。

让我们创建一个简单的游戏模拟来测试这一点。在这个游戏中，通道的每一方都将成为玩家。随着游戏的运行，这些玩家会以随机的时间间隔向对方发送消息。对方会对这些信息做出反应。

首先，我们将在服务器上创建处理程序。像以前一样，我们给`RSocketImpl`加上:

```java
@Override
public Flux<Payload> requestChannel(Publisher<Payload> payloads) {
    Flux.from(payloads)
      .subscribe(gameController::processPayload);
    return Flux.from(gameController);
}
```

**`requestChannel`处理器有`Payload`个流用于输入和输出**。`Publisher<Payload>`输入参数是从客户端接收的有效负载流。当它们到达时，这些有效载荷被传递给`gameController::processPayload`函数。

作为响应，我们向客户端返回一个不同的`Flux`流。这个流是从我们的`gameController`创建的，它也是一个`Publisher`。

下面是对`GameController`类的总结:

```java
public class GameController implements Publisher<Payload> {

    @Override
    public void subscribe(Subscriber<? super Payload> subscriber) {
        // send Payload messages to the subscriber at random intervals
    }

    public void processPayload(Payload payload) {
        // react to messages from the other player
    }
}
```

当`GameController`接收到一个用户时，它开始向该用户发送消息。

接下来，让我们创建客户端:

```java
public class ChannelClient {

    private final RSocket socket;
    private final GameController gameController;

    public ChannelClient() {
        this.socket = RSocketFactory.connect()
          .transport(TcpClientTransport.create("localhost", TCP_PORT))
          .start()
          .block();

        this.gameController = new GameController("Client Player");
    }

    public void playGame() {
        socket.requestChannel(Flux.from(gameController))
          .doOnNext(gameController::processPayload)
          .blockLast();
    }

    public void dispose() {
        this.socket.dispose();
    }
}
```

正如我们在前面的例子中所看到的，客户端以与其他客户端相同的方式连接到服务器。

客户端创建自己的`GameController`实例。

**我们使用`socket.requestChannel()`将我们的`Payload`流发送到服务器**。服务器用它自己的有效负载流来响应。

当从服务器接收到有效载荷时，我们将它们传递给我们的`gameController::processPayload`处理器。

在我们的游戏模拟中，客户端和服务器是彼此的镜像。也就是说，**每一端都在发送一个`Payload`流，并从另一端**接收一个`Payload`流。

这些流独立运行，没有同步。

最后，让我们在测试中运行模拟:

```java
@Test
public void whenRunningChannelGame_thenLogTheResults() {
    ChannelClient client = new ChannelClient();
    client.playGame();
    client.dispose();
}
```

## 5.结论

在这篇介绍性文章中，我们探索了 RSocket 提供的交互模型。示例的完整源代码可以在我们的 [Github 库](https://web.archive.org/web/20221205180251/https://github.com/eugenp/tutorials/tree/master/rsocket)中找到。

请务必查看 RSocket 网站进行更深入的讨论。特别是，[常见问题](https://web.archive.org/web/20221205180251/https://rsocket.io/about/faq)和[动机](https://web.archive.org/web/20221205180251/https://rsocket.io/about/motivations)文档提供了很好的背景。