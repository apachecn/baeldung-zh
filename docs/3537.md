# 删除 Redis 中的所有内容

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/redis-delete-data>

## 1.概观

在 Redis 中进行缓存时，当缓存失效时清除整个缓存会很有用。

在这个简短的教程中，我们将学习如何删除 Redis 中的所有键，包括特定数据库和所有数据库中的键。

首先，我们来看看命令行。然后，我们将看到如何使用 API 和 Java 客户端来完成同样的事情。

## 2.跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑

我们需要安装一个 Redis 来使用。在 [Redis 快速入门](https://web.archive.org/web/20220524114953/https://redis.io/topics/quickstart)中有 Mac 和 Linux 的安装说明。在 docker 中运行 Redis 可能更容易。

让我们启动一个测试 Redis 服务器:

```
docker run --name redis -p 6379:6379 -d redis:latest
```

并且，我们可以运行 [`redis-cli`](https://web.archive.org/web/20220524114953/https://redis.io/topics/rediscli) 来测试该服务器是否工作正常:

```
docker exec -it redis redis-cli
```

这将我们带入 cli shell，其中命令`ping`将测试服务器是否启动:

```
127.0.0.1:6379> ping
PONG
```

我们用 CTRL+C 退出`redis-cli`。

## 3.重复命令

让我们从 Redis 命令开始删除所有内容。

有两个主要的命令可以删除 Redis 中的键: **`FLUSHDB`和`FLUSHALL`** 。我们可以使用 Redis CLI 来执行这些命令。

`FLUSHDB`命令删除数据库中的键。而`FLUSHALL`命令删除所有数据库中的所有键。

**我们可以使用`ASYNC`选项在后台线程中执行这些操作。**如果刷新需要很长时间，这很有用，因为发出命令`ASYNC`会阻止它阻塞，直到它完成。

我们应该注意到，Redis 4.0.0 中提供了`ASYNC`选项。

## 4.使用 Java 客户端

现在，让我们看看如何使用 [Jedis](/web/20220524114953/https://www.baeldung.com/jedis-java-redis-client-library) Java 客户端删除密钥。

### 4.1.属国

首先，我们需要为 [Jedis](https://web.archive.org/web/20220524114953/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22redis.clients%22%20AND%20a%3A%22jedis%22) 添加 Maven 依赖项:

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
```

为了使测试更容易，让我们也使用一个[嵌入式 Redis 服务器](https://web.archive.org/web/20220524114953/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.github.kstyrc%22%20AND%20a%3A%22embedded-redis%22):

```
<dependency>
    <groupId>com.github.kstyrc</groupId>
    <artifactId>embedded-redis</artifactId>
    <version>0.6</version>
</dependency>
```

### 4.2.启动嵌入式 Redis

我们将创建一个嵌入式 Redis 服务器进行测试，在一个可用的端口上运行它:

```
RedisService redisServer = new RedisServer(port);
```

然后，我们的 Jedis 客户机被创建，并使用`localhost`作为主机名，以及相同的端口:

```
Jedis jedis = new Jedis("localhost", port);
```

## 5.刷新单个数据库

让我们将一些数据放入数据库，并检查它是否被记住:

```
String key = "key";
String value = "value";
jedis.set(key, value);
String received = jedis.get(key);

assertEquals(value, received);
```

现在让我们使用`flushDB` 方法来**刷新数据库:**

```
jedis.flushDB();

assertNull(jedis.get(key));
```

如我们所见，尝试在刷新后检索值会返回`null`。

## 6.清除所有数据库

Redis 提供了多个数据库，这些数据库是有编号的。**在添加值之前，我们可以使用`select`命令**将数据添加到不同的数据库中:

```
jedis.select(0);
jedis.set("key1", "value1");
jedis.select(1);
jedis.set("key2", "value2");
```

我们现在应该在两个数据库中各有一个键:

```
jedis.select(0);
assertEquals("value1", jedis.get("key1"));
assertNull(jedis.get("key2"));

jedis.select(1);
assertEquals("value2", jedis.get("key2"));
assertNull(jedis.get("key1"));
```

**`flushDB`方法只会清除当前数据库**。为了清除所有数据库，我们使用`flushAll`方法:

```
jedis.flushAll();
```

我们可以测试这是否有效:

```
jedis.select(0);

assertNull(jedis.get("key1"));
assertNull(jedis.get("key2"));

jedis.select(1);

assertNull(jedis.get("key1"));
assertNull(jedis.get("key2"));
```

## 7.时间复杂度

Redis 是一个可伸缩性很好的快速数据存储。但是，当有更多数据时，刷新操作可能需要更长时间。

`FLUSHDB`操作的[时间复杂度](/web/20220524114953/https://www.baeldung.com/java-algorithm-complexity)为`O(N)`，其中`N`为数据库中的键数。如果我们使用`FLUSHALL`命令，时间复杂度又是`O(N)`，但是在这里，`N`是所有数据库中的键的数量。

## 8.结论

在本文中，我们看到了如何在 Docker 中运行 Redis 和`redis-cli`，以及如何在嵌入式 Redis 服务器上使用 Jedis Java 客户端。

我们看到了如何将数据保存在不同的 Redis 数据库中，以及如何使用 flush 命令清除其中的一个或多个数据库。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524114953/https://github.com/eugenp/tutorials/tree/master/persistence-modules/redis)