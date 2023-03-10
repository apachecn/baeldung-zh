# RabbitMQ 中的交换、队列和绑定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rabbitmq-exchanges-queues-bindings>

## 1.概观

为了更好地理解 RabbitMQ 是如何工作的，我们需要深入了解它的核心组件。

在本文中，我们将研究交换、队列和绑定，以及如何在 Java 应用程序中以编程方式声明它们。

## 2.设置

像往常一样，我们将为 RabbitMQ 服务器使用 Java 客户端和官方客户端。

首先，让我们为 RabbitMQ 客户端添加 [Maven 依赖项:](https://web.archive.org/web/20220930164138/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.rabbitmq%22%20AND%20a%3A%22amqp-client%22)

```java
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.12.0</version>
</dependency>
```

接下来，让我们声明到 RabbitMQ 服务器的连接，并打开一个通信通道:

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();
```

另外，在 RabbitMQ 的[介绍中可以找到更详细的设置示例。](/web/20220930164138/https://www.baeldung.com/rabbitmq)

## 3 .交换

在 RabbitMQ 中，**生产者从不直接向队列**发送消息。相反，它使用交换作为路由中介。

因此，交换决定消息是进入一个队列、多个队列，还是被简单地丢弃。

例如，根据路由策略，我们有**四种交换类型可以从** : 中选择

*   direct–交换根据路由关键字将消息转发到队列
*   扇出–交换忽略路由关键字，并将消息转发到所有有界队列
*   主题–交换使用在交换上定义的模式和附加到队列的路由关键字之间的匹配，将消息路由到有界队列
*   头–在这种情况下，使用消息头属性(而不是路由关键字)将交换绑定到一个或多个队列

此外，**我们还需要申报交易所的性质** :

*   名称–交易所的名称
*   持久性–如果启用，代理将不会在重启时删除交换
*   自动删除–启用此选项时，如果交换没有绑定到队列，代理将删除它
*   可选参数

考虑到所有因素，让我们声明交换的可选参数:

```java
Map<String, Object> exchangeArguments = new HashMap<>();
exchangeArguments.put("alternate-exchange", "orders-alternate-exchange");
```

**当传递`alternate-exchange` 参数时，交换将未路由的消息重定向到另一个交换**，我们可以从参数名中猜到这一点。

接下来，**让我们声明一个启用了持久性并禁用了自动删除的直接交换**:

```java
channel.exchangeDeclare("orders-direct-exchange", BuiltinExchangeType.DIRECT, true, false, exchangeArguments);
```

## 4.行列

与其他消息传递代理类似，RabbitMQ 队列**基于 FIFO 模型**向消费者传递消息。

另外，在创建队列**时，我们可以定义队列**的几个属性:

*   名称–队列的名称。如果没有定义，代理将生成一个
*   持久性–如果启用，代理将不会在重新启动时删除队列
*   exclusive–如果启用，队列将仅由一个连接使用，并将在连接关闭时被删除
*   自动删除–如果启用，代理将在最后一个消费者退订时删除队列
*   可选参数

此外，我们将声明队列的可选参数。

让我们添加两个参数，消息 TTL 和优先级的最大数量:

```java
Map<String, Object> queueArguments = new HashMap<>();
queueArguments.put("x-message-ttl", 60000);
queueArguments.put("x-max-priority", 10);
```

现在，让我们**声明一个持久队列，禁用独占和自动删除属性**:

```java
channel.queueDeclare("orders-queue", true, false, false, queueArguments);
```

## 5.粘合剂

交换使用绑定将消息路由到特定队列。

有时，它们附带一个路由关键字，某些类型的交换使用该关键字来过滤特定的消息，并将它们路由到有界队列。

最后，让我们使用路由关键字将我们创建的队列绑定到交换:

```java
channel.queueBind("orders-queue", "orders-direct-exchange", "orders-routing-key");
```

## 6.结论

在本文中，我们讨论了 RabbitMQ 的核心组件——交换、主题和绑定。我们还了解了它们在消息传递中的作用，以及如何从 Java 应用程序中管理它们。

和往常一样，本教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220930164138/https://github.com/eugenp/tutorials/tree/master/rabbitmq)