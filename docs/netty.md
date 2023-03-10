# Netty 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netty>

## 1。简介

在本文中，我们将了解 Netty——一个异步事件驱动的网络应用框架。

Netty 的主要目的是构建基于 NIO(或者可能是 NIO.2)的高性能协议服务器，实现网络和业务逻辑组件的分离和松散耦合。它可能实现一个广为人知的协议，比如 HTTP，或者您自己的特定协议。

## 2。核心概念

Netty 是一个非阻塞的框架。与阻塞 IO 相比，这导致了高吞吐量。**理解非阻塞 IO 对于理解 Netty 的核心组件及其关系至关重要。**

### 2.1。频道

`Channel`是 Java NIO 的基础。它代表一个开放的连接，能够进行 IO 操作，如读和写。

### 2.2。未来

**Netty 中`Channel`上的每个 IO 操作都是非阻塞的。**

这意味着每个操作都在调用后立即返回。在标准 Java 库中有一个`Future` 接口，但它不方便用于 Netty 目的——我们只能向`Future` 询问操作的完成情况，或者阻塞当前线程，直到操作完成。

这就是为什么 **Netty 有自己的`ChannelFuture` 接口** `.` 我们可以向`ChannelFuture`传递一个回调，在操作完成时调用。

### 2.3。事件和处理程序

Netty 使用事件驱动的应用程序范例，因此数据处理的管道是通过处理程序的一系列事件。事件和处理程序可以与入站和出站数据流相关。入站事件可以是以下事件:

*   通道激活和去激活
*   读取操作事件
*   异常事件
*   用户事件

出站事件更简单，通常与打开/关闭连接和写入/刷新数据有关。

Netty 应用程序由几个网络和应用程序逻辑事件及其处理程序组成。通道事件处理程序的基本接口是`ChannelHandler` 及其后继接口`ChannelOutboundHandler` 和`ChannelInboundHandler`。

Netty 提供了一个巨大的`ChannelHandler.` 实现层次，值得注意的是这些适配器只是空的实现，例如`ChannelInboundHandlerAdapter` 和 `ChannelOutboundHandlerAdapter`。当我们只需要处理所有事件的子集时，我们可以扩展这些适配器。

还有，HTTP 等特定协议有很多实现，比如`HttpRequestDecoder, HttpResponseEncoder, HttpObjectAggregator.` 在 Netty 的 Javadoc 里熟悉一下就好了。

### 2.4。编码器和解码器

当我们使用网络协议时，我们需要执行数据序列化和反序列化。为此，Netty 为能够解码输入数据的**解码器**引入了`ChannelInboundHandler` 的特殊扩展。大多数解码器的基类是`ByteToMessageDecoder.`

为了对输出数据进行编码，Netty 对称为**编码器的`ChannelOutboundHandler` 进行了扩展。** `MessageToByteEncoder`是大多数编码器实现的基础`.`我们可以用编码器和解码器将消息从字节序列转换成 Java 对象，反之亦然。

## 3。示例服务器应用程序

让我们创建一个代表简单协议服务器的项目，该服务器接收请求、执行计算并发送响应。

### 3.1。依赖性

首先，我们需要在我们的`pom.xml`中提供 Netty 依赖:

```java
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.10.Final</version>
</dependency>
```

