# Atomix 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/atomix>

## 1。概述

大多数分布式应用程序要求一些有状态的组件是一致的和容错的。Atomix 是一个嵌入式库，有助于实现分布式资源的容错和一致性。

它提供了一组丰富的 API 来管理其资源，如集合、组和并发工具。

首先，我们需要将以下 Maven 依赖项添加到 pom 中:

```java
<dependency>
    <groupId>io.atomix</groupId>
    <artifactId>atomix-all</artifactId>
    <version>1.0.8</version>
</dependency>
```

这种依赖性提供了其节点相互通信所需的基于 Netty 的传输。

## 2。引导集群

要开始使用 Atomix，我们需要首先引导一个集群。

Atomix 由一组用于创建有状态分布式资源的副本组成。每个副本维护集群中存在的每个资源的状态的副本。

群集中有两种类型的副本:主动副本和被动副本。

分布式资源的状态变化通过主动副本传播，而被动副本保持同步以保持容错。

### 2.1。引导嵌入式集群

为了引导单节点集群，我们需要首先创建一个 *AtomixReplica* 的实例:

```java
AtomixReplica replica = AtomixReplica.builder(
  new Address("localhost", 8700))
   .withStorage(storage)
   .withTransport(new NettyTransport())
   .build();
```

这里副本配置了*存储*和*传输*。声明存储的代码段:

```java
Storage storage = Storage.builder()
  .withDirectory(new File("logs"))
  .withStorageLevel(StorageLevel.DISK)
  .build();
```

一旦副本被声明并配置了存储和传输，我们就可以通过简单地调用 *bootstrap()* 来引导它，这将返回一个 *CompletableFuture* ，它可用于阻塞，直到服务器通过调用相关的阻塞 *join()* 方法来引导:

```java
CompletableFuture<AtomixReplica> future = replica.bootstrap();
future.join();
```

到目前为止，我们已经构建了一个单节点集群。现在我们可以向它添加更多的节点。

为此，我们需要创建其他副本，并将它们加入现有集群；我们需要生成一个新线程来调用 *join(Address)* 方法:

```java
AtomixReplica replica2 = AtomixReplica.builder(
  new Address("localhost", 8701))
    .withStorage(storage)
    .withTransport(new NettyTransport())
    .build();

replica2
  .join(new Address("localhost", 8700))
  .join();

AtomixReplica replica3 = AtomixReplica.builder(
  new Address("localhost", 8702))
    .withStorage(storage)
    .withTransport(new NettyTransport())
    .build();

replica3.join(
  new Address("localhost", 8700), 
  new Address("localhost", 8701))
  .join();
```

现在我们有了一个三节点集群引导。或者，我们可以通过在 *bootstrap(列表<地址> )* 方法中传递地址的*列表*来引导集群:

```java
List<Address> cluster = Arrays.asList(
  new Address("localhost", 8700), 
  new Address("localhost", 8701), 
  new Address("localhsot", 8702));

AtomixReplica replica1 = AtomixReplica
  .builder(cluster.get(0))
  .build();
replica1.bootstrap(cluster).join();

AtomixReplica replica2 = AtomixReplica
  .builder(cluster.get(1))
  .build();

replica2.bootstrap(cluster).join();

AtomixReplica replica3 = AtomixReplica
  .builder(cluster.get(2))
  .build();

replica3.bootstrap(cluster).join();
```

我们需要为每个副本生成一个新线程。

### 2.2。引导独立集群

Atomix 服务器可以作为独立的服务器运行，可以从 Maven Central 下载。简而言之，它是一个 Java 归档文件，可以通过终端运行，只需提供

简而言之——这是一个 Java 归档文件，可以通过在地址标志中提供*主机:端口*参数并使用*-引导程序*标志来通过终端运行。

以下是引导集群的命令:

```java
java -jar atomix-standalone-server.jar 
  -address 127.0.0.1:8700 -bootstrap -config atomix.properties
```

这里的 *atomix.properties* 是配置存储和传输的配置文件。要创建一个多节点集群，我们可以使用 *-join* 标志向现有集群添加节点。

它的格式是:

```java
java -jar atomix-standalone-server.jar 
  -address 127.0.0.1:8701 -join 127.0.0.1:8700
```

## 3。与客户合作

Atomix 支持创建一个客户机，通过 *AtomixClient* API 远程访问它的集群。

因为客户端不需要有状态，所以*atomic client*没有任何存储。我们只需要在创建客户端时配置传输，因为传输将用于与集群通信。

让我们创建一个带有传输的客户机:

```java
AtomixClient client = AtomixClient.builder()
  .withTransport(new NettyTransport())
  .build();
```

我们现在需要将客户端连接到集群。

我们可以声明一个*地址*的*列表*，并将*列表*作为参数传递给客户端的 *connect()* 方法:

```java
client.connect(cluster)
  .thenRun(() -> {
      System.out.println("Client is connected to the cluster!");
  });
```

## 4。处理资源

Atomix 的真正强大之处在于它为创建和管理分布式资源提供了一组强大的 API。**资源在集群中被复制和持久化，并由复制的状态机**支持，该状态机由 Raft 共识协议的底层实现来管理。

分布式资源可以通过它的一个`get()`方法来创建和管理。我们可以从 *AtomixReplica* 创建一个分布式资源实例。

考虑到*副本*是*原子副本*的实例，创建分布式地图资源并为其设置值的代码片段:

```java
replica.getMap("map")
  .thenCompose(m -> m.put("bar", "Hello world!"))
  .thenRun(() -> System.out.println("Value is set in Distributed Map"))
  .join();
```

这里的 *join()* 方法将阻塞程序，直到创建了资源并为其设置了值。我们可以使用*atomic client*获得相同的对象，并使用 *get("bar")* 方法检索值。

我们可以在最后使用`get()` 方法来等待结果:

```java
String value = client.getMap("map"))
  .thenCompose(m -> m.get("bar"))
  .thenApply(a -> (String) a)
  .get();
```

## 5。一致性和容错性

Atomix 用于任务关键型小规模数据集，对于这些数据集，一致性比可用性更重要。

**它通过读取和写入的线性化提供了强大的可配置一致性**。在线性化中，一旦提交写操作，保证所有客户端都知道结果状态。

Atomix 集群中的一致性由底层的 Raft 共识算法保证，在该算法中，当选的领导者将拥有之前成功的所有写入。

所有新写入将通过群集领导者，并在完成前同步复制到服务器的大部分。

为了保持容错，集群的大多数服务器需要保持活动状态。如果少数节点出现故障，这些节点将被标记为非活动状态，并由被动节点或备用节点替代。

如果领导者失败，群集中的其余服务器将开始新的领导者选举。同时，群集将不可用。

在分区的情况下，如果一个领导者在分区的非法定一方，它会下台，而在法定一方会选出一个新的领导者。

而且，如果领导者是多数党，它将继续没有变化。解析分区后，非仲裁端的节点将加入仲裁，并相应地更新它们的日志。

## 6。结论

像 ZooKeeper 一样，Atomix 提供了一组健壮的库来处理分布式计算问题。

和往常一样，这项任务的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/atomix)