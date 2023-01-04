# Netty 中的 HTTP/2

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netty-http2>

## 1.概观

Netty 是一个基于 NIO 的客户机-服务器框架，它让 Java 开发人员能够在网络层上操作。使用这个框架，开发人员可以构建他们自己的任何已知协议的实现，甚至自定义协议。

对于框架的基本理解，[介绍 Netty](/web/20221208143856/https://www.baeldung.com/netty) 是一个很好的开始。

在本教程中，**我们将看到如何在 Netty** 中实现 HTTP/2 服务器和客户端。

## 2.什么是`HTTP/2`？

顾名思义， [HTTP version 2 或简称 HTTP/2](https://web.archive.org/web/20221208143856/https://httpwg.org/specs/rfc7540.html) ，是超文本传输协议的更新版本。

大约在互联网诞生的 1989 年，HTTP/1.0 应运而生。1997 年升级到 1.1 版。然而，直到 2015 年，它才迎来了重大升级，第 2 版。

在撰写本文时， [HTTP/3](https://web.archive.org/web/20221208143856/https://blog.cloudflare.com/http3-the-past-present-and-future/) 也是可用的，尽管并非所有浏览器都默认支持。

HTTP/2 仍然是被广泛接受和实现的协议的最新版本。它与以前的版本有很大的不同，包括多路复用和服务器推送功能。

HTTP/2 中的通信通过一组称为帧的字节进行，多个帧形成一个流。

在我们的代码示例中，**我们将看到 Netty 如何处理[报头](https://web.archive.org/web/20221208143856/https://tools.ietf.org/html/rfc7540#section-6.2)、[数据](https://web.archive.org/web/20221208143856/https://tools.ietf.org/html/rfc7540#section-6.1)和[设置](https://web.archive.org/web/20221208143856/https://tools.ietf.org/html/rfc7540#section-6.5)帧**的交换。

## 3.服务器

现在让我们看看如何在 Netty 中创建一个 HTTP/2 服务器。

### 3.1.`SslContext`

Netty 支持通过 TLS 的 HTTP/2 的 [APN 协商。所以，我们首先需要创建一个服务器](https://web.archive.org/web/20221208143856/https://tools.ietf.org/html/rfc7301) [`SslContext`](https://web.archive.org/web/20221208143856/https://netty.io/4.1/api/io/netty/handler/ssl/SslContext.html) :

```java
SelfSignedCertificate ssc = new SelfSignedCertificate();
SslContext sslCtx = SslContextBuilder.forServer(ssc.certificate(), ssc.privateKey())
  .sslProvider(SslProvider.JDK)
  .ciphers(Http2SecurityUtil.CIPHERS, SupportedCipherSuiteFilter.INSTANCE)
  .applicationProtocolConfig(
    new ApplicationProtocolConfig(Protocol.ALPN, SelectorFailureBehavior.NO_ADVERTISE,
      SelectedListenerFailureBehavior.ACCEPT, ApplicationProtocolNames.HTTP_2))
  .build();
```

这里，我们用 JDK SSL 提供者为服务器创建了一个上下文，添加了一些密码，并为 HTTP/2 配置了应用层协议协商。

这意味着**我们的服务器将只支持 HTTP/2 及其底层[协议标识符 h2](https://web.archive.org/web/20221208143856/https://httpwg.org/specs/rfc7540.html#versioning)** 。

### 3.2.用`ChannelInitializer`引导服务器

接下来，我们需要一个`ChannelInitializer`用于我们的多路复用子通道，以便建立一个 Netty 管道。

我们将使用这个通道中前面的`sslContext`来启动管道，然后引导服务器:

```java
public final class Http2Server {

    static final int PORT = 8443;

    public static void main(String[] args) throws Exception {
        SslContext sslCtx = // create sslContext as described above
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.option(ChannelOption.SO_BACKLOG, 1024);
            b.group(group)
              .channel(NioServerSocketChannel.class)
              .handler(new LoggingHandler(LogLevel.INFO))
              .childHandler(new ChannelInitializer() {
                  @Override
                  protected void initChannel(SocketChannel ch) throws Exception {
                      if (sslCtx != null) {
                          ch.pipeline()
                            .addLast(sslCtx.newHandler(ch.alloc()), Http2Util.getServerAPNHandler());
                      }
                  }
            });
            Channel ch = b.bind(PORT).sync().channel();

            logger.info("HTTP/2 Server is listening on https://127.0.0.1:" + PORT + '/');

            ch.closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

作为该通道初始化的一部分，我们将在我们自己的实用程序类`Http2Util`中定义的实用程序方法`getServerAPNHandler()` 中向管道添加一个 APN 处理程序:

```java
public static ApplicationProtocolNegotiationHandler getServerAPNHandler() {
    ApplicationProtocolNegotiationHandler serverAPNHandler = 
      new ApplicationProtocolNegotiationHandler(ApplicationProtocolNames.HTTP_2) {

        @Override
        protected void configurePipeline(ChannelHandlerContext ctx, String protocol) throws Exception {
            if (ApplicationProtocolNames.HTTP_2.equals(protocol)) {
                ctx.pipeline().addLast(
                  Http2FrameCodecBuilder.forServer().build(), new Http2ServerResponseHandler());
                return;
            }
            throw new IllegalStateException("Protocol: " + protocol + " not supported");
        }
    };
    return serverAPNHandler;
}
```

这个处理程序反过来使用它的构建器添加一个 Netty 提供的`Http2FrameCodec`和一个名为`Http2ServerResponseHandler`的定制处理程序。

我们的定制处理程序扩展了 Netty 的`ChannelDuplexHandler` ,既作为服务器的入站处理程序，也作为服务器的出站处理程序。首先，它准备发送给客户机的响应。

出于本教程的目的，**我们将在`io.netty.buffer.ByteBuf`** 中定义一个静态的`Hello World`响应，这是在 Netty 中读写字节的首选对象:

```java
static final ByteBuf RESPONSE_BYTES = Unpooled.unreleasableBuffer(
  Unpooled.copiedBuffer("Hello World", CharsetUtil.UTF_8));
```

这个缓冲区将在我们的处理程序的`channelRead`方法中被设置为一个数据帧，并被写入`ChannelHandlerContext`:

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    if (msg instanceof Http2HeadersFrame) {
        Http2HeadersFrame msgHeader = (Http2HeadersFrame) msg;
        if (msgHeader.isEndStream()) {
            ByteBuf content = ctx.alloc().buffer();
            content.writeBytes(RESPONSE_BYTES.duplicate());

            Http2Headers headers = new DefaultHttp2Headers().status(HttpResponseStatus.OK.codeAsText());
            ctx.write(new DefaultHttp2HeadersFrame(headers).stream(msgHeader.stream()));
            ctx.write(new DefaultHttp2DataFrame(content, true).stream(msgHeader.stream()));
        }
    } else {
        super.channelRead(ctx, msg);
    }
} 
```

就这样，我们的服务器准备好了`Hello World.`

为了进行快速测试，启动服务器并使用`–http2`选项发出 curl 命令:

```java
curl -k -v --http2 https://127.0.0.1:8443
```

这将给出类似于以下内容的响应:

```java
> GET / HTTP/2
> Host: 127.0.0.1:8443
> User-Agent: curl/7.64.1
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 4294967295)!
< HTTP/2 200 
< 
* Connection #0 to host 127.0.0.1 left intact
Hello World* Closing connection 0
```

## 4.客户

接下来，我们来看看客户端。当然，它的目的是发送请求，然后处理从服务器获得的响应。

我们的客户端代码将由几个处理程序组成，一个初始化器类来在管道中设置它们，最后一个 JUnit 测试来引导客户端并把所有东西放在一起。

### 4.1.`SslContext`

不过还是那句话，首先让我们看看客户端的`SslContext`是怎么设置的。我们将把它作为客户机 JUnit 设置的一部分:

```java
@Before
public void setup() throws Exception {
    SslContext sslCtx = SslContextBuilder.forClient()
      .sslProvider(SslProvider.JDK)
      .ciphers(Http2SecurityUtil.CIPHERS, SupportedCipherSuiteFilter.INSTANCE)
      .trustManager(InsecureTrustManagerFactory.INSTANCE)
      .applicationProtocolConfig(
        new ApplicationProtocolConfig(Protocol.ALPN, SelectorFailureBehavior.NO_ADVERTISE,
          SelectedListenerFailureBehavior.ACCEPT, ApplicationProtocolNames.HTTP_2))
      .build();
}
```

正如我们所见，它与服务器的`slContext`非常相似，只是我们在这里没有提供任何`SelfSignedCertificate`。另一个不同是，我们添加了一个`InsecureTrustManagerFactory`来信任任何证书，而无需任何验证。

**重要的是，该信任管理器仅用于演示目的，不应用于生产**。为了使用可信证书，Netty 的 [SslContextBuilder](https://web.archive.org/web/20221208143856/https://netty.io/4.1/api/io/netty/handler/ssl/class-use/SslContextBuilder.html) 提供了许多替代方案。

我们将在最后回到这个 JUnit 来引导客户端。

### 4.2.经理人

现在，让我们来看看处理程序。

首先，**我们需要一个名为`Http2SettingsHandler`的处理程序来处理 HTTP/2 的设置帧**。它扩展了 Netty 的`SimpleChannelInboundHandler`:

```java
public class Http2SettingsHandler extends SimpleChannelInboundHandler<Http2Settings> {
    private final ChannelPromise promise;

    // constructor

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Http2Settings msg) throws Exception {
        promise.setSuccess();
        ctx.pipeline().remove(this);
    }
}
```

该类只是初始化一个`ChannelPromise`并将其标记为成功。

它还有一个实用方法`awaitSettings`，我们的客户端将使用它来等待初始握手完成:

```java
public void awaitSettings(long timeout, TimeUnit unit) throws Exception {
    if (!promise.awaitUninterruptibly(timeout, unit)) {
        throw new IllegalStateException("Timed out waiting for settings");
    }
}
```

如果通道读取未在规定的超时时间内发生，则抛出`IllegalStateException`。

其次，**我们需要一个处理程序来处理从服务器**获得的响应，我们将其命名为`Http2ClientResponseHandler`:

```java
public class Http2ClientResponseHandler extends SimpleChannelInboundHandler {

    private final Map<Integer, MapValues> streamidMap;

    // constructor
}
```

这个类还扩展了`SimpleChannelInboundHandler`并声明了`MapValues`的`streamidMap`，它是我们`Http2ClientResponseHandler`的内部类:

```java
public static class MapValues {
    ChannelFuture writeFuture;
    ChannelPromise promise;

    // constructor and getters
} 
```

我们添加这个类是为了能够为给定的`Integer`键存储两个值。

当然，处理程序也有一个实用方法`put`，用于将值放入`streamidMap`:

```java
public MapValues put(int streamId, ChannelFuture writeFuture, ChannelPromise promise) {
    return streamidMap.put(streamId, new MapValues(writeFuture, promise));
} 
```

接下来，让我们看看当在管道中读取通道时，这个处理程序做了什么。

基本上，这是我们从服务器获取数据帧或作为`FullHttpResponse`的`ByteBuf`内容的地方，并且可以以我们想要的方式操纵它。

在本例中，我们将只记录它:

```java
@Override
protected void channelRead0(ChannelHandlerContext ctx, FullHttpResponse msg) throws Exception {
    Integer streamId = msg.headers().getInt(HttpConversionUtil.ExtensionHeaderNames.STREAM_ID.text());
    if (streamId == null) {
        logger.error("HttpResponseHandler unexpected message received: " + msg);
        return;
    }

    MapValues value = streamidMap.get(streamId);

    if (value == null) {
        logger.error("Message received for unknown stream id " + streamId);
    } else {
        ByteBuf content = msg.content();
        if (content.isReadable()) {
            int contentLength = content.readableBytes();
            byte[] arr = new byte[contentLength];
            content.readBytes(arr);
            logger.info(new String(arr, 0, contentLength, CharsetUtil.UTF_8));
        }

        value.getPromise().setSuccess();
    }
}
```

在方法结束时，我们将`ChannelPromise`标记为成功，以指示正确完成。

作为我们描述的第一个处理程序，这个类还包含一个供我们的客户端使用的实用方法。该方法让我们的事件循环一直等到`ChannelPromise`成功。或者，换句话说，它一直等到响应处理完成:

```java
public String awaitResponses(long timeout, TimeUnit unit) {
    Iterator<Entry<Integer, MapValues>> itr = streamidMap.entrySet().iterator();        
    String response = null;

    while (itr.hasNext()) {
        Entry<Integer, MapValues> entry = itr.next();
        ChannelFuture writeFuture = entry.getValue().getWriteFuture();

        if (!writeFuture.awaitUninterruptibly(timeout, unit)) {
            throw new IllegalStateException("Timed out waiting to write for stream id " + entry.getKey());
        }
        if (!writeFuture.isSuccess()) {
            throw new RuntimeException(writeFuture.cause());
        }
        ChannelPromise promise = entry.getValue().getPromise();

        if (!promise.awaitUninterruptibly(timeout, unit)) {
            throw new IllegalStateException("Timed out waiting for response on stream id "
              + entry.getKey());
        }
        if (!promise.isSuccess()) {
            throw new RuntimeException(promise.cause());
        }
        logger.info("---Stream id: " + entry.getKey() + " received---");
        response = entry.getValue().getResponse();

        itr.remove();
    }        
    return response;
}
```

### 4.3.`Http2ClientInitializer`

正如我们在服务器的例子中看到的,`ChannelInitializer`的目的是建立一个管道:

```java
public class Http2ClientInitializer extends ChannelInitializer {

    private final SslContext sslCtx;
    private final int maxContentLength;
    private Http2SettingsHandler settingsHandler;
    private Http2ClientResponseHandler responseHandler;
    private String host;
    private int port;

    // constructor

    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        settingsHandler = new Http2SettingsHandler(ch.newPromise());
        responseHandler = new Http2ClientResponseHandler();

        if (sslCtx != null) {
            ChannelPipeline pipeline = ch.pipeline();
            pipeline.addLast(sslCtx.newHandler(ch.alloc(), host, port));
            pipeline.addLast(Http2Util.getClientAPNHandler(maxContentLength, 
              settingsHandler, responseHandler));
        }
    }
    // getters
}
```

在这种情况下，我们使用新的`SslHandler`来启动管道，以便在握手过程开始时添加 [TLS SNI 扩展](https://web.archive.org/web/20221208143856/https://en.wikipedia.org/wiki/Server_Name_Indication)。

然后，`ApplicationProtocolNegotiationHandler`负责在管道中排列连接处理程序和我们的定制处理程序:

```java
public static ApplicationProtocolNegotiationHandler getClientAPNHandler(
  int maxContentLength, Http2SettingsHandler settingsHandler, Http2ClientResponseHandler responseHandler) {
    final Http2FrameLogger logger = new Http2FrameLogger(INFO, Http2ClientInitializer.class);
    final Http2Connection connection = new DefaultHttp2Connection(false);

    HttpToHttp2ConnectionHandler connectionHandler = 
      new HttpToHttp2ConnectionHandlerBuilder().frameListener(
        new DelegatingDecompressorFrameListener(connection, 
          new InboundHttp2ToHttpAdapterBuilder(connection)
            .maxContentLength(maxContentLength)
            .propagateSettings(true)
            .build()))
          .frameLogger(logger)
          .connection(connection)
          .build();

    ApplicationProtocolNegotiationHandler clientAPNHandler = 
      new ApplicationProtocolNegotiationHandler(ApplicationProtocolNames.HTTP_2) {
        @Override
        protected void configurePipeline(ChannelHandlerContext ctx, String protocol) {
            if (ApplicationProtocolNames.HTTP_2.equals(protocol)) {
                ChannelPipeline p = ctx.pipeline();
                p.addLast(connectionHandler);
                p.addLast(settingsHandler, responseHandler);
                return;
            }
            ctx.close();
            throw new IllegalStateException("Protocol: " + protocol + " not supported");
        }
    };
    return clientAPNHandler;
} 
```

现在剩下要做的就是引导客户端并发送请求。

### 4.4.引导客户端

客户端的自举在某种程度上类似于服务器的自举。之后，我们需要添加更多的功能来处理发送请求和接收响应。

如前所述，我们将把它写成一个 JUnit 测试:

```java
@Test
public void whenRequestSent_thenHelloWorldReceived() throws Exception {

    EventLoopGroup workerGroup = new NioEventLoopGroup();
    Http2ClientInitializer initializer = new Http2ClientInitializer(sslCtx, Integer.MAX_VALUE, HOST, PORT);

    try {
        Bootstrap b = new Bootstrap();
        b.group(workerGroup);
        b.channel(NioSocketChannel.class);
        b.option(ChannelOption.SO_KEEPALIVE, true);
        b.remoteAddress(HOST, PORT);
        b.handler(initializer);

        channel = b.connect().syncUninterruptibly().channel();

        logger.info("Connected to [" + HOST + ':' + PORT + ']');

        Http2SettingsHandler http2SettingsHandler = initializer.getSettingsHandler();
        http2SettingsHandler.awaitSettings(60, TimeUnit.SECONDS);

        logger.info("Sending request(s)...");

        FullHttpRequest request = Http2Util.createGetRequest(HOST, PORT);

        Http2ClientResponseHandler responseHandler = initializer.getResponseHandler();
        int streamId = 3;

        responseHandler.put(streamId, channel.write(request), channel.newPromise());
        channel.flush();

        String response = responseHandler.awaitResponses(60, TimeUnit.SECONDS);

        assertEquals("Hello World", response);

        logger.info("Finished HTTP/2 request(s)");
    } finally {
        workerGroup.shutdownGracefully();
    }
} 
```

值得注意的是，这些是我们在服务器引导方面采取的额外步骤:

*   首先，我们利用`Http2SettingsHandler`的`awaitSettings`方法等待初始握手
*   其次，我们将请求创建为一个`FullHttpRequest`
*   第三，我们将`streamId`放在我们的`Http2ClientResponseHandler`的`streamIdMap`中，并调用它的`awaitResponses`方法
*   最后，我们验证了在响应中确实获得了`Hello World`

简而言之，事情是这样的——客户端发送一个报头帧，进行初始 SSL 握手，服务器在报头和数据帧中发送响应。

## 5.结论

在本教程中，**我们看到了如何使用代码示例在 Netty 中实现 HTTP/2 服务器和客户端，以获得使用 HTTP/2 帧的`Hello World`响应。**

我们希望在未来看到 Netty API 在处理 HTTP/2 帧方面有更多的改进，因为它仍在工作中。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/server-modules/netty)