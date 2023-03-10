# 用 Java 在 Kafka 中处理一次

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-exactly-once>

## 1。概述

在本教程中，我们将了解 Kafka 如何通过新引入的事务 API 确保生产者和消费者应用程序之间的一次性交付。

此外，在 WordCount 示例中，我们将使用该 API 实现事务性生产者和消费者，以实现端到端的一次性交付。

## 2。卡夫卡的信息传递

由于各种故障，消息传递系统不能保证生产者和消费者应用程序之间的消息传递。根据客户端应用程序与此类系统的交互方式，以下消息语义是可能的:

*   如果一个消息系统永远不会复制一条消息，但可能会错过偶尔的消息，我们称之为`at-most-once`
*   或者，如果它永远不会错过一条消息，但可能会复制偶尔出现的消息，我们称之为`at-least-once`
*   **但是，如果它总是不重复地传递所有消息，那就是`exactly-once`**

最初，Kafka 只支持最多一次和至少一次消息传递。

然而，**Kafka 经纪人和客户应用程序之间交易的引入确保了 Kafka 中的一次性交付**。为了更好地理解它，让我们快速回顾一下事务性客户端 API。

## 3。Maven 依赖关系

为了使用事务 API，我们将需要 pom 中的 [Kafka 的 Java 客户端](https://web.archive.org/web/20221129002105/https://search.maven.org/search?q=g:org.apache.kafka%20AND%20a:kafka-clients):

```java
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.8.0</version>
</dependency>
```

## 4。一个事务性的`consume-transform-produce` 循环

对于我们的例子，我们将使用来自输入主题`sentences`的消息。

然后，对于每个句子，我们将计算每个单词的数量，并将单个单词的数量发送到输出主题`counts`。

在这个例子中，我们将假设在`sentences` 主题中已经有可用的事务数据。

### 4.1。感知事务的生产者

所以我们先来补充一个典型的卡夫卡制作人。

```java
Properties producerProps = new Properties();
producerProps.put("bootstrap.servers", "localhost:9092");
```

此外，我们需要指定一个`transactional.id`并启用`idempotence`:

```java
producerProps.put("enable.idempotence", "true");
producerProps.put("transactional.id", "prod-1");

KafkaProducer<String, String> producer = new KafkaProducer(producerProps);
```

因为我们已经启用了幂等性，Kafka 将使用这个事务 id 作为其算法的一部分，**对这个生产者** **发送的**的任何消息进行重复删除，从而确保幂等性。

简而言之，如果制作者不小心不止一次地向卡夫卡发送相同的信息，这些设置使它能够注意到。

我们需要做的就是**确保每个生产者**的事务 id 是不同的，尽管在重启之间是一致的。

### 4.2。启用交易的生产者

一旦我们准备好了，那么我们还需要调用`initTransaction `来准备生产者使用的事务:

```java
producer.initTransactions();
```

这将生产者向代理注册为可以使用事务的生产者，**通过它的`transactional.id `和一个序列号或纪元**来标识它。反过来，代理将使用这些将任何操作预写到事务日志中。

因此，**代理将从日志中删除属于具有相同事务 id 和更早的** **时期的生产者的任何动作，**假定它们来自失效的事务。

### 4.3。交易感知型消费者

当我们消费时，我们可以按顺序阅读一个主题分区上的所有消息。虽然，**我们可以用`isolation.level`来表示我们应该等到相关的事务被提交后再读取事务消息**:

```java
Properties consumerProps = new Properties();
consumerProps.put("bootstrap.servers", "localhost:9092");
consumerProps.put("group.id", "my-group-id");
consumerProps.put("enable.auto.commit", "false");
consumerProps.put("isolation.level", "read_committed");
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
consumer.subscribe(singleton(“sentences”));
```

**使用值`read_committed`确保我们在事务完成之前不会读取任何事务消息。**

`isolation.level` 的默认值为`read_uncommitted.`

### 4.4。通过交易消费和转换

既然我们已经将生产者和消费者都配置为事务性地读写，我们就可以从我们的输入主题中消费记录，并计算每个记录中的每个单词:

```java
ConsumerRecords<String, String> records = consumer.poll(ofSeconds(60));
Map<String, Integer> wordCountMap =
  records.records(new TopicPartition("input", 0))
    .stream()
    .flatMap(record -> Stream.of(record.value().split(" ")))
    .map(word -> Tuple.of(word, 1))
    .collect(Collectors.toMap(tuple -> 
      tuple.getKey(), t1 -> t1.getValue(), (v1, v2) -> v1 + v2));
```

注意，上面的代码没有任何事务性。但是，**,因为我们使用了`read_committed,`,这意味着在同一个事务中写入输入主题的任何消息都不会被这个消费者读取，直到它们都被写入。**

现在，我们可以将计算出的字数发送到输出主题。

让我们看看如何产生我们的结果，也是交易。

### 4.5。发送 API

将我们的计数作为新消息发送，但是在同一个事务中，我们调用`beginTransaction`:

```java
producer.beginTransaction();
```

然后，我们可以将每一个都写入我们的“计数”主题，关键字是单词，计数是值:

```java
wordCountMap.forEach((key,value) -> 
    producer.send(new ProducerRecord<String,String>("counts",key,value.toString())));
```

注意，因为生产者可以通过键对数据进行分区，这意味着**事务性消息可以跨越多个分区，每个分区由不同的消费者读取。**因此，Kafka broker 将为一个事务存储所有更新分区的列表。

还要注意，**在一个事务中，一个生产者可以使用多个线程并行发送记录**。

### 4.6。提交偏移量

最后，我们需要提交刚刚消耗完的补偿量。对于事务，我们将偏移量提交回我们从中读取它们的输入主题，就像平常一样。也不过，**我们** **把它们送到生产者的交易中。**

我们可以在一个调用中完成所有这些，但是我们首先需要计算每个主题分区的偏移量:

```java
Map<TopicPartition, OffsetAndMetadata> offsetsToCommit = new HashMap<>();
for (TopicPartition partition : records.partitions()) {
    List<ConsumerRecord<String, String>> partitionedRecords = records.records(partition);
    long offset = partitionedRecords.get(partitionedRecords.size() - 1).offset();
    offsetsToCommit.put(partition, new OffsetAndMetadata(offset + 1));
}
```

**注意，我们提交给事务的是即将到来的偏移量，这意味着我们需要加 1。**

然后，我们可以将计算出的偏移量发送给事务:

```java
producer.sendOffsetsToTransaction(offsetsToCommit, "my-group-id");
```

### 4.7。提交或中止交易

最后，我们可以提交事务，这将自动地向`consumer_offsets `主题以及事务本身写入偏移量:

```java
producer.commitTransaction();
```

这会将所有缓冲的消息刷新到各自的分区。此外，Kafka broker 将该交易中的所有消息提供给消费者。

当然，如果我们在处理过程中出现任何问题，例如，如果我们捕捉到一个异常，我们可以调用`abortTransaction:`

```java
try {
  // ... read from input topic
  // ... transform
  // ... write to output topic
  producer.commitTransaction();
} catch ( Exception e ) {
  producer.abortTransaction();
}
```

并丢弃任何缓冲的消息，从代理中删除事务。

**如果我们在代理配置的`max.transaction.timeout.ms,`之前既没有提交也没有中止，Kafka 代理将自己中止交易。**该属性的默认值是 900，000 毫秒或 15 分钟。

## 5。其他`consume-transform-produce`循环

我们刚刚看到的是一个基本的`consume-transform-produce`循环，它读写同一个 Kafka 集群。

相反，**必须读写不同 Kafka 集群的应用程序必须使用旧的`commitSync `和`commitAsync ` API** 。通常，应用程序会将使用者偏移量存储到它们的外部状态存储中，以保持事务性。

## 6。结论

对于数据关键型应用程序，端到端一次性处理通常是必不可少的。

在本教程中，**我们看到了如何使用 Kafka 通过事务**来实现这一点，并且我们实现了一个基于事务的字数统计示例来说明这一原理。

请随意查看 GitHub 上的所有[代码示例。](https://web.archive.org/web/20221129002105/https://github.com/eugenp/tutorials/tree/master/apache-kafka)