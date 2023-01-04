# 在 Apache Kafka 中配置消息保持期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-message-retention>

## 1.概观

当生成器向 Apache Kafka 发送消息时，它会将消息附加到日志文件中，并在配置的时间段内保留该消息。

在本教程中，我们将学习**为 Kafka 主题**配置基于时间的消息保留属性。

## 2.基于时间的保留

有了保持期属性，**消息就有了 TTL(生存时间)**。到期后，邮件将被标记为删除，从而释放磁盘空间。

相同的保持期属性适用于给定 Kafka 主题中的所有消息。此外，**我们可以在主题创建之前设置这些属性，或者在运行时为预先存在的主题**修改它们。

在接下来的小节中，我们将学习如何通过代理配置来调整这一点，为新主题设置保留期，并通过**主题级配置在运行时控制它**。

## 3.服务器级配置

Apache Kafka 支持**服务器级保留策略，我们可以通过配置三个基于时间的配置属性**中的一个来调整该策略:

*   `log.retention.hours`
*   `log.retention.minutes`
*   `log.retention.ms`

理解 Kafka 用较高的精度值覆盖较低的精度值是很重要的。所以， **`log.retention.ms`会优先于**。

### 3.1.基础

首先，让我们通过从 [Apache Kafka 目录](https://web.archive.org/web/20220524030209/https://kafka.apache.org/documentation/#quickstart)中执行 [`grep`](/web/20220524030209/https://www.baeldung.com/linux/grep-sed-awk-differences#grep) 命令来检查保留的默认值:

```
$ grep -i 'log.retention.[hms].*\=' config/server.properties
log.retention.hours=168 
```

我们可以注意到这里的**默认保留时间是七天**。

要将消息仅保留十分钟，我们可以在`config/server.properties`中设置`log.retention.minutes`属性的值:

```
log.retention.minutes=10 
```

### 3.2.新主题的保持期

Apache Kafka 包包含几个 shell 脚本，我们可以使用它们来执行管理任务。我们将使用它们来**创建一个助手脚本，`functions.sh`** ，我们将在本教程`.`中使用它

让我们从在`functions.sh`到**中添加两个函数开始，分别创建一个主题并描述其配置**:

```
function create_topic {
    topic_name="$1"
    bin/kafka-topics.sh --create --topic ${topic_name} --if-not-exists \
      --partitions 1 --replication-factor 1 \
      --zookeeper localhost:2181
}

function describe_topic_config {
    topic_name="$1"
    ./bin/kafka-configs.sh --describe --all \
      --bootstrap-server=0.0.0.0:9092 \
      --topic ${topic_name}
}
```

接下来，让我们创建两个独立的脚本，`create-topic.sh` 和`get-topic-retention-time.sh`:

```
bash-5.1# cat create-topic.sh
#!/bin/bash
. ./functions.sh
topic_name="$1"
create_topic "${topic_name}"
exit $? 
```

```
bash-5.1# cat get-topic-retention-time.sh
#!/bin/bash
. ./functions.sh
topic_name="$1"
describe_topic_config "${topic_name}" | awk 'BEGIN{IFS="=";IRS=" "} /^[ ]*retention.ms/{print $1}'
exit $? 
```

我们必须注意到，`describe_topic_config`将给出为主题配置的所有属性。因此，我们使用 [`awk`](/web/20220524030209/https://www.baeldung.com/linux/awk-guide) 一行程序为`retention.ms`属性添加一个过滤器。

最后，让我们[启动 Kafka 环境](https://web.archive.org/web/20220524030209/https://kafka.apache.org/documentation/#quickstart_startserver)并验证新样本主题的保留期配置:

```
bash-5.1# ./create-topic.sh test-topic
Created topic test-topic.
bash-5.1# ./get-topic-retention-time.sh test-topic
retention.ms=600000
```

一旦创建并描述了主题，我们会注意到 **`retention.ms`被设置为`600000`** (十分钟)。这实际上是从我们之前在`server.properties`文件中定义的`log.retention.minutes`属性中派生出来的**。**

## 4.主题级配置

**一旦代理服务器启动，`log.retention.{hours|minutes|ms}`服务器级属性就变成只读的**。另一方面，我们可以访问`retention.ms `属性，我们可以在主题级别对其进行调优。

让我们在我们的`functions.sh`脚本中添加一个方法来配置主题的属性:

```
function alter_topic_config {
    topic_name="$1"
    config_name="$2"
    config_value="$3"
    ./bin/kafka-configs.sh --alter \
      --add-config ${config_name}=${config_value} \
      --bootstrap-server=0.0.0.0:9092 \
      --topic ${topic_name}
}
```

然后，我们可以在一个`alter-topic-config.sh`脚本中使用它:

```
#!/bin/sh
. ./functions.sh

alter_topic_retention_config $1 $2 $3
exit $?
```

最后，让我们将`test-topic`的保留时间设置为五分钟，并进行验证:

```
bash-5.1# ./alter-topic-config.sh test-topic retention.ms 300000
Completed updating config for topic test-topic.

bash-5.1# ./get-topic-retention-time.sh test-topic
retention.ms=300000
```

## 5.确认

到目前为止，我们已经看到了如何在 Kafka 主题中配置消息的保持期。是时候验证消息在保留超时后是否确实过期了。

### 5.1.生产者-消费者

让我们在内部的`functions.sh.` 中添加`produce_message`和`consume_message`函数，它们分别使用`kafka-console-producer.sh`和`kafka-console-consumer.sh`来产生/消费一条消息:

```
function produce_message {
    topic_name="$1"
    message="$2"
    echo "${message}" | ./bin/kafka-console-producer.sh \
    --bootstrap-server=0.0.0.0:9092 \
    --topic ${topic_name}
}

function consume_message {
    topic_name="$1"
    timeout="$2"
    ./bin/kafka-console-consumer.sh \
    --bootstrap-server=0.0.0.0:9092 \
    --from-beginning \
    --topic ${topic_name} \
    --max-messages 1 \
    --timeout-ms $timeout
}
```

我们必须注意到**消费者总是从头开始阅读信息**，因为我们需要一个**阅读卡夫卡**中任何可用信息的消费者。

接下来，让我们创建一个独立的消息生成器:

```
bash-5.1# cat producer.sh
#!/bin/sh
. ./functions.sh
topic_name="$1"
message="$2"

produce_message ${topic_name} ${message}
exit $?
```

最后，让我们有一个独立的消息消费者:

```
bash-5.1# cat consumer.sh
#!/bin/sh
. ./functions.sh
topic_name="$1"
timeout="$2"

consume_message ${topic_name} $timeout
exit $?
```

### 5.2.消息过期

现在我们已经准备好了基本的设置，让我们生成一条消息并立即使用它两次:

```
bash-5.1# ./producer.sh "test-topic-2" "message1"
bash-5.1# ./consumer.sh test-topic-2 10000
message1
Processed a total of 1 messages
bash-5.1# ./consumer.sh test-topic-2 10000
message1
Processed a total of 1 messages
```

因此，我们可以看到消费者在重复消费任何可用的信息。

现在，让我们引入一个五分钟的睡眠延迟，然后尝试使用该消息:

```
bash-5.1# sleep 300 && ./consumer.sh test-topic 10000
[2021-02-06 21:55:00,896] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 0 messages
```

正如预期的那样，**消费者没有找到任何要消费的消息，因为消息已经超过了它的保持期**。

## 6.限制

在内部，Kafka 代理维护另一个名为`log.retention.check.interval.ms. `的属性。该属性决定检查消息是否过期的频率。

因此，为了保持保留策略的有效性，我们必须确保对于任何给定的主题，`log.retention.check.interval.ms `的值低于`retention.ms `的属性值。

## 7.结论

在本教程中，我们**探索了 Apache Kafka，以了解消息**的基于时间的保留策略。在这个过程中，我们创建了简单的 shell 脚本来简化管理活动。后来，我们创建了一个独立的消费者和生产者来验证消息在保持期后是否过期。