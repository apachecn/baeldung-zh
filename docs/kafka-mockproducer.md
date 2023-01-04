# 使用卡夫卡 MockProducer

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-mockproducer>

## 1.概观

Kafka 是一个围绕分布式消息队列构建的消息处理系统。它提供了一个 Java 库，这样应用程序就可以向 Kafka 主题写入数据或从中读取数据。

现在，**由于大多数业务领域逻辑都是通过单元测试来验证的，所以应用程序通常会模仿 JUnit 中的所有 I/O 操作**。Kafka 还提供了一个`MockProducer `来模仿生产者应用程序。

在本教程中，我们将首先实现 Kafka producer 应用程序。稍后，我们将使用`MockProducer`实现一个单元测试来验证常见的生产者操作。

## 2.Maven 依赖性

在我们实现生产者应用程序之前，我们将为 [`kafka-clients`](https://web.archive.org/web/20220628150856/https://search.maven.org/artifact/org.apache.kafka/kafka-clients) 添加一个 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.5.0</version>
</dependency>
```

## 3.`MockProducer`

`kafka-clients `库包含一个 Java 库，用于在 Kafka 中发布和使用消息。生产者应用程序可以使用这些 API 向 Kafka 主题发送键值记录:

```java
public class KafkaProducer {

    private final Producer<String, String> producer;

    public KafkaProducer(Producer<String, String> producer) {
        this.producer = producer;
    }

    public Future<RecordMetadata> send(String key, String value) {
        ProducerRecord record = new ProducerRecord("topic_sports_news", key, value);
        return producer.send(record);
    }
}
```

任何 Kafka 制作者都必须在客户端的库中实现`Producer `接口。Kafka 还提供了一个`KafkaProducer` 类，这是一个针对 Kafka 代理执行 I/O 操作的具体实现。

此外，Kafka 提供了一个`MockProducer `，它实现了相同的`Producer `接口，并模仿了在`KafkaProducer`中实现的所有 I/O 操作:

```java
@Test
void givenKeyValue_whenSend_thenVerifyHistory() {

    MockProducer mockProducer = new MockProducer<>(true, new StringSerializer(), new StringSerializer());

    kafkaProducer = new KafkaProducer(mockProducer);
    Future<RecordMetadata> recordMetadataFuture = kafkaProducer.send("soccer", 
      "{\"site\" : \"baeldung\"}");

    assertTrue(mockProducer.history().size() == 1);
}
```

虽然这样的 I/O 操作也可以用 [Mockito](/web/20220628150856/https://www.baeldung.com/mockito-series) ， **`MockProducer` 来模拟，但是它让我们可以访问很多我们需要在 mock 之上实现的特性。**一个这样的特性是`history() `方法。`MockProducer `缓存被调用`send() `的记录，从而允许我们验证生产者的发布行为。

此外，我们还可以验证元数据，如主题名、分区、记录键或值:

```java
assertTrue(mockProducer.history().get(0).key().equalsIgnoreCase("data"));
assertTrue(recordMetadataFuture.get().partition() == 0);
```

## 4.嘲笑一个卡夫卡集群

到目前为止，在我们的模拟测试中，我们假设一个主题只有一个分区。然而，为了实现生产者线程和消费者线程之间的最大并发性，Kafka 主题通常被分成多个分区。

这允许生产者将数据写入多个分区。这通常是通过基于键对记录进行分区并将特定的键映射到特定分区来实现的:

```java
public class EvenOddPartitioner extends DefaultPartitioner {

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, 
      byte[] valueBytes, Cluster cluster) {
        if (((String)key).length() % 2 == 0) {
            return 0;
        }
        return 1;
    }
}
```

因此，所有偶数长度的键将被发布到分区“0”，同样，奇数长度的键将被发布到分区“1”。

**`MockProducer `使我们能够通过模拟具有多个分区的 Kafka 集群来验证这样的分区分配算法:**

```java
@Test
void givenKeyValue_whenSendWithPartitioning_thenVerifyPartitionNumber() 
  throws ExecutionException, InterruptedException {
    PartitionInfo partitionInfo0 = new PartitionInfo(TOPIC_NAME, 0, null, null, null);
    PartitionInfo partitionInfo1 = new PartitionInfo(TOPIC_NAME, 1, null, null, null);
    List<PartitionInfo> list = new ArrayList<>();
    list.add(partitionInfo0);
    list.add(partitionInfo1);

    Cluster cluster = new Cluster("kafkab", new ArrayList<Node>(), list, emptySet(), emptySet());
    this.mockProducer = new MockProducer<>(cluster, true, new EvenOddPartitioner(), 
      new StringSerializer(), new StringSerializer());

    kafkaProducer = new KafkaProducer(mockProducer);
    Future<RecordMetadata> recordMetadataFuture = kafkaProducer.send("partition", 
      "{\"site\" : \"baeldung\"}");

    assertTrue(recordMetadataFuture.get().partition() == 1);
}
```

我们模仿了一个有两个分区的`Cluster `，0 和 1。然后我们可以验证`EvenOddPartitioner `将记录发布到分区 1。

## 5.用`MockProducer`模拟错误

到目前为止，我们只成功地嘲笑了制作人向卡夫卡主题发送记录。但是，如果写记录时出现异常，会发生什么呢？

应用程序通常通过重试或向客户端抛出异常来处理此类异常。

`MockProducer` 允许我们在`send()`期间模拟异常，以便我们可以验证异常处理代码:

```java
@Test
void givenKeyValue_whenSend_thenReturnException() {
    MockProducer<String, String> mockProducer = new MockProducer<>(false, 
      new StringSerializer(), new StringSerializer())

    kafkaProducer = new KafkaProducer(mockProducer);
    Future<RecordMetadata> record = kafkaProducer.send("site", "{\"site\" : \"baeldung\"}");
    RuntimeException e = new RuntimeException();
    mockProducer.errorNext(e);

    try {
        record.get();
    } catch (ExecutionException | InterruptedException ex) {
        assertEquals(e, ex.getCause());
    }
    assertTrue(record.isDone());
}
```

这段代码中有两件值得注意的事情。

首先，我们用`autoComplete `作为`false.` 调用`MockProducer `构造函数，这告诉`MockProducer` 在完成`send()` 方法之前等待输入。

其次，我们将调用`mockProducer.errorNext(e),` ，以便`MockProducer` 为最后一次`send()` 调用返回一个异常。

## 6.用`MockProducer`模拟事务性写入

Kafka 0.11 引入了 Kafka 经纪人、生产者和消费者之间的交易。这允许 Kafka 中端到端[恰好一次的消息传递语义](/web/20220628150856/https://www.baeldung.com/kafka-exactly-once)。简而言之，这意味着事务生产者只能通过[两阶段提交](/web/20220628150856/https://www.baeldung.com/transactions-intro)协议向代理发布记录。

`MockProducer `还支持事务性写入，并允许我们验证这种行为:

```java
@Test
void givenKeyValue_whenSendWithTxn_thenSendOnlyOnTxnCommit() {
    MockProducer<String, String> mockProducer = new MockProducer<>(true, 
      new StringSerializer(), new StringSerializer())

    kafkaProducer = new KafkaProducer(mockProducer);
    kafkaProducer.initTransaction();
    kafkaProducer.beginTransaction();
    Future<RecordMetadata> record = kafkaProducer.send("data", "{\"site\" : \"baeldung\"}");

    assertTrue(mockProducer.history().isEmpty());
    kafkaProducer.commitTransaction();
    assertTrue(mockProducer.history().size() == 1);
}
```

**因为`MockProducer `也支持与具体的`KafkaProducer,`相同的 API，所以它只在我们提交事务后更新`history `。**这样的模仿行为可以帮助应用程序验证每个事务都调用了`commitTransaction() `。

## 7.结论

在本文中，我们查看了`kafka-client `库的`MockProducer `类。我们讨论过`MockProducer `实现了与具体的`KafkaProducer` 相同的层次结构，因此，我们可以用 Kafka 代理模拟所有的 I/O 操作。

**我们还讨论了一些复杂的模拟场景，并能够使用`MockProducer.`** 测试异常、分区和事务

和往常一样，所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628150856/https://github.com/eugenp/tutorials/tree/master/apache-kafka)