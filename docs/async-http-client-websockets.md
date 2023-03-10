# 具有异步客户端的 WebSockets

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/async-http-client-websockets>

## 1.介绍

AsyncHttpClient (AHC)是一个基于 Netty 的库，用于轻松执行异步 HTTP 调用和通过 WebSocket 协议进行通信。

在这个快速教程中，我们将了解如何启动 WebSocket 连接、发送数据和处理各种控制帧。

## 2.设置

这个库的最新版本可以在 [Maven Central](https://web.archive.org/web/20220523235438/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.asynchttpclient%22%20AND%20a%3A%22async-http-client%22) 上找到。我们需要小心使用组 id 为`org.asynchttpclient`的依赖项，而不是组 id 为`com.ning:`的依赖项

```java
<dependency>
    <groupId>org.asynchttpclient</groupId>
    <artifactId>async-http-client</artifactId>
    <version>2.2.0</version>
</dependency>
```

## 3.WebSocket 客户端配置

**要创建一个 WebSocket 客户端，我们首先要获得一个 HTTP 客户端，如这篇[文章](/web/20220523235438/https://www.baeldung.com/async-http-client)中所示，并升级它以支持 WebSocket 协议。**

WebSocket 协议升级的处理由`WebSocketUpgradeHandler`类完成。这个类实现了`AsyncHandler`接口，也为我们提供了一个构建器:

```java
WebSocketUpgradeHandler.Builder upgradeHandlerBuilder
  = new WebSocketUpgradeHandler.Builder();
WebSocketUpgradeHandler wsHandler = upgradeHandlerBuilder
  .addWebSocketListener(new WebSocketListener() {
      @Override
      public void onOpen(WebSocket websocket) {
          // WebSocket connection opened
      }

      @Override
      public void onClose(WebSocket websocket, int code, String reason) {
          // WebSocket connection closed
      }

      @Override
      public void onError(Throwable t) {
          // WebSocket connection error
      }
  }).build();
```

为了获得一个`WebSocket`连接对象，我们使用标准的`AsyncHttpClient` 来创建一个带有首选连接细节的 HTTP 请求，比如头、查询参数或超时:

```java
WebSocket webSocketClient = Dsl.asyncHttpClient()
  .prepareGet("ws://localhost:5590/websocket")
  .addHeader("header_name", "header_value")
  .addQueryParam("key", "value")
  .setRequestTimeout(5000)
  .execute(wsHandler)
  .get();
```

## 4.发送数据

使用`WebSocket`对象**，我们可以使用`isOpen()`方法**检查连接是否成功打开。一旦我们有了一个开放的连接**，我们就可以使用`sendTextFrame()`和`sendBinaryFrame()`方法发送带有字符串或二进制有效载荷的数据帧**:

```java
if (webSocket.isOpen()) {
    webSocket.sendTextFrame("test message");
    webSocket.sendBinaryFrame(new byte[]{'t', 'e', 's', 't'});
}
```

## 5.处理控制帧

WebSocket 协议支持三种类型的控制帧:ping、pong 和 close。

**乒乓帧主要用于实现连接的“保活”机制。**我们可以使用`sendPingFrame()`和`sendPongFrame()`方法发送这些帧:

```java
webSocket.sendPingFrame();
webSocket.sendPongFrame();
```

**关闭现有连接是通过使用`sendCloseFrame()`方法发送关闭帧**来完成的，其中我们可以以文本的形式提供状态代码和关闭连接的原因:

```java
webSocket.sendCloseFrame(404, "Forbidden");
```

## 6.结论

对 WebSocket 协议的支持，加上它提供了一种执行异步 HTTP 请求的简单方法，使 AHC 成为一个非常强大的库。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220523235438/https://github.com/eugenp/tutorials/tree/master/libraries-http)