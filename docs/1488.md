# 获取 Apache Kafka 主题中的消息数量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kafka-count-topic-messages>

## 1.概观

[Apache Kafka](https://web.archive.org/web/20220927105427/https://kafka.apache.org/) 是一个开源的分布式事件流媒体平台。

在这个快速教程中，我们将学习获取 Kafka 主题中消息数量的技巧。我们将展示编程和本地命令技术。

## 2.编程技术

一个 Kafka 主题可以有多个分区。我们的技术应该确保我们已经计算了来自每个分区的消息数量。

我们必须遍历每个分区，检查它们的最新偏移量。为此，我们将介绍一个消费者:

```
KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
```

第二步是**从这个消费者**那里获得所有分区:

```
List<TopicPartition> partitions = consumer.partitionsFor(topic).stream().map(p -> new TopicPartition(topic, p.partition()))
    .collect(Collectors.toList());
```

第三步是**在每个分区的末尾偏移消费者，并将结果记录在一个分区表**中:

```
consumer.assign(partitions);
consumer.seekToEnd(Collections.emptySet());
Map<TopicPartition, Long> endPartitions = partitions.stream().collect(Collectors.toMap(Function.identity(), consumer::position));
```

最后一步是**取每个分区的最后一个位置，对结果求和，得到主题中的消息数:**

```
numberOfMessages = partitions.stream().mapToLong(p -> endPartitions.get(p)).sum();
```

## 3.卡夫卡原生命令

如果我们想对 Kafka 主题的大量消息执行一些自动化任务，编程技术是很好的。然而，如果只是为了分析的目的，创建这些服务并在机器上运行它们将是一种开销。一个简单的选择是使用原生的 Kafka 命令。它会很快见效。

### 3.1.使用`GetoffsetShell`命令

在执行本机命令之前，我们必须导航到机器上 Kafka 的根文件夹。以下命令返回主题`baeldung`上发布的消息数量:

```
$ bin/kafka-run-class.sh kafka.tools.GetOffsetShell   --broker-list localhost:9092   
--topic baeldung   | awk -F  ":" '{sum += $3} END {print "Result: "sum}'
Result: 3
```

### 3.2.使用消费者控制台

如前所述，在执行任何命令之前，我们将导航到 Kafka 的根文件夹。以下命令返回主题`baeldung`上发布的消息数量:

```
$ bin/kafka-console-consumer.sh  --from-beginning  --bootstrap-server localhost:9092 
--property print.key=true --property print.value=false --property print.partition 
--topic baeldung --timeout-ms 5000 | tail -n 10|grep "Processed a total of"
Processed a total of 3 messages
```

## 4.结论

在本文中，我们研究了获取 Kafka 主题中消息数量的技术。我们学习了一种编程技术，将所有分区分配给一个消费者，并检查最新的偏移量。

我们还看到了两种原生的卡夫卡指令技术。一个是 Kafka tools 的`GetoffsetShell` 命令。另一个是在控制台上运行一个消费者，并从头开始打印消息的数量。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220927105427/https://github.com/eugenp/tutorials/tree/master/spring-kafka)