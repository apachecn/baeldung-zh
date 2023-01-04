# 阿帕奇火箭与 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-rocketmq-spring-boot>

## 1.介绍

在本教程中，我们将使用 Spring Boot 和 Apache RocketMQ(一个开源的分布式消息传递和流数据平台)创建一个消息生产者和消费者。

## 2.属国

对于 Maven 项目，我们需要添加 [RocketMQ Spring Boot 启动器](https://web.archive.org/web/20220926190008/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.rocketmq%22%20AND%20a%3A%22rocketmq-spring-boot-starter%22)依赖项:

```
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.0.4</version>
</dependency>
```

## 3.生成消息

对于我们的示例，我们将创建一个基本的消息生成器，每当用户在购物车中添加或删除商品时，它都会发送事件。

首先，让我们在`application.properties`中设置我们的服务器位置和组名:

```
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=cart-producer-group
```

注意，如果我们有不止一个名称服务器，我们可以像`host:port;host:port`一样列出它们。

现在，为了简单起见，我们将创建一个`CommandLineRunner`应用程序，并在应用程序启动期间生成几个事件:

```
@SpringBootApplication
public class CartEventProducer implements CommandLineRunner {

    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    public static void main(String[] args) {
        SpringApplication.run(CartEventProducer.class, args);
    }

    public void run(String... args) throws Exception {
        rocketMQTemplate.convertAndSend("cart-item-add-topic", new CartItemEvent("bike", 1));
        rocketMQTemplate.convertAndSend("cart-item-add-topic", new CartItemEvent("computer", 2));
        rocketMQTemplate.convertAndSend("cart-item-removed-topic", new CartItemEvent("bike", 1));
    }
}
```

`CartItemEvent`只包含两个属性——商品的 id 和数量:

```
class CartItemEvent {
    private String itemId;
    private int quantity;

    // constructor, getters and setters
}
```

在上面的例子中，我们使用`convertAndSend()`方法，一个由`AbstractMessageSendingTemplate`抽象类定义的通用方法，来发送我们的购物车事件。它有两个参数:一个目的地(在我们的例子中是主题名)和一个消息负载。

## 4.消息消费者

消费 RocketMQ 消息就像创建一个用`@RocketMQMessageListener`注释的 Spring 组件并实现`RocketMQListener`接口一样简单:

```
@SpringBootApplication
public class CartEventConsumer {

    public static void main(String[] args) {
        SpringApplication.run(CartEventConsumer.class, args);
    }

    @Service
    @RocketMQMessageListener(
      topic = "cart-item-add-topic",
      consumerGroup = "cart-consumer_cart-item-add-topic"
    )
    public class CardItemAddConsumer implements RocketMQListener<CartItemEvent> {
        public void onMessage(CartItemEvent addItemEvent) {
            log.info("Adding item: {}", addItemEvent);
            // additional logic
        }
    }

    @Service
    @RocketMQMessageListener(
      topic = "cart-item-removed-topic",
      consumerGroup = "cart-consumer_cart-item-removed-topic"
    )
    public class CardItemRemoveConsumer implements RocketMQListener<CartItemEvent> {
        public void onMessage(CartItemEvent removeItemEvent) {
            log.info("Removing item: {}", removeItemEvent);
            // additional logic
        }
    }
}
```

我们需要为我们正在监听的每个消息主题创建一个单独的组件。在每个监听器中，我们通过`@` `RocketMQMessageListener`注释定义主题名和消费者组名。

## 5.同步和异步传输

在前面的例子中，我们使用了`convertAndSend`方法来发送消息。不过，我们还有其他选择。

例如，我们可以调用与`convertAndSend`不同的`syncSend` ，因为它返回`SendResult`对象。

例如，它可用于验证我们的消息是否成功发送或获取其 id:

```
public void run(String... args) throws Exception { 
    SendResult addBikeResult = rocketMQTemplate.syncSend("cart-item-add-topic", 
      new CartItemEvent("bike", 1)); 
    SendResult addComputerResult = rocketMQTemplate.syncSend("cart-item-add-topic", 
      new CartItemEvent("computer", 2)); 
    SendResult removeBikeResult = rocketMQTemplate.syncSend("cart-item-removed-topic", 
      new CartItemEvent("bike", 1)); 
}
```

像`convertAndSend,` 一样，这个方法只在发送过程完成时返回。

在要求高可靠性的情况下，比如重要的通知消息或短信通知，我们应该使用同步传输。

另一方面，我们可能希望异步发送消息，并在发送完成时得到通知。

我们可以用`asyncSend`来做这件事，它接受一个`SendCallback` 作为参数并立即返回:

```
rocketMQTemplate.asyncSend("cart-item-add-topic", new CartItemEvent("bike", 1), new SendCallback() {
    @Override
    public void onSuccess(SendResult sendResult) {
        log.error("Successfully sent cart item");
    }

    @Override
    public void onException(Throwable throwable) {
        log.error("Exception during cart item sending", throwable);
    }
});
```

在需要高吞吐量的情况下，我们使用异步传输。

最后，对于吞吐量要求非常高的场景，我们可以使用`sendOneWay`而不是`asyncSend`。`sendOneWay `与`asyncSend `不同，它不保证消息被发送出去。

单向传输也可以用于普通的可靠性情况，比如收集日志。

## 6.在**交易中发送消息**

RocketMQ 为我们提供了在一个事务中发送消息的能力。我们可以通过使用 `sendInTransaction()`方法来实现:

```
MessageBuilder.withPayload(new CartItemEvent("bike", 1)).build();
rocketMQTemplate.sendMessageInTransaction("test-transaction", "topic-name", msg, null);
```

此外，我们必须实现一个`RocketMQLocalTransactionListener`接口:

```
@RocketMQTransactionListener(txProducerGroup="test-transaction")
class TransactionListenerImpl implements RocketMQLocalTransactionListener {
      @Override
      public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
          // ... local transaction process, return ROLLBACK, COMMIT or UNKNOWN
          return RocketMQLocalTransactionState.UNKNOWN;
      }

      @Override
      public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
          // ... check transaction status and return ROLLBACK, COMMIT or UNKNOWN
          return RocketMQLocalTransactionState.COMMIT;
      }
}
```

在`sendMessageInTransaction()`中，第一个参数是交易名称。它的 必须与`@RocketMQTransactionListener`的成员字段`txProducerGroup.` 相同

## 7.消息生产者配置

我们还可以配置消息生成器本身的各个方面:

*   `rocketmq.producer.send-message-timeout`:消息发送超时，单位为毫秒——默认值为 3000
*   `rocketmq.producer.compress-message-body-threshold`:阈值，超过该阈值，RocketMQ 将压缩消息——默认值为 1024。
*   `rocketmq.producer.max-message-size`:最大消息大小，以字节为单位，默认值为 4096。
*   `rocketmq.producer.retry-times-when-send-async-failed`:发送失败前在异步模式下内部执行的最大重试次数——默认值为 2。
*   `rocketmq.producer.retry-next-server`:表示内部发送失败时是否重试另一个代理，默认值为`false`。
*   `rocketmq.producer.retry-times-when-send-failed`:发送失败前在异步模式下内部执行的最大重试次数——默认值为 2。

## 8.结论

在本文中，我们学习了如何使用 Apache RocketMQ 和 Spring Boot 发送和使用消息。像往常一样，所有的源代码都可以在 GitHub 上获得。