我们可以在 Maven Central 上找到超过[的最新版本。](https://web.archive.org/web/20220627184440/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.netty%22%20AND%20a%3A%22netty-all%22)

### 3.2。数据模型

请求数据类将具有以下结构:

```java
public class RequestData {
    private int intValue;
    private String stringValue;

    // standard getters and setters
}
```

让我们假设服务器收到请求并返回乘以 2 的`intValue`。响应将只有一个 int 值:

```java
public class ResponseData {
    private int intValue;

    // standard getters and setters
}
```

### 3.3。请求解码器

现在我们需要为我们的协议消息创建编码器和解码器。

应该注意的是， **Netty 使用套接字接收缓冲区**，它不是以队列的形式表示，而是以一组字节的形式表示。这意味着当服务器没有收到完整的消息时，可以调用我们的入站处理程序。

在处理之前，我们必须确保收到了完整的消息，有很多方法可以做到这一点。

首先，我们可以创建一个临时的`ByteBuf` ,并将所有的入站字节附加到它上面，直到我们得到所需的字节数:

```java
public class SimpleProcessingHandler 
  extends ChannelInboundHandlerAdapter {
    private ByteBuf tmp;

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        System.out.println("Handler added");
        tmp = ctx.alloc().buffer(4);
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        System.out.println("Handler removed");
        tmp.release();
        tmp = null;
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        tmp.writeBytes(m);
        m.release();
        if (tmp.readableBytes() >= 4) {
            // request processing
            RequestData requestData = new RequestData();
            requestData.setIntValue(tmp.readInt());
            ResponseData responseData = new ResponseData();
            responseData.setIntValue(requestData.getIntValue() * 2);
            ChannelFuture future = ctx.writeAndFlush(responseData);
            future.addListener(ChannelFutureListener.CLOSE);
        }
    }
}
```

上面的例子看起来有点奇怪，但有助于我们理解 Netty 是如何工作的。当相应的事件发生时，我们的处理程序的每个方法都会被调用。因此，我们在添加处理程序时初始化缓冲区，在接收新字节时用数据填充它，并在获得足够的数据时开始处理它。

我们故意不使用`stringValue` ——以这种方式解码将是不必要的复杂。这就是为什么 Netty 提供了有用的解码器类，它们是`ChannelInboundHandler` : `ByteToMessageDecoder` 和 `ReplayingDecoder.` 的实现

正如我们上面提到的，我们可以用 Netty 创建一个通道处理管道。因此，我们可以将解码器作为第一个处理程序，处理逻辑处理程序紧随其后。

RequestData 的解码器如下所示:

```java
public class RequestDecoder extends ReplayingDecoder<RequestData> {

    private final Charset charset = Charset.forName("UTF-8");

    @Override
    protected void decode(ChannelHandlerContext ctx, 
      ByteBuf in, List<Object> out) throws Exception {

        RequestData data = new RequestData();
        data.setIntValue(in.readInt());
        int strLen = in.readInt();
        data.setStringValue(
          in.readCharSequence(strLen, charset).toString());
        out.add(data);
    }
}
```

这个解码器的想法很简单。它使用了`ByteBuf` 的实现，当缓冲区中没有足够的数据用于读取操作时，它会抛出一个异常。

当捕捉到异常时，缓冲区被倒回起始处，解码器等待新的数据部分。当`decode` 执行后`out` 列表不为空时，解码停止。

### 3.4。响应编码器

除了解码之外，我们还需要对信息进行编码。这个操作更简单，因为在写操作发生时，我们有完整的消息数据。

我们可以在我们的主处理程序中向`Channel`写入数据，或者我们可以分离逻辑并创建一个扩展`MessageToByteEncoder`的处理程序，它将捕获写`ResponseData` 操作:

```java
public class ResponseDataEncoder 
  extends MessageToByteEncoder<ResponseData> {

    @Override
    protected void encode(ChannelHandlerContext ctx, 
      ResponseData msg, ByteBuf out) throws Exception {
        out.writeInt(msg.getIntValue());
    }
}
```

### 3.5。请求处理

因为我们在单独的处理程序中执行解码和编码，所以我们需要更改我们的`ProcessingHandler`:

```java
public class ProcessingHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) 
      throws Exception {

        RequestData requestData = (RequestData) msg;
        ResponseData responseData = new ResponseData();
        responseData.setIntValue(requestData.getIntValue() * 2);
        ChannelFuture future = ctx.writeAndFlush(responseData);
        future.addListener(ChannelFutureListener.CLOSE);
        System.out.println(requestData);
    }
}
```

### 3.6。服务器引导程序

现在，让我们把它们放在一起，运行我们的服务器:

```java
public class NettyServer {

    private int port;

    // constructor

    public static void main(String[] args) throws Exception {

        int port = args.length > 0
          ? Integer.parseInt(args[0]);
          : 8080;

        new NettyServer(port).run();
    }

    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) 
                  throws Exception {
                    ch.pipeline().addLast(new RequestDecoder(), 
                      new ResponseDataEncoder(), 
                      new ProcessingHandler());
                }
            }).option(ChannelOption.SO_BACKLOG, 128)
              .childOption(ChannelOption.SO_KEEPALIVE, true);

            ChannelFuture f = b.bind(port).sync();
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
}
```

在上面的服务器引导示例中使用的类的细节可以在它们的 Javadoc 中找到。最有趣的部分是这一行:

```java
ch.pipeline().addLast(
  new RequestDecoder(), 
  new ResponseDataEncoder(), 
  new ProcessingHandler());
```

这里我们定义了入站和出站处理程序，它们将按照正确的顺序处理请求和输出。

## 4。客户端应用程序

客户端应该执行反向编码和解码，所以我们需要有一个`RequestDataEncoder` 和`ResponseDataDecoder`:

```java
public class RequestDataEncoder 
  extends MessageToByteEncoder<RequestData> {

    private final Charset charset = Charset.forName("UTF-8");

    @Override
    protected void encode(ChannelHandlerContext ctx, 
      RequestData msg, ByteBuf out) throws Exception {

        out.writeInt(msg.getIntValue());
        out.writeInt(msg.getStringValue().length());
        out.writeCharSequence(msg.getStringValue(), charset);
    }
}
```

```java
public class ResponseDataDecoder 
  extends ReplayingDecoder<ResponseData> {

    @Override
    protected void decode(ChannelHandlerContext ctx, 
      ByteBuf in, List<Object> out) throws Exception {

        ResponseData data = new ResponseData();
        data.setIntValue(in.readInt());
        out.add(data);
    }
}
```

此外，我们需要定义一个`ClientHandler` ，它将发送请求并从服务器接收响应:

```java
public class ClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) 
      throws Exception {

        RequestData msg = new RequestData();
        msg.setIntValue(123);
        msg.setStringValue(
          "all work and no play makes jack a dull boy");
        ChannelFuture future = ctx.writeAndFlush(msg);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) 
      throws Exception {
        System.out.println((ResponseData)msg);
        ctx.close();
    }
}
```

现在让我们引导客户端:

```java
public class NettyClient {
    public static void main(String[] args) throws Exception {

        String host = "localhost";
        int port = 8080;
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            Bootstrap b = new Bootstrap();
            b.group(workerGroup);
            b.channel(NioSocketChannel.class);
            b.option(ChannelOption.SO_KEEPALIVE, true);
            b.handler(new ChannelInitializer<SocketChannel>() {

                @Override
                public void initChannel(SocketChannel ch) 
                  throws Exception {
                    ch.pipeline().addLast(new RequestDataEncoder(), 
                      new ResponseDataDecoder(), new ClientHandler());
                }
            });

            ChannelFuture f = b.connect(host, port).sync();

            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

正如我们所看到的，服务器引导有许多共同的细节。

现在我们可以运行客户端的 main 方法，并查看控制台输出。不出所料，我们得到了`intValue` 等于 246 的`ResponseData` 。

## 5。结论

在本文中，我们快速介绍了 Netty。我们展示了它的核心部件如`Channel` 和 `ChannelHandler`。此外，我们还为它制作了一个简单的非阻塞协议服务器和一个客户机。

和往常一样，所有的代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627184440/https://github.com/eugenp/tutorials/tree/master/libraries-server)