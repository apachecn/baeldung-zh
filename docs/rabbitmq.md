# RabbitMQ 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rabbitmq>

## 1.概观

软件组件的解耦是软件设计中最重要的部分之一。实现这一点的一种方法是使用消息传递系统，它提供了组件(服务)之间的异步通信方式。在本文中，我们将介绍这样一个系统:RabbitMQ。

RabbitMQ 是一个消息代理，它实现了高级消息队列协议( [AMQP](https://web.archive.org/web/20220930021900/https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) )。它为主要的编程语言提供了客户端库。

除了用于解耦软件组件，RabbitMQ 还可用于:

*   执行后台操作
*   执行异步操作

## 2.消息传递模型

首先，让我们快速地、高层次地了解一下消息传递是如何工作的。

简单地说，有两种应用程序与消息传递系统交互:生产者和消费者。生产者是向代理发送(发布)消息的人，而消费者从代理接收消息。通常，这些程序(软件组件)运行在不同的机器上，RabbitMQ 充当它们之间的通信中间件。

在本文中，我们将讨论一个简单的例子，其中两个服务将使用 RabbitMQ 进行通信。其中一个服务将向 RabbitMQ 发布消息，另一个服务将消费。

## 3.设置

首先，让我们使用官方安装指南[在这里](https://web.archive.org/web/20220930021900/https://www.rabbitmq.com/download.html)运行 RabbitMQ。

我们自然会使用 Java 客户端与 RabbitMQ 服务器进行交互；该客户端的 [Maven 依赖关系](https://web.archive.org/web/20220930021900/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.rabbitmq%22%20AND%20a%3A%22amqp-client%22)为:

```java
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>4.0.0</version>
</dependency>
```

在使用官方指南运行 RabbitMQ 代理之后，我们需要使用 java 客户端连接到它:

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
Connection connection = factory.newConnection();
Channel channel = connection.createChannel(); 
```

我们使用`ConnectionFactory`来建立与服务器的连接，它负责协议(AMQP)和认证。这里我们连接到`localhost`上的服务器，我们可以通过使用 *setHost* 函数来修改主机名。

如果 RabbitMQ 服务器不使用默认端口，我们可以使用 *setPort* 来设置端口；RabbitMQ 的默认端口是`15672`:

```java
factory.setPort(15678);
```

我们可以设置用户名和密码:

```java
factory.setUsername("user1");
factory.setPassword("MyPassword");
```

此外，我们将使用这个连接来发布和使用消息。

## 4.生产者

**考虑一个简单的场景**，web 应用程序允许用户向网站添加新产品。任何时候有新产品加入，我们都需要给客户发电子邮件。

首先，让我们定义一个队列:

```java
channel.queueDeclare("products_queue", false, false, false, null);
```

每次用户添加新产品时，我们都会向队列发布一条消息:

```java
String message = "product details"; 
channel.basicPublish("", "products_queue", null, message.getBytes());
```

最后，我们关闭通道和连接:

```java
channel.close();
connection.close();
```

这条消息将由另一个服务使用，该服务负责向客户发送电子邮件。

## 5.消费者

看看我们能在消费端实现什么；我们将声明同一个队列:

```java
channel.queueDeclare("products_queue", false, false, false, null);
```

下面是我们如何定义将异步处理来自队列的消息的使用者:

```java
DefaultConsumer consumer = new DefaultConsumer(channel) {
    @Override
     public void handleDelivery(
        String consumerTag,
        Envelope envelope, 
        AMQP.BasicProperties properties, 
        byte[] body) throws IOException {

            String message = new String(body, "UTF-8");
            // process the message
     }
};
channel.basicConsume("products_queue", true, consumer);
```

## 6.结论

这篇简单的文章介绍了 RabbitMQ 的基本概念，并讨论了一个使用它的简单例子。

本教程的完整实现可以在 GitHub 项目中找到。