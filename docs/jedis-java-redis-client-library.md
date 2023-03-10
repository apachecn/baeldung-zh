# intro to jedis–Java redis 客户端库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jedis-java-redis-client-library>

## 1。概述

本文是对 Jedis T1 的介绍，这是一个用于 T2 Redis T3 的 Java 客户端库，这是一种流行的内存数据结构存储，也可以保存在磁盘上。它由基于密钥库的数据结构驱动来持久存储数据，并可用作数据库、缓存、消息代理等。

首先，我们将解释 Jedis 在什么样的情况下有用，以及它是关于什么的。

在接下来的章节中，我们将详细阐述各种数据结构，并解释事务、管道和发布/订阅功能。我们以连接池和 Redis 集群作为结论。

## 2 .为什么是 Jedis？

Redis 在其官方网站上列出了最著名的客户端库。杰迪斯有多种选择，但目前只有两种更值得他们推荐的明星，[莴苣](https://web.archive.org/web/20220529014815/https://paluch.biz/)和[雷迪森](https://web.archive.org/web/20220529014815/https://github.com/mrniko/redisson/)。

这两个客户端确实有一些独特的特性，比如线程安全、透明的重新连接处理和异步 API，所有这些特性都是 Jedis 所缺乏的。

然而，它很小，而且比另外两个要快得多。此外，它是 Spring 框架开发人员的首选客户端库，并且它拥有这三者中最大的社区。

## 3。Maven 依赖关系

让我们从声明我们将在`pom.xml`中需要的唯一依赖项开始:

```java
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.8.1</version>
</dependency> 
```

如果你正在寻找这个库的最新版本，请查看这个页面。

## 4 .Redis 安装〔t1〕

你需要安装并启动最新版本的 Redis。我们此刻运行的是最新的稳定版(3.2.1)，不过任何 3.x 以后的版本应该都没问题。

找到[这里](https://web.archive.org/web/20220529014815/http://redis.io/topics/quickstart)更多关于 Redis for Linux 和 Macintosh 的信息，它们有非常相似的基本安装步骤。Windows 没有官方支持，但是这个[端口](https://web.archive.org/web/20220529014815/https://github.com/MSOpenTech/redis)维护的很好。

之后，我们可以直接从 Java 代码中进入并连接到它:

```java
Jedis jedis = new Jedis();
```

默认的构造函数可以正常工作，除非您已经在非默认端口或远程机器上启动了服务，在这种情况下，您可以通过将正确的值作为参数传递给构造函数来正确地配置它。

## 5。Redis 数据结构〔t1〕

大多数本机操作命令都受支持，而且非常方便的是，它们通常共享相同的方法名。

### 5.1。字符串

字符串是最基本的 Redis 值，在需要持久存储简单的键值数据类型时非常有用:

```java
jedis.set("events/city/rome", "32,15,223,828");
String cachedResponse = jedis.get("events/city/rome");
```

变量`cachedResponse`将保存值`32,15,223,828`。再加上到期支持(稍后讨论),它可以像闪电一样快速、简单地使用缓存层来处理 web 应用程序收到的 HTTP 请求和其他缓存需求。

### 5.2。列表

Redis 列表是简单的字符串列表，按插入顺序排序，是实现例如消息队列的理想工具:

```java
jedis.lpush("queue#tasks", "firstTask");
jedis.lpush("queue#tasks", "secondTask");

String task = jedis.rpop("queue#tasks");
```

变量`task`将保存值`firstTask`。请记住，您可以序列化任何对象并将其作为字符串保存，因此队列中的消息可以在需要时携带更复杂的数据。

### 5.3。设置

Redis 集是一个无序的字符串集合，当您想要排除重复的成员时会很方便:

```java
jedis.sadd("nicknames", "nickname#1");
jedis.sadd("nicknames", "nickname#2");
jedis.sadd("nicknames", "nickname#1");

Set<String> nicknames = jedis.smembers("nicknames");
boolean exists = jedis.sismember("nicknames", "nickname#1");
```

Java 集合`nicknames`的大小为 2，第二次添加的`nickname#1`被忽略。同样，`exists`变量的值为`true`，方法`sismember`使您能够快速检查特定成员的存在。

### 5.4。哈希

Redis 散列在*字符串*字段和*字符串*值之间进行映射:

```java
jedis.hset("user#1", "name", "Peter");
jedis.hset("user#1", "job", "politician");

String name = jedis.hget("user#1", "name");

Map<String, String> fields = jedis.hgetAll("user#1");
String job = fields.get("job");
```

如您所见，当您想要单独访问对象的属性时，散列是一种非常方便的数据类型，因为您不需要检索整个对象。

### 5.5。有序集合

已排序集合类似于这样一个集合，其中每个成员都有一个关联的排名，用于对其进行排序:

```java
Map<String, Double> scores = new HashMap<>();

scores.put("PlayerOne", 3000.0);
scores.put("PlayerTwo", 1500.0);
scores.put("PlayerThree", 8200.0);

scores.entrySet().forEach(playerScore -> {
    jedis.zadd(key, playerScore.getValue(), playerScore.getKey());
});

String player = jedis.zrevrange("ranking", 0, 1).iterator().next();
long rank = jedis.zrevrank("ranking", "PlayerOne");
```

变量`player`将保存值`PlayerThree`,因为我们正在检索排名前 1 的玩家，他是得分最高的玩家。`rank`变量的值为 1，因为`PlayerOne`是排名中的第二位，并且排名是从零开始的。

## 6。交易

事务保证原子性和线程安全操作，这意味着在 Redis 事务期间，来自其他客户端的请求将永远不会被并发处理:

```java
String friendsPrefix = "friends#";
String userOneId = "4352523";
String userTwoId = "5552321";

Transaction t = jedis.multi();
t.sadd(friendsPrefix + userOneId, userTwoId);
t.sadd(friendsPrefix + userTwoId, userOneId);
t.exec();
```

您甚至可以在实例化您的`Transaction`之前通过“观察”特定的键来使事务成功:

```java
jedis.watch("friends#deleted#" + userOneId);
```

如果该项的值在事务执行之前发生了变化，则事务将不会成功完成。

## 7。流水线作业

当我们必须发送多个命令时，我们可以将它们打包在一个请求中，并通过使用管道来节省连接开销，这本质上是一种网络优化。只要操作是相互独立的，我们就可以利用这种技术:

```java
String userOneId = "4352523";
String userTwoId = "4849888";

Pipeline p = jedis.pipelined();
p.sadd("searched#" + userOneId, "paris");
p.zadd("ranking", 126, userOneId);
p.zadd("ranking", 325, userTwoId);
Response<Boolean> pipeExists = p.sismember("searched#" + userOneId, "paris");
Response<Set<String>> pipeRanking = p.zrange("ranking", 0, -1);
p.sync();

String exists = pipeExists.get();
Set<String> ranking = pipeRanking.get();
```

注意，我们不能直接访问命令响应，相反，我们得到了一个`Response`实例，在管道同步后，我们可以从该实例请求底层响应。

## 8。发布/订阅

我们可以使用 Redis messaging broker 功能在系统的不同组件之间发送消息。确保订阅者线程和发布者线程不共享同一个 Jedis 连接。

### 8.1。订户

订阅并收听发送到频道的消息:

```java
Jedis jSubscriber = new Jedis();
jSubscriber.subscribe(new JedisPubSub() {
    @Override
    public void onMessage(String channel, String message) {
        // handle message
    }
}, "channel");
```

Subscribe 是一个阻塞方法，你需要明确地从`JedisPubSub`中取消订阅。我们已经覆盖了`onMessage`方法，但是还有更多[有用的方法](https://web.archive.org/web/20220529014815/http://javadox.com/redis.clients/jedis/2.8.0/redis/clients/jedis/JedisPubSub.html)可以覆盖。

### 8.2。发布者

然后简单地从发布者的线程向相同的通道发送消息:

```java
Jedis jPublisher = new Jedis();
jPublisher.publish("channel", "test message");
```

## 9。连接池

重要的是要知道，我们处理 Jedis 实例的方式是幼稚的。在现实世界中，您不希望在多线程环境中使用单个实例，因为单个实例不是线程安全的。

幸运的是，我们可以很容易地创建一个到 Redis 的连接池，供我们按需重用，这个池是线程安全和可靠的，只要您在使用完资源后将它返回到池中。

让我们创建`JedisPool`:

```java
final JedisPoolConfig poolConfig = buildPoolConfig();
JedisPool jedisPool = new JedisPool(poolConfig, "localhost");

private JedisPoolConfig buildPoolConfig() {
    final JedisPoolConfig poolConfig = new JedisPoolConfig();
    poolConfig.setMaxTotal(128);
    poolConfig.setMaxIdle(128);
    poolConfig.setMinIdle(16);
    poolConfig.setTestOnBorrow(true);
    poolConfig.setTestOnReturn(true);
    poolConfig.setTestWhileIdle(true);
    poolConfig.setMinEvictableIdleTimeMillis(Duration.ofSeconds(60).toMillis());
    poolConfig.setTimeBetweenEvictionRunsMillis(Duration.ofSeconds(30).toMillis());
    poolConfig.setNumTestsPerEvictionRun(3);
    poolConfig.setBlockWhenExhausted(true);
    return poolConfig;
}
```

因为池实例是线程安全的，所以您可以将它静态地存储在某个地方，但是您应该注意在应用程序关闭时销毁池以避免泄漏。

现在，我们可以在需要时从应用程序的任何地方使用我们的池:

```java
try (Jedis jedis = jedisPool.getResource()) {
    // do operations with jedis resource
}
```

我们使用 Java try-with-resources 语句来避免手动关闭 Jedis 资源，但是如果不能使用该语句，也可以在 *finally* 子句中手动关闭资源。

如果您不想面对讨厌的多线程问题，请确保使用我们在您的应用程序中描述的池。显然，您可以调整池配置参数，使其适应您系统中的最佳设置。

## 10。Redis 集群〔t1〕

这个 Redis 实现提供了简单的可伸缩性和高可用性，如果您不熟悉它，我们鼓励您阅读他们的官方规范。我们将不讨论 Redis 集群设置，因为这超出了本文的范围，但是当您阅读完它的文档后，您应该不会有任何问题。

一旦我们准备好了，我们就可以开始在我们的应用程序中使用它:

```java
try (JedisCluster jedisCluster = new JedisCluster(new HostAndPort("localhost", 6379))) {
    // use the jedisCluster resource as if it was a normal Jedis resource
} catch (IOException e) {}
```

我们只需要提供一个主实例的主机和端口详细信息，它将自动发现集群中的其余实例。

这当然是一个非常强大的功能，但它不是灵丹妙药。使用 Redis 集群时，不能执行事务，也不能使用管道，这是许多应用程序赖以确保数据完整性的两个重要特性。

事务被禁用，因为在集群环境中，键将跨多个实例持久化。对于涉及不同实例中的命令执行的操作，不能保证操作原子性和线程安全性。

一些高级的键创建策略将确保您感兴趣的数据在同一个实例中得到持久化。理论上，这应该使您能够使用 Redis 集群的一个底层 Jedis 实例成功地执行事务。

不幸的是，目前您无法使用 Jedis(实际上 Redis 本身就支持 Jedis)找到特定键保存在哪个 Redis 实例中，因此您不知道必须在哪个实例中执行事务操作。如果你对此感兴趣，可以在这里找到更多信息[。](https://web.archive.org/web/20220529014815/https://groups.google.com/forum/#!topic/redis-db/4I0ELYnf3bk)

## 11。结论

Redis 的绝大多数特性在 Jedis 中已经可以使用，并且它的开发进展很快。

它使您能够毫不费力地将强大的内存存储引擎集成到您的应用程序中，只是不要忘记设置连接池以避免线程安全问题。

你可以在 [GitHub 项目](https://web.archive.org/web/20220529014815/https://github.com/eugenp/tutorials/tree/master/persistence-modules/redis)中找到代码示例。