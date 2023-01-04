# 使用 Nats Java 客户端发布和接收消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/nats-java-client>

## 1。概述

在本教程中，我们将使用用于 NAT 的 Java 客户端连接到 T2 的 NATs 服务器并发布和接收消息。

NATS 提供了三种主要的消息交换模式。发布/订阅语义将消息传递给主题的所有订阅者。请求/回复消息通过主题发送请求，并将响应路由回请求者。

订阅者还可以在订阅主题时加入消息队列组。发送到关联主题的消息只传递给队列组中的一个订阅者。

## 2。设置

### 2.1。Maven 依赖关系

首先，我们需要将 NATS 图书馆添加到我们的`pom.xml:`

```
<dependency>
    <groupId>io.nats</groupId>
    <artifactId>jnats</artifactId>
    <version>1.0</version>
</dependency>
```

最新版本的库[可以在这里](https://web.archive.org/web/20220625082214/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jnats%22)找到，Github 项目是[在这里](https://web.archive.org/web/20220625082214/https://github.com/nats-io/java-nats)。

### 2.2。NATS 服务器

其次，我们需要一个 NATS 服务器来交换消息。这里有所有主要平台的说明。

我们假设有一个服务器运行在 localhost:4222 上。

## 3。连接和交换消息

### 3.1。连接到 NATS

**静态 NATS 类中的`connect()` 方法创建了`Connections`。**

如果我们想使用带有默认选项的连接，并在本地主机的端口 4222 上监听，我们可以使用默认方法:

```
Connection natsConnection = Nats.connect(); 
```

但是`Connections`有许多可配置的选项，其中一些我们想要覆盖。

我们将创建一个`Options`对象并将其传递给`Nats`:

```
private Connection initConnection() {
    Options options = new Options.Builder()
      .errorCb(ex -> log.error("Connection Exception: ", ex))
      .disconnectedCb(event -> log.error("Channel disconnected: {}", event.getConnection()))
      .reconnectedCb(event -> log.error("Reconnected to server: {}", event.getConnection()))
      .build();

    return Nats.connect(uri, options);
}
```

**NATS `Connections`耐用。**API 将尝试重新连接丢失的连接。

我们已经安装了回调来通知我们什么时候断开连接，什么时候连接恢复。在这个例子中，我们使用 lambdas，但是对于需要做的不仅仅是记录事件的应用程序，我们可以安装实现所需接口的对象。

我们可以做一个快速测试。创建一个连接，并添加一个 60 秒的休眠，以保持进程运行:

```
Connection natsConnection = initConnection();
Thread.sleep(60000); 
```

运行这个。然后停止并启动您的 NATS 服务器:

```
[jnats-callbacks] ERROR com.baeldung.nats.NatsClient 
  - Channel disconnected: [[email protected]](/web/20220625082214/https://www.baeldung.com/cdn-cgi/l/email-protection)
[reconnect] WARN io.nats.client.ConnectionImpl 
  - couldn't connect to nats://localhost:4222 (nats: connection read error)
[jnats-callbacks] ERROR com.baeldung.nats.NatsClient 
  - Reconnected to server: [[email protected]](/web/20220625082214/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

我们可以看到回调记录了断开和重新连接。

### 3.2。订阅消息

现在我们有了一个连接，我们可以处理消息了。

NATS `Message`是一个`bytes[]`数组的容器。除了预期的`setData(byte[])`和`byte[] getData()`方法之外，还有设置和获取消息目的地以及回复主题的方法。

我们订阅的主题是`Strings.`

NATS 支持同步和异步订阅。

让我们来看看异步订阅:

```
AsyncSubscription subscription = natsConnection
  .subscribe( topic, msg -> log.info("Received message on {}", msg.getSubject())); 
```

API 在其线程中将`Messages`传递给我们的`MessageHandler(),` 。

一些应用程序可能希望控制处理消息的线程:

```
SyncSubscription subscription = natsConnection.subscribeSync("foo.bar");
Message message = subscription.nextMessage(1000); 
```

`SyncSubscription`有一个阻塞`nextMessage()`方法，它将阻塞指定的毫秒数。我们将在测试中使用同步订阅来保持测试用例的简单性。

`AsyncSubscription` 和 `SyncSubscription`都有一个`unsubscribe()`方法，我们可以用它来关闭订阅。

```
subscription.unsubscribe(); 
```

### 3.3。发布消息

发布`Messages`可以有几种方式。

最简单的方法只需要一个主题`String`和消息`bytes`:

```
natsConnection.publish("foo.bar", "Hi there!".getBytes()); 
```

如果发布者希望得到回复或提供有关消息来源的特定信息，也可以发送包含回复主题的消息:

```
natsConnection.publish("foo.bar", "bar.foo", "Hi there!".getBytes()); 
```

还有一些其他组合的重载，比如传入一个`Message`而不是`bytes`。

### 3.4。简单的消息交换

给定一个有效的`Connection`，我们可以编写一个测试来验证消息交换:

```
SyncSubscription fooSubscription = natsConnection.subscribe("foo.bar");
SyncSubscription barSubscription = natsConnection.subscribe("bar.foo");
natsConnection.publish("foo.bar", "bar.foo", "hello there".getBytes());

Message message = fooSubscription.nextMessage();
assertNotNull("No message!", message);
assertEquals("hello there", new String(message.getData()));

natsConnection
  .publish(message.getReplyTo(), message.getSubject(), "hello back".getBytes());

message = barSubscription.nextMessage();
assertNotNull("No message!", message);
assertEquals("hello back", new String(message.getData())); 
```

我们从使用同步订阅来订阅两个主题开始，因为它们在 JUnit 测试中工作得更好。然后，我们向其中一个发送消息，将另一个指定为`replyTo`地址。

在阅读了来自第一个目的地的消息后，我们“翻转”主题来发送响应。

### 3.5。通配符订阅

NATS 服务器支持主题通配符。

通配符作用于以“.”分隔的主题标记性格。星号字符“*”匹配单个标记。大于号“>”是一个通配符，用于匹配主题的剩余部分，它可以是多个标记。

例如:

*   福。*匹配 foo.bar，foo.requests，**但不匹配`foo.bar.requests`**
*   福。>匹配 foo.bar、foo.requests、foo.bar.requests、foo.bar.baeldung 等。

让我们尝试几个测试:

```
SyncSubscription fooSubscription = client.subscribeSync("foo.*");

client.publishMessage("foo.bar", "bar.foo", "hello there");

Message message = fooSubscription.nextMessage(200);
assertNotNull("No message!", message);
assertEquals("hello there", new String(message.getData()));

client.publishMessage("foo.bar.plop", "bar.foo", "hello there");
message = fooSubscription.nextMessage(200);
assertNull("Got message!", message);

SyncSubscription barSubscription = client.subscribeSync("foo.>");

client.publishMessage("foo.bar.plop", "bar.foo", "hello there");

message = barSubscription.nextMessage(200);
assertNotNull("No message!", message);
assertEquals("hello there", new String(message.getData()));
```

## 4。请求/回复消息

我们的消息交换测试类似于发布/订阅消息系统上的常见习语；请求/答复。 **NATS 明确支持这种请求/回复消息传递**。

发布者可以使用我们上面使用的异步订阅方法安装请求处理程序:

```
AsyncSubscription subscription = natsConnection
  .subscribe("foo.bar.requests", new MessageHandler() {
    @Override
    public void onMessage(Message msg) {
        natsConnection.publish(message.getReplyTo(), reply.getBytes());
    }
}); 
```

或者他们可以在请求到达时响应请求。

API 提供了一个`request()`方法:

```
Message reply = natsConnection.request("foo.bar.requests", request.getBytes(), 100); 
```

这个方法为响应创建一个临时邮箱，并为我们写下回复地址。

`Request()`返回响应，如果请求超时，则返回`null`。最后一个参数是等待的毫秒数。

我们可以修改请求/回复的测试:

```
natsConnection.subscribe(salary.requests", message -> {
    natsConnection.publish(message.getReplyTo(), "denied!".getBytes());
});
Message reply = natsConnection.request("salary.requests", "I need a raise.", 100);
assertNotNull("No message!", reply);
assertEquals("denied!", new String(reply.getData())); 
```

## 5。消息队列

订阅方可以在订阅时指定队列组。当消息发布到群组时，NATS 会将它发送给一个且只有一个订阅者。

**队列组不保存消息。**如果没有可用的监听器，消息将被丢弃。

### 5.1。订阅队列

订户将队列组名指定为`String:`

```
SyncSubscription subscription = natsConnection.subscribe("topic", "queue name");
```

当然还有一个异步版本:

```
SyncSubscription subscription = natsConnection
  .subscribe("topic", "queue name", new MessageHandler() {
    @Override
    public void onMessage(Message msg) {
        log.info("Received message on {}", msg.getSubject());
    }
}); 
```

订阅在 NATS 服务器上创建队列。

### 5.2。发布到队列

将消息发布到队列组只需发布到相关主题:

```
natsConnection.publish("foo",  "queue message".getBytes());
```

NATS 服务器会将消息路由到队列，并选择一个消息接收者。

我们可以通过测试来验证这一点:

```
SyncSubscription queue1 = natsConnection.subscribe("foo", "queue name");
SyncSubscription queue2 = natsConnection.subscribe("foo", "queue name");

natsConnection.publish("foo", "foobar".getBytes());

List<Message> messages = new ArrayList<>();

Message message = queue1.nextMessage(200);
if (message != null) messages.add(message);

message = queue2.nextMessage(200);
if (message != null) messages.add(message);

assertEquals(1, messages.size());
```

我们只收到一条信息。

如果我们将前两行改为普通订阅:

```
SyncSubscription queue1 = natsConnection.subscribe("foo");
SyncSubscription queue2 = natsConnection.subscribe("foo"); 
```

测试失败，因为消息被传递给两个订户。

## 6。结论

在这个简短的介绍中，我们连接到一个 NATS 服务器，发送发布/订阅消息和负载平衡队列消息。我们研究了 NATS 对通配符订阅的支持。我们还使用了请求/回复消息。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220625082214/https://github.com/eugenp/tutorials/tree/master/libraries-5)