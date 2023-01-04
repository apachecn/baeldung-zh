# 列出 Kafka 消费者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/listing-kafka-consumers>

## 1.概观

在这个快速教程中，我们将学习如何列出卡夫卡的消费群体，并一窥他们的细节。

## 2.先决条件

为了运行本教程中的例子，我们需要一个 Kafka 集群来发送我们的请求。这可以是运行在生产环境中的成熟 Kafka 集群，也可以是特定于测试的单实例 Kafka 集群。

为了简单起见，我们假设我们有一个监听端口 9092 的单节点集群，一个 [Zookeeper](/web/20220628091626/https://www.baeldung.com/java-zookeeper) 实例监听本地主机上的 2181 端口。

此外，请注意，我们正在从 Kafka 安装目录运行所有示例命令。

## 3.添加主题和消费者

在列出特定 Kafka 集群上的消费者之前，让我们先使用`kafka-topics.sh` shell 脚本[添加几个主题](https://web.archive.org/web/20220628091626/https://kafka.apache.org/documentation/#basic_ops_add_topic):

```java
$ ./bin/kafka-topics.sh --create --topic users.registrations --replication-factor 1 \ 
  --partitions 2 --zookeeper localhost:2181
$ ./bin/kafka-topics.sh --create --topic users.verfications --replication-factor 1 \ 
  --partitions 2 --zookeeper localhost:2181
```

现在，我们需要**添加一些消费群体**。最简单的方法是**使用捆绑在 Kafka 发行版中的控制台消费者**:

```java
$ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.registrations --group new-user
$ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.registrations --group new-user
```

这里，我们使用了`kafka-console-consumer.sh ` shell 脚本来添加两个收听同一主题的消费者。**这些消费者在同一个组中，所以来自主题分区的消息将在组的成员中传播**。这样我们可以在卡夫卡中实现[竞争消费者](https://web.archive.org/web/20220628091626/https://www.enterpriseintegrationpatterns.com/patterns/messaging/CompetingConsumers.html)模式。

让我们从另一个话题开始:

```java
$ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic users.verifications
```

由于我们没有为消费者指定一个组，**控制台消费者创建了一个新组，它自己是唯一的成员**。

我们将在下一节看到这个新组，在这里我们将学习如何在 Kafka 集群上列出消费者和消费者组。

## 4.列出消费者

要列出 Kafka 集群中的消费者，我们可以使用`kafka-consumer-groups.sh ` shell 脚本。**`–list `选项将列出所有的消费群体**:

```java
$ ./bin/kafka-consumer-groups.sh --list --bootstrap-server localhost:9092
new-user
console-consumer-40123
```

除了`–list `选项，我们还传递了`–bootstrap-server `选项来指定 Kafka 集群地址。我们在两组中有三个消费者，所以结果只包含两组。

**要查看第一组**的成员，我们可以使用`“–group <name> –describe –members”` 选项:

```java
$ ./bin/kafka-consumer-groups.sh --describe --group new-user --members --bootstrap-server localhost:9092
GROUP           CONSUMER-ID                    HOST            CLIENT-ID            #PARTITIONS
new-user        consumer-new-user-1-b90...     /127.0.0.1      consumer-new-user-1  1
new-user        consumer-new-user-1-af8...     /127.0.0.1      consumer-new-user-1  1
```

在这里，我们可以看到在我们的`new-user`组中有两个单独的消费者，每个人从一个分区消费。

如果我们省略`–members `选项，它将列出组中的用户、每个用户正在监听的分区号以及它们的偏移量:

```java
$ ./bin/kafka-consumer-groups.sh --describe --group new-user --bootstrap-server localhost:9092
GROUP           TOPIC                       PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG          
new-user        users.registrations         1          3               3               0              
new-user        users.registrations         0          5               5               0 
```

还有一点需要注意，这个命令需要集群或引导服务器地址。**如果我们忽略集群连接信息，shell 脚本将抛出一个错误**:

```java
$ ./bin/kafka-consumer-groups.sh --list
Missing required argument "[bootstrap-server]"
// truncated
```

## 5.结论

在这个简短的教程中，我们一开始增加了几个卡夫卡的话题和消费群体。然后，我们学习了如何列出消费者群体并查看每个群体的详细信息。