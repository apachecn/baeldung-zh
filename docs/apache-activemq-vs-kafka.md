# Apache ActiveMQ 与 Kafka

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-activemq-vs-kafka>

## 1.概观

在分布式体系结构中，应用程序通常需要在它们之间交换数据。一方面，这可以通过彼此直接交流来实现。另一方面，为了达到高可用性和分区容忍度，并获得应用程序之间的松散耦合，消息传递是一个合适的解决方案。

因此，我们可以在多个产品中进行选择。Apache 基金会提供了 ActiveMQ 和 Kafka，我们将在本文中对它们进行比较。

## 2.一般事实

### 2.1.活动 MQ

Active MQ 是传统的消息代理之一，其目标是确保应用程序之间以安全可靠的方式进行数据交换。它处理少量数据，因此专门用于定义明确的消息格式和事务性消息传递。

我们必须注意到，除了这个“经典”版本之外，还有另一个版本:Active MQ Artemis。这个下一代代理基于 HornetQ，其代码由 RedHat 在 2015 年提供给 Apache 基金会。在[活跃的 MQ 网站](https://web.archive.org/web/20220811180315/https://activemq.apache.org/)上，据说:

> 一旦 Artemis 达到与“经典”代码库相当的功能水平，它将成为 ActiveMQ 的下一个主要版本。

所以，为了比较，我们需要考虑两个版本。我们将通过使用术语`“Active MQ”`和`“Artemis”`来区分它们。

### 2.2.卡夫卡

与 Active MQ 相反，Kafka 是一个分布式系统，用于处理大量数据。我们可以将它用于传统的信息传递以及:

*   网站活动跟踪
*   韵律学
*   日志聚合
*   流处理
*   活动采购
*   提交日志

随着使用微服务构建的典型云架构的出现，这些需求变得非常重要。

### 2.3.JMS 的角色和消息传递的发展

Java 消息服务(JMS)是用于在 Java EE 应用程序中发送和接收消息的通用 API。它是消息传递系统早期发展的一部分，今天仍然是一个标准。在雅加达 EE，它被采用为`Jakarta Messaging`。因此，理解核心概念可能会有所帮助:

*   一个 Java 原生但独立于供应商的 API
*   需要一个`JCA Resource Adapter`来实现特定于供应商的通信协议
*   消息目标模型:
    *   `Queues` ( `P2P`)确保消息排序和一次性消息处理，即使在多个消费者的情况下
    *   `Topics` ( `PubSub`)作为发布-订阅模式的实现，这意味着多个消费者将在订阅主题期间接收消息
*   消息格式:
    *   作为代理处理的标准化元信息(如优先级或截止日期)
    *   `Properties`作为消费者可用于消息处理的非标准化元信息
    *   包含有效负载——JMS 的`Body` 声明了五种类型的消息，但这仅与使用 API 相关，与此比较无关

**然而，进化朝着一个开放和独立的方向发展——独立于消费者和生产者的平台，独立于消息代理的供应商。**有一些协议定义了它们自己的目的地模型:

*   [AMQP](https://web.archive.org/web/20220811180315/https://www.amqp.org/)——用于独立于供应商的消息传递的二进制协议——使用`generic nodes`
*   [MQTT](https://web.archive.org/web/20220811180315/https://mqtt.org/)——嵌入式系统和物联网的轻量级二进制协议——用途`topics`
*   [STOMP](https://web.archive.org/web/20220811180315/https://stomp.github.io/)——一种简单的基于文本的协议，甚至允许从浏览器发送消息——使用`generic destinations`

**另一项发展是通过云架构的推广，根据“一劳永逸”原则，将之前可靠的单个消息传输(“传统消息”)添加到大量数据的处理中。**我们可以说，主动 MQ 与卡夫卡的比较，就是这两种进路的典范代表的比较。例如，卡夫卡的替代品可能是 [NATS](https://web.archive.org/web/20220811180315/https://nats.io/) 。

## 3.比较

在这一节中，我们将比较 Active MQ 和 Kafka 之间最有趣的架构和开发特性。

### 3.1.消息目的地模型、协议和 API

Active MQ 完全实现了 JMS 消息目的地模型`Queues`和`Topics`，并将 AMQP、MQTT 和 STOMP 消息映射到它们。例如，STOMP 消息被映射到`Topic`内的 JMS `BytesMessage`。此外，它支持 [OpenWire](https://web.archive.org/web/20220811180315/https://activemq.apache.org/openwire) ，允许跨语言访问活动 MQ。

Artemis 定义了独立于标准 API 和协议的自己的消息目的地模型，并且还需要将它们映射到该模型:

*   `Messages`被发送到一个`Address`，它被赋予一个唯一的名称、一个`Routing Type`和零个或多个`Queues`。
*   一个`Routing Type`决定消息如何从一个地址路由到绑定到该地址的队列。定义了两种类型:
    *   `ANYCAST`:消息被路由到地址上的单个队列
    *   `MULTICAST`:消息被路由到地址上的每个队列

卡夫卡只定义了`Topics`，由多个`Partitions`(至少 1 个)和`Replicas`组成，可以放在不同的经纪人身上。找到划分主题的最佳策略是一个挑战。我们必须注意到:

*   一条消息被分发到一个分区中。
*   仅对一个分区内的消息确保排序。
*   默认情况下，后续消息在主题的分区中循环分发。
*   如果我们使用消息键，那么具有相同键的消息将到达相同的分区。

卡夫卡有自己的[API](https://web.archive.org/web/20220811180315/https://kafka.apache.org/documentation/#api)。尽管 JMS 也有一个[资源适配器，但我们应该意识到这些概念并不完全兼容。官方不支持 AMQP、MQTT 和 STOMP，但是有用于](https://web.archive.org/web/20220811180315/https://docs.payara.fish/enterprise/docs/documentation/ecosystem/cloud-connectors/apache-kafka.html) [AMQP](https://web.archive.org/web/20220811180315/https://github.com/ppatierno/kafka-connect-amqp) 和 [MQTT](https://web.archive.org/web/20220811180315/https://github.com/johanvandevenne/kafka-connect-mqtt) 的[连接器](/web/20220811180315/https://www.baeldung.com/kafka-connectors-guide)。

### 3.2.消息格式和处理

Active MQ 支持 JMS 标准消息格式，包括消息头、属性和消息体(如上所述)。代理必须维护每个消息的交付状态，这导致了较低的吞吐量。由于 JMS 支持，消费者可以从目的地同步拉取消息，或者消息可以由代理异步推送。

卡夫卡没有定义任何信息格式——这完全是制作者的责任。每个消息没有任何交付状态，只有每个消费者和分区的一个`Offset`。一个`Offset`是传递的最后一条消息的索引。这不仅更快，而且还允许通过重置偏移量来重新发送消息，而无需询问生产者。

### 3.3.Spring 和 CDI 集成

JMS 是 Java/Jakarta EE 标准，因此完全集成到 Java/Jakarta EE 应用程序中。因此，应用服务器很容易管理到活动 MQ 和 Artemis 的连接。有了 Artemis，我们甚至可以使用一个[嵌入式代理](https://web.archive.org/web/20220811180315/https://activemq.apache.org/components/artemis/documentation/latest/cdi-integration.html)。对于 Kafka，托管连接仅在使用 JMS 的[资源适配器](https://web.archive.org/web/20220811180315/https://docs.payara.fish/enterprise/docs/documentation/ecosystem/cloud-connectors/apache-kafka.html)或[Eclipse micro profile Reactive](https://web.archive.org/web/20220811180315/https://dzone.com/articles/using-jakarta-eemicroprofile-to-connect-to-apache)时可用。

Spring 集成了 [JMS](https://web.archive.org/web/20220811180315/https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms) 以及 [AMQP](https://web.archive.org/web/20220811180315/https://spring.io/projects/spring-amqp) 、 [MQTT、](https://web.archive.org/web/20220811180315/https://docs.spring.io/spring-integration/reference/html/mqtt.html#mqtt)和 [STOMP](https://web.archive.org/web/20220811180315/https://docs.spring.io/spring-integration/reference/html/stomp.html#stomp) 。[卡夫卡](https://web.archive.org/web/20220811180315/https://spring.io/projects/spring-kafka)也支持。有了 Spring Boot，我们可以为 [Active MQ](https://web.archive.org/web/20220811180315/https://memorynotfound.com/spring-boot-embedded-activemq-configuration-example/) 、 [Artemis、](https://web.archive.org/web/20220811180315/https://activemq.apache.org/components/artemis/documentation/latest/spring-integration.html)和 [Kafka](/web/20220811180315/https://www.baeldung.com/spring-boot-kafka-testing) 使用嵌入式经纪人。

## 4.主动 MQ/Artemis 和 Kafka 的使用案例

以下几点为我们指明了何时使用哪种产品最好。

### 4.1.主动 MQ/Artemis 的使用案例

*   每天仅处理少量邮件
*   高度的可靠性和事务性
*   动态数据转换，ETL 作业

### 4.2.卡夫卡的用例

*   处理大量数据
    *   实时数据处理
    *   应用程序活动跟踪
    *   记录和监控
*   没有数据转换的消息传递(这是可能的，但不容易)
*   没有传输保证的消息传递(这是可能的，但不容易)

## 5.结论

正如我们已经看到的，活跃的 MQ/阿耳忒弥斯和卡夫卡都有他们的目的，因此也有他们的理由。了解它们之间的差异非常重要，这样才能为正确的案例选择正确的产品。下表再次简要解释了这些差异:

Differences between Active MQ and Kafka
     
| 标准 | 活动 MQ 经典 | 主动 MQ Artemis | 卡夫卡 |
| --- | --- | --- | --- |
| 用例 | 传统消息传递(可靠、事务性) | 分布式事件流 |
| P2P 消息传递 | 行列 | 路由类型为任播的地址 | – |
| 发布订阅消息 | 主题 | 路由类型为多播的地址 | 主题 |
| APIs 协议 | JMS，AMQP。MQTT，STOMP，OpenWire | Kafka 客户端，AMQP 和 MQTT 的连接器，JMS 资源适配器 |
| 基于拉和推的消息传递 | 基于推送的 | 基于拉动的 |
| 消息传递的责任 | 生产者必须确保信息被传递 | 消费者消费它应该消费的消息 |
| 交易支持 | JMS，XA | [自定义交易管理器](/web/20220811180315/https://www.baeldung.com/kafka-exactly-once) |
| 可量测性 | [经纪人网络](https://web.archive.org/web/20220811180315/https://activemq.apache.org/networks-of-brokers.html) | [集群](https://web.archive.org/web/20220811180315/https://activemq.apache.org/components/artemis/documentation/1.0.0/clusters.html) | 高度可伸缩(分区和副本) |
| 消费者越多… | …性能越慢 | …不会减速 |