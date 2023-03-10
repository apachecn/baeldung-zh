# AMQP 之春的信息传递

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-amqp>

## 1。概述

在本教程中，我们将使用 Spring AMQP 框架探索 AMQP 上基于消息的通信。首先，我们将介绍消息传递的一些关键概念。然后，我们将继续讨论一个实际的例子。

## 2。基于消息的通信

消息传递是应用程序之间进行通信的一种技术。它依赖于异步消息传递，而不是基于同步请求响应的架构。消息的生产者和消费者通过一个称为消息代理的中间消息层来分离。消息代理提供诸如消息的持久存储、消息过滤和消息转换等功能。

在用 Java 编写的应用程序之间传递消息的情况下，通常使用 JMS (Java 消息服务)API。对于不同供应商和平台之间的互操作性，我们将无法使用 JMS 客户机和代理。这就是 AMQP 派上用场的地方。

## 3。AMQP——高级消息队列协议

AMQP 是异步消息通信的开放标准线路规范。它描述了应该如何构造消息。

### 3.1。Amqp 与 Jms 有何不同

由于 AMQP 是一个平台中立的二进制协议标准，库可以用不同的编程语言编写，并运行在不同的环境中。

不像从一个 JMS 代理迁移到另一个时那样，存在基于供应商的协议锁定。更多详情请参考 [JMS vs AMQP](https://web.archive.org/web/20221208143841/https://www.linkedin.com/pulse/jms-vs-amqp-eran-shaham) 和[了解 AMQP](https://web.archive.org/web/20221208143841/https://docs.spring.io/spring-amqp/reference/html/) 。一些广泛使用的 AMQP 经纪人是 RabbitMQ `,` OpenAMQ 和 StormMQ。

### 3.2。AMQP 实体

简言之，AMQP 由交换、队列和绑定组成:

*   就像邮局或邮箱，客户向 AMQP 交易所发布信息。有四种内置的交换类型
    *   直接交换–通过匹配完整的路由关键字将消息路由到队列
    *   扇出交换–将消息路由到与其绑定的所有队列
    *   主题交换–通过将路由关键字与模式匹配，将消息路由到多个队列
    *   头交换–根据消息头路由消息
*   使用路由关键字绑定到一个交换
*   `Messages`被发送到带有路由关键字的交换机。然后，交换机会将邮件副本分发到队列中

更多细节，请看一下 [AMQP 概念](https://web.archive.org/web/20221208143841/https://www.rabbitmq.com/tutorials/amqp-concepts.html)和[路由拓扑。](https://web.archive.org/web/20221208143841/https://spring.io/blog/2011/04/01/routing-topologies-for-performance-and-scalability-with-rabbitmq/)

### 3.3。春天的 AMQP

弹簧 AMQP 由两个模块组成:`spring-amqp`和`spring-rabbit`。这些模块共同提供了对以下内容的抽象:

*   AMQP 实体——我们用`Message, Queue, Binding, and Exchange` 类创建实体
*   连接管理——我们通过使用`CachingConnectionFactory`连接到我们的 RabbitMQ 代理
*   消息发布——我们使用一个`RabbitTemplate`来发送消息
*   消息消费——我们使用一个`@RabbitListener`从队列中读取消息

## 4。设置 Rabbitmq 代理

我们需要一个 RabbitMQ 代理可供我们连接。最简单的方法是使用 Docker 为我们获取并运行一个 RabbitMQ 映像:

```java
docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
```

我们公开端口 5672，以便我们的应用程序可以连接到 RabbitMQ。

我们公开端口 15672，这样我们就可以通过管理 UI: `[http://localhost:15672](https://web.archive.org/web/20221208143841/http://localhost:15672/)`或 HTTP API: `[http://localhost:15672/api/index.html](https://web.archive.org/web/20221208143841/http://localhost:15672/api/index.html)`看到我们的 RabbitMQ 代理正在做什么。

## 5。创建 Spring Amqp 应用程序

所以，现在让我们创建我们的应用程序来发送和接收一个简单的“Hello，world！”使用 Spring AMQP 的消息。

### 5.1。Maven 依赖关系

为了将`spring-amqp`和`spring-rabbit`模块添加到我们的项目中，我们将`spring-boot-starter-amqp`依赖项添加到我们的`pom.xml`中:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
        <version>2.2.2.RELEASE</version>
    </dependency>
</dependencies>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221208143841/https://search.maven.org/classic/#search%7Cga%7C1%7C(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-amqp%22)) 找到最新版本。

### 5.2。连接到我们的 Rabbitmq 代理

**我们将使用 Spring Boot 的自动配置来创建我们的`ConnectionFactory`、`RabbitTemplate`和`RabbitAdmin`bean**。因此，我们使用默认用户名和密码“guest”在端口 5672 上连接到我们的 RabbitMQ 代理。因此，我们只是用`@SpringBootApplication`来注释我们的应用程序:

```java
@SpringBootApplication
public class HelloWorldMessageApp {
   // ...
}
```

### 5.3。创建我们的队列

**为了创建我们的队列，我们简单地定义了一个类型为`Queue`的 bean。** `RabbitAdmin`会找到这个，并使用路由关键字“myQueue”将其绑定到默认交换机:

```java
@Bean
public Queue myQueue() {
    return new Queue("myQueue", false);
}
```

我们将队列设置为非持久的，这样当 RabbitMQ 停止时，队列和其中的任何消息都将被删除。但是，请注意，重新启动我们的应用程序不会对队列产生任何影响。

### 5.4。发送我们的消息

让我们**用`RabbitTemplate`向**发送我们的“你好，世界！”消息:

```java
rabbitTemplate.convertAndSend("myQueue", "Hello, world!");
```

### 5.5。消费我们的消息

我们将通过用`@RabbitListener`注释一个方法来实现一个消息消费者:

```java
@RabbitListener(queues = "myQueue")
public void listen(String in) {
    System.out.println("Message read from myQueue : " + in);
}
```

## 6。运行我们的应用程序

首先，我们启动 RabbitMQ 代理:

```java
docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
```

然后，我们通过运行`HelloWorldMessage.java`来运行 spring boot 应用程序，执行`main()`方法:

```java
mvn spring-boot:run -Dstart-class=com.baeldung.springamqp.simple.HelloWorldMessageApp
```

当应用程序运行时，我们将看到:

*   应用程序向默认交换发送一条消息，将“myQueue”作为路由关键字
*   然后，队列“myQueue”接收消息
*   最后，`listen`方法使用来自“myQueue”的消息，并将其打印在控制台上

我们还可以使用位于`[http://localhost:15672](https://web.archive.org/web/20221208143841/http://localhost:15672/)`的 RabbitMQ 管理页面来查看我们的消息是否已经被发送和使用。

## 7。结论

在本教程中，我们介绍了使用 Spring AMQP 在应用程序之间进行通信的基于消息传递的 AMQP 协议架构。

本教程的完整源代码和所有代码片段可以在 GitHub 项目上获得。