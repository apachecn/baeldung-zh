# 清除阿帕奇卡夫卡主题指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-purge-topic>

## 1.概观

在本文中，我们将探索一些从 Apache Kafka 主题中清除数据的**策略。**

## 2.清理场景

在我们学习清理数据的策略之前，让我们先了解一个需要清理活动的简单场景。

### 2.1.方案

Apache Kafka **中的消息在配置的[保留时间](/web/20221102005738/https://www.baeldung.com/kafka-message-retention)** 后自动过期。尽管如此，在一些情况下，我们可能希望立即删除消息。

让我们假设在 Kafka 主题中产生消息的应用程序代码中引入了一个缺陷。当一个 bug 修复被集成时，我们已经在 Kafka 主题中有许多**损坏的消息可供消费。**

这样的问题在开发环境中是最常见的，我们想要快速的结果。因此，批量删除邮件是合理的做法。

### 2.2.模拟

为了模拟这个场景，让我们从从 Kafka 安装目录创建一个`purge-scenario` 主题开始:

```java
$ bin/kafka-topics.sh \
  --create --topic purge-scenario --if-not-exists \
  --partitions 2 --replication-factor 1 \
  --zookeeper localhost:2181
```

接下来，让我们使用 [`shuf`](/web/20221102005738/https://www.baeldung.com/linux/read-random-line-from-file#using-shuf) 命令来让**生成随机数据并将其馈送给`kafka-console-producer.sh`** 脚本:

```java
$ /usr/bin/shuf -i 1-100000 -n 50000000 \
  | tee -a /tmp/kafka-random-data \
  | bin/kafka-console-producer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --topic purge-scenario
```

我们必须注意，我们已经使用了 [`tee`](/web/20221102005738/https://www.baeldung.com/linux/tee-command) 命令来保存仿真数据以备后用。

最后，让我们验证一个消费者可以使用来自主题的消息:

```java
$ bin/kafka-console-consumer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario \
  --max-messages 3
76696
49425
1744
Processed a total of 3 messages
```

## 3.消息过期

在`purge-scenario `主题中生成的消息将有一个七天的[默认保留期](/web/20221102005738/https://www.baeldung.com/kafka-message-retention#basics)。为了清除消息，我们可以**暂时将 [`retention.ms`](https://web.archive.org/web/20221102005738/http://log.retention.minutes/) 主题级属性**重置为 10 秒，然后等待消息过期:

```java
$ bin/kafka-configs.sh --alter \
  --add-config retention.ms=10000 \
  --bootstrap-server=0.0.0.0:9092 \
  --topic purge-scenario \
  && sleep 10
```

接下来，让我们验证主题中的消息是否已过期:

```java
$ bin/kafka-console-consumer.sh  \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario \
  --max-messages 1 --timeout-ms 1000
[2021-02-28 11:20:15,951] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 0 messages 
```

最后，我们可以将主题的保留期恢复为七天:

```java
$ bin/kafka-configs.sh --alter \
  --add-config retention.ms=604800000 \
  --bootstrap-server=0.0.0.0:9092 \
  --topic purge-scenario
```

使用这种方法，Kafka 将为`purge-scenario`主题清除所有分区中的消息。

## 4.选择性记录删除

有时，我们可能希望从特定主题的一个或多个分区中有选择地删除记录。我们可以通过使用`kafka-delete-records.sh`脚本来满足这样的需求。

首先，我们需要在`delete-config.json`配置文件中指定分区级别的偏移量。

让我们使用`offset=-1`清除来自`partition=1`的所有消息:

```java
{
  "partitions": [
    {
      "topic": "purge-scenario",
      "partition": 1,
      "offset": -1
    }
  ],
  "version": 1
}
```

接下来，让我们继续删除记录:

```java
$ bin/kafka-delete-records.sh \
  --bootstrap-server localhost:9092 \
  --offset-json-file delete-config.json
```

我们可以验证我们仍然能够从`partition=0`开始读取:

```java
$ bin/kafka-console-consumer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario --partition=0 \
  --max-messages 1 --timeout-ms 1000
  44017
  Processed a total of 1 messages
```

但是，当我们从`partition=1`开始读取时，将没有要处理的记录:

```java
$ bin/kafka-console-consumer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario \
  --partition=1 \
  --max-messages 1 --timeout-ms 1000
[2021-02-28 11:48:03,548] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 0 messages 
```

## 5.删除并重新创建主题

清除 Kafka 主题的所有消息的另一个解决方法是删除并重新创建它。然而，只有当我们在启动 Kafka 服务器时将`delete.topic.enable`属性设置为 `**true**` **时，这才有可能**:****

```java
$ bin/kafka-server-start.sh config/server.properties \
  --override delete.topic.enable=true
```

要删除主题，我们可以使用`kafka-topics.sh`脚本:

```java
$ bin/kafka-topics.sh \
  --delete --topic purge-scenario \
  --zookeeper localhost:2181
Topic purge-scenario is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true. 
```

我们来列个题目验证一下:

```java
$ bin/kafka-topics.sh --zookeeper localhost:2181 --list
```

确认该主题不再列出后，我们现在可以继续创建它。

## 6.结论

在本教程中，我们模拟了一个需要清除 Apache Kafka 主题的场景。此外，我们**探索了** **多种策略来完全或有选择地跨分区**清除它。