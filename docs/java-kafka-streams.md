# Java 中的 KafkaStreams 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kafka-streams>

## 1。概述

在本文中，我们将关注 [`KafkaStreams` 库](https://web.archive.org/web/20221126234120/https://kafka.apache.org/documentation/streams/)。

由 Apache Kafka 的创造者设计,`.` 这款软件的主要目标是允许程序员创建高效、实时、流应用程序，这些应用程序可以作为微服务工作。

**`KafkaStreams` 使我们能够从 Kafka 主题中消费、分析或转换数据，并可能将其发送到另一个 Kafka 主题。**

为了演示`KafkaStreams,` ,我们将创建一个简单的应用程序，它从一个主题中读取句子，计算单词的出现次数，并打印每个单词的数量。

值得注意的是，`KafkaStreams` 库不是反应式的，不支持异步操作和反压处理。

## 2。Maven 依赖关系

要开始使用`KafkaStreams,` 编写流处理逻辑，我们需要添加对 [`kafka-streams`](https://web.archive.org/web/20221126234120/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.kafka%22%20AND%20a%3A%22kafka-streams%22) 和 [`kafka-clients`](https://web.archive.org/web/20221126234120/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.kafka%22%20AND%20a%3A%22kafka-clients%22) 的依赖:

```java
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.8.0</version>
</dependency> 
```

我们还需要安装并启动 Apache Kafka，因为我们将使用 Kafka 主题。这个主题将是我们流式作业的数据源。

我们可以从[官网](https://web.archive.org/web/20221126234120/https://www.confluent.io/download/)下载卡夫卡和其他需要的依赖。

## 3。配置 KafkaStreams 输入

**我们首先要做的是输入卡夫卡主题的定义。**

我们可以使用我们下载的`Confluent` 工具——它包含一个 Kafka 服务器。它还包含我们可以用来向 Kafka 发布消息的`kafka-console-producer`。

首先，让我们运行 Kafka 集群:

```java
./confluent start
```

一旦 Kafka 启动，我们就可以使用`APPLICATION_ID_CONFIG`来定义我们的数据源和应用程序的名称:

```java
String inputTopic = "inputTopic";
```

```java
Properties streamsConfiguration = new Properties();
streamsConfiguration.put(
  StreamsConfig.APPLICATION_ID_CONFIG, 
  "wordcount-live-test");
```

一个至关重要的配置参数是`BOOTSTRAP_SERVER_CONFIG.` ,这是我们刚刚启动的本地 Kafka 实例的 URL:

```java
private String bootstrapServers = "localhost:9092";
streamsConfiguration.put(
  StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, 
  bootstrapServers);
```

接下来，我们需要传递从`inputTopic:`开始消费的密钥类型和值

```java
streamsConfiguration.put(
  StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, 
  Serdes.String().getClass().getName());
streamsConfiguration.put(
  StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, 
  Serdes.String().getClass().getName());
```

**流处理通常是有状态的。当我们想要保存中间结果时，我们需要指定`STATE_DIR_CONFIG` 参数**。

在我们的测试中，我们使用本地文件系统:

```java
this.stateDirectory = Files.createTempDirectory("kafka-streams");
streamsConfiguration.put(
  StreamsConfig.STATE_DIR_CONFIG, this.stateDirectory.toAbsolutePath().toString()); 
```

## 4。构建流拓扑

一旦我们定义了输入主题，我们就可以创建一个流拓扑——这是一个事件应该如何处理和转换的定义。

在我们的例子中，我们想要实现一个单词计数器。对于发送到`inputTopic,` 的每一个句子，我们希望将其拆分成单词，并计算每个单词的出现次数。

我们可以使用`KStreamsBuilder` 类的一个实例来开始构建我们的拓扑:

```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> textLines = builder.stream(inputTopic);
Pattern pattern = Pattern.compile("\\W+", Pattern.UNICODE_CHARACTER_CLASS);

KTable<String, Long> wordCounts = textLines
  .flatMapValues(value -> Arrays.asList(pattern.split(value.toLowerCase())))
  .groupBy((key, word) -> word)
  .count();
```

为了实现字数统计，首先，我们需要使用正则表达式拆分值。

split 方法返回一个数组。我们使用`flatMapValues()`来展平它。否则，我们会得到一个数组列表，使用这样的结构编写代码会很不方便。

最后，我们聚合每个单词的值，并调用`count()` 来计算特定单词的出现次数。

## 5。处理结果

我们已经计算了输入消息的字数。**现在让我们使用`foreach()`方法:**在标准输出上打印结果

```java
wordCounts.toStream()
  .foreach((word, count) -> System.out.println("word: " + word + " -> " + count));
```

在生产中，这样的流作业通常会将输出发布到另一个 Kafka 主题。

我们可以使用`to() method:`来完成这项工作

```java
String outputTopic = "outputTopic";
wordCounts.toStream()
  .to(outputTopic, Produced.with(Serdes.String(), Serdes.Long()));
```

`Serde` 类为我们提供了用于 Java 类型的预配置序列化器，这些序列化器将用于将对象序列化为字节数组。然后，字节数组将被发送到 Kafka 主题。

我们使用`String`作为主题的关键字，使用`Long`作为实际计数的值。`to()` 方法会将结果数据保存到`outputTopic`。

## 6。正在启动 KafkaStream 作业

至此，我们构建了一个可以执行的拓扑。然而，工作还没有开始。

**我们需要通过调用`KafkaStreams` 实例上的`start()` 方法来显式地开始我们的工作:**

```java
Topology topology = builder.build();
KafkaStreams streams = new KafkaStreams(topology, streamsConfiguration);
streams.start();

Thread.sleep(30000);
streams.close();
```

请注意，我们等待 30 秒钟，等待作业完成。在现实世界的场景中，该作业将一直运行，在事件到达时处理来自 Kafka 的事件。

我们可以通过向我们的 Kafka 主题发布一些事件来测试我们的工作。

让我们启动一个`kafka-console-producer` 并手动发送一些事件到我们的`inputTopic:`

```java
./kafka-console-producer --topic inputTopic --broker-list localhost:9092
>"this is a pony"
>"this is a horse and pony" 
```

这样，我们发表了两个事件给卡夫卡。我们的应用程序将使用这些事件，并打印以下输出:

```java
word:  -> 1
word: this -> 1
word: is -> 1
word: a -> 1
word: pony -> 1
word:  -> 2
word: this -> 2
word: is -> 2
word: a -> 2
word: horse -> 1
word: and -> 1
word: pony -> 2
```

我们可以看到，当第一条消息到达时，`pony` 这个词只出现了一次。但是当我们发出第二条信息时，`pony` 这个词发生了第二次印刷:“`word: pony -> 2″`”。

## 6。结论

本文讨论如何使用 Apache Kafka 作为数据源，使用`KafkaStreams` 库作为流处理库来创建主要的流处理应用程序。

所有这些例子和代码片段都可以在 [GitHub 项目](https://web.archive.org/web/20221126234120/https://github.com/eugenp/tutorials/tree/master/apache-kafka)中找到——这是一个 Maven 项目，所以它应该很容易导入和运行。