# 在 Cassandra 中请求路由和告密者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-request-routing-snitches>

## 1.概观

在本教程中，我们将了解一个`snitch`的工作以及[卡珊德拉](https://web.archive.org/web/20220524031658/https://cassandra.apache.org/_/index.html)如何使用它来有效地路由请求。我们还会看看卡珊德拉中各种类型的告密者。

## 2.什么是告密者？

告密者只是报告每个节点所属的[机架和数据中心](/web/20220524031658/https://www.baeldung.com/cassandra-cluster-datacenters-racks-nodes)——本质上，它决定并通知 Cassandra 集群的网络拓扑。

了解了集群的拓扑结构，包括节点之间的相对接近度，Cassandra 就能够高效地将请求路由到集群中的适当节点。

### 2.1.写操作时告密

Cassandra 使用来自飞贼的信息将节点分组到机架和数据中心。因此，为了避免写操作期间的相关故障，Cassandra 尽一切努力不将副本存储在同一个机架中。

### 2.2.告发读取操作

我们知道，在读操作期间，Cassandra 必须根据它们的[一致性级别](/web/20220524031658/https://www.baeldung.com/cassandra-consistency-levels)联系许多节点和副本。**为了提高读取操作的效率，卡珊德拉使用金色飞贼的信息来识别最快返回副本的节点。**

然后查询该节点以获得完整的行信息。然后，Cassandra 向其他副本节点查询哈希值，以确保返回最新的数据。

## 3.告密者的类型

**默认的金色飞贼`SimpleSnitch`，不支持拓扑。也就是说，它不知道集群的机架和数据中心。**因此，它不适合多数据中心部署。

正因如此，卡珊德拉想出了各种符合我们要求的告密者。通常情况下，我们可以通过属性名`endpoint_snitch`在配置文件`cassandra.yml`中配置告密者的类型。

让我们看看几种告密者及其工作方式。

### 3.1.`PropertyFileSnitch`

`PropertyFileSnitch`是一种机架感知型飞贼。我们可以在名为`cassandra-topology.properties`的文件中提供集群拓扑信息作为键值属性。在这个属性文件中，我们提供了每个节点所属的机架和数据中心名称。

此外，我们可以为机架和数据中心使用任何名称。但是**我们需要确保数据中心名称与在`keyspace`定义**中定义为`NetworkTopologyStrategy`的名称相匹配。

下面是`cassandra-topology.properties`的内容示例:

```
# Cassandra Node IP=Data Center:Rack 
172.86.22.125=DC1:RAC1 
172.80.23.120=DC1:RAC1 
172.84.25.127=DC1:RAC1 

192.53.34.122=DC1:RAC2 
192.55.36.134=DC1:RAC2 
192.57.302.112=DC1:RAC2 

# default for unknown nodes 
default=DC1:RAC1
```

在上面的示例中，有两个机架(RAC1、RAC2)和一个数据中心(DC1)。任何未涵盖的节点 IP 都将归入默认数据中心(DC1)和机架(RAC1)。

这个 snitch 的一个缺点是，我们需要确保`cassandra-topology.properties`文件与集群中的所有节点同步。

### 3.2.`GossipingPropertyFileSnitch`

`GossipingPropertyFileSnitch`也是一个机架感知型飞贼。为了避免`PropertyFileSnitch`中要求的机架和数据中心的手动同步，在这个 snitch 中，我们只需在`cassandra-rackdc.properties` 文件中为每个节点单独定义机架和数据中心的名称。

使用 gossip 协议与所有节点交换这些机架和数据中心的信息。

下面是`cassandra-rackdc.properties`文件的内容示例:

```
dc=DC1
rack=RAC1
```

### 3.3.`Ec2Snitch`

顾名思义，`Ec2Snitch `与亚马逊 Web 服务(AWS) EC2 中的集群**部署相关。在单个 AWS 区域部署中，区域名称被视为数据中心，可用性区域名称被视为机架。**

如果我们只需要一个单数据中心部署，则不需要任何属性配置。然而，在多数据中心部署的情况下，我们可以在`cassandra-rackdc.properties` 文件中指定`dc_suffix`。

例如，在`us-east`区域，如果我们需要几个数据中心，我们可以提供如下的`dc_suffix`配置:

```
dc_suffix=_1_DC1
dc_suffix=_1_DC2
```

上面给出的每个配置进入两个不同的节点。因此，数据中心的名称是`us_east_1_DC1`和`us_east_1_DC2`。

### 3.4.`Ec2MultiRegionSnitch`

在 AWS 中多区域集群部署的情况下，我们应该使用`Ec2MultiRegionSnitch`。此外，我们需要配置`cassandra.yaml`和`cassandra-rackdc.properties.`

我们必须将公共 IP 地址配置为`broadcast_address`，如果需要，还可以将其用作`cassandra.yaml`中的种子节点。此外，我们必须配置一个私有 IP 地址作为`listen_address.`，最后，我们必须打开公共 IP 防火墙上的`session_port `或`ssl_session_port `。

这些配置允许节点跨 AWS 区域进行通信，从而支持跨区域的多数据中心部署。在单个区域的情况下，Cassandra 节点在建立连接后切换到私有 IP 地址进行通信。

跨区域的每个节点在`cassandra_rackdc.properties`中的`dc_suffix`数据中心配置类似于`Ec2Snitch`。

### 3.5.`GoogleCloudSnitch`

顾名思义，GoogleCloudSnitch 用于跨一个或多个地区的 Google 云平台中的集群部署。与 AWS 类似，区域名称被认为是数据中心，可用性区域是机架。

在单数据中心部署的情况下，我们不需要任何配置。相反，在多数据中心部署的情况下，类似于`Ec2Snitch`，我们可以在`cassandra-rackdc.properties`文件中设置`dc_suffix`。

### 3.6.`RackInferringSnitch`

`RackInferringSnitch`根据节点 IP 地址的第三和第二个八位字节推断节点与机架和数据中心的接近程度。

## 4.动态窃密

默认情况下，Cassandra 将我们在`cassandra.yml`文件中配置的任何金色飞贼类型与另一种称为`DynamicEndpointSnitch.`的金色飞贼类型包装在一起。这种动态金色飞贼从已经配置好的底层金色飞贼中获取集群拓扑的基本信息。然后，它监控节点的读取延迟，甚至跟踪任何节点中的压缩操作。

然后，动态 snitch 使用这些性能数据为任何读取查询选择最佳副本节点。通过这种方式，Cassandra 避免了将读取请求重新路由到坏的或繁忙的(运行缓慢的)副本节点。

动态飞贼使用 gossip 使用的`Phi accrual failure detection`机制的修改版本来确定读请求的最佳副本节点。`badness threshold `是一个可配置的参数，它确定为了失去其优先状态，优选节点与最佳执行节点相比必须执行得多差。

Cassandra 定期重置每个节点的性能分数，以允许表现不佳的节点恢复并表现更好，从而重新获得其优先状态。

## 5.结论

在本教程中，我们学习了什么是金色飞贼，并且还学习了我们可以在 Cassandra 集群部署中配置的一些金色飞贼类型。