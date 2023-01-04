# 咖啡因简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-caching-caffeine>

## 1。简介

在本文中，我们将看看[Caffeine](https://web.archive.org/web/20221206104739/https://github.com/ben-manes/caffeine)——一个用于 Java 的**高性能缓存库。**

缓存和`Map`的一个根本区别是缓存会驱逐存储的项目。

一个**驱逐策略决定在任何给定的时间应该删除哪些对象**。这个策略**直接影响缓存的命中率**——缓存库的一个重要特征。

咖啡因使用`Window TinyLfu`驱逐策略，这提供了**接近最优的命中率**。

## 2。依赖性

我们需要将`caffeine`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.5.5</version>
</dependency>
```

你可以在 Maven Central 上找到最新版本的*咖啡因* [。](https://web.archive.org/web/20221206104739/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.github.ben-manes.caffeine%22%20AND%20a%3A%22caffeine%22)

## 3。填充缓存

让我们来关注一下 Caffeine 的**缓存填充的三种策略**:手动、同步加载和异步加载。

首先，让我们为将要存储在缓存中的值类型编写一个类:

```java
class DataObject {
    private final String data;

    private static int objectCounter = 0;
    // standard constructors/getters

    public static DataObject get(String data) {
        objectCounter++;
        return new DataObject(data);
    }
}
```

### 3.1。手动填充

在这种策略中，我们手动将值放入缓存，稍后再检索它们。

让我们初始化缓存:

```java
Cache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .maximumSize(100)
  .build();
```

现在，**我们可以使用`getIfPresent` 方法**从缓存中获取一些值。如果值不在缓存中，该方法将返回`null` :

```java
String key = "A";
DataObject dataObject = cache.getIfPresent(key);

assertNull(dataObject);
```

我们可以使用`put`方法手动填充缓存:

```java
cache.put(key, dataObject);
dataObject = cache.getIfPresent(key);

assertNotNull(dataObject);
```

**我们也可以使用`get`方法**来获取值，该方法使用一个`Function` 和一个键作为参数。如果缓存中不存在关键字，此函数将用于提供回退值，该值将在计算后插入缓存中:

```java
dataObject = cache
  .get(key, k -> DataObject.get("Data for A"));

assertNotNull(dataObject);
assertEquals("Data for A", dataObject.getData());
```

`get`方法自动执行计算。这意味着计算将只进行一次——即使几个线程同时请求该值。这就是为什么**用`get` 比`getIfPresent`** 更可取。

有时我们需要手动使一些缓存值无效:

```java
cache.invalidate(key);
dataObject = cache.getIfPresent(key);

assertNull(dataObject);
```

### 3.2。同步加载

这种加载缓存的方法需要一个用于初始化值的`Function,` ，类似于手动策略的`get` 方法。让我们看看如何利用这一点。

首先，我们需要初始化我们的缓存:

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

现在我们可以使用`get` 方法检索这些值:

```java
DataObject dataObject = cache.get(key);

assertNotNull(dataObject);
assertEquals("Data for " + key, dataObject.getData());
```

我们还可以使用`getAll`方法获得一组值:

```java
Map<String, DataObject> dataObjectMap 
  = cache.getAll(Arrays.asList("A", "B", "C"));

assertEquals(3, dataObjectMap.size());
```

从传递给`build`方法的底层后端初始化`Function`中检索值。**这使得使用缓存作为访问值的主要门面成为可能。**

### 3.3。异步加载

这个策略**的工作方式与前面的相同，但是异步执行操作，并返回一个保存实际值的`CompletableFuture`** :

```java
AsyncLoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .buildAsync(k -> DataObject.get("Data for " + k));
```

我们可以用同样的方式**使用`get`和`getAll`方法**，考虑到它们返回`CompletableFuture`:

```java
String key = "A";

cache.get(key).thenAccept(dataObject -> {
    assertNotNull(dataObject);
    assertEquals("Data for " + key, dataObject.getData());
});

cache.getAll(Arrays.asList("A", "B", "C"))
  .thenAccept(dataObjectMap -> assertEquals(3, dataObjectMap.size()));
```

`CompletableFuture`拥有丰富而有用的 API，你可以在本文中了解更多关于[的内容。](/web/20221206104739/https://www.baeldung.com/java-completablefuture)

## 4。数值驱逐

咖啡因有三种价值驱逐策略:基于大小、基于时间和基于参考。

### 4.1。基于大小的驱逐

这种类型的驱逐假设当超过高速缓存的配置大小限制时发生**驱逐。有两种**方法可以获得大小**——计算缓存中的对象，或者获得它们的权重。**

让我们看看如何对缓存中的对象进行计数。初始化缓存时，其大小等于零:

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(1)
  .build(k -> DataObject.get("Data for " + k));

assertEquals(0, cache.estimatedSize());
```

当我们增加一个值时，大小明显增加:

```java
cache.get("A");

assertEquals(1, cache.estimatedSize());
```

我们可以将第二个值添加到缓存中，这将导致删除第一个值:

```java
cache.get("B");
cache.cleanUp();

assertEquals(1, cache.estimatedSize());
```

值得一提的是，在获取缓存大小之前，我们**调用了`cleanUp` 方法。这是因为缓存回收是异步执行的，这个方法**有助于等待回收**的完成。**

我们还可以通过**传递一个`weigher`** `**Function**` 来获得缓存的大小:

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumWeight(10)
  .weigher((k,v) -> 5)
  .build(k -> DataObject.get("Data for " + k));

assertEquals(0, cache.estimatedSize());

cache.get("A");
assertEquals(1, cache.estimatedSize());

cache.get("B");
assertEquals(2, cache.estimatedSize());
```

当权重超过 10:

```java
cache.get("C");
cache.cleanUp();

assertEquals(2, cache.estimatedSize());
```

### 4.2。基于时间的驱逐

该驱逐策略**基于条目**的到期时间，并且具有三种类型:

*   **访问后过期** —条目在自最后一次读取或写入发生起经过一段时间后过期
*   **写入后过期** —条目在自最后一次写入发生起经过一段时间后过期
*   **自定义策略**——由`Expiry` 实现为每个条目单独计算到期时间

让我们使用`expireAfterAccess`方法配置访问后过期策略:

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterAccess(5, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

为了配置写后过期策略，我们使用了`expireAfterWrite`方法:

```java
cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .weakKeys()
  .weakValues()
  .build(k -> DataObject.get("Data for " + k));
```

为了初始化定制策略，我们需要实现`Expiry` 接口:

```java
cache = Caffeine.newBuilder().expireAfter(new Expiry<String, DataObject>() {
    @Override
    public long expireAfterCreate(
      String key, DataObject value, long currentTime) {
        return value.getData().length() * 1000;
    }
    @Override
    public long expireAfterUpdate(
      String key, DataObject value, long currentTime, long currentDuration) {
        return currentDuration;
    }
    @Override
    public long expireAfterRead(
      String key, DataObject value, long currentTime, long currentDuration) {
        return currentDuration;
    }
}).build(k -> DataObject.get("Data for " + k));
```

### 4.3。基于引用的驱逐

我们可以配置我们的缓存来允许缓存键和/或值的**垃圾收集**。为此，我们将为键和值配置`WeakRefence`的用法，并且我们可以为值的垃圾收集配置`SoftReference`。

当没有任何对对象的强引用时，`WeakRefence`用法允许对象的垃圾收集。`SoftReference`允许基于 JVM 的全局最近最少使用策略对对象进行垃圾收集。关于 Java 中引用的更多细节可以在[这里](/web/20221206104739/https://www.baeldung.com/java-weakhashmap)找到。

我们应该使用`Caffeine.weakKeys()`、`Caffeine.weakValues(),`和`Caffeine.softValues()` 来启用每个选项:

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .weakKeys()
  .weakValues()
  .build(k -> DataObject.get("Data for " + k));

cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .softValues()
  .build(k -> DataObject.get("Data for " + k));
```

## 5。刷新

可以将缓存配置为在定义的时间段后自动刷新条目。让我们看看如何使用`refreshAfterWrite`方法来实现这一点:

```java
Caffeine.newBuilder()
  .refreshAfterWrite(1, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

这里我们要了解一下`expireAfter`和`refreshAfter` 的一个**区别。当过期的条目被请求时，一个执行阻塞，直到新的值被构建`Function`计算出来。**

但是如果条目符合刷新条件，那么高速缓存将返回一个旧值并且**异步地重新加载值**。

## 6。统计数据

咖啡因有一种方法**记录关于缓存使用的统计数据**:

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .recordStats()
  .build(k -> DataObject.get("Data for " + k));
cache.get("A");
cache.get("A");

assertEquals(1, cache.stats().hitCount());
assertEquals(1, cache.stats().missCount());
```

我们还可以传递给`recordStats` supplier，它创建了一个`StatsCounter.` 的实现，这个对象将随着每一个与统计相关的变化而被推送。

## 7。结论

在本文中，我们熟悉了面向 Java 的咖啡因缓存库。我们看到了如何配置和填充缓存，以及如何根据我们的需求选择适当的过期或刷新策略。

这里显示的源代码可以在 Github 上的[处获得。](https://web.archive.org/web/20221206104739/https://github.com/eugenp/tutorials/tree/master/libraries-5)