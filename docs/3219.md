# 监测阿帕奇卡夫卡中的消费者滞后

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kafka-consumer-lag>

## 1.概观

Kafka 消费群体滞后是任何基于 Kafka 的事件驱动系统的一个**关键绩效指标。**

在本教程中，我们将构建一个分析器应用程序来监控 Kafka 消费者滞后。

## 2.消费者滞后

消费者滞后就是消费者最后提交的偏移量和生产者在日志中的结束偏移量之间的**增量。换句话说，消费者滞后衡量的是任何生产者-消费者系统中生产和消费消息之间的延迟。**

在本节中，让我们了解如何确定偏移值。

### 2.1 .Kaka admin 客户端

**为了检查一个消费群体的偏移值**，**我们需要管理 Kafka 客户端**。因此，让我们在`LagAnalyzerService`类中编写一个方法来创建`AdminClient`类的实例:

```
private AdminClient getAdminClient(String bootstrapServerConfig) {
    Properties config = new Properties();
    config.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServerConfig);
    return AdminClient.create(config);
}
```

我们必须注意使用 [`@Value`注释](/web/20221208143830/https://www.baeldung.com/spring-value-annotation)从属性文件中检索引导服务器列表。同样，我们将使用这个注释来获取其他值，如`groupId and topicName`。

### 2.2.消费者群体补偿

首先，**可以使用`AdminClient`类的`listConsumerGroupOffsets()` 方法来获取特定消费群 id 的偏移信息**。

接下来，我们主要关注偏移值，因此我们可以调用`partitionsToOffsetAndMetadata()`方法来获得 TopicPartition 与`OffsetAndMetadata` 值的映射:

```
private Map<TopicPartition, Long> getConsumerGrpOffsets(String groupId) 
  throws ExecutionException, InterruptedException {
    ListConsumerGroupOffsetsResult info = adminClient.listConsumerGroupOffsets(groupId);
    Map<TopicPartition, OffsetAndMetadata> topicPartitionOffsetAndMetadataMap = info.partitionsToOffsetAndMetadata().get();

    Map<TopicPartition, Long> groupOffset = new HashMap<>();
    for (Map.Entry<TopicPartition, OffsetAndMetadata> entry : topicPartitionOffsetAndMetadataMap.entrySet()) {
        TopicPartition key = entry.getKey();
        OffsetAndMetadata metadata = entry.getValue();
        groupOffset.putIfAbsent(new TopicPartition(key.topic(), key.partition()), metadata.offset());
    }
    return groupOffset;
}
```

最后，我们可以注意到在`topicPartitionOffsetAndMetadataMap`上的迭代将我们获取的结果限制为每个主题和分区的偏移值。

### 2.3.生产者补偿

找到消费者组滞后的唯一方法是获得末端偏移值。为此，我们可以使用 [`KafkaConsumer`](https://web.archive.org/web/20221208143830/https://kafka.apache.org/25/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html) 类的`endOffsets() `方法。

让我们从在`LagAnalyzerService`类中创建一个`KafkaConsumer `类的实例开始:

```
private KafkaConsumer<String, String> getKafkaConsumer(String bootstrapServerConfig) {
    Properties properties = new Properties();
    properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServerConfig);
    properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    return new KafkaConsumer<>(properties);
}
```

接下来，让我们从需要计算滞后的使用者组偏移量中聚合所有相关的 TopicPartition 值，以便将它作为参数提供给`endOffsets()`方法:

```
private Map<TopicPartition, Long> getProducerOffsets(Map<TopicPartition, Long> consumerGrpOffset) {
    List<TopicPartition> topicPartitions = new LinkedList<>();
    for (Map.Entry<TopicPartition, Long> entry : consumerGrpOffset.entrySet()) {
        TopicPartition key = entry.getKey();
        topicPartitions.add(new TopicPartition(key.topic(), key.partition()));
    }
    return kafkaConsumer.endOffsets(topicPartitions);
}
```

最后，让我们编写一个方法，使用消费者补偿和生产者的`endoffsets`来生成每个`TopicPartition`的延迟:

```
private Map<TopicPartition, Long> computeLags(
  Map<TopicPartition, Long> consumerGrpOffsets,
  Map<TopicPartition, Long> producerOffsets) {
    Map<TopicPartition, Long> lags = new HashMap<>();
    for (Map.Entry<TopicPartition, Long> entry : consumerGrpOffsets.entrySet()) {
        Long producerOffset = producerOffsets.get(entry.getKey());
        Long consumerOffset = consumerGrpOffsets.get(entry.getKey());
        long lag = Math.abs(producerOffset - consumerOffset);
        lags.putIfAbsent(entry.getKey(), lag);
    }
    return lags;
}
```

## 3.滞后分析器

现在，让我们通过在`LagAnalyzerService`类中编写`analyzeLag() `方法来编排滞后分析:

```
public void analyzeLag(String groupId) throws ExecutionException, InterruptedException {
    Map<TopicPartition, Long> consumerGrpOffsets = getConsumerGrpOffsets(groupId);
    Map<TopicPartition, Long> producerOffsets = getProducerOffsets(consumerGrpOffsets);
    Map<TopicPartition, Long> lags = computeLags(consumerGrpOffsets, producerOffsets);
    for (Map.Entry<TopicPartition, Long> lagEntry : lags.entrySet()) {
        String topic = lagEntry.getKey().topic();
        int partition = lagEntry.getKey().partition();
        Long lag = lagEntry.getValue();
        System.out.printf("Time=%s | Lag for topic = %s, partition = %s is %d\n",
          MonitoringUtil.time(),
          topic,
          partition,
          lag);
    }
}
```

然而，在监控滞后指标时，我们需要一个**几乎实时的滞后值，以便我们可以采取任何管理措施来恢复系统性能**。

实现这一点的一个直接方法是通过**以规则的时间间隔**轮询滞后值。因此，让我们创建一个将调用`LagAnalyzerService:`的`analyzeLag()`方法的`LiveLagAnalyzerService`服务

```
@Scheduled(fixedDelay = 5000L)
public void liveLagAnalysis() throws ExecutionException, InterruptedException {
    lagAnalyzerService.analyzeLag(groupId);
}
```

出于我们的目的，我们使用 [`@Scheduled`](/web/20221208143830/https://www.baeldung.com/spring-scheduled-tasks) 注释将轮询频率设置为`5 seconds`。然而，对于实时监控，我们可能需要通过 [JMX](/web/20221208143830/https://www.baeldung.com/java-management-extensions) 来实现。

## 4.模拟

在本节中，我们将**为[本地 Kafka 设置](/web/20221208143830/https://www.baeldung.com/ops/kafka-docker-setup)** 模拟 Kafka 生产者和消费者，以便我们可以在不依赖外部 Kafka 生产者和消费者的情况下看到`LagAnalyzer`的运行。

### 4.1.模拟模式

由于**模拟模式仅用于演示目的**，我们应该有一个机制，当我们想要运行真实场景的 Lag Analyzer 应用程序时，可以关闭它。

我们可以将它作为一个可配置属性保存在`application.properties`资源文件中:

```
monitor.producer.simulate=true
monitor.consumer.simulate=true
```

我们将把这些属性插入 Kafka 生产者和消费者，并控制他们的行为。

此外，让我们定义生产者`startTime`、`endTime,`和一个助手方法`time() `来获得监控期间的当前时间:

```
public static final Date startTime = new Date();
public static final Date endTime = new Date(startTime.getTime() + 30 * 1000);

public static String time() {
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
    LocalDateTime now = LocalDateTime.now();
    String date = dtf.format(now);
    return date;
}
```

### 4.2.生产者-消费者配置

我们需要为实例化 Kafka 消费者和生产者模拟器的实例定义一些核心配置值。

首先，让我们在`KafkaConsumerConfig`类中定义消费者模拟器的配置:

```
public ConsumerFactory<String, String> consumerFactory(String groupId) {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
    if (enabled) {
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
    } else {
        props.put(ConsumerConfig.GROUP_ID_CONFIG, simulateGroupId);
    }
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 0);
    return new DefaultKafkaConsumerFactory<>(props);
}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
    if (enabled) {
        factory.setConsumerFactory(consumerFactory(groupId));
    } else {
        factory.setConsumerFactory(consumerFactory(simulateGroupId));
    }
    return factory;
} 
```

接下来，我们可以在`KafkaProducerConfig`类中定义生产者模拟器的配置:

```
@Bean
public ProducerFactory<String, String> producerFactory() {
    Map<String, Object> configProps = new HashMap<>();
    configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
    configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    return new DefaultKafkaProducerFactory<>(configProps);
}

@Bean
public KafkaTemplate<String, String> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
}
```

此外，让我们使用`[@KafkaListener](https://web.archive.org/web/20221208143830/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html) `注释来指定目标监听器，当然，只有当`monitor.consumer.simulate`被设置为`true`时才启用:

```
@KafkaListener(
  topics = "${monitor.topic.name}",
  containerFactory = "kafkaListenerContainerFactory",
  autoStartup = "${monitor.consumer.simulate}")
public void listen(String message) throws InterruptedException {
    Thread.sleep(10L);
}
```

因此，我们添加了 10 毫秒的睡眠时间来创建一个人为的消费者延迟。

最后，让我们写一个 **`sendMessage()`方法来模拟制作人**:

```
@Scheduled(fixedDelay = 1L, initialDelay = 5L)
public void sendMessage() throws ExecutionException, InterruptedException {
    if (enabled) {
        if (endTime.after(new Date())) {
            String message = "msg-" + time();
            SendResult<String, String> result = kafkaTemplate.send(topicName, message).get();
        }
    }
}
```

我们可以注意到，生成器将以 1 条消息/毫秒的速率生成消息。此外，在模拟的`startTime` 之后的`30 seconds`的`endTime`之后，它将停止生成消息。

### 4.3.实时监控

现在，让我们运行`LagAnalyzerApplication:`中的主方法

```
public static void main(String[] args) {
    SpringApplication.run(LagAnalyzerApplication.class, args);
    while (true) ;
}
```

每 30 秒后，我们将看到主题的每个分区的当前延迟:

```
Time=2021/06/06 11:07:24 | Lag for topic = baeldungTopic, partition = 0 is 93
Time=2021/06/06 11:07:29 | Lag for topic = baeldungTopic, partition = 0 is 290
Time=2021/06/06 11:07:34 | Lag for topic = baeldungTopic, partition = 0 is 776
Time=2021/06/06 11:07:39 | Lag for topic = baeldungTopic, partition = 0 is 1159
Time=2021/06/06 11:07:44 | Lag for topic = baeldungTopic, partition = 0 is 1559
Time=2021/06/06 11:07:49 | Lag for topic = baeldungTopic, partition = 0 is 2015
Time=2021/06/06 11:07:54 | Lag for topic = baeldungTopic, partition = 0 is 1231
Time=2021/06/06 11:07:59 | Lag for topic = baeldungTopic, partition = 0 is 731
Time=2021/06/06 11:08:04 | Lag for topic = baeldungTopic, partition = 0 is 231
Time=2021/06/06 11:08:09 | Lag for topic = baeldungTopic, partition = 0 is 0
```

因此，生产者生成消息的速率是 1 条消息/毫秒，高于消费者消费消息的速率。因此， **lag 将在第一个 30 秒开始构建，之后生产者停止生产，因此 lag 将逐渐下降到 0** 。

## 5.结论

在本教程中，我们了解了如何找到卡夫卡主题的消费滞后。此外，我们使用这些知识在 spring 中创建了一个 **`LagAnalyzer`应用程序，它几乎实时地显示了滞后**。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/spring-kafka)