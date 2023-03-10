# Apache Pulsar 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-pulsar>

## 1.介绍

**[Apache Pulsar](https://web.archive.org/web/20220626085600/https://pulsar.apache.org/) 是雅虎**开发的基于发布/订阅的分布式开源消息系统。

它的创建是为了支持雅虎的关键应用，如雅虎邮箱、雅虎财经、雅虎体育等。然后在 2016 年，在 Apache 软件基金会下开源。

## 2.体系结构

**Pulsar 是一个多租户、高性能的服务器到服务器消息传递解决方案**。它由一组经纪人和博彩公司以及内置的 [`Apache ZooKeeper`](https://web.archive.org/web/20220626085600/https://zookeeper.apache.org/) 组成，用于配置和管理。赌注登记经纪人来自`[Apache BookKeeper](https://web.archive.org/web/20220626085600/https://bookkeeper.apache.org/)`，为消息提供存储，直到它们被消费。

在一个集群中，我们将拥有:

*   多个集群代理处理来自生产者的传入消息，并将消息分派给消费者
*   Apache BookKeeper 支持消息持久性
*   Apache ZooKeeper 来存储集群配置

为了更好地理解这一点，让我们看看来自[文档](https://web.archive.org/web/20220626085600/https://pulsar.apache.org/docs/en/concepts-architecture-overview/)的架构图:

[![pulsar system architecture](img/bc6cd4846c2b16f0a844cecd9989690c.png)](/web/20220626085600/https://www.baeldung.com/wp-content/uploads/2018/10/pulsar-system-architecture.png)

## 3.关键特征

让我们先来快速了解一些关键特性:

*   内置对多个集群的支持
*   支持跨多个集群的邮件地理复制
*   多种订阅模式
*   可扩展到数百万个主题
*   使用 Apache BookKeeper 来保证消息传递。
*   低延迟

现在，让我们详细讨论一些关键特性。

### 3.1.消息传递模型

该框架提供了灵活的消息传递模型。一般来说，消息传递体系结构有两种消息传递模型，即队列和发布者/订阅者。发布者/订阅者是一个广播消息传递系统，其中消息被发送给所有消费者。另一方面，排队是点对点的通信。

**Pulsar 在一个通用 API** 中结合了这两个概念。发布者将消息发布到不同的主题。然后将这些消息广播给所有订阅。

消费者订阅获取消息。该库允许消费者选择不同的方式来使用同一订阅中的消息，包括独占、共享和故障转移。我们将在后面的小节中详细讨论这些订阅类型。

### 3.2.部署模式

**Pulsar 内置了对不同环境部署的支持**。这意味着我们可以在标准的本地机器上使用它，或者在 Kubernetes 集群、Google 或 AWS 云中部署它。

出于开发和测试目的，它可以作为单个节点来执行。在这种情况下，所有组件(broker、BookKeeper 和 ZooKeeper)都在一个进程中运行。

### 3.3.地理复制

该库为数据的地理复制提供现成的支持。我们可以通过配置不同的地理区域来实现多个集群之间的消息复制。

邮件数据几乎是实时复制的。在跨集群的网络故障的情况下，数据总是安全的并且存储在簿记员中。复制系统继续重试，直到复制成功。

**地理复制功能还允许组织跨不同的云提供商部署 Pulsar 并复制数据**。这有助于他们避免使用专有的云提供商 API。

### 3.4.持久

**Pulsar 读取并确认数据后，保证无数据丢失**。数据持久性与配置用于存储数据的磁盘数量有关。

Pulsar 通过使用存储节点中运行的 bookies (Apache BookKeeper 实例)来确保持久性。每当赌注登记经纪人收到一条消息，它会在内存中保存一份副本，并将数据写入 WAL(预写日志)。该日志的工作方式与数据库墙相同。博彩公司根据数据库交易原则运作，确保即使在机器故障的情况下数据也不会丢失。

除此之外，Pulsar 还可以承受多个节点故障。该库将数据复制到多个博彩公司，然后向生产者发送确认消息。这种机制保证了即使在多个硬件故障的情况下也不会丢失数据。

## 4.单节点设置

现在让我们看看如何设置 Apache Pulsar 的单节点集群。

**Apache 还提供了一个简单的[客户端 API](https://web.archive.org/web/20220626085600/https://pulsar.apache.org/docs/en/concepts-clients/) ，绑定了 Java、Python 和 C++** 。稍后，我们将创建一个简单的 Java 生成器和订阅示例。

### 4.1.装置

Apache Pulsar 以二进制发行版的形式提供。让我们从下载开始:

```java
wget https://archive.apache.org/dist/incubator/pulsar/pulsar-2.1.1-incubating/apache-pulsar-2.1.1-incubating-bin.tar.gz
```

下载完成后，我们可以对 zip 文件进行解压缩。未归档的发行版将包含`bin, conf, example, licenses`和 `lib`文件夹。

之后，我们需要下载内置的连接器。这些现在作为一个单独的包提供:

```java
wget https://archive.apache.org/dist/incubator/pulsar/pulsar-2.1.1-incubating/apache-pulsar-io-connectors-2.1.1-incubating-bin.tar.gz
```

让我们将连接器解压缩，并将`Connectors `文件夹复制到 Pulsar 文件夹中。

### 4.2.启动实例

要启动独立实例，我们可以执行:

```java
bin/pulsar standalone
```

## 5.Java 客户端

现在我们将创建一个 Java 项目来产生和使用消息。我们还将为不同的订阅类型创建示例。

### 5.1.设置项目

我们首先将 [pulsar-client](https://web.archive.org/web/20220626085600/https://search.maven.org/search?q=a:pulsar-client%20AND%20g:org.apache.pulsar) 依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>org.apache.pulsar</groupId>
    <artifactId>pulsar-client</artifactId>
    <version>2.1.1-incubating</version>
</dependency>
```

### 5.2.生产者

让我们继续创建一个`Producer`示例。在这里，我们将创建一个主题和一个生成器。

**首先，我们需要创建一个`PulsarClient `，它将使用自己的协议连接到特定主机和端口**上的 Pulsar 服务。许多生产者和消费者可以共享一个客户端对象。

现在，我们将创建一个具有特定主题名称的`Producer`:

```java
private static final String SERVICE_URL = "pulsar://localhost:6650";
private static final String TOPIC_NAME = "test-topic"; 
```

```java
PulsarClient client = PulsarClient.builder()
  .serviceUrl(SERVICE_URL)
  .build();

Producer<byte[]> producer = client.newProducer()
  .topic(TOPIC_NAME)
  .compressionType(CompressionType.LZ4)
  .create();
```

生产者将发送 5 条消息:

```java
IntStream.range(1, 5).forEach(i -> {
    String content = String.format("hi-pulsar-%d", i);

    Message<byte[]> msg = MessageBuilder.create()
      .setContent(content.getBytes())
      .build();
    MessageId msgId = producer.send(msg);
});
```

### 5.3.消费者

接下来，我们将创建消费者来获取生产者创建的消息。消费者也需要相同的`PulsarClient`来连接我们的服务器:

```java
Consumer<byte[]> consumer = client.newConsumer()
  .topic(TOPIC_NAME)
  .subscriptionType(SubscriptionType.Shared)
  .subscriptionName(SUBSCRIPTION_NAME)
  .subscribe(); 
```

在这里，我们创建了一个订阅类型为`. `的客户端，它允许多个用户连接到同一个订阅并获取消息。

### 5.4.消费者的订阅类型

在上面的消费者示例中，**我们创建了一个类型为`shared`的订阅。我们还可以创建`exclusive`和`failover`订阅。**

`exclusive`订阅只允许一个消费者订阅。

另一方面，f `ailover subscription`允许用户定义后备消费者，以防一个消费者失败，如下图所示:

[![pulsar subscription modes](img/006e64680a349bb61e194ad37047bc86.png)](/web/20220626085600/https://www.baeldung.com/wp-content/uploads/2018/10/pulsar-subscription-modes.png)

## 6.结论

在本文中，我们重点介绍了 Pulsar 消息传递系统的特性，如消息传递模型、地理复制和强大的耐用性保证。

我们还学习了如何设置单个节点以及如何使用 Java 客户端。

一如既往，本教程的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220626085600/https://github.com/eugenp/tutorials/tree/master/apache-libraries)