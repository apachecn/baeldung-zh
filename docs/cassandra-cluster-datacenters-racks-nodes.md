# Cassandra 中的集群、数据中心、机架和节点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-cluster-datacenters-racks-nodes>

## 1.介绍

在本教程中，我们将仔细看看卡珊德拉的建筑。我们将了解分布式体系结构中的数据存储，并讨论基本的体系结构组件。

## 延伸阅读:

## [使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20221101081710/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra(一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务)构建仪表板。[阅读更多](/web/20221101081710/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20221101081710/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20221101081710/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [用卡珊德拉、阿斯特拉和 CQL 构建一个仪表板——绘制事件数据](/web/20221101081710/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据存储在阿斯特拉数据库中的数据在交互式地图上显示事件。[阅读更多](/web/20221101081710/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2.Cassandra 概述

Apache Cassandra 是一个 NoSQL，分布式数据库管理系统。**Cassandra 的主要优势在于它可以跨商用服务器处理大量结构化数据。此外，它还提供了高可用性，不会出现单点故障。** Cassandra 通过使用环形架构来实现这一点，其中最小的逻辑单元是一个节点。它使用数据分区来优化查询。

每一段数据都有一个分区键。对每一行的分区键进行哈希运算。因此，我们将为每条数据获得一个唯一的令牌。对于每个节点，都有一个分配的令牌范围。因此，具有相同令牌的数据存储在相同的节点上。节点的[环形架构](https://web.archive.org/web/20221101081710/https://cassandra.apache.org/cassandra-basics)如下图所示:

[![](img/465094496cad2a1b1fb0b415db5df513.png)](/web/20221101081710/https://www.baeldung.com/wp-content/uploads/2021/06/apache-cassandra-diagrams-02-1330x1536-1.jpg)

## 3.Cassandra 组件

### 3.1.结节

节点是 Cassandra 的基本基础结构组件。**这是一台功能齐全的机器，通过内部网络与集群中的其他节点相连。**

这个网络的名字叫`Gossip Protocol`。澄清一下，这个机器可以是一个物理服务器或者一个 [EC2](/web/20221101081710/https://www.baeldung.com/ec2-java) 实例，或者一个虚拟机。所有节点都以环形网络拓扑结构组织。重要的是，每个节点都是独立的，并且在环中具有相同的角色。Cassandra 在一个`peer-to-peer structure.`中排列节点，该节点包含实际数据。

群集中的每个节点都可以接受读写请求。因此，数据在集群中的实际位置并不重要。我们将始终获得最新版本的数据。

### 3.2.虚拟节点

Cassandra 的新版本使用虚拟节点或简称为`vnodes`。**虚拟节点是服务器内的数据存储层。**

默认情况下，每台服务器有 256 个虚拟节点。正如我们在上一段中所讨论的，每个节点都分配有一个令牌范围。每个虚拟节点使用来自它们所属节点的令牌的子范围。

这些虚拟节点为系统提供了更大的灵活性。因此，当我们需要时，Cassandra 可以更容易地向集群添加新节点。当我们的数据在节点之间具有不均匀分布的令牌时，我们可以通过将虚拟节点扩展到负载更重的节点来轻松扩展存储容量。

### 3.4.计算机网络服务器

**当我们使用术语`server`时，我们指的是安装了 Cassandra 软件的** **机器。**每个节点都有一个 Cassandra 的实例，从技术上来说是一个服务器。如前所述，Cassandra 的每个实例已经发展到包含 256 个虚拟节点。Cassandra 服务器运行核心进程。例如，像在节点周围散布副本或路由请求这样的过程。

### 3.5.行李架

**Cassandra 机架是环**内节点的逻辑分组。换句话说，机架是服务器的集合。数据库使用机架，这样可以确保副本分布在不同的逻辑分组中。因此，它不仅可以向一个节点发送操作。多个节点，每个节点位于独立的机架上，可以提供更高的容错能力和可用性。

### 3.6.数据中心

数据中心是一组逻辑机架。数据中心应至少包含一个机架。我们可以说，Cassandra 数据中心是一组相关的节点，在集群中进行配置以实现复制目的。因此，它有助于减少延迟，防止事务受到其他工作负载和相关影响的影响。此外，复制因子还可以设置为写入多个数据中心。因此，Cassandra 可以在建筑设计和组织方面提供额外的灵活性。

### 3.7.串

**集群是包含一个或多个数据中心的组件。**它是数据库中最外层的存储容器。一个数据库包含一个或多个集群。Cassandra 集群中元素的层次结构如下:

[![](img/e898226089f23427f5db3abc71ff7836.png)](/web/20221101081710/https://www.baeldung.com/wp-content/uploads/2021/06/cluster2.png)

首先，我们有由数据中心组成的集群。在数据中心内部，我们的节点默认包含 256 个虚拟节点。

## 4.数据复制

现在，当我们知道了卡珊德拉的基本组成部分。我们来谈谈 Cassandra 是如何围绕其结构管理数据的。一些系统不允许数据传输中的数据丢失或中断。解决方案是在出现问题时提供备份。例如，可能是硬件问题，也可能是链路在数据处理过程中随时中断。Cassandra 将数据副本存储在多个节点上，以确保可靠性和容错性。

### 5.1.复制因子

我们可以通过复制因子和复制策略来确定副本的数量及其位置。**复制因子是集群中副本的总数**。当我们将这个因子设置为 1 时，这意味着在一个集群中每行只有一个副本，依此类推**。**我们可以在数据中心级别和机架级别设置这一因素。

### 5.1.复制策略

**复制策略控制如何选择副本**。复制品的重要性是一样的。Cassandra 有两种策略来确定哪些节点包含复制的数据。第一个称为 S `impleStrategy,` ，它不知道数据中心和机架节点的逻辑划分。第二个是更复杂的`NetworkTopologyStrategy `，它既支持机架感知，也支持数据中心感知。我们可以通过使用`NetworkTopologyStrategy`来定义在不同的数据中心放置多少副本。此外，它试图避免将两个副本放在同一个机架上的情况。

## 5.结论

本教程介绍了卡桑德拉的架构的基本组成部分。我们讨论了这个数据库的关键概念，这些概念确保了它的高可用性和分区容差。我们还讨论了数据分区和数据复制。