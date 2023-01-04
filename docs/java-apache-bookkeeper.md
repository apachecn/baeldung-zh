# 阿帕奇簿记员指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-bookkeeper>

## 1.概观

在本文中，我们将介绍[簿记员](https://web.archive.org/web/20221126220445/https://bookkeeper.apache.org/)，一个实现了**分布式容错记录存储系统**的服务。

## 2.什么是`BookKeeper`？

BookKeeper 最初是由雅虎作为[动物园管理员](/web/20221126220445/https://www.baeldung.com/java-zookeeper)子项目开发的，并于 2015 年毕业成为顶级项目。在其核心，簿记员的目标是成为一个可靠和高性能的系统，在称为`Ledgers`的数据结构中存储`Log Entries`(又名`Records`)的序列。

分类账的一个重要特征是它们是只可追加的，不可改变的。这使得 BookKeeper 成为某些应用程序的良好候选，例如分布式日志记录系统、发布-订阅消息应用程序和实时流处理。

## 3.簿记员概念

### 3.1.日志条目

日志条目包含一个不可分割的数据单元，客户端应用程序存储到簿记员或从簿记员读取数据。当存储在分类帐中时，每个条目包含提供的数据和一些元数据字段。

这些元数据字段包括一个`entryId,` ，它在给定的分类帐中必须是唯一的。还有一个验证码，BookKeeper 用它来检测一个条目何时被破坏或被篡改。

BookKeeper 本身不提供序列化特性，因此客户必须设计自己的方法来将更高级别的结构转换成数组。

### 3.2.分类账

分类账是由簿记员管理的基本存储单位，存储日志条目的有序序列。如前所述，分类帐具有仅追加语义，这意味着记录一旦被添加就不能被修改。

**此外，一旦客户停止写入分类账并关闭它，簿记员`seals`就会关闭它，我们不能再向它添加数据，甚至在以后的时间**。在围绕 BookKeeper 设计应用程序时，这是需要记住的重要一点。**分类账并不适合直接实现更高层的结构**，比如队列。相反，我们看到分类帐更多地用于创建更基本的数据结构，以支持那些更高级的概念。

例如， [Apache 的分布式日志](https://web.archive.org/web/20221126220445/https://bookkeeper.apache.org/distributedlog/)项目使用分类帐作为日志段。这些段被聚合到分布式日志中，但是底层分类帐对普通用户是透明的。

BookKeeper 通过跨多个服务器实例复制日志条目来实现分类帐弹性。三个参数控制保留多少服务器和副本:

*   总体规模:用于写入分类帐数据的服务器数量
*   写入仲裁大小:用于复制给定日志条目的服务器数量
*   Ack 仲裁大小:必须确认给定日志条目写入操作的服务器数量

通过调整这些参数，我们可以调整给定分类帐的性能和弹性特征。写入分类帐时，簿记员只有在最低法定人数的集群成员确认时，才会认为操作成功。

除了内部元数据之外，BookKeeper 还支持向分类帐添加自定义元数据。这些是客户端在创建时传递的键/值对的映射，簿记员将其存储在 ZooKeeper 中，与自己的映射放在一起。

### 3.3.博彩公司

庄家是持有一个或多个分类账的服务器。簿记员集群由许多在给定环境中运行的庄家组成，通过普通的 TCP 或 TLS 连接向客户机提供服务。

博彩公司使用 ZooKeeper 提供的集群服务来协调行动。这意味着，如果我们想要实现一个完全容错的系统，我们至少需要一个 3 实例 ZooKeeper 和一个 3 实例 BookKeeper 设置。这样的设置将能够承受任何单个实例失败时的损失，并且仍然能够正常操作，至少对于默认分类帐设置是这样的:3 节点集合大小、2 节点写仲裁和 2 节点确认仲裁。

## 4.本地设置

在本地运行簿记员的基本要求是相当适度的。首先，我们需要启动并运行一个 ZooKeeper 实例，为 BookKeeper 提供分类帐元数据存储。接下来，我们部署一个庄家，它为客户提供实际的服务。

虽然手动完成这些步骤是完全可能的，但是这里我们将使用一个使用官方 Apache 映像的`docker-compose`文件来简化这项任务:

```java
$ cd <path to docker-compose.yml>
$ docker-compose up
```

这个`docker-compose`创建了三个博彩公司和一个动物园管理员实例。因为所有的赌场都在同一台机器上运行，所以它只对测试有用。官方文档包含配置完全容错集群的必要步骤。

让我们使用 bookkeeper 的 shell 命令`listbookies`做一个基本测试，检查它是否按预期工作:

```java
$ docker exec -it apache-bookkeeper_bookie_1 /opt/bookkeeper/bin/bookkeeper \
  shell listbookies -readwrite
ReadWrite Bookies :
192.168.99.101(192.168.99.101):4181
192.168.99.101(192.168.99.101):4182
192.168.99.101(192.168.99.101):3181 
```

输出显示了可用的`bookies`列表，由三个赌注登记经纪人组成。请注意，显示的 IP 地址将根据本地 Docker 安装的具体情况而变化。

## 5.使用分类帐 API

**总账 API 是与记账员**接口的最基本方式。它允许我们直接与`Ledger`对象交互，但另一方面，它缺乏对更高级抽象(如流)的直接支持。对于这些用例，BookKeeper 项目提供了另一个支持这些特性的库 DistributedLog。

使用分类帐 API 需要将 [`bookkeeper-server`](https://web.archive.org/web/20221126220445/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.bookkeeper%22%20AND%20a%3A%22bookkeeper-server%22) 依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>org.apache.bookkeeper</groupId>
    <artifactId>bookkeeper-server</artifactId>
    <version>4.10.0</version>
</dependency>
```

注意:如文档中所述，使用这个依赖关系也将包括对 [protobuf](/web/20221126220445/https://www.baeldung.com/google-protocol-buffer) 和 [guava](/web/20221126220445/https://www.baeldung.com/category/guava/) 库的依赖关系。如果我们的项目也需要这些库，但是版本不同于 BookKeeper 使用的版本，我们可以使用一个[替代依赖来隐藏这些库](https://web.archive.org/web/20221126220445/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.bookkeeper%22%20AND%20a%3A%22bookkeeper-server-shaded%22):

```java
<dependency>
    <groupId>org.apache.bookkeeper</groupId>
    <artifactId>bookkeeper-server-shaded</artifactId>
    <version>4.10.0</version>
</dependency> 
```

### 5.1.连接到庄家

**`BookKeeper` 类是分类帐 API** 的主要入口点，提供了一些连接到我们的簿记员服务的方法。最简单的形式是，我们只需要创建这个类的一个新实例，传递 BookKeeper 使用的一个 ZooKeeper 服务器的地址:

```java
BookKeeper client = new BookKeeper("zookeeper-host:2131"); 
```

这里，`zookeeper-host`应该设置为保存簿记员集群配置的 ZooKeeper 服务器的 IP 地址或主机名。在我们的例子中，这通常是“localhost”或者 DOCKER_HOST 环境变量指向的主机。

如果我们需要对一些参数进行更多的控制来微调我们的客户端，我们可以使用一个`ClientConfiguration`实例并使用它来创建我们的客户端:

```java
ClientConfiguration cfg = new ClientConfiguration();
cfg.setMetadataServiceUri("zk+null://zookeeper-host:2131");

// ... set other properties

BookKeeper.forConfig(cfg).build();
```

### 5.2.创建分类帐

一旦我们有了一个`BookKeeper`实例，创建一个新的分类帐就很简单了:

```java
LedgerHandle lh = bk.createLedger(BookKeeper.DigestType.MAC,"password".getBytes());
```

这里，我们使用了这种方法的最简单的变体。它将使用默认设置创建一个新的分类帐，使用 MAC 摘要类型来确保条目的完整性。

如果我们想要将自定义元数据添加到分类帐中，我们需要使用一个接受所有参数的变量:

```java
LedgerHandle lh = bk.createLedger(
  3,
  2,
  2,
  DigestType.MAC,
  "password".getBytes(),
  Collections.singletonMap("name", "my-ledger".getBytes()));
```

这一次，我们使用了完整版本的`createLedger()`方法。前三个参数分别是总体大小、写入仲裁和确认仲裁值。接下来，我们有和以前一样的摘要参数。最后，我们传递一个带有自定义元数据的`Map`。

在上述两种情况下，`createLedger`都是同步操作。BookKeeper 还提供使用回调的异步分类帐创建:

```java
bk.asyncCreateLedger(
  3,
  2,
  2,
  BookKeeper.DigestType.MAC, "passwd".getBytes(),
  (rc, lh, ctx) -> {
      // ... use lh to access ledger operations
  },
  null,
  Collections.emptyMap()); 
```

较新版本的 BookKeeper (>= 4.6)也支持流畅风格的 API 和`CompletableFuture`来实现相同的目标:

```java
CompletableFuture<WriteHandle> cf = bk.newCreateLedgerOp()
  .withDigestType(org.apache.bookkeeper.client.api.DigestType.MAC)
  .withPassword("password".getBytes())
  .execute(); 
```

注意，在这种情况下，我们得到的是一个`WriteHandle`而不是一个`LedgerHandle`。稍后我们会看到，当`LedgerHandle` 实现`WriteHandle.`时，我们可以使用它们中的任何一个来访问我们的分类帐

### 5.3.写入数据

一旦我们获得了一个`LedgerHandle`或`WriteHandle`，我们就使用一个`append()`方法变量将数据写入相关的分类帐。让我们从同步变体开始:

```java
for(int i = 0; i < MAX_MESSAGES; i++) {
    byte[] data = new String("message-" + i).getBytes();
    lh.append(data);
} 
```

这里，我们使用一个接受`byte`数组的变量。该 API 还支持 Netty 的`ByteBuf`和 Java NIO 的`ByteBuffer`，这允许在时间关键的场景中进行更好的内存管理。

对于异步操作，API 会根据我们获得的特定句柄类型有所不同。`WriteHandle`使用`CompletableFuture, `，而`LedgerHandle `也支持基于回调的方法:

```java
// Available in WriteHandle and LedgerHandle
CompletableFuture<Long> f = lh.appendAsync(data);

// Available only in LedgerHandle
lh.asyncAddEntry(
  data,
  (rc,ledgerHandle,entryId,ctx) -> {
      // ... callback logic omitted
  },
  null);
```

选择哪一个很大程度上是个人的选择，但是一般来说，使用基于`CompletableFuture`的 API 更容易阅读。此外，还有一个附带的好处，我们可以直接从它构造一个`Mono`，使得在反应式应用程序中集成 BookKeeper 变得更加容易。

### 5.4.读取数据

从簿记员分类账中读取数据的工作方式与书写类似。首先，我们使用我们的`BookKeeper `实例创建一个`LedgerHandle` **:**

```java
LedgerHandle lh = bk.openLedger(
  ledgerId, 
  BookKeeper.DigestType.MAC,
  ledgerPassword); 
```

除了`ledgerId`参数，我们将在后面介绍，这段代码看起来很像我们以前见过的`createLedger()`方法。**不过，这有一个重要的区别；这个方法返回一个只读的`LedgerHandle`实例**。如果我们试图使用任何可用的`append()`方法，我们得到的都是一个异常。

或者，更安全的方法是使用流畅风格的 API:

```java
ReadHandle rh = bk.newOpenLedgerOp()
  .withLedgerId(ledgerId)
  .withDigestType(DigestType.MAC)
  .withPassword("password".getBytes())
  .execute()
  .get(); 
```

`ReadHandle`拥有从我们的分类账中读取数据所需的方法:

```java
long lastId = lh.readLastConfirmed();
rh.read(0, lastId).forEach((entry) -> {
    // ... do something 
});
```

在这里，我们只是使用同步`read`变量请求这个分类帐中的所有可用数据。正如所料，还有一个异步变体:

```java
rh.readAsync(0, lastId).thenAccept((entries) -> {
    entries.forEach((entry) -> {
        // ... process entry
    });
});
```

如果我们选择使用旧的`openLedger()`方法，我们会发现额外的方法支持异步方法的回调风格:

```java
lh.asyncReadEntries(
  0,
  lastId,
  (rc,lh,entries,ctx) -> {
      while(entries.hasMoreElements()) {
          LedgerEntry e = ee.nextElement();
      }
  },
  null);
```

### 5.5.列出分类帐

我们之前已经看到，我们需要分类帐的`id`来打开和读取它的数据。那么，我们如何得到一个呢？**一种方法是使用`LedgerManager`接口，我们可以从我们的`BookKeeper `实例**中访问它。这个接口主要处理分类帐元数据，但也有`asyncProcessLedgers()`方法。使用这种方法以及一些帮助形成并发原语的方法，我们可以枚举所有可用的分类帐:

```java
public List listAllLedgers(BookKeeper bk) {
    List ledgers = Collections.synchronizedList(new ArrayList<>());
    CountDownLatch processDone = new CountDownLatch(1);

    bk.getLedgerManager()
      .asyncProcessLedgers(
        (ledgerId, cb) -> {
            ledgers.add(ledgerId);
            cb.processResult(BKException.Code.OK, null, null);
        }, 
        (rc, s, obj) -> {
            processDone.countDown();
        },
        null,
        BKException.Code.OK,
        BKException.Code.ReadException);

    try {
        processDone.await(1, TimeUnit.MINUTES);
        return ledgers;
    } catch (InterruptedException ie) {
        throw new RuntimeException(ie);
    }
} 
```

让我们来消化一下这段代码，对于一个看似琐碎的任务来说，这段代码比预期的要长一些。**`asyncProcessLedgers()`方法需要两次回调**。

第一个在一个列表中收集所有分类帐 id。我们在这里使用同步列表，因为这个回调可以从多个线程调用。除了分类帐 id，这个回调还接收一个回调参数。我们必须调用它的`processResult()`方法来确认我们已经处理了数据，并表示我们已经准备好获取更多的数据。

当所有分类帐都被发送到处理器回调或出现故障时，调用第二个回调。在我们的例子中，我们省略了错误处理。相反，我们只是减少一个`CountDownLatch`，这反过来将完成`await`操作，并允许该方法返回所有可用分类帐的列表。

## 6.结论

在本文中，我们介绍了 Apache BookKeeper 项目，了解了它的核心概念，并使用它的低级 API 来访问分类帐和执行读/写操作。

像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20221126220445/https://github.com/eugenp/tutorials/tree/master/persistence-modules/apache-bookkeeper)