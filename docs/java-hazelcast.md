# 使用 Java 的 Hazelcast 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hazelcast>

## 1。概述

这是一篇关于 Hazelcast 的介绍性文章，在这篇文章中，我们将看到如何创建一个集群成员，一个在集群节点之间共享数据的分布式`Map`，以及创建一个连接和查询集群中数据的 Java 客户端。

## 2。什么是 Hazelcast？

Hazelcast 是一个面向 Java 的分布式内存数据网格平台。该架构支持集群环境中的高可扩展性和数据分发。它支持节点的自动发现和智能同步。

Hazelcast 有[个不同版本](https://web.archive.org/web/20221020172042/http://docs.hazelcast.org/docs/latest/manual/html-single/index.html#hazelcast-editions.)。要查看所有 Hazelcast 版本的功能，我们可以参考下面的[链接](https://web.archive.org/web/20221020172042/https://hazelcast.org/imdg/imdg-features/)。在本教程中，我们将使用开源版本。

同样，Hazelcast 提供了各种特性，如分布式数据结构、分布式计算、分布式查询等。出于本文的目的，我们将关注分布式的`Map`。

## 3。Maven 依赖关系

Hazelcast 提供了许多不同的库来处理各种需求。我们可以在 Maven Central 的 [com.hazelcast](https://web.archive.org/web/20221020172042/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.hazelcast%22) 群下找到他们。

然而，在本文中，我们将只使用创建独立的 Hazelcast 集群成员和 Hazelcast Java 客户端所需的核心依赖项:

```java
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>4.0.2</version>
</dependency> 
```

当前版本可在 [maven 中央存储库](https://web.archive.org/web/20221020172042/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.hazelcast%22%20AND%20a%3A%22hazelcast%22)中获得。

## 4。第一个危险广播应用程序

### 4.1。创建一个 Hazelcast 成员

成员(也称为节点)自动结合在一起形成集群。这种自动加入是通过各种发现机制发生的，成员使用这些机制来找到彼此。

让我们创建一个在 Hazelcast 分布式地图中存储数据的成员:

```java
public class ServerNode {

    HazelcastInstance hzInstance = Hazelcast.newHazelcastInstance();
    ...
}
```

当我们启动`ServerNode` 应用程序时，我们可以在控制台中看到流动的文本，这意味着我们在 JVM 中创建了一个新的 Hazelcast 节点，它必须加入集群。

```java
Members [1] {
    Member [192.168.1.105]:5701 - 899898be-b8aa-49aa-8d28-40917ccba56c this
} 
```

为了创建多个节点，我们可以启动`ServerNode` 应用程序的多个实例。因此，Hazelcast 将自动创建一个新成员并将其添加到集群中。

例如，如果我们再次运行`ServerNode`应用程序，我们将在控制台中看到下面的日志，它指出集群中有两个成员。

```java
Members [2] {
  Member [192.168.1.105]:5701 - 899898be-b8aa-49aa-8d28-40917ccba56c
  Member [192.168.1.105]:5702 - d6b81800-2c78-4055-8a5f-7f5b65d49f30 this
}
```

### 4.2。创建一个分布式`Map`

接下来，让我们创建一个分布式的`Map.` 我们需要前面创建的`HazelcastInstance` 的实例来构建一个分布式的`Map`，它扩展了`java.util.concurrent.ConcurrentMap`接口。

```java
Map<Long, String> map = hazelcastInstance.getMap("data");
...
```

最后，让我们给`map`添加一些条目:

```java
FlakeIdGenerator idGenerator = hazelcastInstance.getFlakeIdGenerator("newid");
for (int i = 0; i < 10; i++) {
    map.put(idGenerator.newId(), "message" + i);
}
```

正如我们在上面看到的，我们已经向`map`添加了 10 个条目。我们使用了`FlakeIdGenerator` 来确保我们获得了地图的惟一键。关于`FlakeIdGenerator`的更多细节，我们可以查看以下[链接](https://web.archive.org/web/20221020172042/https://javadoc.io/doc/com.hazelcast/hazelcast/4.0.2/com/hazelcast/flakeidgen/FlakeIdGenerator.html)。

虽然这可能不是一个真实的例子，但我们只是用它来演示我们可以应用于分布式地图的许多操作之一。稍后，我们将看到如何从 Hazelcast Java 客户端检索集群成员添加的条目。

在内部，Hazelcast 对`map` 条目进行分区，并在集群成员之间分发和复制这些条目。关于 Hazelcast `Map`的更多细节，我们可以查看下面的[链接](https://web.archive.org/web/20221020172042/https://docs.hazelcast.org/docs/4.0.2/manual/html-single/index.html#map)。

### 4.3。创建一个 Hazelcast Java 客户端

Hazelcast 客户端允许我们在不成为集群成员的情况下执行所有的 Hazelcast 操作。它连接到一个集群成员，并将所有集群范围的操作委托给它。

让我们创建一个本地客户端:

```java
ClientConfig config = new ClientConfig();
config.setClusterName("dev");
HazelcastInstance hazelcastInstanceClient = HazelcastClient.newHazelcastClient(config); 
```

就这么简单。

### 4.4。从 Java 客户端访问分布式`Map`

接下来，我们将使用之前创建的`HazelcastInstance` 的实例来访问分布式`Map`:

```java
Map<Long, String> map = hazelcastInstanceClient.getMap("data");
...
```

现在，我们可以在不成为集群成员的情况下对`map` 进行操作。例如，让我们尝试迭代这些条目:

```java
for (Entry<Long, String> entry : map.entrySet()) {
    ...
}
```

## 5。配置 Hazelcast

在本节中，我们将重点介绍如何以声明方式(XML)和编程方式(API)配置 Hazelcast 网络，并使用 Hazelcast 管理中心来监控和管理正在运行的节点。

当 Hazelcast 启动时，它会寻找一个`hazelcast.config`系统属性。如果设置了它，它的值将被用作路径。否则，Hazelcast 会在工作目录或类路径中搜索一个`hazelcast.xml`文件。

如果以上都不起作用，Hazelcast 加载默认配置，即`hazelcast.jar.`附带的`hazelcast-default.xml`

### 5.1。网络配置

默认情况下，Hazelcast 使用多播来发现可以形成集群的其他成员。如果多播不是我们环境中首选的发现方式，那么我们可以为完整的 TCP/IP 集群配置 Hazelcast。

让我们使用声明性配置来配置 TCP/IP 集群:

```java
<?xml version="1.0" encoding="UTF-8"?>
<hazelcast 
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.hazelcast.com/schema/config
                               http://www.hazelcast.com/schema/config/hazelcast-config-4.0.xsd";
    <network>
        <port auto-increment="true" port-count="20">5701</port>
        <join>
            <multicast enabled="false"/>
            <tcp-ip enabled="true">
                <member>machine1</member>
                <member>localhost</member>
            </tcp-ip>
        </join>
    </network>
</hazelcast>
```

或者，我们可以使用 Java 配置方法:

```java
Config config = new Config();
NetworkConfig network = config.getNetworkConfig();
network.setPort(5701).setPortCount(20);
network.setPortAutoIncrement(true);
JoinConfig join = network.getJoin();
join.getMulticastConfig().setEnabled(false);
join.getTcpIpConfig()
  .addMember("machine1")
  .addMember("localhost").setEnabled(true);
```

默认情况下，Hazelcast 会尝试绑定 100 个端口。在上面的示例中，如果我们将端口的值设置为 5701，并将端口数量限制为 20，当成员加入集群时，Hazelcast 会尝试查找 5701 和 5721 之间的端口。

如果我们想选择只使用一个端口，我们可以通过将`auto-increment`设置为 `false`来禁用自动递增功能。

### 5.2。管理中心配置

管理中心允许我们监控集群的整体状态，我们还可以详细分析和浏览数据结构，更新映射配置，并从节点获取线程转储。

要使用 Hazelcast 管理中心，我们可以部署`mancenter-version`。`war`应用到我们的 Java 应用服务器/容器中，或者我们可以从命令行启动 Hazelcast 管理中心。我们可以从 hazelcast.org 的[下载最新的 Hazelcast ZIP。ZIP 包含了`mancenter-version`。`war` 文件。](https://web.archive.org/web/20221020172042/https://hazelcast.org/imdg/download/)

我们可以通过将 web 应用程序的 URL 添加到`hazelcast.xml` 来配置我们的 Hazelcast 节点，然后让 Hazelcast 成员与管理中心通信。

现在，让我们使用声明式配置来配置管理中心:

```java
<management-center enabled="true">
    http://localhost:8080/mancenter
</management-center>
```

同样，下面是编程配置:

```java
ManagementCenterConfig manCenterCfg = new ManagementCenterConfig();
manCenterCfg.setEnabled(true).setUrl("http://localhost:8080/mancenter");
```

## 6。结论

在本文中，我们讨论了关于 Hazelcast 的介绍性概念。更多细节可以看一下[参考手册](https://web.archive.org/web/20221020172042/http://docs.hazelcast.org/docs/3.7/manual/html-single/index.html)。

像往常一样，这篇文章的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221020172042/https://github.com/eugenp/tutorials/tree/master/hazelcast)