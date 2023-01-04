# Netty 中的异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netty-exception-handling>

## 1。概述

在这篇简短的文章中，我们将了解 Netty 中的异常处理。

简单地说，Netty 是一个用于构建高性能异步和事件驱动的网络应用程序的框架。I/O 操作在其生命周期内使用回调方法进行处理。

关于这个框架以及如何开始使用它的更多细节可以在我们之前的[文章中找到。](/web/20220628151920/https://www.baeldung.com/netty)

## 2。处理 Netty 中的异常

如前所述， **Netty 是一个事件驱动的系统，有针对特定事件的回调方法。例外也是这样的事件。**

在处理从客户端接收的数据时或在 I/O 操作期间，可能会出现异常。发生这种情况时，会触发一个专用的异常捕获事件。

### 2.1。处理始发通道中的异常

**被异常捕获的事件被触发时，由`ChannelInboundHandler`的`exceptionsCaught()`方法**或其适配器和子类处理。

请注意，回调在`ChannelHandler` 接口中已被否决。现在仅限于`ChannelInboudHandler` 接口。

该方法接受一个`Throwable` 对象和一个`ChannelHandlerContext`对象作为参数。`Throwable` 对象可用于打印堆栈跟踪或获取本地化的错误消息。

因此，让我们创建一个通道处理程序`ChannelHandlerA`，并用我们的实现覆盖它的`exceptionCaught()` :

```java
public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) 
  throws Exception {

    logger.info(cause.getLocalizedMessage());
    //do more exception handling
    ctx.close();
}
```

在上面的代码片段中，我们记录了异常消息，还调用了`ChannelHandlerContext`的`close()`。

这将关闭服务器和客户端之间的通道。实质上导致客户端断开连接并终止。

### 2.2。传播异常

在上一节中，我们处理了异常的来源渠道。然而，我们实际上可以将异常传播到管道中的另一个通道处理程序。

我们将使用`ChannelHandlerContext`对象手动触发另一个异常捕获事件，而不是记录错误消息并调用`ctx.close()`。

这将导致调用管道中下一个通道处理程序的`exceptionCaught()`。

让我们修改`ChannelHandlerA`中的代码片段，通过调用`ctx.fireExceptionCaught()`来传播事件:

```java
public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) 
  throws Exception {

    logger.info("Exception Occurred in ChannelHandler A");
    ctx.fireExceptionCaught(cause);
}
```

此外，让我们创建另一个通道处理程序`ChannelHandlerB`，并用这个实现覆盖它的`exceptionCaught()` :

```java
@Override
public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) 
  throws Exception {

    logger.info("Exception Handled in ChannelHandler B");
    logger.info(cause.getLocalizedMessage());
    //do more exception handling
    ctx.close();
}
```

在`Server`类中，通道按以下顺序添加到管道中:

```java
ch.pipeline().addLast(new ChannelHandlerA(), new ChannelHandlerB());
```

在所有异常都由一个指定的通道处理程序处理的情况下，手动传播异常捕获事件非常有用。

## 3。结论

在本教程中，我们已经了解了如何使用回调方法处理 Netty 中的异常，以及如何在需要时传播异常。

完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220628151920/https://github.com/eugenp/tutorials/tree/master/libraries-server)