# 番石榴缓存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-cache>

## 1。概述

在本教程中，我们将重点关注 **Guava 缓存**的实现，包括基本用法、驱逐策略、刷新缓存和一些有趣的批量操作。

最后，我们将讨论如何使用缓存能够发出的删除通知。

## 2。如何使用番石榴缓存

让我们从缓存大写形式的`String`实例的简单例子开始。

首先，我们将创建用于计算存储在缓存中的值的`CacheLoader,`。从这里开始，我们将使用方便的`CacheBuilder`按照给定的规范构建我们的缓存:

```java
@Test
public void whenCacheMiss_thenValueIsComputed() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder().build(loader);

    assertEquals(0, cache.size());
    assertEquals("HELLO", cache.getUnchecked("hello"));
    assertEquals(1, cache.size());
}
```

请注意，缓存中没有“hello”键的值，所以计算并缓存该值。

还要注意，我们使用的是`getUnchecked()`操作，如果值不存在，它会计算并加载到缓存中。

## 3。驱逐政策

每个缓存都需要在某个时候删除值。让我们讨论使用不同标准将值逐出缓存的机制。

### 3.1。按大小驱逐

我们可以使用`maximumSize()`来**限制我们缓存**的大小。如果缓存达到了限制，它会驱逐最旧的项目。

在下面的代码中，我们将把缓存大小限制为三条记录:

```java
@Test
public void whenCacheReachMaxSize_thenEviction() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };
    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder().maximumSize(3).build(loader);

    cache.getUnchecked("first");
    cache.getUnchecked("second");
    cache.getUnchecked("third");
    cache.getUnchecked("forth");
    assertEquals(3, cache.size());
    assertNull(cache.getIfPresent("first"));
    assertEquals("FORTH", cache.getIfPresent("forth"));
}
```

### 3.2。按重量驱逐

我们还可以使用自定义的权重函数来限制缓存大小(T2)。在下面的代码中，我们将使用`length`作为我们的自定义权重函数:

```java
@Test
public void whenCacheReachMaxWeight_thenEviction() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    Weigher<String, String> weighByLength;
    weighByLength = new Weigher<String, String>() {
        @Override
        public int weigh(String key, String value) {
            return value.length();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder()
      .maximumWeight(16)
      .weigher(weighByLength)
      .build(loader);

    cache.getUnchecked("first");
    cache.getUnchecked("second");
    cache.getUnchecked("third");
    cache.getUnchecked("last");
    assertEquals(3, cache.size());
    assertNull(cache.getIfPresent("first"));
    assertEquals("LAST", cache.getIfPresent("last"));
}
```

注意:缓存可能会删除多个记录，以便为新的大记录留出空间。

### 3.3。按时间驱逐

除了使用大小来驱逐旧记录，我们还可以使用时间。在下面的例子中，我们将定制我们的缓存来**删除空闲了 2ms** 的记录:

```java
@Test
public void whenEntryIdle_thenEviction()
  throws InterruptedException {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder()
      .expireAfterAccess(2,TimeUnit.MILLISECONDS)
      .build(loader);

    cache.getUnchecked("hello");
    assertEquals(1, cache.size());

    cache.getUnchecked("hello");
    Thread.sleep(300);

    cache.getUnchecked("test");
    assertEquals(1, cache.size());
    assertNull(cache.getIfPresent("hello"));
}
```

我们还可以根据记录的总生存时间来驱逐记录。在下面的示例中，缓存将在记录存储 2 毫秒后删除它们:

```java
@Test
public void whenEntryLiveTimeExpire_thenEviction()
  throws InterruptedException {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder()
      .expireAfterWrite(2,TimeUnit.MILLISECONDS)
      .build(loader);

    cache.getUnchecked("hello");
    assertEquals(1, cache.size());
    Thread.sleep(300);
    cache.getUnchecked("test");
    assertEquals(1, cache.size());
    assertNull(cache.getIfPresent("hello"));
}
```

## 4。弱键

接下来，我们将演示如何使我们的缓存键具有弱引用，允许垃圾收集器收集其他地方没有引用的缓存键。

默认情况下，缓存键和值都有强引用，但是我们可以通过使用`weakKeys()`让缓存使用弱引用来存储键:

```java
@Test
public void whenWeakKeyHasNoRef_thenRemoveFromCache() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder().weakKeys().build(loader);
}
```

## 5。软值

我们还可以允许垃圾收集器通过使用`softValues()`来收集我们的缓存值:

