# 《春云流》简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-stream>

## 1。概述

Spring Cloud Stream 是建立在 Spring Boot 和 Spring 集成之上的框架，**帮助创建事件驱动或消息驱动的微服务**。

在本文中，我们将通过一些简单的例子来介绍 Spring Cloud Stream 的概念和构造。

## 2。Maven 依赖关系

首先，我们需要将带有代理 RabbitMQ Maven 依赖的 [Spring Cloud Starter Stream 作为消息中间件添加到我们的`pom.xml`:](https://web.archive.org/web/20220628063223/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-stream-rabbit%22)

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    <version>3.1.3</version>
</dependency>
```

我们将添加来自 Maven Central 的[模块依赖项，以启用 JUnit 支持:](https://web.archive.org/web/20220628063223/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-stream-test-support%22)

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-test-support</artifactId>
    <version>3.1.3</version>
    <scope>test</scope>
</dependency>
```

## 3。主要概念

微服务架构遵循“[智能端点和哑管道](https://web.archive.org/web/20220628063223/https://martinfowler.com/articles/microservices.html#SmartEndpointsAndDumbPipes)原则。端点之间的通信是由 RabbitMQ 或 Apache Kafka 等消息中间件方驱动的。**服务通过这些端点或通道发布域事件进行通信**。

让我们浏览一下构成 Spring Cloud Stream 框架的概念，以及构建消息驱动服务必须了解的基本范例。

### 3.1。构造

让我们看看 Spring Cloud Stream 中的一个简单服务，它监听`input`绑定并向`output`绑定发送响应:

```java
@SpringBootApplication
@EnableBinding(Processor.class)
public class MyLoggerServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyLoggerServiceApplication.class, args);
    }

    @StreamListener(Processor.INPUT)
    @SendTo(Processor.OUTPUT)
    public LogMessage enrichLogMessage(LogMessage log) {
        return new LogMessage(String.format("[1]: %s", log.getMessage()));
    }
}
```

注释`@EnableBinding`将应用程序配置为绑定接口`Processor`中定义的通道`INPUT`和`OUTPUT`。**两个通道都是绑定，可以配置成使用具体的消息中间件或绑定器。**

让我们来看看所有这些概念的定义:

*   `Bindings` —以声明方式标识输入和输出通道的接口集合
*   `Binder` —消息中间件实现，如 Kafka 或 RabbitMQ
*   `Channel` —表示消息中间件和应用程序之间的通信管道
*   `StreamListeners`—bean 中的消息处理方法，在`MessageConverter`完成特定于中间件的事件和域对象类型/POJO 之间的序列化/反序列化之后，将自动调用来自通道的消息
*   `Mes` `sage` `Schemas` —用于消息的序列化和反序列化，这些模式可以从某个位置静态读取或动态加载，支持域对象类型的演化

### 3.2。通信模式

**指定到目的地的消息由`Publish-Subscribe`消息模式传递。**发布者将信息分类成主题，每个主题都有一个名称。订户表达对一个或多个主题的兴趣。中间件过滤消息，将感兴趣的主题传递给订阅者。

现在，用户可以被分组。一个`consumer group`是一组订户或消费者，由一个`group id`标识，在其中来自一个主题或主题分区的消息以负载平衡的方式被传递。

## 4。编程模式

本节描述构建 Spring Cloud Stream 应用程序的基础。

### 4.1。功能测试

测试支持是一个 binder 实现，它允许与通道交互并检查消息。

让我们向上面的`enrichLogMessage`服务发送一条消息，并检查响应是否在消息的开头包含文本`“[1]: “`:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = MyLoggerServiceApplication.class)
@DirtiesContext
public class MyLoggerApplicationTests {

    @Autowired
    private Processor pipe;

    @Autowired
    private MessageCollector messageCollector;

    @Test
    public void whenSendMessage_thenResponseShouldUpdateText() {
        pipe.input()
          .send(MessageBuilder.withPayload(new LogMessage("This is my message"))
          .build());

        Object payload = messageCollector.forChannel(pipe.output())
          .poll()
          .getPayload();

        assertEquals("[1]: This is my message", payload.toString());
    }
}
```

### 4.2。自定义频道

在上面的例子中，我们使用了 Spring Cloud 提供的`Processor`接口，它只有一个输入和一个输出通道。

如果我们需要不同的东西，比如一个输入和两个输出通道，我们可以创建一个定制的处理器:

```java
public interface MyProcessor {
    String INPUT = "myInput";

    @Input
    SubscribableChannel myInput();

    @Output("myOutput")
    MessageChannel anOutput();

    @Output
    MessageChannel anotherOutput();
}
```

Spring 将为我们提供这个接口的正确实现。可以使用类似`@Output(“myOutput”)`中的注释来设置通道名称。

否则，Spring 将使用方法名作为通道名。因此，我们有三个通道，称为`myInput`、`myOutput`和`anotherOutput`。

现在，假设如果值小于 10，我们希望将消息路由到一个输出，如果值大于或等于 10，则路由到另一个输出:

```java
@Autowired
private MyProcessor processor;

@StreamListener(MyProcessor.INPUT)
public void routeValues(Integer val) {
    if (val < 10) {
        processor.anOutput().send(message(val));
    } else {
        processor.anotherOutput().send(message(val));
    }
}

private static final <T> Message<T> message(T val) {
    return MessageBuilder.withPayload(val).build();
}
```

### 4.3。有条件调度

使用`@StreamListener`注释，我们还可以**使用我们用 [SpEL 表达式](/web/20220628063223/https://www.baeldung.com/spring-expression-language)定义的任何条件过滤我们期望在消费者**中得到的消息。

例如，我们可以使用条件调度作为将消息路由到不同输出的另一种方法:

```java
@Autowired
private MyProcessor processor;

@StreamListener(
  target = MyProcessor.INPUT, 
  condition = "payload < 10")
public void routeValuesToAnOutput(Integer val) {
    processor.anOutput().send(message(val));
}

@StreamListener(
  target = MyProcessor.INPUT, 
  condition = "payload >= 10")
public void routeValuesToAnotherOutput(Integer val) {
    processor.anotherOutput().send(message(val));
}
```

这种方法的唯一限制是这些方法不能返回值。

## 5。设置

让我们设置处理来自 RabbitMQ 代理的消息的应用程序。

### 5.1。活页夹配置

我们可以通过`META-INF/spring.binders`配置我们的应用程序使用默认的绑定器实现:

```java
rabbit:\
org.springframework.cloud.stream.binder.rabbit.config.RabbitMessageChannelBinderConfiguration
```

或者我们可以通过包含`[this dependency](https://web.archive.org/web/20220628063223/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-stream-binder-rabbit%22)`将 RabbitMQ 的绑定器库添加到类路径中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
    <version>1.3.0.RELEASE</version>
</dependency>
```

如果没有提供绑定器实现，Spring 将使用通道间的直接消息通信。

### 5.2 .rabbitmq 配置〔t1〕

为了配置 3.1 节中的例子来使用 RabbitMQ 绑定器，我们需要更新位于`src/main/resources`的`application.yml`:

```java
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: queue.log.messages
          binder: local_rabbit
        output:
          destination: queue.pretty.log.messages
          binder: local_rabbit
      binders:
        local_rabbit:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host>
                port: 5672
                username: <username>
                password: <password>
                virtual-host: /
```

`input`绑定将使用名为`queue.log.messages`的交换，`output`绑定将使用交换`queue.pretty.log.messages`。两种绑定都将使用名为`local_rabbit`的绑定器。

注意，我们不需要提前创建 RabbitMQ 交换或队列。运行应用程序时，**两个交换都自动创建**。

为了测试应用程序，我们可以使用 RabbitMQ 管理站点发布一条消息。在交易所`queue.log.messages`的`Publish Message`面板中，我们需要输入 JSON 格式的请求。

### 5.3。定制消息转换

Spring Cloud Stream 允许我们为特定的内容类型应用消息转换。在上面的例子中，我们希望提供纯文本，而不是使用 JSON 格式。

为此，我们将对**使用`MessageConverter`** 对`LogMessage`应用一个自定义转换:

```java
@SpringBootApplication
@EnableBinding(Processor.class)
public class MyLoggerServiceApplication {
    //...

    @Bean
    public MessageConverter providesTextPlainMessageConverter() {
        return new TextPlainMessageConverter();
    }

    //...
}
```

```java
public class TextPlainMessageConverter extends AbstractMessageConverter {

    public TextPlainMessageConverter() {
        super(new MimeType("text", "plain"));
    }

    @Override
    protected boolean supports(Class<?> clazz) {
        return (LogMessage.class == clazz);
    }

    @Override
    protected Object convertFromInternal(Message<?> message, 
        Class<?> targetClass, Object conversionHint) {
        Object payload = message.getPayload();
        String text = payload instanceof String 
          ? (String) payload 
          : new String((byte[]) payload);
        return new LogMessage(text);
    }
}
```

应用这些更改后，返回到`Publish Message`面板，如果我们将标题“`contentTypes`”设置为“`text/plain`”，将有效载荷设置为“【T3”)，它应该像以前一样工作。

### 5.4。消费群体

当运行我们的应用程序的多个实例时，**每次在一个输入通道中有一个新消息时，所有的订阅者都会得到通知**。

大多数时候，我们只需要处理一次消息。Spring Cloud Stream 通过消费群体实现这种行为。

为了实现这种行为，每个消费者绑定可以使用`spring.cloud.stream.bindings.<CHANNEL>.group`属性来指定一个组名:

```java
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: queue.log.messages
          binder: local_rabbit
          group: logMessageConsumers
          ...
```

## 6。消息驱动的微服务

在本节中，我们将介绍在微服务环境中运行 Spring Cloud Stream 应用程序所需的所有特性。

### 6.1。向上扩展

当运行多个应用程序时，确保数据在消费者之间正确划分是很重要的。为此，Spring Cloud Stream 提供了两个属性:

*   **`spring.cloud.stream.instanceCount`** —正在运行的应用数量
*   **`spring.cloud.stream.instanceIndex`** —当前应用的索引

例如，如果我们已经部署了上述`MyLoggerServiceApplication`应用程序的两个实例，那么这两个应用程序的属性`spring.cloud.stream.instanceCount` 应该是 2，属性`spring.cloud.stream.instanceIndex`应该分别是 0 和 1。

如果我们使用 Spring 数据流部署 Spring Cloud Stream 应用程序，这些属性会自动设置，如本文中的[所述。](/web/20220628063223/https://www.baeldung.com/spring-cloud-data-flow-stream-processing)

### 6.2。分区

域事件可以是`Partitioned`消息。这有助于我们**扩展存储和提高应用程序性能**。

域事件通常有一个分区键，以便它与相关消息在同一个分区中结束。

假设我们希望日志消息按照消息中的第一个字母(也就是分区键)进行分区，并分成两个分区。

对于以`A-M`开始的日志消息将有一个分区，对于`N-Z.`将有另一个分区。这可以使用两个属性进行配置:

*   `spring.cloud.stream.bindings.output.producer.partitionKeyExpression` —划分有效载荷的表达式
*   `spring.cloud.stream.bindings.output.producer.partitionCount` —组的数量

**有时候要分区的表达式太复杂，用一行就写不出来。**对于这些情况，我们可以使用属性`spring.cloud.stream.bindings.output.producer.partitionKeyExtractorClass`编写自定义的分区策略。

### 6.3。健康指示器

在微服务环境中，我们还需要**检测服务何时关闭或开始失效**。Spring Cloud Stream 提供了属性`management.health.binders.enabled`来启用绑定器的健康指示器。

运行应用程序时，我们可以查询`http://<host>:<port>/health`的健康状态。

## 7。结论

在本教程中，我们介绍了 Spring Cloud Stream 的主要概念，并通过 RabbitMQ 上的一些简单示例展示了如何使用它。更多关于春季云流的信息可以在[这里](https://web.archive.org/web/20220628063223/https://docs.spring.io/spring-cloud-stream/docs/current/reference/htmlsingle/)找到。

这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628063223/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-stream/spring-cloud-stream-rabbit)