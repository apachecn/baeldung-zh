# 带 Netty 的 HTTP 服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-netty-http-server>

## 1.概观

在本教程中，我们将通过 Netty 在 [HTTP](https://web.archive.org/web/20220630012938/http://<a href="https//www.w3.org/Protocols/HTTP/1.1/draft-ietf-http-v11-spec-01.html">) 上**实现一个简单的大写服务器，Netty**是一个异步框架，它给了我们用 Java 开发网络应用程序的灵活性。

## 2.服务器引导

在开始之前，我们应该了解 Netty 的[基础概念，比如通道、处理器、编码器和解码器。](/web/20220630012938/https://www.baeldung.com/netty#core-concepts)

在这里，我们将直接开始引导服务器，这与简单协议服务器基本相同:

```
public class HttpServer {

    private int port;
    private static Logger logger = LoggerFactory.getLogger(HttpServer.class);

    // constructor

    // main method, same as simple protocol server

    public void run() throws Exception {
        ...
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup, workerGroup)
          .channel(NioServerSocketChannel.class)
          .handler(new LoggingHandler(LogLevel.INFO))
          .childHandler(new ChannelInitializer() {
            @Override
            protected void initChannel(SocketChannel ch) throws Exception {
                ChannelPipeline p = ch.pipeline();
                p.addLast(new HttpRequestDecoder());
                p.addLast(new HttpResponseEncoder());
                p.addLast(new CustomHttpServerHandler());
            }
          });
        ...
    }
} 
```

所以，这里**只有`childHandler`根据我们想要实现的协议**不同，对我们来说是 HTTP。

我们将向服务器的管道中添加三个处理程序:

1.  内蒂的[`HttpResponseEncoder`](https://web.archive.org/web/20220630012938/https://netty.io/4.0/api/io/netty/handler/codec/http/HttpResponseEncoder.html)——连载
2.  内蒂的[`HttpRequestDecoder`](https://web.archive.org/web/20220630012938/https://netty.io/4.0/api/io/netty/handler/codec/http/HttpRequestDecoder.html)–用于反序列化
3.  我们自己的`CustomHttpServerHandler`–用于定义我们服务器的行为

接下来让我们详细看看最后一个处理程序。

## 3.`CustomHttpServerHandler`

我们的自定义处理程序的工作是处理入站数据并发送响应。

让我们把它分解一下，以了解它的工作原理。

### 3.1.处理程序的结构

`CustomHttpServerHandler`扩展 Netty 的抽象`SimpleChannelInboundHandler`并实现其生命周期方法:

```
public class CustomHttpServerHandler extends SimpleChannelInboundHandler {
    private HttpRequest request;
    StringBuilder responseData = new StringBuilder();

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.flush();
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) {
       // implementation to follow
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

顾名思义，`channelReadComplete`在通道中的最后一条消息被使用后，刷新处理程序上下文，以便它可用于下一条传入的消息。方法`exceptionCaught`用于处理任何异常。

到目前为止，我们看到的都是样板代码。

现在让我们继续有趣的事情，实现`channelRead0`。

### 3.2.阅读频道

我们的用例很简单，服务器会简单地将请求体和查询参数(如果有的话)转换成大写。关于在响应中反映请求数据，这里需要注意一点——我们这样做只是为了演示，以理解我们如何使用 Netty 来实现 HTTP 服务器。

这里，**我们将消费消息或请求，并将其响应设置为协议** 推荐的[(注意，`RequestUtils` 是我们马上要写的内容):](https://web.archive.org/web/20220630012938/https://www.w3.org/Protocols/HTTP/1.1/draft-ietf-http-v11-spec-01.html#Response)

```
if (msg instanceof HttpRequest) {
    HttpRequest request = this.request = (HttpRequest) msg;

    if (HttpUtil.is100ContinueExpected(request)) {
        writeResponse(ctx);
    }
    responseData.setLength(0);            
    responseData.append(RequestUtils.formatParams(request));
}
responseData.append(RequestUtils.evaluateDecoderResult(request));

if (msg instanceof HttpContent) {
    HttpContent httpContent = (HttpContent) msg;
    responseData.append(RequestUtils.formatBody(httpContent));
    responseData.append(RequestUtils.evaluateDecoderResult(request));

    if (msg instanceof LastHttpContent) {
        LastHttpContent trailer = (LastHttpContent) msg;
        responseData.append(RequestUtils.prepareLastResponse(request, trailer));
        writeResponse(ctx, trailer, responseData);
    }
} 
```

正如我们所看到的，当我们的通道接收到一个`HttpRequest`时，它首先检查请求是否期望一个 [100 继续](https://web.archive.org/web/20220630012938/https://www.w3.org/Protocols/HTTP/1.1/draft-ietf-http-v11-spec-01.html#Status-Codes)状态。在这种情况下，我们立即写回一个状态为`CONTINUE`的空响应:

```
private void writeResponse(ChannelHandlerContext ctx) {
    FullHttpResponse response = new DefaultFullHttpResponse(HTTP_1_1, CONTINUE, 
      Unpooled.EMPTY_BUFFER);
    ctx.write(response);
}
```

之后，处理程序初始化一个要作为响应发送的字符串，并将请求的查询参数添加到该字符串中，然后按原样发送回去。

现在让我们定义方法`formatParams`并把它放在一个`RequestUtils`助手类中来完成:

```
StringBuilder formatParams(HttpRequest request) {
    StringBuilder responseData = new StringBuilder();
    QueryStringDecoder queryStringDecoder = new QueryStringDecoder(request.uri());
    Map<String, List<String>> params = queryStringDecoder.parameters();
    if (!params.isEmpty()) {
        for (Entry<String, List<String>> p : params.entrySet()) {
            String key = p.getKey();
            List<String> vals = p.getValue();
            for (String val : vals) {
                responseData.append("Parameter: ").append(key.toUpperCase()).append(" = ")
                  .append(val.toUpperCase()).append("\r\n");
            }
        }
        responseData.append("\r\n");
    }
    return responseData;
}
```

接下来，在接收到一个`HttpContent`，**时，我们获取请求体并将其转换为大写**:

```
StringBuilder formatBody(HttpContent httpContent) {
    StringBuilder responseData = new StringBuilder();
    ByteBuf content = httpContent.content();
    if (content.isReadable()) {
        responseData.append(content.toString(CharsetUtil.UTF_8).toUpperCase())
          .append("\r\n");
    }
    return responseData;
}
```

同样，如果接收的`HttpContent`是一个`LastHttpContent`，我们添加一个再见消息和尾部报头，如果有的话:

```
StringBuilder prepareLastResponse(HttpRequest request, LastHttpContent trailer) {
    StringBuilder responseData = new StringBuilder();
    responseData.append("Good Bye!\r\n");

    if (!trailer.trailingHeaders().isEmpty()) {
        responseData.append("\r\n");
        for (CharSequence name : trailer.trailingHeaders().names()) {
            for (CharSequence value : trailer.trailingHeaders().getAll(name)) {
                responseData.append("P.S. Trailing Header: ");
                responseData.append(name).append(" = ").append(value).append("\r\n");
            }
        }
        responseData.append("\r\n");
    }
    return responseData;
}
```

### 3.3.撰写回应

既然我们要发送的数据已经准备好了，我们可以写对`ChannelHandlerContext`的响应了:

```
private void writeResponse(ChannelHandlerContext ctx, LastHttpContent trailer,
  StringBuilder responseData) {
    boolean keepAlive = HttpUtil.isKeepAlive(request);
    FullHttpResponse httpResponse = new DefaultFullHttpResponse(HTTP_1_1, 
      ((HttpObject) trailer).decoderResult().isSuccess() ? OK : BAD_REQUEST,
      Unpooled.copiedBuffer(responseData.toString(), CharsetUtil.UTF_8));

    httpResponse.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain; charset=UTF-8");

    if (keepAlive) {
        httpResponse.headers().setInt(HttpHeaderNames.CONTENT_LENGTH, 
          httpResponse.content().readableBytes());
        httpResponse.headers().set(HttpHeaderNames.CONNECTION, 
          HttpHeaderValues.KEEP_ALIVE);
    }
    ctx.write(httpResponse);

    if (!keepAlive) {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }
}
```

在这个方法中，我们用 HTTP/1.1 版本创建了一个`FullHttpResponse`，添加了我们之前准备的数据。

如果一个请求要保持活动，或者换句话说，如果不关闭连接，我们将响应的`connection`头设置为`keep-alive`。否则，我们关闭连接。

## 4.测试服务器

为了测试我们的服务器，让我们发送一些 [cURL](/web/20220630012938/https://www.baeldung.com/curl-rest) 命令并查看响应。

当然，**我们需要通过运行这个**之前的类`HttpServer`来启动服务器。

### 4.1.获取请求

让我们首先调用服务器，提供一个带有请求的 cookie:

```
curl http://127.0.0.1:8080?param1=one
```

作为回应，我们得到:

```
Parameter: PARAM1 = ONE

Good Bye! 
```

我们也可以在任何浏览器中点击`[http://127.0.0.1:8080?param1=one](https://web.archive.org/web/20220630012938/http://127.0.0.1:8080/?param1=one)`来查看相同的结果。

### 4.2.发布请求

作为我们的第二个测试，让我们发送一个正文为`sample content`的帖子:

```
curl -d "sample content" -X POST http://127.0.0.1:8080
```

以下是回应:

```
SAMPLE CONTENT
Good Bye!
```

这一次，因为我们的请求包含了一个主体，**服务器用大写字母**将其发回。

## 5.结论

在本教程中，我们看到了如何实现 HTTP 协议，特别是使用 Netty 的 HTTP 服务器。

Netty 中的 HTTP/2 演示了 HTTP/2 协议的客户端-服务器实现。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630012938/https://github.com/eugenp/tutorials/tree/master/netty)