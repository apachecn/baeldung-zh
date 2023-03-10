# Java NIO2 异步通道 API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio-2-async-channels>

## 1。概述

在本文中，我们将探索 Java 7- **异步通道 API**中新 I/O (NIO2)的关键附加 API 之一的基础。

这是涵盖这一特定主题的系列文章的第一篇。

异步通道 API 是对 Java 1.4 附带的早期新 I/O(NIO)API 的增强。要了解 NIO 选择器，请点击[这个链接](/web/20221013193919/https://www.baeldung.com/java-nio-selector)。

NIO APIs 的另一个增强是新的文件系统 API。你也可以在这个网站上读到更多关于它的文件操作和路径操作的信息。

为了在我们的项目中使用 NIO2 异步通道，我们必须导入 `java.nio.channels`包，因为其中捆绑了所需的类:

```java
import java.nio.channels.*;
```

## 2。异步通道 API 如何工作

简单地说，异步通道 API 被引入到现有的`java.nio.channels`包中——通过在类名前面加上单词`Asynchronous`。

一些核心类包括:`AsynchronousSocketChannel`、`AsynchronousServerSocketChannel`和`AsynchronousFileChannel`。

您可能已经注意到，这些类在风格上类似于标准的 NIO 通道 API。

而且，NIO 通道类可用的大多数 API 操作在新的异步版本中也可用。主要区别在于**新的通道使得一些操作能够异步执行**。

当操作启动时，异步通道 API 为我们提供了两种监视和控制未决操作的方法。该操作可以返回`java.util.concurrent.Future`对象，或者我们可以传递给它一个`java.nio.channels.CompletionHandler`。

## 3。`Future`接近

**`Future`对象表示异步计算的结果。**假设我们想要创建一个服务器来监听客户端连接，我们调用`AsynchronousServerSocketChannel`上的静态`open` API，并可选地将返回的套接字通道绑定到一个地址:

```java
AsynchronousServerSocketChannel server 
  = AsynchronousServerSocketChannel.open().bind(null);
```

我们已经传入了`null`以便系统可以自动分配一个地址。然后，我们在返回的服务器`SocketChannel`上调用`accept`方法:

```java
Future<AsynchronousSocketChannel> future = server.accept();
```

当我们调用旧 IO 中的`ServerSocketChannel`的`accept`方法时，它会一直阻塞，直到从客户端收到一个传入的连接。但是`AsynchronousServerSocketChannel`的`accept`方法会立即返回一个`Future`对象。

`Future`对象的泛型是操作的返回类型。在我们上面的例子中，它是`AsynchronousSocketChannel`，但也可能是`Integer`或`String`，这取决于操作的最终返回类型。

我们可以使用`Future`对象来查询操作的状态:

```java
future.isDone();
```

如果底层操作已经完成，这个 API 返回`true`。请注意，在这种情况下，完成可能意味着正常终止、异常或取消。

我们还可以明确检查操作是否已被取消:

```java
future.isCancelled();
```

如果操作在正常完成前被取消，它只返回`true`，否则返回`false`。取消通过`cancel`方法执行:

```java
future.cancel(true);
```

该调用取消了由`Future`对象表示的操作。参数指示即使操作已经开始，也可以被中断。操作一旦完成，就不能取消

为了检索计算的结果，我们使用了`get`方法:

```java
AsynchronousSocketChannel client= future.get();
```

如果我们在操作完成之前调用这个 API，它将一直阻塞到操作完成，然后返回操作的结果。

## 4。`CompletionHandler`接近

使用 Future 处理操作的替代方法是使用`CompletionHandler`类的回调机制。异步通道允许指定完成处理程序来使用操作的结果:

```java
AsynchronousServerSocketChannel listener
  = AsynchronousServerSocketChannel.open().bind(null);

listener.accept(
  attachment, new CompletionHandler<AsynchronousSocketChannel, Object>() {
    public void completed(
      AsynchronousSocketChannel client, Object attachment) {
          // do whatever with client
      }
    public void failed(Throwable exc, Object attachment) {
          // handle failure
      }
  });
```

当 I/O 操作成功完成时，调用`completed` 回调 API。如果操作失败，则调用`failed`回调。

这些回调方法接受其他参数——允许我们传入任何我们认为可能适合与操作一起标记的数据。第一个参数可以作为回调方法的第二个参数。

最后，一个清晰的场景是——对不同的异步操作使用相同的`CompletionHandler`。在这种情况下，我们将受益于标记每个操作，以便在处理结果时提供上下文，我们将在下一节中看到这一点。

## 5。结论

在本文中，我们探索了 Java NIO2 的异步通道 API 的介绍性方面。

要获得本文的所有代码片段和完整源代码，可以访问 [GitHub 项目](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)。