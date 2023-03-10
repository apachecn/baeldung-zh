# 用 Spring AMQP 调度 RabbitMQ 消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rabbitmq-spring-amqp>

## 1.介绍

在本教程中，我们将探索`fanout`的概念以及与[春天的 AMQP](https://web.archive.org/web/20220627175834/https://spring.io/projects/spring-amqp) 和 [RabbitMQ](https://web.archive.org/web/20220627175834/https://www.rabbitmq.com/) 的话题交流。

在高层次上，**扇出交换机**将**向所有绑定队列**广播相同的消息，而**主题交换机**使用路由关键字**将消息传递到特定的一个或多个绑定队列**。

对于本教程，建议事先阅读使用 Spring AMQP 的[消息。](/web/20220627175834/https://www.baeldung.com/spring-amqp)

## 2.设置扇出交换

让我们设置一个扇出交换，并绑定两个队列。当我们向此交换发送消息时，两个队列都将收到该消息。我们的扇出交换会忽略消息中包含的任何路由关键字。

**Spring AMQP 允许我们将队列、交换和绑定的所有声明聚集在一个`Declarables`对象中:**

```java
@Bean
public Declarables fanoutBindings() {
    Queue fanoutQueue1 = new Queue("fanout.queue1", false);
    Queue fanoutQueue2 = new Queue("fanout.queue2", false);
    FanoutExchange fanoutExchange = new FanoutExchange("fanout.exchange");

    return new Declarables(
      fanoutQueue1,
      fanoutQueue2,
      fanoutExchange,
      bind(fanoutQueue1).to(fanoutExchange),
      BindingBuilder.bind(fanoutQueue2).to(fanoutExchange));
}
```

## 3.建立话题交流

现在，我们还将建立一个带有两个队列的主题交换，每个队列都有不同的绑定模式:

```java
@Bean
public Declarables topicBindings() {
    Queue topicQueue1 = new Queue(topicQueue1Name, false);
    Queue topicQueue2 = new Queue(topicQueue2Name, false);

    TopicExchange topicExchange = new TopicExchange(topicExchangeName);

    return new Declarables(
      topicQueue1,
      topicQueue2,
      topicExchange,
      BindingBuilder
        .bind(topicQueue1)
        .to(topicExchange).with("*.important.*"),
      BindingBuilder
        .bind(topicQueue2)
        .to(topicExchange).with("#.error"));
}
```

主题交换允许我们用不同的键模式将队列绑定到它。这非常灵活，允许我们将多个具有相同模式的队列，甚至多个模式绑定到同一个队列。

当消息的路由关键字与模式匹配时，它将被放入队列中。**如果一个队列有多个与消息路由关键字匹配的绑定，则只有一个消息副本放在队列中。**

我们的绑定模式可以使用星号(“*”)来匹配特定位置的单词，或者使用井号(“#”)来匹配零个或多个单词。

因此，我们的`topicQueue1`将接收路由关键字为三字模式的消息，中间的字是“重要的”——例如:`“user.important.error”`或`“blog.important.notification”.`

而且，我们的`topicQueue2`将接收路由关键字以单词 error 结尾的消息；匹配的例子有`“error”`、`“user.important.error”`或`“blog.post.save.error”.`

## 4.建立一个生产者

我们将使用`RabbitTemplate`的`convertAndSend`方法来发送我们的示例消息:

```java
 String message = " payload is broadcast";
    return args -> {
        rabbitTemplate.convertAndSend(FANOUT_EXCHANGE_NAME, "", "fanout" + message);
        rabbitTemplate.convertAndSend(TOPIC_EXCHANGE_NAME, ROUTING_KEY_USER_IMPORTANT_WARN, 
            "topic important warn" + message);
        rabbitTemplate.convertAndSend(TOPIC_EXCHANGE_NAME, ROUTING_KEY_USER_IMPORTANT_ERROR, 
            "topic important error" + message);
    };
```

**`RabbitTemplate`为不同的交换类型提供了许多重载的`convertAndSend()`方法。**

当我们向扇出交换机发送消息时，路由关键字被忽略，消息被传递到所有绑定的队列。

当我们向主题交换发送消息时，我们需要传递一个路由键。基于这个路由关键字，消息将被传递到特定的队列。

## 5.配置消费者

最后，让我们设置四个消费者——每个队列一个——来接收生成的消息:

```java
 @RabbitListener(queues = {FANOUT_QUEUE_1_NAME})
    public void receiveMessageFromFanout1(String message) {
        System.out.println("Received fanout 1 message: " + message);
    }

    @RabbitListener(queues = {FANOUT_QUEUE_2_NAME})
    public void receiveMessageFromFanout2(String message) {
        System.out.println("Received fanout 2 message: " + message);
    }

    @RabbitListener(queues = {TOPIC_QUEUE_1_NAME})
    public void receiveMessageFromTopic1(String message) {
        System.out.println("Received topic 1 (" + BINDING_PATTERN_IMPORTANT + ") message: " + message);
    }

    @RabbitListener(queues = {TOPIC_QUEUE_2_NAME})
    public void receiveMessageFromTopic2(String message) {
        System.out.println("Received topic 2 (" + BINDING_PATTERN_ERROR + ") message: " + message);
    }
```

**我们使用`@RabbitListener`注释来配置消费者。**这里传递的唯一参数是队列的名称。消费者在这里不知道交换或路由键。

## 6.运行示例

我们的示例项目是一个 Spring Boot 应用程序，因此它将初始化该应用程序以及到 RabbitMQ 的连接，并设置所有队列、交换和绑定。

默认情况下，我们的应用程序期望在本地主机的端口 5672 上运行一个 RabbitMQ 实例。我们可以在`application.yaml`中修改这个和其他默认值。

我们的项目公开了 URI 上的 HTTP 端点–`/broadcast`,它接受在请求体中带有消息的帖子。

当我们用 body“Test”向这个 URI 发送请求时，我们应该在输出中看到类似这样的内容:

```java
Received fanout 1 message: fanout payload is broadcast
Received topic 1 (*.important.*) message: topic important warn payload is broadcast
Received topic 2 (#.error) message: topic important error payload is broadcast
Received fanout 2 message: fanout payload is broadcast
Received topic 1 (*.important.*) message: topic important error payload is broadcast
```

当然，我们看到这些消息的顺序是不确定的。

## 7.结论

在这个快速教程中，我们讨论了与 Spring AMQP 和 RabbitMQ 的扇出和主题交换。

本教程的完整源代码和所有代码片段可以在 [GitHub 资源库](https://web.archive.org/web/20220627175834/https://github.com/eugenp/tutorials/tree/master/spring-amqp)上找到。