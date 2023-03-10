# 用 Apache Kafka 进行数据建模

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-kafka-data-modeling>

## 1.概观

在本教程中，我们将使用 Apache Kafka 进入事件驱动架构的**数据建模领域。**

## 2.设置

Kafka 集群由注册到 Zookeeper 集群的多个 Kafka 代理组成。为了简单起见，**我们将使用现成的 Docker 图像和 [docker-compose](https://web.archive.org/web/20220630003934/https://baeldung.com/docker-compose)** 配置[由 confuent](https://web.archive.org/web/20220630003934/https://docs.confluent.io/5.0.0/installation/docker/docs/installation/clustered-deployment.html#docker-compose-setting-up-a-three-node-kafka-cluster)发布。

首先，让我们下载一个 3 节点 Kafka 集群的`docker-compose.yml `:

```java
$ BASE_URL="https://raw.githubusercontent.com/confluentinc/cp-docker-images/5.3.3-post/examples/kafka-cluster"
$ curl -Os "$BASE_URL"/docker-compose.yml
```

接下来，让我们旋转动物园管理员和卡夫卡经纪人节点:

```java
$ docker-compose up -d
```

最后，我们可以验证所有的卡夫卡经纪人都起来了:

```java
$ docker-compose logs kafka-1 kafka-2 kafka-3 | grep started
kafka-1_1      | [2020-12-27 10:15:03,783] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
kafka-2_1      | [2020-12-27 10:15:04,134] INFO [KafkaServer id=2] started (kafka.server.KafkaServer)
kafka-3_1      | [2020-12-27 10:15:03,853] INFO [KafkaServer id=3] started (kafka.server.KafkaServer)
```

## 3.事件基础

在我们开始为事件驱动系统进行数据建模之前，我们需要理解一些概念，比如事件、事件流、生产者-消费者和主题。

### 3.1.事件

卡夫卡世界中的事件是在领域世界中发生的事情的信息记录。它通过**将信息记录为一个键-值对**消息，以及一些其他属性，如时间戳、元信息和头。

让我们假设我们正在模拟一场国际象棋比赛；那么事件可以是移动:

[![](img/5a44b7ab8e371a36fb6e1b5afb01113e.png)](/web/20220630003934/https://www.baeldung.com/wp-content/uploads/2021/01/kafka-img1-v1.png)

我们可以注意到，**事件包含了行动者、行动以及事件发生时间**的关键信息。在这种情况下，`Player1 `是演员，动作是在`12/2020/25 00:08:30`将车从单元格`a1`移动到`a5`。

### 3.2.消息流

Apache Kafka 是一个流处理系统，它将事件捕获为消息流。在我们的国际象棋游戏中，我们可以将事件流视为玩家所走棋的日志。

在每个事件发生时，板的快照将代表其状态。通常使用传统的表模式存储对象的最新静态状态。

另一方面，事件流可以帮助我们以事件的形式捕捉两个连续状态之间的动态变化。如果我们播放一系列这些不变的事件，我们可以从一种状态转换到另一种状态。这就是事件流和传统表之间的关系，通常被称为**流表二元性**。

让我们设想棋盘上只有两个连续事件的事件流:

[![](img/86697f65e73ebd2150aaf707f37acf60.png)](/web/20220630003934/https://www.baeldung.com/wp-content/uploads/2021/01/kafka-img2-v1.png)

## 4.主题

在这一节中，我们将学习如何对通过 Apache Kafka 路由的消息进行分类。

### 4.1.分门别类

在像 Apache Kafka 这样的消息传递系统中，任何产生事件的东西通常被称为生产者。而阅读和消费这些信息的人被称为消费者。

在现实世界的场景中，每个生产者都可以生成不同类型的事件，所以如果我们希望消费者过滤与他们相关的消息，而忽略其他消息，那将是对消费者的极大浪费。

为了解决这个基本问题， **Apache Kafka 使用主题，这些主题本质上是属于一起的消息组**。因此，消费者在使用事件消息时可以更有效率。

在我们的棋盘示例中，一个主题可以用来将所有的走法分组到`chess-moves`主题下:

```java
$ docker run \
  --net=host --rm confluentinc/cp-kafka:5.0.0 \
  kafka-topics --create --topic chess-moves \
  --if-not-exists \
  --partitions 1 --replication-factor 1 \
  --zookeeper localhost:32181
Created topic "chess-moves".
```

### 4.2.生产者-消费者

现在，让我们看看生产者和消费者是如何利用卡夫卡的主题进行信息处理的。我们将使用 Kafka 发行版附带的`kafka-console-producer`和`kafka-console-consumer`实用程序来演示这一点。

让我们[旋转一个名为`kafka-producer `的容器](https://web.archive.org/web/20220630003934/https://docs.docker.com/engine/reference/commandline/run/)，我们将在其中调用生产者实用程序:

```java
$ docker run \
--net=host \
--name=kafka-producer \
-it --rm \
confluentinc/cp-kafka:5.0.0 /bin/bash
# kafka-console-producer --broker-list localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves \
--property parse.key=true --property key.separator=: 
```

同时，我们可以启动一个名为`kafka-consumer`的容器，在其中我们将调用消费者实用程序:

```java
$ docker run \
--net=host \
--name=kafka-consumer \
-it --rm \
confluentinc/cp-kafka:5.0.0 /bin/bash
# kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves --from-beginning \
--property print.key=true --property print.value=true --property key.separator=: 
```

现在，让我们通过制作人记录一些游戏动作:

```java
>{Player1 : Rook, a1->a5}
```

由于消费者是活跃的，它将通过关键字`Player1`获得该消息:

```java
{Player1 : Rook, a1->a5}
```

## 5.划分

接下来，让我们看看如何使用分区创建进一步的消息分类，并提高整个系统的性能。

### 5.1.并发

我们可以**将一个主题分成多个分区，并调用多个消费者来消费来自不同分区的消息**。通过启用这种并发行为，系统的整体性能可以得到改善。

**默认情况下，在主题创建期间支持`–bootstrap-server`选项的 Kafka 版本将创建主题**的单个分区，除非在主题创建时明确指定。但是，对于一个预先存在的主题，我们可以增加分区的数量。让我们将`chess-moves`主题的分区号设置为`3`:

```java
$ docker run \
--net=host \
--rm confluentinc/cp-kafka:5.0.0 \
bash -c "kafka-topics --alter --zookeeper localhost:32181 --topic chess-moves --partitions 3"
WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affected
Adding partitions succeeded!
```

### 5.2.分区键

在一个主题中，Kafka 使用一个分区键跨多个分区处理消息。一端，生产者使用它隐式地将消息路由到其中一个分区。另一方面，每个消费者可以从特定的分区读取消息。

默认情况下，**生成器会生成一个键的哈希值，后跟一个带有分区数量**的模数。然后，它会将消息发送到由计算出的标识符标识的分区。

让我们用`kafka-console-producer`实用程序创建新的事件消息，但这次我们将记录两个玩家的移动:

```java
# kafka-console-producer --broker-list localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves \
--property parse.key=true --property key.separator=:
>{Player1: Rook, a1 -> a5}
>{Player2: Bishop, g3 -> h4}
>{Player1: Rook, a5 -> e5}
>{Player2: Bishop, h4 -> g3}
```

现在，我们可以有两个消费者，一个从分区 1 读取数据，另一个从分区 2 读取数据:

```java
# kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves --from-beginning \
--property print.key=true --property print.value=true \
--property key.separator=: \
--partition 1
{Player2: Bishop, g3 -> h4}
{Player2: Bishop, h4 -> g3}
```

我们可以看到玩家 2 的所有移动都被记录到分区 1 中。同样，我们可以检查玩家 1 的移动是否被记录到分区 0 中。

## 6.缩放比例

我们如何概念化主题和分区对于水平扩展至关重要。一方面，**主题更像是数据**的预定义分类。另一方面，分区是动态发生的数据的动态分类。

此外，我们在一个主题中可以配置多少个分区是有实际限制的。这是因为**每个分区都被映射到代理节点的文件系统中的一个目录**。**当我们增加分区数量时，我们也增加了操作系统上打开的文件句柄**的数量。

根据经验， **[Confluent 的专家建议将每个代理的分区数量](https://web.archive.org/web/20220630003934/https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster/)限制为`100` x `b` x `r`** ，其中`b`是 Kafka 集群中的代理数量，`r`是复制因子。

## 7.结论

在本文中，我们**使用了一个 Docker 环境来介绍使用 Apache Kafka** 进行消息处理的系统的数据建模基础。有了对事件、主题和分区的基本理解，我们现在准备概念化事件流并进一步使用这个架构范例。