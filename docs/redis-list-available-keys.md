# 列出所有可用的重定向关键点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/redis-list-available-keys>

## 1.概观

集合是几乎所有现代应用程序中常见的基本构件。因此， **Redis 提供各种[流行的数据结构](https://web.archive.org/web/20220703150911/https://redis.io/topics/data-types)** 供我们使用也就不足为奇了，比如列表、集合、散列和有序集合。

在本教程中，我们将学习如何有效地读取所有匹配特定模式的可用 Redis 键。

## 2.探索收藏

假设我们的**应用程序使用 Redis 来存储不同运动中使用的球**的信息。我们应该能够从 Redis 集合中看到每个球的信息。为简单起见，我们将数据集限制为三个球:

*   重量为 160 克的板球
*   重量为 450 克的足球
*   重量为 270 克的排球

像往常一样，让我们首先通过研究一种探索 Redis 集合的简单方法来理清我们的基础知识。

## 3.使用 redis-cli 的简单方法

在我们开始编写 Java 代码来探索这些集合之前，我们应该清楚地知道如何使用 [`redis-cli`](https://web.archive.org/web/20220703150911/https://redis.io/topics/rediscli) 接口来实现。让我们假设我们的 Redis 实例在端口`6379`的`127.0.0.1`可用，让我们用命令行界面探索每种集合类型。

### 3.1.链表

首先，让我们借助于 [`rpush`](https://web.archive.org/web/20220703150911/https://redis.io/commands/rpush) 命令，将数据集以`sports-name` _ `ball-weight `的格式存储在一个名为`balls`的 Redis 链表中:

```java
% redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> RPUSH balls "cricket_160"
(integer) 1
127.0.0.1:6379> RPUSH balls "football_450"
(integer) 2
127.0.0.1:6379> RPUSH balls "volleyball_270"
(integer) 3
```

我们可以注意到，**成功插入列表输出了列表的新长度**。然而，在大多数情况下，我们对数据插入活动视而不见。因此，我们可以使用 [`llen`](https://web.archive.org/web/20220703150911/https://redis.io/commands/llen) 命令找出链表的长度:

```java
127.0.0.1:6379> llen balls
(integer) 3
```

当我们已经知道列表的长度时，很方便**使用 [`lrange`](https://web.archive.org/web/20220703150911/https://redis.io/commands/lrange) 命令**轻松检索整个数据集:

```java
127.0.0.1:6379> lrange balls 0 2
1) "cricket_160"
2) "football_450"
3) "volleyball_270"
```

### 3.2.一组

接下来，让我们看看当我们决定将数据集存储在 Redis 集中时，我们如何浏览数据集。为此，我们首先需要使用 [`sadd`](https://web.archive.org/web/20220703150911/https://redis.io/commands/sadd) 命令将数据集填充到名为 balls 的 Redis 集中:

```java
127.0.0.1:6379> sadd balls "cricket_160" "football_450" "volleyball_270" "cricket_160"
(integer) 3
```

哎呀！我们的命令中有重复的值。但是，由于我们是向集合中添加值，所以不需要担心重复。当然，我们可以从输出响应值中看到添加的项数。

现在，我们可以通过**利用 [`smembers`](https://web.archive.org/web/20220703150911/https://redis.io/commands/smembers) 命令查看所有集合成员**:

```java
127.0.0.1:6379> smembers balls
1) "volleyball_270"
2) "cricket_160"
3) "football_450"
```

### 3.3.混杂

现在，让我们使用 Redis 的散列数据结构将数据集存储在名为 balls 的散列关键字中，这样散列的字段就是运动名称，字段值就是球的重量。我们可以在`[hmset](https://web.archive.org/web/20220703150911/https://redis.io/commands/hmset)`命令的帮助下做到这一点:

```java
127.0.0.1:6379> hmset balls cricket 160 football 450 volleyball 270
OK
```

为了查看存储在我们 hash 中的信息，我们可以使用**命令`[hgetall](https://web.archive.org/web/20220703150911/https://redis.io/commands/hgetall)`T2:**

```java
127.0.0.1:6379> hgetall balls
1) "cricket"
2) "160"
3) "football"
4) "450"
5) "volleyball"
6) "270"
```

### 3.4.排序集合

除了唯一的成员值之外，有序集合允许我们在它们旁边保留一个分数。在我们的用例中，我们可以将运动的名称作为成员值，将球的重量作为分数。让我们使用`[zadd](https://web.archive.org/web/20220703150911/https://redis.io/commands/zadd)`命令来存储我们的数据集:

```java
127.0.0.1:6379> zadd balls 160 cricket 450 football 270 volleyball
(integer) 3
```

现在，我们可以首先使用`[zcard](https://web.archive.org/web/20220703150911/https://redis.io/commands/zcard)`命令来查找有序集的长度，然后使用 **`[zrange](https://web.archive.org/web/20220703150911/https://redis.io/commands/zrange)`命令来探索完整集**:

```java
127.0.0.1:6379> zcard balls
(integer) 3
127.0.0.1:6379> zrange balls 0 2
1) "cricket"
2) "volleyball"
3) "football"
```

### 3.5.用线串

我们也可以把**通常的键值字符串看作是一个肤浅的项目集合**。让我们首先使用`[mset](https://web.archive.org/web/20220703150911/https://redis.io/commands/mset)`命令填充数据集:

```java
127.0.0.1:6379> mset balls:cricket 160 balls:football 450 balls:volleyball 270
OK
```

我们必须注意，我们添加了前缀“balls: `”` ,这样我们就可以从 Redis 数据库中的其他键中识别出这些键。此外，这种命名策略允许我们使用 [`keys`](https://web.archive.org/web/20220703150911/https://redis.io/commands/keys) 命令在前缀模式匹配的帮助下探索我们的数据集:

```java
127.0.0.1:6379> keys balls*
1) "balls:cricket"
2) "balls:volleyball"
3) "balls:football"
```

## 4.简单的 Java 实现

现在我们已经对相关的 Redis 命令有了一个基本的概念，我们可以使用这些命令来探索不同类型的集合，现在是时候让我们用代码来弄脏我们的手了。

### 4.1.Maven 依赖性

在本节中，我们将在我们的实现中使用`[Jedis](/web/20220703150911/https://www.baeldung.com/jedis-java-redis-client-library)`客户端库来实现 Redis:

```java
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 4.2 .客户端重定向

Jedis 库附带了与 Redis-CLI 同名的方法。然而，建议我们**创建一个包装 Redis 客户端，它将在内部调用 Jedis 函数调用**。

每当我们使用 Jedis 库时，我们必须记住**单个 Jedis 实例不是线程安全的**。因此，要在我们的应用程序中获得一个 Jedis 资源，我们可以**利用 [`JedisPool`](/web/20220703150911/https://www.baeldung.com/jedis-java-redis-client-library#Connection) ，这是一个网络连接的线程安全池**。

而且，由于我们不希望在应用程序生命周期中的任何给定时间出现多个 Redis 客户端实例，我们应该根据[单例设计模式](/web/20220703150911/https://www.baeldung.com/java-singleton-double-checked-locking)的原则创建我们的`RedisClient`类。

首先，让我们为我们的客户机创建一个私有构造函数，当`RedisClient`类的实例被创建时，它将在内部初始化`JedisPool`:

```java
private static JedisPool jedisPool;

private RedisClient(String ip, int port) {
    try {
        if (jedisPool == null) {
            jedisPool = new JedisPool(new URI("http://" + ip + ":" + port));
        }
    } catch (URISyntaxException e) {
        log.error("Malformed server address", e);
    }
}
```

接下来，我们需要一个单点客户端的访问点。因此，让我们为此创建一个静态方法`getInstance()`:

```java
private static volatile RedisClient instance = null;

public static RedisClient getInstance(String ip, final int port) {
    if (instance == null) {
        synchronized (RedisClient.class) {
            if (instance == null) {
                instance = new RedisClient(ip, port);
            }
        }
    }
    return instance;
}
```

最后，让我们看看如何在 Jedis 的`lrange method`之上创建一个包装器方法:

```java
public List lrange(final String key, final long start, final long stop) {
    try (Jedis jedis = jedisPool.getResource()) {
        return jedis.lrange(key, start, stop);
    } catch (Exception ex) {
        log.error("Exception caught in lrange", ex);
    }
    return new LinkedList();
}
```

当然，我们可以遵循相同的策略来创建其余的包装器方法，比如`lpush`、`hmset`、`hgetall`、`sadd`、`smembers`、`keys`、`zadd`和`zrange`。

### 4.3.分析

在最好的情况下，我们可以使用所有 Redis 命令**在一次运行中探索一个集合，自然会有 O(n)的时间复杂度。**

我们可能有点开明，称这种方法为幼稚。在 Redis 的实际生产实例中，在一个集合中有数千或数百万个键是很常见的。此外， [Redis 的单线程特性](https://web.archive.org/web/20220703150911/https://redis.io/topics/latency#single-threaded-nature-of-redis)带来了更多的痛苦，我们的方法可能会灾难性地阻塞其他更高优先级的操作。

所以，我们应该强调的是，我们将我们幼稚的方法限制为仅用于调试目的。

## 5.迭代器基础

我们天真的实现中的主要缺陷是，我们要求 Redis 一次给我们单一 fetch-query 的所有结果。为了克服这个问题，我们可以将原始的 fetch 查询分成多个顺序 fetch 查询，这些查询对整个数据集的较小块进行操作。

让我们假设我们有一本 1000 页的书要读。如果我们按照我们天真的方法，我们将不得不一口气读完这本大部头的书。这对我们的幸福来说是致命的，因为它会耗尽我们的精力，阻止我们做任何其他更重要的事情。

当然，正确的方法是通过多次阅读来完成这本书。在每节课中，**我们从上一节课**停止的地方继续——我们可以**通过使用页面书签**来跟踪我们的进度。

尽管两种情况下的总阅读时间价值相当，但第二种方法更好，因为它给了我们喘息的空间。

让我们看看如何使用基于迭代器的方法来探索 Redis 集合。

## 6.重定向扫描

Redis 提供了几种扫描策略，使用基于光标的方法从集合中读取键，这在原理上类似于页面书签。

### 6.1.扫描策略

我们可以使用`[Scan](https://web.archive.org/web/20220703150911/https://redis.io/commands/scan)`命令扫描整个键值集合存储。但是，如果我们想通过集合类型来限制数据集，那么我们可以使用其中一个变量:

*   `[Sscan](https://web.archive.org/web/20220703150911/https://redis.io/commands/sscan)`可用于迭代集合
*   [`Hscan`](https://web.archive.org/web/20220703150911/https://redis.io/commands/hscan) 帮助我们遍历散列中的字段值对
*   [`Zscan`](https://web.archive.org/web/20220703150911/https://redis.io/commands/zscan) 允许对存储在一个有序集合中的成员进行迭代

我们必须注意，我们**并不真的需要专门为链表**设计的服务器端扫描策略。这是因为我们可以使用`[lindex](https://web.archive.org/web/20220703150911/https://redis.io/commands/lindex)`或 [`lrange`](https://web.archive.org/web/20220703150911/https://redis.io/commands/lrange) 命令通过索引访问链表的成员。另外，我们可以找出元素的数量，并在一个简单的循环中使用`lrange`以小块的形式迭代整个列表。

让我们使用`SCAN`命令扫描字符串类型的键。**要开始扫描，我们需要使用光标值为" 0"** ，匹配模式字符串为" ball* ":

```java
127.0.0.1:6379> mset balls:cricket 160 balls:football 450 balls:volleyball 270
OK
127.0.0.1:6379> SCAN 0 MATCH ball* COUNT 1
1) "2"
2) 1) "balls:cricket"
127.0.0.1:6379> SCAN 2 MATCH ball* COUNT 1
1) "3"
2) 1) "balls:volleyball"
127.0.0.1:6379> SCAN 3 MATCH ball* COUNT 1
1) "0"
2) 1) "balls:football"
```

随着每一次扫描的完成，我们得到下一个游标值，用于后续的迭代。最终，我们知道当下一个光标值为“0”时，我们已经扫描了整个集合。

## 7.用 Java 扫描

到目前为止，我们已经对我们的方法有了足够的了解，可以开始用 Java 实现它了。

### 7.1.扫描策略

如果我们深入了解由`Jedis`类提供的核心扫描功能，我们会发现扫描不同集合类型的策略:

```java
public ScanResult<String> scan(final String cursor, final ScanParams params);
public ScanResult<String> sscan(final String key, final String cursor, final ScanParams params);
public ScanResult<Map.Entry<String, String>> hscan(final String key, final String cursor,
  final ScanParams params);
public ScanResult<Tuple> zscan(final String key, final String cursor, final ScanParams params);
```

`Jedis`需要两个可选参数，**搜索模式和结果大小，以有效地控制扫描-`ScanParams`使其发生**。为此，它依赖于`match()`和`count()`方法，这两种方法大致基于[构建器设计模式](/web/20220703150911/https://www.baeldung.com/creational-design-patterns#builder):

```java
public ScanParams match(final String pattern);
public ScanParams count(final Integer count);
```

既然我们已经掌握了关于`Jedis's`扫描方法的基础知识，让我们通过`ScanStrategy`界面来模拟这些策略:

```java
public interface ScanStrategy<T> {
    ScanResult<T> scan(Jedis jedis, String cursor, ScanParams scanParams);
}
```

首先，让我们研究最简单的`scan`策略，它独立于集合类型，读取键，但不读取键的值:

```java
public class Scan implements ScanStrategy<String> {
    public ScanResult<String> scan(Jedis jedis, String cursor, ScanParams scanParams) {
        return jedis.scan(cursor, scanParams);
    }
}
```

接下来，让我们来看看`hscan`策略，它是为读取特定散列键的所有字段键和字段值而定制的:

```java
public class Hscan implements ScanStrategy<Map.Entry<String, String>> {

    private String key;

    @Override
    public ScanResult<Entry<String, String>> scan(Jedis jedis, String cursor, ScanParams scanParams) {
        return jedis.hscan(key, cursor, scanParams);
    }
}
```

最后，让我们为集合和有序集合建立策略。`sscan`策略可以读取一个集合的所有成员，而`zscan`策略可以读取成员以及他们以`Tuple` s:

```java
public class Sscan implements ScanStrategy<String> {

    private String key;

    public ScanResult<String> scan(Jedis jedis, String cursor, ScanParams scanParams) {
        return jedis.sscan(key, cursor, scanParams);
    }
}

public class Zscan implements ScanStrategy<Tuple> {

    private String key;

    @Override
    public ScanResult<Tuple> scan(Jedis jedis, String cursor, ScanParams scanParams) {
        return jedis.zscan(key, cursor, scanParams);
    }
}
```

### 7.2 .Redis 迭代器

接下来，让我们勾画出构建我们的`RedisIterator`类所需的构件:

*   基于字符串的光标
*   扫描策略如`scan`、sscan、`hscan`、`zscan`
*   扫描参数的占位符
*   访问`JedisPool`以获得`Jedis`资源

我们现在可以在我们的`RedisIterator`类中定义这些成员:

```java
private final JedisPool jedisPool;
private ScanParams scanParams;
private String cursor;
private ScanStrategy<T> strategy;
```

我们的阶段是为迭代器定义特定于迭代器的功能。为此，我们的`RedisIterator`类必须实现 [`Iterator`](/web/20220703150911/https://www.baeldung.com/java-iterator) 接口:

```java
public class RedisIterator<T> implements Iterator<List<T>> {
}
```

自然，我们需要覆盖从`Iterator`接口继承的`hasNext()`和`next()`方法。

首先，让我们挑选最容易摘到的果子——`hasNext()`方法——因为它的基本逻辑很简单。**当光标值变为“0”时，我们知道已经完成了**扫描。那么，让我们看看如何用一行代码实现它:

```java
@Override
public boolean hasNext() {
    return !"0".equals(cursor);
}
```

接下来，让我们来研究执行繁重扫描的`next()`方法:

```java
@Override
public List next() {
    if (cursor == null) {
        cursor = "0";
    }
    try (Jedis jedis = jedisPool.getResource()) {
        ScanResult scanResult = strategy.scan(jedis, cursor, scanParams);
        cursor = scanResult.getCursor();
        return scanResult.getResult();
    } catch (Exception ex) {
        log.error("Exception caught in next()", ex);
    }
    return new LinkedList();
}
```

我们必须注意到 **`ScanResult`不仅给出了扫描结果，还给出后续扫描所需的下一个光标值**。

最后，我们可以启用在`RedisClient`类中创建`RedisIterator`的功能:

```java
public RedisIterator iterator(int initialScanCount, String pattern, ScanStrategy strategy) {
    return new RedisIterator(jedisPool, initialScanCount, pattern, strategy);
}
```

### 7.3 .用递归迭代器读取

由于我们已经在`Iterator`接口的帮助下设计了 Redis 迭代器，所以只要`hasNext()`返回`true` ，那么**就可以很直观地在`next()`方法的帮助下读取集合值。**

为了完整和简单起见，我们首先将与运动球相关的数据集存储在 Redis 散列中。之后，我们将使用`RedisClient`创建一个使用`Hscan`扫描策略的迭代器。让我们通过实际操作来测试我们的实现:

```java
@Test
public void testHscanStrategy() {
    HashMap<String, String> hash = new HashMap<String, String>();
    hash.put("cricket", "160");
    hash.put("football", "450");
    hash.put("volleyball", "270");
    redisClient.hmset("balls", hash);

    Hscan scanStrategy = new Hscan("balls");
    int iterationCount = 2;
    RedisIterator iterator = redisClient.iterator(iterationCount, "*", scanStrategy);
    List<Map.Entry<String, String>> results = new LinkedList<Map.Entry<String, String>>();
    while (iterator.hasNext()) {
        results.addAll(iterator.next());
    }
    Assert.assertEquals(hash.size(), results.size());
}
```

我们可以遵循相同的思维过程，只需稍加修改，就可以测试和实现扫描和读取不同类型集合中可用的键的剩余策略。

## 8.结论

我们开始本教程的目的是学习如何在 Redis 中读取所有匹配的键。

我们发现 Redis 提供了一种简单的方法来一次性读取密钥。虽然简单，但我们讨论了这是如何对资源造成压力的，因此不适合生产系统。深入挖掘后，我们知道有一种基于**迭代器的方法，可以通过匹配 Redis 键来扫描**,以进行读取查询。

和往常一样，本文中使用的 Java 实现的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220703150911/https://github.com/eugenp/tutorials/tree/master/persistence-modules/redis)