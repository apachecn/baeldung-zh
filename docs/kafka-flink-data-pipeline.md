# 用 Flink 和 Kafka 构建数据管道

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-flink-data-pipeline>

## 1。概述

Apache Flink 是一个流处理框架，可以很容易地与 Java 一起使用。 [Apache Kafka](https://web.archive.org/web/20221126224609/https://kafka.apache.org/) 是一个支持高容错的分布式流处理系统。

在本教程中，我们将了解如何使用这两种技术构建数据管道。

## 2。安装

要安装和配置 Apache Kafka，请参考[官方指南](https://web.archive.org/web/20221126224609/https://kafka.apache.org/quickstart)。安装后，我们可以使用以下命令来创建名为`flink_input`和`flink_output:`的新主题

```java
 bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic flink_output

 bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic flink_input
```

出于本教程的考虑，我们将使用 Apache Kafka 的默认配置和默认端口。

## 3。Flink 用法

Apache Flink 允许实时流处理技术。该框架允许使用多个第三方系统作为流源或汇。

在 Flink 中，有各种连接器可用:

*   阿帕奇卡夫卡(源/汇)
*   阿帕奇卡桑德拉(水槽)
*   亚马逊 Kinesis 流(源/汇)
*   Elasticsearch (sink)
*   Hadoop 文件系统(接收器)
*   RabbitMQ(源/宿)
*   Apache NiFi(源/宿)
*   Twitter 流媒体应用编程接口(来源)

要将 Flink 添加到我们的项目中，我们需要包含以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-core</artifactId>
    <version>1.5.0</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
    <version>1.5.0</version>
</dependency>
```

添加这些依赖项将允许我们消费和生产 Kafka 主题。你可以在 [Maven Central](https://web.archive.org/web/20221126224609/https://search.maven.org/search?q=g:org.apache.flink) 上找到 Flink 的当前版本。

## 4。卡夫卡字符串消费者

要使用 Flink 从 Kafka 获取数据，我们需要提供一个主题和一个 Kafka 地址。我们还应该提供一个用于保存偏移量的组 id，这样我们就不会总是从头开始读取整个数据。

让我们创建一个静态方法，它将使`FlinkKafkaConsumer `的创建更加容易:

```java
public static FlinkKafkaConsumer011<String> createStringConsumerForTopic(
  String topic, String kafkaAddress, String kafkaGroup ) {

    Properties props = new Properties();
    props.setProperty("bootstrap.servers", kafkaAddress);
    props.setProperty("group.id",kafkaGroup);
    FlinkKafkaConsumer011<String> consumer = new FlinkKafkaConsumer011<>(
      topic, new SimpleStringSchema(), props);

    return consumer;
}
```

这个方法采用一个`topic, kafkaAddress,` 和`kafkaGroup`并创建一个`FlinkKafkaConsumer`，它将从给定主题中消费数据作为一个`String`，因为我们已经使用了`SimpleStringSchema`来解码数据。

课名中的数字`011`指的是卡夫卡版本。

## 5。卡夫卡弦乐制作人

为了向 Kafka 生成数据，我们需要提供我们想要使用的 Kafka 地址和主题。同样，我们可以创建一个静态方法，帮助我们为不同的主题创建生产者:

```java
public static FlinkKafkaProducer011<String> createStringProducer(
  String topic, String kafkaAddress){

    return new FlinkKafkaProducer011<>(kafkaAddress,
      topic, new SimpleStringSchema());
}
```

这个方法只接受`topic`和`kafkaAddress`作为参数，因为当我们生成 Kafka 主题时，不需要提供组 id。

## 6。字符串流处理

当我们有一个完全工作的消费者和生产者时，我们可以尝试处理来自 Kafka 的数据，然后将我们的结果保存回 Kafka。可用于流处理的函数的完整列表可在[这里](/web/20221126224609/https://www.baeldung.com/apache-flink)找到。

在这个例子中，我们将大写每个 Kafka 条目中的单词，然后将其写回 Kafka。

为此，我们需要创建一个自定义的`MapFunction`:

```java
public class WordsCapitalizer implements MapFunction<String, String> {
    @Override
    public String map(String s) {
        return s.toUpperCase();
    }
}
```

创建函数后，我们可以在流处理中使用它:

```java
public static void capitalize() {
    String inputTopic = "flink_input";
    String outputTopic = "flink_output";
    String consumerGroup = "baeldung";
    String address = "localhost:9092";
    StreamExecutionEnvironment environment = StreamExecutionEnvironment
      .getExecutionEnvironment();
    FlinkKafkaConsumer011<String> flinkKafkaConsumer = createStringConsumerForTopic(
      inputTopic, address, consumerGroup);
    DataStream<String> stringInputStream = environment
      .addSource(flinkKafkaConsumer);

    FlinkKafkaProducer011<String> flinkKafkaProducer = createStringProducer(
      outputTopic, address);

    stringInputStream
      .map(new WordsCapitalizer())
      .addSink(flinkKafkaProducer);
}
```

**应用程序将从`flink_input`主题读取数据，在流上执行操作，然后将结果保存到 Kafka 中的 `flink_output`主题。**

我们已经看到如何使用 Flink 和 Kafka 处理字符串。但是通常需要对自定义对象执行操作。我们将在下一章中看到如何做到这一点。

## 7 .**。自定义对象反序列化**

下面的类表示一条简单的消息，其中包含发件人和收件人的信息:

```java
@JsonSerialize
public class InputMessage {
    String sender;
    String recipient;
    LocalDateTime sentAt;
    String message;
}
```

以前，我们使用`SimpleStringSchema`来反序列化来自 Kafka 的消息，但是现在**我们希望直接将数据反序列化到自定义对象**。

为此，我们需要一个自定义的`DeserializationSchema:`

```java
public class InputMessageDeserializationSchema implements
  DeserializationSchema<InputMessage> {

    static ObjectMapper objectMapper = new ObjectMapper()
      .registerModule(new JavaTimeModule());

    @Override
    public InputMessage deserialize(byte[] bytes) throws IOException {
        return objectMapper.readValue(bytes, InputMessage.class);
    }

    @Override
    public boolean isEndOfStream(InputMessage inputMessage) {
        return false;
    }

    @Override
    public TypeInformation<InputMessage> getProducedType() {
        return TypeInformation.of(InputMessage.class);
    }
}
```

我们在这里假设消息在 Kafka 中被保存为 JSON。

因为我们有一个类型为`LocalDateTime`的字段，我们需要指定负责将`LocalDateTime`对象映射到 JSON 的`JavaTimeModule, `。

**Flink 模式不能有不可序列化的字段**，因为所有操作符(如模式或函数)在作业开始时都是序列化的。

Apache Spark 也有类似的问题。这个问题的一个已知解决方案是将字段初始化为`static`，就像我们在上面对`ObjectMapper`所做的一样。这不是最漂亮的解决方案，但它相对简单，而且很管用。

方法`isEndOfStream`可用于特殊情况，即只在接收到某些特定数据时才对流进行处理。但是在我们的例子中不需要。

## 8。自定义对象序列化

现在，让我们假设我们希望我们的系统有可能创建消息的备份。我们希望该过程是自动的，并且每个备份应该由一整天内发送的消息组成。

此外，应该为备份消息分配一个唯一的 id。

为此，我们可以创建以下类:

```java
public class Backup {
    @JsonProperty("inputMessages")
    List<InputMessage> inputMessages;
    @JsonProperty("backupTimestamp")
    LocalDateTime backupTimestamp;
    @JsonProperty("uuid")
    UUID uuid;

    public Backup(List<InputMessage> inputMessages, 
      LocalDateTime backupTimestamp) {
        this.inputMessages = inputMessages;
        this.backupTimestamp = backupTimestamp;
        this.uuid = UUID.randomUUID();
    }
}
```

请注意，UUID 生成机制并不完美，因为它允许重复。然而，对于这个例子的范围来说，这已经足够了。

我们希望将我们的`Backup`对象保存为 Kafka 的 JSON，因此我们需要创建我们的`SerializationSchema`:

```java
public class BackupSerializationSchema
  implements SerializationSchema<Backup> {

    ObjectMapper objectMapper;
    Logger logger = LoggerFactory.getLogger(BackupSerializationSchema.class);

    @Override
    public byte[] serialize(Backup backupMessage) {
        if(objectMapper == null) {
            objectMapper = new ObjectMapper()
              .registerModule(new JavaTimeModule());
        }
        try {
            return objectMapper.writeValueAsString(backupMessage).getBytes();
        } catch (com.fasterxml.jackson.core.JsonProcessingException e) {
            logger.error("Failed to parse JSON", e);
        }
        return new byte[0];
    }
}
```

## 9。时间戳消息

因为我们想要为每天的所有消息创建备份，所以消息需要时间戳。

Flink 提供了三种不同的时间特性`EventTime, ProcessingTime, `和`IngestionTime.`

在我们的例子中，我们需要使用消息发送的时间，所以我们将使用`EventTime.`

为了使用`EventTime` **，我们需要一个`TimestampAssigner`，它将从我们的输入数据**中提取时间戳:

```java
public class InputMessageTimestampAssigner 
  implements AssignerWithPunctuatedWatermarks<InputMessage> {

    @Override
    public long extractTimestamp(InputMessage element, 
      long previousElementTimestamp) {
        ZoneId zoneId = ZoneId.systemDefault();
        return element.getSentAt().atZone(zoneId).toEpochSecond() * 1000;
    }

    @Nullable
    @Override
    public Watermark checkAndGetNextWatermark(InputMessage lastElement, 
      long extractedTimestamp) {
        return new Watermark(extractedTimestamp - 1500);
    }
}
```

我们需要将我们的`LocalDateTime`转换成`EpochSecond`,因为这是 Flink 期望的格式。分配时间戳后，所有基于时间的操作将使用来自`sentAt`字段的时间进行操作。

由于 Flink 期望时间戳以毫秒为单位，而`toEpochSecond()`返回以秒为单位的时间，我们需要将它乘以 1000，因此 Flink 将正确创建窗口。

Flink 定义了`Watermark. ` **水印的概念，在数据没有按照发送顺序到达的情况下，水印非常有用。**水印定义了元素被允许处理的最大延迟。

时间戳低于水印的元素根本不会被处理。

## 10。创建时间窗口

为了确保我们的备份只收集一天内发送的消息，我们可以在流上使用`timeWindowAll`方法，这将把消息分成窗口。

然而，我们仍然需要聚集来自每个窗口的消息，并将它们作为`Backup`返回。

为此，我们需要一个自定义的`AggregateFunction`:

```java
public class BackupAggregator 
  implements AggregateFunction<InputMessage, List<InputMessage>, Backup> {

    @Override
    public List<InputMessage> createAccumulator() {
        return new ArrayList<>();
    }

    @Override
    public List<InputMessage> add(
      InputMessage inputMessage,
      List<InputMessage> inputMessages) {
        inputMessages.add(inputMessage);
        return inputMessages;
    }

    @Override
    public Backup getResult(List<InputMessage> inputMessages) {
        return new Backup(inputMessages, LocalDateTime.now());
    }

    @Override
    public List<InputMessage> merge(List<InputMessage> inputMessages,
      List<InputMessage> acc1) {
        inputMessages.addAll(acc1);
        return inputMessages;
    }
}
```

## 11。聚合备份

在分配了适当的时间戳并实现了我们的`AggregateFunction`之后，我们终于可以获取我们的 Kafka 输入并处理它了:

```java
public static void createBackup () throws Exception {
    String inputTopic = "flink_input";
    String outputTopic = "flink_output";
    String consumerGroup = "baeldung";
    String kafkaAddress = "192.168.99.100:9092";
    StreamExecutionEnvironment environment
      = StreamExecutionEnvironment.getExecutionEnvironment();
    environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
    FlinkKafkaConsumer011<InputMessage> flinkKafkaConsumer
      = createInputMessageConsumer(inputTopic, kafkaAddress, consumerGroup);
    flinkKafkaConsumer.setStartFromEarliest();

    flinkKafkaConsumer.assignTimestampsAndWatermarks(
      new InputMessageTimestampAssigner());
    FlinkKafkaProducer011<Backup> flinkKafkaProducer
      = createBackupProducer(outputTopic, kafkaAddress);

    DataStream<InputMessage> inputMessagesStream
      = environment.addSource(flinkKafkaConsumer);

    inputMessagesStream
      .timeWindowAll(Time.hours(24))
      .aggregate(new BackupAggregator())
      .addSink(flinkKafkaProducer);

    environment.execute();
}
```

## 12。结论

在本文中，我们介绍了如何使用 Apache Flink 和 Apache Kafka 创建简单的数据管道。

和往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20221126224609/https://github.com/eugenp/tutorials/tree/master/apache-kafka)