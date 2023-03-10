# 列出 Kafka 主题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kafka-list-topics>

## 1.概观

在这个快速教程中，我们将学习如何列出 Apache Kafka 集群中的所有主题。

首先，我们将建立一个单节点 Apache Kafka 和 [Zookeeper](/web/20221109150434/https://www.baeldung.com/java-zookeeper) 集群。然后，我们将询问该集群的主题。

## 2.建立卡夫卡

在列出 Kafka 集群中的所有主题之前，让我们分三步设置一个测试单节点 Kafka 集群:

*   下载卡夫卡和动物园管理员
*   启动 Zookeeper 服务
*   启动 Kafka 服务

首先，我们应该确保从 Apache 网站下载正确的 Kafka 版本。下载完成后，我们应该提取下载的归档文件:

```java
$ tar xvf kafka_2.13-2.6.0.tgz
```

Kafka 正在使用 Apache Zookeeper 来管理它的集群元数据，所以我们需要一个正在运行的 Zookeeper 集群。

出于测试目的，我们可以使用`bin`目录中的`zookeeper-server-start.sh`脚本运行一个单节点 Zookeeper 实例:

```java
$ cd kafka_2.13-2.6.0 # extracted directory
$ ./bin/zookeeper-server-start.sh config/zookeeper.properties
```

这将启动 Zookeeper 服务监听端口 2181。之后，我们可以使用另一个脚本来运行 Kafka 服务器:

```java
$ ./bin/kafka-server-start.sh config/server.properties
```

一段时间后，一个卡夫卡经纪人将开始。让我们[向这个简单的集群添加几个主题](https://web.archive.org/web/20221109150434/https://kafka.apache.org/documentation/#basic_ops_add_topic):

```java
$ bin/kafka-topics.sh --create --topic users.registrations --replication-factor 1 \
  --partitions 2  --zookeeper localhost:2181
$ bin/kafka-topics.sh --create --topic users.verfications --replication-factor 1 \
  --partitions 2  --zookeeper localhost:2181
```

现在一切都准备好了，接下来看看如何列出卡夫卡的话题。

## 3.列出主题

要列出一个集群中的所有 Kafka 主题，我们可以使用下载的 Kafka 发行版中捆绑的`bin/kafka-topics.sh ` shell 脚本。**我们所要做的就是传递`–list `选项，以及关于集群**的信息。例如，我们可以传递动物园管理员服务地址:

```java
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
users.registrations
users.verfications
```

如上所示，`–list`选项告诉`kafka-topics.sh` shell 脚本列出所有主题。在这种情况下，我们有两个主题来存储用户相关的事件。如果集群中没有主题，那么该命令将无声地返回，没有任何结果。

另外，**为了与 Kafka 集群对话，我们需要使用`–zookeeper `选项**传递 Zookeeper 服务 URL。

**甚至可以使用`–bootstrap-server`选项**直接传递 Kafka 集群地址:

```java
$ ./bin/kafka-topics.sh --bootstrap-server=localhost:9092 --list
users.registrations
users.verfications
```

我们的单实例 Kafka 集群监听 9092 端口，因此我们将`“localhost:9092”`指定为引导服务器。简单地说，引导服务器是 Kafka 代理。

如果我们不传递与 Kafka 集群对话所需的信息，`kafka-topics.sh` shell 脚本将报错:

```java
$ ./bin/kafka-topics.sh --list
Exception in thread "main" java.lang.IllegalArgumentException: Only one of --bootstrap-server or --zookeeper must be specified
        at kafka.admin.TopicCommand$TopicCommandOptions.checkArgs(TopicCommand.scala:721)
        at kafka.admin.TopicCommand$.main(TopicCommand.scala:52)
        at kafka.admin.TopicCommand.main(TopicCommand.scala)
```

如上所示，shell 脚本要求我们传递`–bootstrap-server`或`–zookeeper`选项。

## 4.主题详细信息

一旦我们找到了一个主题列表，我们就可以浏览一个特定主题的细节。为此，**我们可以使用`“` `–describe –topic <topic name>”`选项的组合**:

```java
$ ./bin/kafka-topics.sh --bootstrap-server=localhost:9092 --describe --topic users.registrations
Topic: users.registrations      PartitionCount: 2       ReplicationFactor: 1    Configs: segment.bytes=1073741824
        Topic: users.registrations      Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: users.registrations      Partition: 1    Leader: 0       Replicas: 0     Isr: 0
```

这些详细信息包括关于指定主题的信息，例如分区和副本的数量等。与其他命令类似，我们必须传递集群信息或 Zookeeper 地址。否则，我们将无法与集群通信。

## 5.结论

在这篇简短的文章中，我们学习了如何列出 Kafka 集群中的所有主题。在这个过程中，我们看到了如何建立一个简单的单节点 Kafka 集群。

目前，Apache Kafka 使用 Zookeeper 来管理其集群元数据。然而，作为 KIP-500 的一部分，这将很快改变，因为 Kafka 将拥有自己的元数据定额。