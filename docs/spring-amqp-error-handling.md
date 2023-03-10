# 用 Spring AMQP 处理错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-amqp-error-handling>

## 1.介绍

异步消息传递是一种松散耦合的分布式通信，它越来越流行于实现[事件驱动架构](/web/20221126223148/https://www.baeldung.com/cqrs-event-sourced-architecture-resources)。幸运的是， [Spring 框架](/web/20221126223148/https://www.baeldung.com/spring-intro)提供了 [Spring AMQP](/web/20221126223148/https://www.baeldung.com/spring-amqp) 项目，允许我们构建基于 AMQP 的消息传递解决方案。

另一方面，**在这样的环境中处理错误可能是一项重要的任务**。因此，在本教程中，我们将讨论处理错误的不同策略。

## 2.环境设置

对于本教程，**我们将使用实现 AMQP 标准的 [RabbitMQ](/web/20221126223148/https://www.baeldung.com/rabbitmq) 。**另外，Spring AMQP 提供了`spring-rabbit`模块，使得集成变得非常容易。

让我们将 RabbitMQ 作为一个独立的服务器来运行。我们将通过执行以下命令在 [Docker 容器](/web/20221126223148/https://www.baeldung.com/dockerizing-spring-boot-application)中运行它:

```java
docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
```

关于详细的配置和项目依赖设置，请参考我们的 [Spring AMQP 文章](/web/20221126223148/https://www.baeldung.com/spring-amqp)。

## 3.故障场景

通常，由于分布式的本质，基于消息传递的系统与整体或单一封装的应用程序相比，会出现更多类型的错误。

我们可以指出一些例外的类型:

*   `Network-` 或 `I/O-related`–网络连接和 I/O 操作的一般故障
*   `Protocol-` 或`infrastructure-related`–通常表示消息传递基础设施配置错误的错误
*   `Broker-related`–警告客户和 AMQP 经纪人之间配置不当的故障。例如，达到定义的限制或阈值、身份验证或无效的策略配置
*   `Application-` 和`message-related`–通常表示违反某些业务或应用程序规则的异常

当然，这个失败列表并不详尽，但包含了最常见的错误类型。

**我们应该注意到，Spring AMQP 开箱即用地处理与连接相关的低级问题，例如通过应用重试或重新排队策略**。此外，大多数失败和故障都被转换成一个`AmqpException`或它的一个子类。

在接下来的部分中，我们将主要关注特定于应用程序的高级错误，然后讨论全局错误处理策略。

## 4.项目设置

现在，让我们先定义一个简单的队列和交换配置:

```java
public static final String QUEUE_MESSAGES = "baeldung-messages-queue";
public static final String EXCHANGE_MESSAGES = "baeldung-messages-exchange";

@Bean
Queue messagesQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES)
      .build();
}

@Bean
DirectExchange messagesExchange() {
    return new DirectExchange(EXCHANGE_MESSAGES);
}

@Bean
Binding bindingMessages() {
    return BindingBuilder.bind(messagesQueue()).to(messagesExchange()).with(QUEUE_MESSAGES);
}
```

接下来，让我们创建一个简单的生成器:

```java
public void sendMessage() {
    rabbitTemplate
      .convertAndSend(SimpleDLQAmqpConfiguration.EXCHANGE_MESSAGES,
        SimpleDLQAmqpConfiguration.QUEUE_MESSAGES, "Some message id:" + messageNumber++);
}
```

最后，一个消费者抛出一个异常:

```java
@RabbitListener(queues = SimpleDLQAmqpConfiguration.QUEUE_MESSAGES)
public void receiveMessage(Message message) throws BusinessException {
    throw new BusinessException();
}
```

**默认情况下，所有失败的消息将立即在目标队列的头部重新排队。**

让我们通过执行下一个 Maven 命令来运行我们的示例应用程序:

```java
mvn spring-boot:run -Dstart-class=com.baeldung.springamqp.errorhandling.ErrorHandlingApp
```

现在，我们应该看到类似的结果输出:

```java
WARN 22260 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler :
  Execution of Rabbit message listener failed.
Caused by: com.baeldung.springamqp.errorhandling.errorhandler.BusinessException: null
```

因此，默认情况下，我们将在输出中看到无限多的此类消息。

要改变这种行为，我们有两个选择:

*   在监听器端将`default-requeue-rejected`选项设置为`false`—`spring.rabbitmq.listener.simple.default-requeue-rejected=false`
*   抛出一个`AmqpRejectAndDontRequeueException – t` his 可能对将来没有意义的消息有用，所以它们可以被丢弃。

现在，让我们来看看如何以更智能的方式处理失败的消息。

## 5.死信队列

**死信队列(DLQ)是保存未送达或失败消息的队列**。DLQ 允许我们处理错误或坏消息，监控故障模式，并从系统异常中恢复。

更重要的是，这有助于防止队列中出现无限循环，这些循环会不断地处理坏消息并降低系统性能。

总之，有两个主要概念:死信交换(DLX)和死信队列(DLQ)本身。事实上， **DLX 是一个普通的交易所，我们可以将其定义为常见的类型** : `direct`，`topic`或`fanout`之一。

很重要的一点是要明白，制作人对队列一无所知。它只知道交换，所有生成的消息都根据交换配置和消息路由键进行路由。

现在让我们看看如何通过应用死信队列方法来处理异常。

### 5.1.基本配置

为了配置 DLQ，我们需要在定义队列时指定额外的参数:

```java
@Bean
Queue messagesQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES)
      .withArgument("x-dead-letter-exchange", "")
      .withArgument("x-dead-letter-routing-key", QUEUE_MESSAGES_DLQ)
      .build();
}

@Bean
Queue deadLetterQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES_DLQ).build();
}
```

在上面的例子中，我们使用了两个附加参数:`x-dead-letter-exchange`和`x-dead-letter-routing-key`。**`x-dead-letter-exchange`选项的空字符串值告诉代理使用默认的交易所**。

第二个参数与为简单消息设置路由键同样重要。此选项更改消息的初始路由关键字，以便由 DLX 进一步路由。

### 5.2.失败的消息路由

因此，当消息无法传递时，它会被路由到死信交换。但是正如我们已经指出的，DLX 是一个正常的交易所。因此，如果失败的消息路由关键字与交换不匹配，它将不会被传递到 DLQ。

```java
Exchange: (AMQP default)
Routing Key: baeldung-messages-queue.dlq
```

因此，如果我们在示例中省略了`x-dead-letter-routing-key`参数，失败的消息将陷入无限的重试循环中。

此外，消息的原始元信息可在`x-death`报头中获得:

```java
x-death:
  count: 1
  exchange: baeldung-messages-exchange
  queue: baeldung-messages-queue 
  reason: rejected
  routing-keys: baeldung-messages-queue 
  time: 1571232954 
```

上面的**信息可以在 RabbitMQ 管理控制台**中获得，通常运行在本地端口 15672 上。

除了这个配置，如果我们使用 [Spring Cloud Stream](/web/20221126223148/https://www.baeldung.com/spring-cloud-stream) ，我们甚至可以通过利用配置属性`republishToDlq`和`autoBindDlq`来简化配置过程。

### 5.3.死信交换

在上一节中，我们已经看到，当消息被路由到死信交换时，路由键会发生变化。但是这种行为并不总是可取的。我们可以通过自己配置 DLX 并使用`fanout`类型定义它来改变它:

```java
public static final String DLX_EXCHANGE_MESSAGES = QUEUE_MESSAGES + ".dlx";

@Bean
Queue messagesQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES)
      .withArgument("x-dead-letter-exchange", DLX_EXCHANGE_MESSAGES)
      .build();
}

@Bean
FanoutExchange deadLetterExchange() {
    return new FanoutExchange(DLX_EXCHANGE_MESSAGES);
}

@Bean
Queue deadLetterQueue() {
    return QueueBuilder.durable(QUEUE_MESSAGES_DLQ).build();
}

@Bean
Binding deadLetterBinding() {
    return BindingBuilder.bind(deadLetterQueue()).to(deadLetterExchange());
}
```

**这次我们定义了一个`fanout`类型的自定义交换，因此消息将被发送到所有有界队列**。此外，我们已经将参数`x-dead-letter-exchange`的值设置为 DLX 的名称。与此同时，我们已经删除了`x-dead-letter-routing-key`的论点。

现在，如果我们运行我们的示例，失败的消息应该被传递到 DLQ，但是不改变初始路由键:

```java
Exchange: baeldung-messages-queue.dlx
Routing Key: baeldung-messages-queue 
```

### 5.4.处理死信队列消息

当然，我们将它们移动到死信队列的原因是，它们可以在另一个时间被重新处理。

让我们为死信队列定义一个监听器:

```java
@RabbitListener(queues = QUEUE_MESSAGES_DLQ)
public void processFailedMessages(Message message) {
    log.info("Received failed message: {}", message.toString());
}
```

如果我们现在运行我们的代码示例，我们应该会看到日志输出:

```java
WARN 11752 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler :
  Execution of Rabbit message listener failed.
INFO 11752 --- [ntContainer#1-1] c.b.s.e.consumer.SimpleDLQAmqpContainer  : 
  Received failed message:
```

我们得到了一条失败的消息，但是我们下一步该怎么办呢？答案取决于具体的系统需求、异常的种类或消息的类型。

例如，我们可以将消息重新排队到原始目的地:

```java
@RabbitListener(queues = QUEUE_MESSAGES_DLQ)
public void processFailedMessagesRequeue(Message failedMessage) {
    log.info("Received failed message, requeueing: {}", failedMessage.toString());
    rabbitTemplate.send(EXCHANGE_MESSAGES, 
      failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
}
```

但是这种异常逻辑与默认的重试策略没有什么不同:

```java
INFO 23476 --- [ntContainer#0-1] c.b.s.e.c.RoutingDLQAmqpContainer        :
  Received message: 
WARN 23476 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler :
  Execution of Rabbit message listener failed.
INFO 23476 --- [ntContainer#1-1] c.b.s.e.c.RoutingDLQAmqpContainer        : 
  Received failed message, requeueing:
```

常见的策略可能需要重试处理消息`n`次，然后拒绝它。让我们通过利用消息头来实现这一策略:

```java
public void processFailedMessagesRetryHeaders(Message failedMessage) {
    Integer retriesCnt = (Integer) failedMessage.getMessageProperties()
      .getHeaders().get(HEADER_X_RETRIES_COUNT);
    if (retriesCnt == null) retriesCnt = 1;
    if (retriesCnt > MAX_RETRIES_COUNT) {
        log.info("Discarding message");
        return;
    }
    log.info("Retrying message for the {} time", retriesCnt);
    failedMessage.getMessageProperties()
      .getHeaders().put(HEADER_X_RETRIES_COUNT, ++retriesCnt);
    rabbitTemplate.send(EXCHANGE_MESSAGES, 
      failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
}
```

首先，我们获取`x-retries-count`头的值，然后将这个值与最大允许值进行比较。随后，如果计数器达到尝试次数限制，消息将被丢弃:

```java
WARN 1224 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : 
  Execution of Rabbit message listener failed.
INFO 1224 --- [ntContainer#1-1] c.b.s.e.consumer.DLQCustomAmqpContainer  : 
  Retrying message for the 1 time
WARN 1224 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : 
  Execution of Rabbit message listener failed.
INFO 1224 --- [ntContainer#1-1] c.b.s.e.consumer.DLQCustomAmqpContainer  : 
  Retrying message for the 2 time
WARN 1224 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : 
  Execution of Rabbit message listener failed.
INFO 1224 --- [ntContainer#1-1] c.b.s.e.consumer.DLQCustomAmqpContainer  : 
  Discarding message
```

我们应该补充的是，我们还可以利用`x-message-ttl`头来设置一个时间，在这个时间之后消息应该被丢弃。这可能有助于防止队列无限增长。

### 5.5.停车场排队

另一方面，考虑这样一种情况，当我们不能丢弃一个消息时，例如，它可能是银行领域中的一个事务。或者，有时消息可能需要手动处理，或者我们只需要记录失败超过`n`次的消息。

对于这种情况，有一个停车场排队的概念。我们可以**将来自 DLQ 的失败次数超过允许次数的所有消息转发到停车场队列，以便进一步处理**。

现在让我们来实现这个想法:

```java
public static final String QUEUE_PARKING_LOT = QUEUE_MESSAGES + ".parking-lot";
public static final String EXCHANGE_PARKING_LOT = QUEUE_MESSAGES + "exchange.parking-lot";

@Bean
FanoutExchange parkingLotExchange() {
    return new FanoutExchange(EXCHANGE_PARKING_LOT);
}

@Bean
Queue parkingLotQueue() {
    return QueueBuilder.durable(QUEUE_PARKING_LOT).build();
}

@Bean
Binding parkingLotBinding() {
    return BindingBuilder.bind(parkingLotQueue()).to(parkingLotExchange());
}
```

其次，让我们重构侦听器逻辑，向停车场队列发送一条消息:

```java
@RabbitListener(queues = QUEUE_MESSAGES_DLQ)
public void processFailedMessagesRetryWithParkingLot(Message failedMessage) {
    Integer retriesCnt = (Integer) failedMessage.getMessageProperties()
      .getHeaders().get(HEADER_X_RETRIES_COUNT);
    if (retriesCnt == null) retriesCnt = 1;
    if (retriesCnt > MAX_RETRIES_COUNT) {
        log.info("Sending message to the parking lot queue");
        rabbitTemplate.send(EXCHANGE_PARKING_LOT, 
          failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
        return;
    }
    log.info("Retrying message for the {} time", retriesCnt);
    failedMessage.getMessageProperties()
      .getHeaders().put(HEADER_X_RETRIES_COUNT, ++retriesCnt);
    rabbitTemplate.send(EXCHANGE_MESSAGES, 
      failedMessage.getMessageProperties().getReceivedRoutingKey(), failedMessage);
}
```

最终，我们还需要处理到达停车场队列的消息:

```java
@RabbitListener(queues = QUEUE_PARKING_LOT)
public void processParkingLotQueue(Message failedMessage) {
    log.info("Received message in parking lot queue");
    // Save to DB or send a notification.
}
```

现在，我们可以将失败的消息保存到数据库中，或者发送电子邮件通知。

让我们通过运行我们的应用程序来测试这个逻辑:

```java
WARN 14768 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : 
  Execution of Rabbit message listener failed.
INFO 14768 --- [ntContainer#1-1] c.b.s.e.c.ParkingLotDLQAmqpContainer     : 
  Retrying message for the 1 time
WARN 14768 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : 
  Execution of Rabbit message listener failed.
INFO 14768 --- [ntContainer#1-1] c.b.s.e.c.ParkingLotDLQAmqpContainer     : 
  Retrying message for the 2 time
WARN 14768 --- [ntContainer#0-1] s.a.r.l.ConditionalRejectingErrorHandler : 
  Execution of Rabbit message listener failed.
INFO 14768 --- [ntContainer#1-1] c.b.s.e.c.ParkingLotDLQAmqpContainer     : 
  Sending message to the parking lot queue
INFO 14768 --- [ntContainer#2-1] c.b.s.e.c.ParkingLotDLQAmqpContainer     : 
  Received message in parking lot queue
```

从输出中我们可以看到，在几次尝试失败后，消息被发送到停车场队列。

## 6.自定义错误处理

在上一节中，我们已经看到了如何用专用队列和交换来处理故障。然而，有时我们可能需要捕捉所有错误，例如将它们记录或保存到数据库中。

### 6.1.全球`ErrorHandler`

到目前为止，我们一直使用默认的`SimpleRabbitListenerContainerFactory`，这个工厂默认使用`ConditionalRejectingErrorHandler`。这个处理程序捕捉不同的异常，并将它们转换成`AmqpException`层次结构中的一个异常。

值得一提的是，如果我们需要处理连接错误，那么我们需要实现`ApplicationListener`接口。

简单来说， **`ConditionalRejectingErrorHandler`决定是否拒绝某个特定的消息。**当导致异常的消息被拒绝时，它将不会被重新排队。

让我们定义一个自定义的`ErrorHandler`,它将只对`BusinessException`进行重新排队:

```java
public class CustomErrorHandler implements ErrorHandler {
    @Override
    public void handleError(Throwable t) {
        if (!(t.getCause() instanceof BusinessException)) {
            throw new AmqpRejectAndDontRequeueException("Error Handler converted exception to fatal", t);
        }
    }
}
```

此外，当我们在侦听器方法中抛出异常时，它被包装在一个`ListenerExecutionFailedException`中。因此，我们需要调用`getCause`方法来获取一个源异常。

### 6.2.`FatalExceptionStrategy`

在幕后，这个处理程序使用`FatalExceptionStrategy`来检查一个异常是否应该被认为是致命的。如果是，失败的消息将被拒绝。

默认情况下，这些异常是致命的:

*   `MessageConversionException`
*   `MessageConversionException`
*   `MethodArgumentNotValidException`
*   `MethodArgumentTypeMismatchException`
*   `NoSuchMethodException`
*   `ClassCastException`

**不用实现`ErrorHandler`接口，我们只需提供我们的`FatalExceptionStrategy`** :

```java
public class CustomFatalExceptionStrategy 
      extends ConditionalRejectingErrorHandler.DefaultExceptionStrategy {
    @Override
    public boolean isFatal(Throwable t) {
        return !(t.getCause() instanceof BusinessException);
    }
}
```

最后，我们需要将我们的定制策略传递给`ConditionalRejectingErrorHandler`构造函数:

```java
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
  ConnectionFactory connectionFactory,
  SimpleRabbitListenerContainerFactoryConfigurer configurer) {
      SimpleRabbitListenerContainerFactory factory = 
        new SimpleRabbitListenerContainerFactory();
      configurer.configure(factory, connectionFactory);
      factory.setErrorHandler(errorHandler());
      return factory;
}

@Bean
public ErrorHandler errorHandler() {
    return new ConditionalRejectingErrorHandler(customExceptionStrategy());
}

@Bean
FatalExceptionStrategy customExceptionStrategy() {
    return new CustomFatalExceptionStrategy();
}
```

## 7.结论

在本教程中，我们讨论了使用 Spring AMQP 时处理错误的不同方式，特别是 RabbitMQ。

每个系统都需要特定的错误处理策略。我们已经讨论了事件驱动架构中最常见的错误处理方式。此外，我们已经看到，我们可以结合多种策略来构建更全面、更强大的解决方案。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221126223148/https://github.com/eugenp/tutorials/tree/master/messaging-modules/spring-amqp)