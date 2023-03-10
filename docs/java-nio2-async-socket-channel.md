# NIO2 异步套接字通道指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio2-async-socket-channel>

## 1。概述

在本文中，我们将演示如何使用 Java 7 NIO.2 通道 API 构建一个简单的服务器及其客户端。

我们将看看`AsynchronousServerSocketChannel` 和`AsynchronousSocketChannel`类，它们是分别用于实现服务器和客户端的关键类。

如果您不熟悉 NIO.2 通道 API，我们在这个站点上有一篇介绍性文章。跟着这个[链接](/web/20221013193920/https://www.baeldung.com/java-nio-2-async-channels)就可以看了。

使用 NIO.2 通道 API 所需的所有类都打包在`java.nio.channels`包中:

```java
import java.nio.channels.*;
```

## 2。服务器用`Future`

通过调用其类上的静态 open API 来创建`AsynchronousServerSocketChannel`的实例:

```java
AsynchronousServerSocketChannel server
  = AsynchronousServerSocketChannel.open();
```

新创建的异步服务器套接字通道是打开的，但尚未绑定，因此我们必须将其绑定到一个本地地址，并选择一个端口:

```java
server.bind(new InetSocketAddress("127.0.0.1", 4555));
```

我们也可以传入 null，以便它使用本地地址并绑定到任意端口:

```java
server.bind(null);
```

一旦绑定，`accept` API 用于启动接受到通道套接字的连接:

```java
Future<AsynchronousSocketChannel> acceptFuture = server.accept();
```

与异步通道操作一样，上面的调用会立即返回并继续执行。

接下来，我们可以使用`get` API 来查询来自`Future`对象的响应:

```java
AsynchronousSocketChannel worker = future.get();
```

如果需要等待来自客户端的连接请求，这个调用将被阻塞。或者，如果我们不想永远等待，我们可以指定一个超时:

```java
AsynchronousSocketChannel worker = acceptFuture.get(10, TimeUnit.SECONDS);
```

在上面的调用返回并且操作成功之后，我们可以创建一个循环，在这个循环中我们监听传入的消息并将它们回显到客户端。

让我们创建一个名为`runServer`的方法，在这个方法中我们将等待并处理任何传入的消息:

```java
public void runServer() {
    clientChannel = acceptResult.get();
    if ((clientChannel != null) && (clientChannel.isOpen())) {
        while (true) {
            ByteBuffer buffer = ByteBuffer.allocate(32);
            Future<Integer> readResult  = clientChannel.read(buffer);

            // perform other computations

            readResult.get();

            buffer.flip();
            Future<Integer> writeResult = clientChannel.write(buffer);

            // perform other computations

            writeResult.get();
            buffer.clear();
        } 
        clientChannel.close();
        serverChannel.close();
    }
}
```

在循环内部，我们所做的就是创建一个缓冲区，根据操作进行读写。

然后，每次我们进行读或写时，我们可以继续执行任何其他代码，当我们准备好处理结果时，我们调用`Future`对象上的`get()` API。

为了启动服务器，我们调用它的构造函数，然后调用`main`中的`runServer`方法:

```java
public static void main(String[] args) {
    AsyncEchoServer server = new AsyncEchoServer();
    server.runServer();
}
```

## 3。服务器用`CompletionHandler`

在这一节中，我们将看到如何使用`CompletionHandler`方法而不是`Future` 方法来实现相同的服务器。

在构造函数内部，我们创建了一个`AsynchronousServerSocketChannel`,并像以前一样将它绑定到一个本地地址:

```java
serverChannel = AsynchronousServerSocketChannel.open();
InetSocketAddress hostAddress = new InetSocketAddress("localhost", 4999);
serverChannel.bind(hostAddress);
```

接下来，仍然在构造函数中，我们创建了一个 while 循环，在这个循环中，我们接受来自客户端的任何传入连接。这个 while 循环严格用于**防止服务器在与客户端**建立连接之前退出。

为了**防止循环无休止地运行**，我们在循环结束时调用`System.in.read()`来阻止执行，直到从标准输入流中读取一个传入的连接:

```java
while (true) {
    serverChannel.accept(
      null, new CompletionHandler<AsynchronousSocketChannel,Object>() {

        @Override
        public void completed(
          AsynchronousSocketChannel result, Object attachment) {
            if (serverChannel.isOpen()){
                serverChannel.accept(null, this);
            }

            clientChannel = result;
            if ((clientChannel != null) && (clientChannel.isOpen())) {
                ReadWriteHandler handler = new ReadWriteHandler();
                ByteBuffer buffer = ByteBuffer.allocate(32);

                Map<String, Object> readInfo = new HashMap<>();
                readInfo.put("action", "read");
                readInfo.put("buffer", buffer);

                clientChannel.read(buffer, readInfo, handler);
             }
         }
         @Override
         public void failed(Throwable exc, Object attachment) {
             // process error
         }
    });
    System.in.read();
}
```

当一个连接建立后，accept 操作的`CompletionHandler`中的`completed`回调方法被调用。

它的返回类型是一个`AsynchronousSocketChannel`的实例。如果服务器套接字通道仍然打开，我们再次调用`accept` API 来准备另一个传入的连接，同时重用同一个处理程序。

接下来，我们将返回的套接字通道分配给一个全局实例。然后，在对它执行操作之前，我们检查它是否不为空并且是打开的。

我们可以开始读写操作的地方是在`accept`操作的处理程序的`completed`回调 API 内部。这一步取代了之前用`get` API 轮询通道的方法。

注意**在连接建立之后，服务器将不再退出**，除非我们明确地关闭它。

还要注意，我们创建了一个单独的内部类来处理读写操作；`ReadWriteHandler`。我们将看到附件对象在这一点上是如何派上用场的。

首先，我们来看一下`ReadWriteHandler`类:

```java
class ReadWriteHandler implements 
  CompletionHandler<Integer, Map<String, Object>> {

    @Override
    public void completed(
      Integer result, Map<String, Object> attachment) {
        Map<String, Object> actionInfo = attachment;
        String action = (String) actionInfo.get("action");

        if ("read".equals(action)) {
            ByteBuffer buffer = (ByteBuffer) actionInfo.get("buffer");
            buffer.flip();
            actionInfo.put("action", "write");

            clientChannel.write(buffer, actionInfo, this);
            buffer.clear();

        } else if ("write".equals(action)) {
            ByteBuffer buffer = ByteBuffer.allocate(32);

            actionInfo.put("action", "read");
            actionInfo.put("buffer", buffer);

            clientChannel.read(buffer, actionInfo, this);
        }
    }

    @Override
    public void failed(Throwable exc, Map<String, Object> attachment) {
        // 
    }
}
```

在`ReadWriteHandler`类中，我们附件的一般类型是一个地图。我们特别需要通过它传递两个重要的参数——操作(动作)的类型和缓冲区。

接下来，我们将了解如何使用这些参数。

我们执行的第一个操作是`read` ,因为这是一个只对客户端消息做出反应的 echo 服务器。在`ReadWriteHandler`的`completed`回调方法中，我们检索附加的数据并相应地决定做什么。

如果是一个已经完成的`read`操作，我们检索缓冲区，更改附件的动作参数，并立即执行一个`write`操作，将消息回显到客户端。

如果是一个刚刚完成的`write`操作，我们再次调用`read` API 来准备服务器接收另一个输入消息。

## 4。客户端

设置好服务器之后，我们现在可以通过调用`AsyncronousSocketChannel`类上的`open` API 来设置客户端。这个调用创建了一个客户机套接字通道的新实例，然后我们用它来连接服务器:

```java
AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
InetSocketAddress hostAddress = new InetSocketAddress("localhost", 4999)
Future<Void> future = client.connect(hostAddress);
```

`connect`操作成功时不返回任何内容。然而，我们仍然可以使用`Future`对象来监控异步操作的状态。

让我们调用`get` API 来等待连接:

```java
future.get()
```

在这一步之后，我们可以开始向服务器发送消息并接收相同的回显。`sendMessage`方法如下所示:

```java
public String sendMessage(String message) {
    byte[] byteMsg = new String(message).getBytes();
    ByteBuffer buffer = ByteBuffer.wrap(byteMsg);
    Future<Integer> writeResult = client.write(buffer);

    // do some computation

    writeResult.get();
    buffer.flip();
    Future<Integer> readResult = client.read(buffer);

    // do some computation

    readResult.get();
    String echo = new String(buffer.array()).trim();
    buffer.clear();
    return echo;
}
```

## 5。测试

为了确认我们的服务器和客户端应用程序按照预期执行，我们可以使用一个测试:

```java
@Test
public void givenServerClient_whenServerEchosMessage_thenCorrect() {
    String resp1 = client.sendMessage("hello");
    String resp2 = client.sendMessage("world");

    assertEquals("hello", resp1);
    assertEquals("world", resp2);
}
```

## 6。结论

在本文中，我们探索了 Java NIO.2 异步套接字通道 API。我们已经能够使用这些新的 API 逐步完成构建服务器和客户机的过程。

您可以在 [Github 项目](https://web.archive.org/web/20221013193920/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)中访问本文的完整源代码。