```java
@Test
public void whenSoftValue_thenRemoveFromCache() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder().softValues().build(loader);
}
```

注意:过多的软引用可能会影响系统性能，所以首选是使用`maximumSize()`。

## 6。处理`null`值

现在让我们看看如何处理缓存`null`值。默认情况下，如果我们试图加载一个`null`值，**番石榴缓存**将抛出异常，因为缓存一个`null`没有任何意义。

但是如果一个`null`值在我们的代码中意味着什么，那么我们可以很好地利用`Optional`类:

```java
@Test
public void whenNullValue_thenOptional() {
    CacheLoader<String, Optional<String>> loader;
    loader = new CacheLoader<String, Optional<String>>() {
        @Override
        public Optional<String> load(String key) {
            return Optional.fromNullable(getSuffix(key));
        }
    };

    LoadingCache<String, Optional<String>> cache;
    cache = CacheBuilder.newBuilder().build(loader);

    assertEquals("txt", cache.getUnchecked("text.txt").get());
    assertFalse(cache.getUnchecked("hello").isPresent());
}
private String getSuffix(final String str) {
    int lastIndex = str.lastIndexOf('.');
    if (lastIndex == -1) {
        return null;
    }
    return str.substring(lastIndex + 1);
}
```

## 7。刷新缓存

接下来，我们将了解如何刷新缓存值。

### 7.1.手动刷新

我们可以借助`LoadingCache.refresh(key):`手动刷新单个密钥

```java
String value = loadingCache.get("key");
loadingCache.refresh("key");
```

这将迫使`CacheLoader`为`key.`加载新值

直到新值被成功加载，**`**get(key)**.`将返回`key`的先前值**

 **### 7.2.自动刷新

我们还可以使用`CacheBuilder.refreshAfterWrite(duration)`来自动刷新缓存值:

```java
@Test
public void whenLiveTimeEnd_thenRefresh() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder()
      .refreshAfterWrite(1,TimeUnit.MINUTES)
      .build(loader);
}
```

重要的是要明白 **`refreshAfterWrite(duration)`只是在指定的持续时间**之后为刷新生成一个密钥`eligible`。只有当`get(key).`查询到相应的条目时，该值才会被刷新

## 8。预加载缓存

我们可以使用`putAll()`方法在缓存中插入多条记录。在下面的例子中，我们将使用一个`Map`将多个记录添加到我们的缓存中:

```java
@Test
public void whenPreloadCache_thenUsePutAll() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return key.toUpperCase();
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder().build(loader);

    Map<String, String> map = new HashMap<String, String>();
    map.put("first", "FIRST");
    map.put("second", "SECOND");
    cache.putAll(map);

    assertEquals(2, cache.size());
}
```

## 9。移除通知

有时我们需要在记录从缓存中移除时采取行动，所以我们将讨论`RemovalNotification`。

我们可以注册一个`RemovalListener`来获得记录被删除的通知。我们还可以通过`getCause()`方法了解移除的原因。

在下面的例子中，当高速缓存中的第四个元素由于其大小而被移除时，接收到一个`RemovalNotification`:

```java
@Test
public void whenEntryRemovedFromCache_thenNotify() {
    CacheLoader<String, String> loader;
    loader = new CacheLoader<String, String>() {
        @Override
        public String load(final String key) {
            return key.toUpperCase();
        }
    };

    RemovalListener<String, String> listener;
    listener = new RemovalListener<String, String>() {
        @Override
        public void onRemoval(RemovalNotification<String, String> n){
            if (n.wasEvicted()) {
                String cause = n.getCause().name();
                assertEquals(RemovalCause.SIZE.toString(),cause);
            }
        }
    };

    LoadingCache<String, String> cache;
    cache = CacheBuilder.newBuilder()
      .maximumSize(3)
      .removalListener(listener)
      .build(loader);

    cache.getUnchecked("first");
    cache.getUnchecked("second");
    cache.getUnchecked("third");
    cache.getUnchecked("last");
    assertEquals(3, cache.size());
}
```

## 10。注释

最后，这里有一些关于 Guava 缓存实现的附加快速注释:

*   这是线程安全的
*   我们可以使用`put(key,value)`手动将值插入缓存
*   我们可以使用`CacheStats` ( `hitRate()`，`missRate()`，..)

## 11。结论

在本文中，我们探索了许多番石榴缓存的用例。我们讨论的主题包括简单使用、驱逐元素、刷新和预加载缓存以及移除通知。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221024234817/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)**