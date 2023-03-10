# JCache 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jcache>

## 1。概述

简单来说， [JCache](https://web.archive.org/web/20221128113145/https://www.javadoc.io/doc/javax.cache/cache-api/1.0.0) 是 Java 的标准缓存 API。在本教程中，我们将了解什么是 JCache 以及如何使用它。

## 2。Maven 依赖关系

要使用 JCache，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
    <version>1.1.1</version>
</dependency>
```

注意，我们可以在 [Maven 中央存储库](https://web.archive.org/web/20221128113145/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22cache-api%22)中找到该库的最新版本。

我们还需要将 API 的实现添加到我们的`pom.xml`中；我们将在这里使用 Hazelcast:

```java
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>5.2.0</version>
</dependency>
```

我们还可以在 Hazelcast 的 [Maven 中央存储库](https://web.archive.org/web/20221128113145/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hazelcast%22)中找到它的最新版本。

## 3。JCache 实现

JCache 由各种缓存解决方案实现:

*   JCache 参考实现
*   黑泽尔卡斯特
*   Oracle Coherence
*   超高速公路
*   无限跨度

注意，与其他参考实现不同，**不建议在生产中使用 JCache 参考实现，因为它会导致一些并发问题。**

## 4。主要部件

### 4.1。`Cache`

`Cache`接口有以下有用的方法:

*   `get()`–将元素的键作为参数，返回元素的值；如果`Cache`中不存在该键，则返回`null`
*   `getAll()`–多个键可以作为一个`Set; t`传递给该方法。该方法将给定的键和相关值作为一个`Map`返回
*   `getAndRemove()`–该方法使用其键检索一个值，并从`Cache`中删除该元素
*   `put()`–在`Cache`中插入一个新项目
*   `clear()`–删除`Cache`中的所有元素
*   `containsKey()`–检查`Cache`是否包含特定的键

正如我们所看到的，这些方法的名字是不言自明的。有关这些方法和其他方法的更多信息，请访问 [Javadoc](https://web.archive.org/web/20221128113145/https://www.javadoc.io/doc/javax.cache/cache-api/1.0.0) 。

### 4.2。`CacheManager`

`CacheManager`是 API 最重要的接口之一。它使我们能够建立、配置和关闭`Caches`。

### 4.3。`CachingProvider`

`CachingProvider`是一个界面，允许我们创建和管理`CacheManagers`的生命周期。

### 4.4。`Configuration`

`Configuration`是一个使我们能够配置`Caches`的界面。它有一个具体的实现——`MutableConfiguration`和一个子接口——`CompleteConfiguration`。

## 5。创造一个`Cache`

让我们看看如何创建一个简单的`Cache`:

```java
CachingProvider cachingProvider = Caching.getCachingProvider();
CacheManager cacheManager = cachingProvider.getCacheManager();
MutableConfiguration<String, String> config
  = new MutableConfiguration<>();
Cache<String, String> cache = cacheManager
  .createCache("simpleCache", config);
cache.put("key1", "value1");
cache.put("key2", "value2");
cacheManager.close();
```

我们所做的就是:

*   创建一个`CachingProvider`对象，我们用它来构造一个`CacheManager`对象
*   创建一个`MutableConfiguration`对象，它是`Configuration`接口的一个实现
*   使用我们之前创建的`CacheManager`对象创建一个`Cache`对象
*   将所有条目放入，我们需要缓存到我们的`Cache`对象中
*   关闭`CacheManager`释放`Cache`使用的资源

**如果我们在`pom.xml`中没有提供 JCache 的任何实现，将会抛出以下异常:**

```java
javax.cache.CacheException: No CachingProviders have been configured
```

**这是因为 JVM 找不到`getCacheManager()`方法的任何具体实现。**

## 6。`EntryProcessor`

`EntryProcessor`允许我们使用原子操作修改`Cache`条目，而不必将它们重新添加到`Cache`。要使用它，我们需要实现`EntryProcessor`接口:

```java
public class SimpleEntryProcessor
  implements EntryProcessor<String, String, String>, Serializable {

    public String process(MutableEntry<String, String> entry, Object... args)
      throws EntryProcessorException {

        if (entry.exists()) {
            String current = entry.getValue();
            entry.setValue(current + " - modified");
            return current;
        }
        return null;
    }
}
```

现在，让我们使用我们的`EntryProcessor`实现:

```java
@Test
public void whenModifyValue_thenCorrect() {
    this.cache.invoke("key", new SimpleEntryProcessor());

    assertEquals("value - modified", cache.get("key"));
}
```

## 7 .**。事件监听器**

**事件监听器允许我们在触发`EventType`枚举中定义的任何事件类型时采取动作**，这些事件类型是:

*   `CREATED`
*   `UPDATED`
*   `REMOVED`
*   `EXPIRED`

首先，我们需要实现将要使用的事件的接口。

例如，如果我们想使用`CREATED`和`UPDATED`事件类型，那么我们应该实现接口`CacheEntryCreatedListener`和`CacheEntryUpdatedListener`。

让我们看一个例子:

```java
public class SimpleCacheEntryListener implements
  CacheEntryCreatedListener<String, String>,
  CacheEntryUpdatedListener<String, String>,
  Serializable {

    private boolean updated;
    private boolean created;

    // standard getters

    public void onUpdated(
      Iterable<CacheEntryEvent<? extends String,
      ? extends String>> events) throws CacheEntryListenerException {
        this.updated = true;
    }

    public void onCreated(
      Iterable<CacheEntryEvent<? extends String,
      ? extends String>> events) throws CacheEntryListenerException {
        this.created = true;
    }
}
```

现在，让我们运行我们的测试:

```java
@Test
public void whenRunEvent_thenCorrect() throws InterruptedException {
    this.listenerConfiguration
      = new MutableCacheEntryListenerConfiguration<String, String>(
        FactoryBuilder.factoryOf(this.listener), null, false, true);
    this.cache.registerCacheEntryListener(this.listenerConfiguration);

    assertEquals(false, this.listener.getCreated());

    this.cache.put("key", "value");

    assertEquals(true, this.listener.getCreated());
    assertEquals(false, this.listener.getUpdated());

    this.cache.put("key", "newValue");

    assertEquals(true, this.listener.getUpdated());
}
```

## 8。`CacheLoader`

**`CacheLoader`允许** **用户使用读通模式** **将缓存作为主数据存储，并从中读取数据**。

在现实场景中，我们可以让缓存从实际存储中读取数据。

让我们看一个例子。首先，我们应该实现`CacheLoader`接口:

```java
public class SimpleCacheLoader
  implements CacheLoader<Integer, String> {

    public String load(Integer key) throws CacheLoaderException {
        return "fromCache" + key;
    }

    public Map<Integer, String> loadAll(Iterable<? extends Integer> keys)
      throws CacheLoaderException {
        Map<Integer, String> data = new HashMap<>();
        for (int key : keys) {
            data.put(key, load(key));
        }
        return data;
    }
}
```

现在，让我们使用我们的`CacheLoader`实现:

```java
public class CacheLoaderIntegrationTest {

    private Cache<Integer, String> cache;

    @Before
    public void setup() {
        CachingProvider cachingProvider = Caching.getCachingProvider();
        CacheManager cacheManager = cachingProvider.getCacheManager();
        MutableConfiguration<Integer, String> config
          = new MutableConfiguration<>()
            .setReadThrough(true)
            .setCacheLoaderFactory(new FactoryBuilder.SingletonFactory<>(
              new SimpleCacheLoader()));
        this.cache = cacheManager.createCache("SimpleCache", config);
    }

    @Test
    public void whenReadingFromStorage_thenCorrect() {
        for (int i = 1; i < 4; i++) {
            String value = cache.get(i);

            assertEquals("fromCache" + i, value);
        }
    }
}
```

## 9。结论

在本教程中，我们已经了解了什么是 JCache，并在一些实际场景中探索了它的一些重要特性。

一如既往，本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221128113145/https://github.com/eugenp/tutorials/tree/master/libraries-data)