# 莴苣介绍 Java Redis 客户机

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-redis-lettuce>

## 1。概述

这篇文章是对[莴苣](https://web.archive.org/web/20221126235007/https://lettuce.io/)，一个 [Redis](https://web.archive.org/web/20221126235007/http://redis.io/) Java 客户端的介绍。

Redis 是一个内存中的键值存储，可以用作数据库、缓存或消息代理。使用对 Redis 内存数据结构中的键进行操作的[命令](https://web.archive.org/web/20221126235007/https://redis.io/commands)来添加、查询、修改和删除数据。

**莴苣支持完整 Redis API 的同步和异步通信使用，包括其数据结构、发布/订阅消息和高可用性服务器连接。**

## 2。为什么是生菜？

我们在之前的一篇文章中已经讨论过 Jedis [。](/web/20221126235007/https://www.baeldung.com/jedis-java-redis-client-library)是什么让生菜与众不同？

最显著的区别是它通过 Java 8 的`CompletionStage`接口的异步支持和对反应流的支持。正如我们将在下面看到的，莴苣为从 Redis 数据库服务器发出异步请求和创建流提供了一个自然的接口。

它还使用  与服务器通信。这使得 API“更重”，但也使它更适合与多个线程共享一个连接。

## 3。设置

### 3.1。依赖性

让我们从声明我们在`pom.xml`中需要的唯一依赖项开始:

```
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>5.0.1.RELEASE</version>
</dependency> 
```

最新版本的库可以在 Github 库或 T2 的 Maven Central 上查看。

### 3.2 .Redis 安装〔t1〕

我们需要安装并运行至少一个 Redis 实例，如果我们希望测试集群或 sentinel 模式，则需要两个(尽管 sentinel 模式需要三台服务器才能正常工作。)对于本文，我们使用的是目前最新的稳定版本 4.0.x。

关于 Redis 入门的更多信息可以在这里找到，包括 Linux 和 MacOS 的下载。

Redis 官方不支持 Windows，但是这里有一个服务器的端口[这里](https://web.archive.org/web/20221126235007/https://github.com/MicrosoftArchive/redis)。我们也可以在 [Docker](https://web.archive.org/web/20221126235007/https://hub.docker.com/_/redis/) 中运行 Redis，这是 Windows 10 更好的替代方案，也是快速启动和运行的一种方式。

## 4。连接

### 4.1。连接到服务器

**连接 Redis 包括四个步骤:**

1.  创建重定向 URI
2.  使用 URI 连接到`RedisClient`
3.  重新连接打开
4.  生成一组`RedisCommands`

让我们来看看实现:

```
RedisClient redisClient = RedisClient
  .create("redis://[[email protected]](/web/20221126235007/https://www.baeldung.com/cdn-cgi/l/email-protection):6379/");
StatefulRedisConnection<String, String> connection
 = redisClient.connect();
```

A `StatefulRedisConnection`就是它听起来的样子；到 Redis 服务器的线程安全连接，它将保持与服务器的连接，并在需要时重新连接。一旦有了连接，我们就可以用它来同步或异步地执行 Redis 命令。

`RedisClient`使用大量系统资源，因为它拥有与 Redis 服务器通信的净资源。需要多个连接的应用程序应该使用单个`RedisClient.`

### 4.2 .瑞迪斯〔t1〕

**我们通过向静态工厂方法传递一个 URI 来创建一个`RedisClient`。**

莴苣利用了 Redis URIs 的自定义语法。这是模式:

```
redis :// [[[email protected]](/web/20221126235007/https://www.baeldung.com/cdn-cgi/l/email-protection)] host [: port] [/ database]
  [? [timeout=timeout[d|h|m|s|ms|us|ns]]
  [&_database=database_]] 
```

有四种 URI 计划:

*   `redis`独立重定向服务器
*   `rediss`–通过 SSL 连接的独立 Redis 服务器
*   `redis-socket`–通过 Unix 域套接字的独立 Redis 服务器
*   `redis-sentinel`—重定向哨兵服务器

Redis 数据库实例可以被指定为 URL 路径的一部分或附加参数。如果两者都提供了，则参数具有较高的优先级。

在上面的例子中，我们使用了一个`String`表示。莴苣也有一个用于建立连接的`RedisURI`类。它提供了`Builder`模式:

```
RedisURI.Builder
  .redis("localhost", 6379).auth("password")
  .database(1).build(); 
```

和一个构造函数:

```
new RedisURI("localhost", 6379, 60, TimeUnit.SECONDS); 
```

### 4.3。同步命令

**类似于 Jedis，莴苣以方法的形式提供了完整的 Redis 命令集。**

然而，莴苣实现了同步和异步版本。我们将简要地看一下同步版本，然后在本教程的剩余部分使用异步实现。

创建连接后，我们使用它来创建命令集:

```
RedisCommands<String, String> syncCommands = connection.sync(); 
```

现在我们有了一个与 Redis 交流的直观界面。

我们可以设置并获取`String values:`

```
syncCommands.set("key", "Hello, Redis!");

String value = syncommands.get(“key”); 
```

我们可以使用哈希:

```
syncCommands.hset("recordName", "FirstName", "John");
syncCommands.hset("recordName", "LastName", "Smith");
Map<String, String> record = syncCommands.hgetall("recordName"); 
```

我们将在本文后面讨论更多的 Redis。

莴苣同步 API 使用异步 API。阻止是在命令级别为我们完成的。这意味着不止一个客户端可以共享一个同步连接。

### 4.4。异步命令

让我们来看看异步命令:

```
RedisAsyncCommands<String, String> asyncCommands = connection.async(); 
```

我们从连接中检索一组`RedisAsyncCommands`，类似于我们检索同步组的方式。这些命令返回一个`RedisFuture`(内部是一个`CompletableFuture`)`:`

```
RedisFuture<String> result = asyncCommands.get("key"); 
```

使用`CompletableFuture` 的指南可以在[这里找到。](/web/20221126235007/https://www.baeldung.com/java-completablefuture)

### 4.5。反应式 API

最后，让我们看看如何使用非阻塞反应式 API:

```
RedisStringReactiveCommands<String, String> reactiveCommands = connection.reactive(); 
```

**这些命令从[项目反应堆`.`](https://web.archive.org/web/20221126235007/https://github.com/reactor/reactor-core)** 返回包裹在`Mono`或`Flux` 中的结果

使用 Project Reactor 的指南可以在这里找到。

## 5。Redis 数据结构〔t1〕

我们在上面简要地看了字符串和散列，让我们看看莴苣是如何实现 Redis 的其余数据结构的。正如我们所料，每个 Redis 命令都有一个类似命名的方法。

### 5.1。列表

**列表是保留插入顺序的`Strings`列表。**从两端插入或检索值:

```
asyncCommands.lpush("tasks", "firstTask");
asyncCommands.lpush("tasks", "secondTask");
RedisFuture<String> redisFuture = asyncCommands.rpop("tasks");

String nextTask = redisFuture.get(); 
```

在本例中，`nextTask`等于`firstTask`。`Lpush`将值推到列表的开头，然后`rpop`从列表的末尾弹出值。

我们也可以从另一端弹出元素:

```
asyncCommands.del("tasks");
asyncCommands.lpush("tasks", "firstTask");
asyncCommands.lpush("tasks", "secondTask");
redisFuture = asyncCommands.lpop("tasks");

String nextTask = redisFuture.get(); 
```

我们从用`del`删除列表开始第二个例子。然后我们再次插入相同的值，但是我们使用`lpop`从列表的头部弹出值，所以`nextTask`保存了`secondTask`文本。

### 5.2。设置

**Redis 集合是类似 Java `Sets`的`Strings`的无序集合；没有重复的元素:**

```
asyncCommands.sadd("pets", "dog");
asyncCommands.sadd("pets", "cat");
asyncCommands.sadd("pets", "cat");

RedisFuture<Set<String>> pets = asyncCommands.smembers("nicknames");
RedisFuture<Boolean> exists = asyncCommands.sismember("pets", "dog"); 
```

当我们检索 Redis 集合作为一个`Set`时，大小是 2，因为重复的`“cat”`被忽略了。当我们用`sismember,` 向 Redis 查询`“dog”`的存在时，得到的回应是`true.`

### 5.3。哈希

我们之前简要地看了一个散列的例子。它们值得一个简短的解释。

**Redis 散列是具有`String`字段和值的记录。**每个记录在主索引中也有一个键:

```
asyncCommands.hset("recordName", "FirstName", "John");
asyncCommands.hset("recordName", "LastName", "Smith");

RedisFuture<String> lastName 
  = syncCommands.hget("recordName", "LastName");
RedisFuture<Map<String, String>> record 
  = syncCommands.hgetall("recordName"); 
```

我们使用`hset`将字段添加到散列中，传递散列的名称、字段的名称和一个值。

然后，我们用记录和字段的名称`hget,`检索一个单独的值。最后，我们用`hgetall.`获取整个记录作为散列

### 5.4。有序集合

**有序集合包含值和一个等级，它们是按照这个等级进行排序的。**秩为 64 位浮点值。

添加带有等级的项目，并在一个范围内检索:

```
asyncCommands.zadd("sortedset", 1, "one");
asyncCommands.zadd("sortedset", 4, "zero");
asyncCommands.zadd("sortedset", 2, "two");

RedisFuture<List<String>> valuesForward = asyncCommands.zrange(key, 0, 3);
RedisFuture<List<String>> valuesReverse = asyncCommands.zrevrange(key, 0, 3); 
```

`zadd` 的第二个参数是等级。我们按等级检索一个范围，用`zrange`表示升序，用`zrevrange`表示降序。

我们添加了排名为 4 的“`zero`”，因此它将出现在`valuesForward`的末尾和`valuesReverse.`的开头

## 6。交易

事务允许在单个原子步骤中执行一组命令。这些命令保证按顺序且排他地执行。在事务完成之前，不会执行来自另一个用户的命令。

要么执行所有命令，要么都不执行。**如果其中一个失败，Redis 不会执行回滚。**一旦`exec()`被调用，所有命令按指定的顺序执行。

让我们看一个例子:

```
asyncCommands.multi();

RedisFuture<String> result1 = asyncCommands.set("key1", "value1");
RedisFuture<String> result2 = asyncCommands.set("key2", "value2");
RedisFuture<String> result3 = asyncCommands.set("key3", "value3");

RedisFuture<TransactionResult> execResult = asyncCommands.exec();

TransactionResult transactionResult = execResult.get();

String firstResult = transactionResult.get(0);
String secondResult = transactionResult.get(0);
String thirdResult = transactionResult.get(0); 
```

对`multi`的调用启动了事务。当一个事务开始时，在调用`exec()`之前不会执行后续命令。

在同步模式下，命令返回`null.`，在异步模式下，命令返回`RedisFuture`。`Exec`返回一个包含响应列表的`TransactionResult`。

**由于`RedisFutures`也接收它们的结果，异步 API 客户端在两个地方接收事务结果。**

## 7。配料

在正常情况下，只要 API 客户机调用命令，莴苣就会执行它们。

这是大多数普通应用程序所希望的，尤其是当它们依赖于串行接收命令结果时。

然而，如果应用程序不需要立即得到结果，或者如果大量数据被批量上传，这种行为是没有效率的。

异步应用程序可以覆盖这种行为:

```
commands.setAutoFlushCommands(false);

List<RedisFuture<?>> futures = new ArrayList<>();
for (int i = 0; i < iterations; i++) {
    futures.add(commands.set("key-" + i, "value-" + i);
}
commands.flushCommands();

boolean result = LettuceFutures.awaitAll(5, TimeUnit.SECONDS,
  futures.toArray(new RedisFuture[0])); 
```

setAutoFlushCommands 设置为`false`时，应用程序必须手动调用`flushCommands`。在这个例子中，我们将多个`set` 命令排队，然后刷新通道。`AwaitAll`等待所有的`RedisFutures`完成。

这个状态是基于每个连接设置的，并且影响所有使用该连接的线程。此功能不适用于同步命令。

## 8。发布/订阅

Redis 提供了一个简单的发布/订阅消息系统。订户使用`subscribe`命令消费来自通道的消息。邮件不会持久保存；只有当用户订阅了某个频道时，才会将它们发送给用户。

Redis 使用发布/订阅系统来通知 Redis 数据集，使客户端能够接收关于键被设置、删除、过期等事件。

更多详细信息，请参见此处的文档[。](https://web.archive.org/web/20221126235007/https://redis.io/topics/notifications)

### 8.1。订户

A `RedisPubSubListener`接收发布/订阅消息。这个接口定义了几个方法，但是我们在这里只展示接收消息的方法:

```
public class Listener implements RedisPubSubListener<String, String> {

    @Override
    public void message(String channel, String message) {
        log.debug("Got {} on channel {}",  message, channel);
        message = new String(s2);
    }
} 
```

我们使用`RedisClient`连接一个发布/订阅通道并安装监听器:

```
StatefulRedisPubSubConnection<String, String> connection
 = client.connectPubSub();
connection.addListener(new Listener())

RedisPubSubAsyncCommands<String, String> async
 = connection.async();
async.subscribe("channel"); 
```

安装了监听器后，我们检索一组`RedisPubSubAsyncCommands`并订阅一个频道。

### 8.2。发布者

发布只是连接发布/订阅通道并检索命令的问题:

```
StatefulRedisPubSubConnection<String, String> connection 
  = client.connectPubSub();

RedisPubSubAsyncCommands<String, String> async 
  = connection.async();
async.publish("channel", "Hello, Redis!"); 
```

发布需要渠道和消息。

### 8.3。被动订阅

莴苣还提供了一个用于订阅发布/订阅消息的反应界面:

```
StatefulRedisPubSubConnection<String, String> connection = client
  .connectPubSub();

RedisPubSubAsyncCommands<String, String> reactive = connection
  .reactive();

reactive.observeChannels().subscribe(message -> {
    log.debug("Got {} on channel {}",  message, channel);
    message = new String(s2);
});
reactive.subscribe("channel").subscribe(); 
```

由`observeChannels`返回的`Flux`接收所有通道的消息，但是由于这是一个流，过滤很容易做到。

## 9。高可用性

Redis 为高可用性和可伸缩性提供了几个选项。完全理解需要 Redis 服务器配置的知识，但是我们将简要概述一下莴苣是如何支持它们的。

### 9.1。主/从

Redis 服务器在主/从配置中进行自我复制。主服务器向从服务器发送将主缓存复制到从服务器的命令流。Redis 不支持双向复制，所以从站是只读的。

莴苣可以连接到主/从系统，查询它们的拓扑结构，然后选择从系统进行读取操作，这可以提高吞吐量:

```
RedisClient redisClient = RedisClient.create();

StatefulRedisMasterSlaveConnection<String, String> connection
 = MasterSlave.connect(redisClient, 
   new Utf8StringCodec(), RedisURI.create("redis://localhost"));

connection.setReadFrom(ReadFrom.SLAVE); 
```

### 9.2。哨兵

Redis Sentinel 监控主实例和从实例，并在主实例发生故障转移时协调从实例的故障转移。

**莴苣可以连接到哨兵，用它来发现当前主人的地址，然后返回一个连接到它。**

为此，我们构建了一个不同的`RedisURI`并将我们的`RedisClient`与它连接起来:

```
RedisURI redisUri = RedisURI.Builder
  .sentinel("sentinelhost1", "clustername")
  .withSentinel("sentinelhost2").build();
RedisClient client = new RedisClient(redisUri);

RedisConnection<String, String> connection = client.connect(); 
```

我们用第一个 sentinel 的主机名(或地址)和一个集群名构建了 URI，后跟第二个 Sentinel 地址。当我们连接到 Sentinel 时，莴苣向它查询拓扑结构，并为我们返回一个到当前主服务器的连接。

完整的文档可在此处获得[。](https://web.archive.org/web/20221126235007/https://redis.io/topics/sentinel)

### 9.3。集群

Redis 集群使用分布式配置来提供高可用性和高吞吐量。

**集群共享多达 1000 个节点的密钥，因此集群中的事务不可用:**

```
RedisURI redisUri = RedisURI.Builder.redis("localhost")
  .withPassword("authentication").build();
RedisClusterClient clusterClient = RedisClusterClient
  .create(rediUri);
StatefulRedisClusterConnection<String, String> connection
 = clusterClient.connect();
RedisAdvancedClusterCommands<String, String> syncCommands = connection
  .sync(); 
```

`RedisAdvancedClusterCommands`保存集群支持的 Redis 命令集，将它们路由到保存密钥的实例。

完整规格可在[这里](https://web.archive.org/web/20221126235007/https://redis.io/topics/cluster-spec)获得。

## 10。结论

在本教程中，我们研究了如何使用莴苣从我们的应用程序中连接和查询 Redis 服务器。

莴苣支持完整的 Redis 特性集，以及一个完全线程安全的异步接口。它还大量使用 Java 8 的`CompletionStage`接口，让应用程序对如何接收数据进行细粒度控制。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20221126235007/https://github.com/eugenp/tutorials/tree/master/persistence-modules/redis)