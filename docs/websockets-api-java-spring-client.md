# WebSockets API 的 Java 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/websockets-api-java-spring-client>

## 1。简介

HTTP(超文本传输协议)是一种无状态的请求-响应协议。其简单的设计使其具有很强的可伸缩性，但对于高度交互的实时 web 应用程序来说不合适且效率低下，因为需要随每个请求/响应一起传输大量的开销。

由于 HTTP 是同步的，而实时应用程序需要是异步的，所以任何像轮询或长时间轮询( [Comet](https://web.archive.org/web/20221129014048/https://en.wikipedia.org/wiki/Comet_(programming)) )这样的解决方案往往都是复杂且低效的。

为了解决上述问题，我们需要一个基于标准的双向全双工协议，服务器和客户端都可以使用，这导致了 [JSR 356 API](https://web.archive.org/web/20221129014048/http://www.oracle.com/technetwork/articles/java/jsr356-1937161.html) 的引入——在本文中，我们将展示它的一个使用示例。

## 2。设置

让我们将 Spring WebSocket 依赖项包含到我们的项目中:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-websocket</artifactId>
    <version>5.2.2.RELEASE</version>
 </dependency>
 <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-messaging</artifactId>
    <version>5.2.2.RELEASE</version>
 </dependency>
```

我们总能从 Maven Central 获得最新版本的 spring-websocket 和 T2 spring-messaging 的依赖关系。

## 3。跺脚

面向流文本的消息传递协议(STOMP)是一种简单、可互操作的有线格式，它允许客户端和服务器与几乎所有的消息代理进行通信。它是 AMQP(高级消息队列协议)和 JMS (Java 消息服务)的替代方案。

STOMP 为客户机/服务器使用消息传递语义进行通信定义了一个协议。语义位于 WebSockets 之上，定义了映射到 WebSockets 框架上的框架。

使用 STOMP 给了我们用不同的编程语言开发客户机和服务器的灵活性。在当前的例子中，我们将使用 STOMP 在客户机和服务器之间传递消息。

## 4。WebSocket 服务器

你可以在这篇文章中阅读更多关于构建 WebSocket 服务器的内容。

## 5。WebSocket 客户端

为了与 WebSocket 服务器通信，客户端必须通过向服务器发送一个 HTTP 请求来启动 WebSocket 连接，该请求带有正确设置的`Upgrade`头:

```java
GET ws://websocket.example.com/ HTTP/1.1
Origin: http://example.com
Connection: Upgrade
Host: websocket.example.com
Upgrade: websocket
```

请注意，WebSocket URLs 使用 *ws* 和`wss`方案，第二个方案表示安全 WebSocket。

如果启用了 WebSockets 支持，服务器通过在响应中发送`Upgrade`头来响应。

```java
HTTP/1.1 101 WebSocket Protocol Handshake
Date: Wed, 16 Oct 2013 10:07:34 GMT
Connection: Upgrade
Upgrade: WebSocket
```

一旦这个过程(也称为 WebSocket 握手)完成，初始的 HTTP 连接就被相同 TCP/IP 连接之上的 WebSocket 连接所取代，此后双方都可以共享数据。

该客户端连接由`WebSocketStompClient` 实例发起。

### 5.1。`WebSocketStompClient`

如第 3 节所述，我们首先需要建立一个 WebSocket 连接，这是使用`WebSocketClient`类完成的。

`WebSocketClient`可通过以下方式进行配置:

*   由像 Tyrus 这样的任何 JSR-356 实现提供
*   由 Jetty 9+本地 WebSocket API 提供
*   Spring 的`WebSocketClient`的任何实现

我们将使用`StandardWebSocketClient`，在我们的例子中是`WebSocketClient`的一个实现:

```java
WebSocketClient client = new StandardWebSocketClient();

WebSocketStompClient stompClient = new WebSocketStompClient(client);
stompClient.setMessageConverter(new MappingJackson2MessageConverter());

StompSessionHandler sessionHandler = new MyStompSessionHandler();
stompClient.connect(URL, sessionHandler);

new Scanner(System.in).nextLine(); // Don't close immediately. 
```

默认情况下，`WebSocketStompClient`支持`SimpleMessageConverter`。因为我们处理的是 JSON 消息，所以我们将消息转换器设置为`MappingJackson2MessageConverter`，以便将 JSON 有效负载转换为 object。

当连接到一个端点时，我们传递一个`StompSessionHandler`的实例，它处理像`afterConnected`和`handleFrame`这样的事件。

如果我们的服务器支持 SockJs，那么我们可以修改客户端来使用 [`SockJsClient`](https://web.archive.org/web/20221129014048/https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-fallback-sockjs-client) 而不是`StandardWebSocketClient.`

### 5.2。`StompSessionHandler`

我们可以使用一个`StompSession`来订阅一个 WebSocket 主题。这可以通过创建一个`StompSessionHandlerAdapter`的实例来完成，该实例又实现了`StompSessionHandler`。

`StompSessionHandler`为 STOMP 会话提供生命周期事件。这些事件包括会话建立时的回调和失败时的通知。

一旦 WebSocket 客户端连接到端点，就会通知`StompSessionHandler`并调用`afterConnected()`方法，我们使用`StompSession`来订阅主题:

```java
@Override
public void afterConnected(
  StompSession session, StompHeaders connectedHeaders) {
    session.subscribe("/topic/messages", this);
    session.send("/app/chat", getSampleMessage());
}
@Override
public void handleFrame(StompHeaders headers, Object payload) {
    Message msg = (Message) payload;
    logger.info("Received : " + msg.getText()+ " from : " + msg.getFrom());
}
```

确保 WebSocket 服务器正在运行并且正在运行客户端，消息将显示在控制台上:

```java
INFO o.b.w.client.MyStompSessionHandler - New session established : 53b993eb-7ad6-4470-dd80-c4cfdab7f2ba
INFO o.b.w.client.MyStompSessionHandler - Subscribed to /topic/messages
INFO o.b.w.client.MyStompSessionHandler - Message sent to websocket server
INFO o.b.w.client.MyStompSessionHandler - Received : Howdy!! from : Nicky 
```

## 6。结论

在这个快速教程中，我们实现了一个基于 Spring 的 WebSocket 客户端。

完整的实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221129014048/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-client)