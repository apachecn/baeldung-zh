# 阿帕奇卡夫卡与春天简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-kafka>

## 1。概述

[Apache Kafka](https://web.archive.org/web/20220712173816/https://kafka.apache.org/) 是一个分布式容错流处理系统。

在本教程中，我们将介绍 Spring 对 Kafka 的支持，以及它在原生 Kafka Java 客户端 API 上提供的抽象级别。

Spring Kafka 带来了简单而典型的 Spring 模板编程模型，带有一个`KafkaTemplate`和通过`@KafkaListener`注释的消息驱动的 POJOs。

## 延伸阅读:

## [用 Flink 和 Kafka 搭建数据管道](/web/20220712173816/https://www.baeldung.com/kafka-flink-data-pipeline)

Learn how to process stream data with Flink and Kafka[Read more](/web/20220712173816/https://www.baeldung.com/kafka-flink-data-pipeline) →

## [Kafka 与 MQTT 和 MongoDB 的连接示例](/web/20220712173816/https://www.baeldung.com/kafka-connect-mqtt-mongodb)

Have a look at a practical example using Kafka connectors.[Read more](/web/20220712173816/https://www.baeldung.com/kafka-connect-mqtt-mongodb) →

## 2。安装和设置

下载安装 Kafka 请参考官方指南[这里](https://web.archive.org/web/20220712173816/https://kafka.apache.org/quickstart)。

我们还需要将`spring-kafka`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.7.2</version>
</dependency>
```

这个产品的最新版本可以在[这里](https://web.archive.org/web/20220712173816/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.kafka%22%20AND%20a%3A%22spring-kafka%22)找到。

我们的示例应用程序将是一个 Spring Boot 应用程序。

本文假设服务器使用默认配置启动，并且没有更改服务器端口。

## 3.配置主题

以前，我们在 Kafka 中运行命令行工具来创建主题:

```java
$ bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic mytopic
```

但是随着 Kafka 中`AdminClient`的引入，我们现在可以通过编程来创建主题。

**我们需要添加`KafkaAdmin` Spring bean，它将自动为所有类型为`NewTopic`** 的 bean 添加主题:

```java
@Configuration
public class KafkaTopicConfig {

    @Value(value = "${kafka.bootstrapAddress}")
    private String bootstrapAddress;

    @Bean
    public KafkaAdmin kafkaAdmin() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        return new KafkaAdmin(configs);
    }

    @Bean
    public NewTopic topic1() {
         return new NewTopic("baeldung", 1, (short) 1);
    }
}
```

## 4。生成消息

要创建消息，我们首先需要配置一个 [`ProducerFactory`](https://web.archive.org/web/20220712173816/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/core/ProducerFactory.html) 。这设定了创建卡夫卡 [`Producer`](https://web.archive.org/web/20220712173816/https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/producer/Producer.html) 实例的策略。

**然后我们需要一个 [`KafkaTemplate`](https://web.archive.org/web/20220712173816/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/core/KafkaTemplate.html) ，它包装了一个`Producer`实例，并提供了向卡夫卡主题发送消息的便利方法。**

实例是线程安全的。因此，在整个应用程序上下文中使用单个实例会带来更高的性能。因此，`KakfaTemplate`实例也是线程安全的，建议使用一个实例。

### 4.1。生产者配置

```java
@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(
          ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, 
          bootstrapAddress);
        configProps.put(
          ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
          StringSerializer.class);
        configProps.put(
          ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
          StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

### 4.2。发布消息

我们可以使用`KafkaTemplate`类发送消息:

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public void sendMessage(String msg) {
    kafkaTemplate.send(topicName, msg);
}
```

**`send`API 返回一个`ListenableFuture`对象。**如果我们想阻塞发送线程并得到关于发送消息的结果，我们可以调用`ListenableFuture`对象的`get` API。线程将等待结果，但这会降低生成器的速度。

Kafka 是一个快速的流处理平台。因此，最好异步处理结果，这样后续消息就不会等待前一条消息的结果。

我们可以通过回调来做到这一点:

```java
public void sendMessage(String message) {

    ListenableFuture<SendResult<String, String>> future = 
      kafkaTemplate.send(topicName, message);

    future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {

        @Override
        public void onSuccess(SendResult<String, String> result) {
            System.out.println("Sent message=[" + message + 
              "] with offset=[" + result.getRecordMetadata().offset() + "]");
        }
        @Override
        public void onFailure(Throwable ex) {
            System.out.println("Unable to send message=[" 
              + message + "] due to : " + ex.getMessage());
        }
    });
}
```

## 5。消费信息

### 5.1。消费者配置

对于消费消息，我们需要配置一个`[ConsumerFactory](https://web.archive.org/web/20220712173816/https://docs.spring.io/autorepo/docs/spring-kafka-dist/1.1.3.RELEASE/api/org/springframework/kafka/core/ConsumerFactory.html)`和一个 [`KafkaListenerContainerFactory`](https://web.archive.org/web/20220712173816/https://docs.spring.io/autorepo/docs/spring-kafka-dist/1.1.3.RELEASE/api/org/springframework/kafka/config/KafkaListenerContainerFactory.html) 。一旦这些 bean 在 Spring bean 工厂中可用，就可以使用 [`@KafkaListener`](https://web.archive.org/web/20220712173816/https://docs.spring.io/autorepo/docs/spring-kafka-dist/1.1.3.RELEASE/api/org/springframework/kafka/annotation/KafkaListener.html) 注释来配置基于 POJO 的消费者。

配置类上需要 **[`@EnableKafka`](https://web.archive.org/web/20220712173816/https://docs.spring.io/autorepo/docs/spring-kafka-dist/1.1.3.RELEASE/api/org/springframework/kafka/annotation/EnableKafka.html) 注释，以便能够检测 spring 管理的 beans 上的`@KafkaListener`注释**:

```java
@EnableKafka
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(
          ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, 
          bootstrapAddress);
        props.put(
          ConsumerConfig.GROUP_ID_CONFIG, 
          groupId);
        props.put(
          ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
          StringDeserializer.class);
        props.put(
          ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
          StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> 
      kafkaListenerContainerFactory() {

        ConcurrentKafkaListenerContainerFactory<String, String> factory =
          new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

### 5.2。消费信息

```java
@KafkaListener(topics = "topicName", groupId = "foo")
public void listenGroupFoo(String message) {
    System.out.println("Received Message in group foo: " + message);
}
```

我们可以为一个主题实现多个监听器，每个监听器有一个不同的组 Id。此外，消费者可以收听各种主题的消息:

```java
@KafkaListener(topics = "topic1, topic2", groupId = "foo")
```

Spring 还支持使用监听器中的 [`@Header`](https://web.archive.org/web/20220712173816/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/handler/annotation/Header.html) 注释来检索一个或多个消息头:

```java
@KafkaListener(topics = "topicName")
public void listenWithHeaders(
  @Payload String message, 
  @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
      System.out.println(
        "Received Message: " + message"
        + "from partition: " + partition);
}
```

### 5.3。消费来自特定分区的消息

注意，我们创建的主题`baeldung`只有一个分区。

然而，对于具有多个分区的主题，`@KafkaListener`可以显式地订阅具有初始偏移的主题的特定分区:

```java
@KafkaListener(
  topicPartitions = @TopicPartition(topic = "topicName",
  partitionOffsets = {
    @PartitionOffset(partition = "0", initialOffset = "0"), 
    @PartitionOffset(partition = "3", initialOffset = "0")}),
  containerFactory = "partitionsKafkaListenerContainerFactory")
public void listenToPartition(
  @Payload String message, 
  @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
      System.out.println(
        "Received Message: " + message"
        + "from partition: " + partition);
}
```

由于这个监听器中的`initialOffset`已经被设置为 0，所以每次初始化这个监听器时，来自分区 0 和 3 的所有先前使用的消息都将被重新使用。

如果我们不需要设置偏移量，我们可以使用`@TopicPartition`注释的`partitions`属性只设置没有偏移量的分区:

```java
@KafkaListener(topicPartitions 
  = @TopicPartition(topic = "topicName", partitions = { "0", "1" }))
```

### 5.4。为监听器添加消息过滤器

我们可以通过添加自定义过滤器来配置侦听器，以使用特定类型的消息。这可以通过将`[RecordFilterStrategy](https://web.archive.org/web/20220712173816/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/listener/adapter/RecordFilterStrategy.html)`设置为`KafkaListenerContainerFactory`来实现:

```java
@Bean
public ConcurrentKafkaListenerContainerFactory<String, String>
  filterKafkaListenerContainerFactory() {

    ConcurrentKafkaListenerContainerFactory<String, String> factory =
      new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setRecordFilterStrategy(
      record -> record.value().contains("World"));
    return factory;
}
```

然后，我们可以配置一个侦听器来使用这个容器工厂:

```java
@KafkaListener(
  topics = "topicName", 
  containerFactory = "filterKafkaListenerContainerFactory")
public void listenWithFilter(String message) {
    System.out.println("Received Message in filtered listener: " + message);
}
```

在这个监听器中，所有与过滤器匹配的**消息都将被丢弃。**

## 6。定制消息转换器

到目前为止，我们只讨论了将字符串作为消息发送和接收。然而，我们也可以发送和接收定制的 Java 对象。这需要在`ProducerFactory`中配置适当的串行化器，在`ConsumerFactory`中配置去串行化器。

让我们看一个简单的 bean 类`,`，我们将把它作为消息发送:

```java
public class Greeting {

    private String msg;
    private String name;

    // standard getters, setters and constructor
}
```

### 6.1。生成自定义消息

在这个例子中，我们将使用`[JsonSerializer](https://web.archive.org/web/20220712173816/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/support/serializer/JsonSerializer.html)`。

让我们看看`ProducerFactory`和 `KafkaTemplate`的代码:

```java
@Bean
public ProducerFactory<String, Greeting> greetingProducerFactory() {
    // ...
    configProps.put(
      ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
      JsonSerializer.class);
    return new DefaultKafkaProducerFactory<>(configProps);
}

@Bean
public KafkaTemplate<String, Greeting> greetingKafkaTemplate() {
    return new KafkaTemplate<>(greetingProducerFactory());
}
```

我们可以使用这个新的`KafkaTemplate`来发送`Greeting`消息:

```java
kafkaTemplate.send(topicName, new Greeting("Hello", "World"));
```

### 6.2。消费自定义消息

类似地，让我们修改`ConsumerFactory`和`KafkaListenerContainerFactory`来正确地反序列化问候消息:

```java
@Bean
public ConsumerFactory<String, Greeting> greetingConsumerFactory() {
    // ...
    return new DefaultKafkaConsumerFactory<>(
      props,
      new StringDeserializer(), 
      new JsonDeserializer<>(Greeting.class));
}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, Greeting> 
  greetingKafkaListenerContainerFactory() {

    ConcurrentKafkaListenerContainerFactory<String, Greeting> factory =
      new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(greetingConsumerFactory());
    return factory;
}
```

spring-kafka JSON 序列化器和反序列化器使用 [Jackson](/web/20220712173816/https://www.baeldung.com/jackson) 库，这也是 spring-kafka 项目的可选 Maven 依赖项。

所以，让我们把它添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.7</version>
</dependency>
```

不要用杰克逊的最新版本，建议用春天-卡夫卡的`pom.xml`加的版本。

最后，我们需要编写一个监听器来使用`Greeting`消息:

```java
@KafkaListener(
  topics = "topicName", 
  containerFactory = "greetingKafkaListenerContainerFactory")
public void greetingListener(Greeting greeting) {
    // process greeting message
}
```

## 7 .**。结论**

在本文中，我们介绍了 Apache Kafka 的 Spring 支持的基础知识。我们简单看了一下用于发送和接收消息的类。

本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220712173816/https://github.com/eugenp/tutorials/tree/master/spring-kafka)

在运行代码之前，请确保 Kafka 服务器正在运行，并且主题是手动创建的。