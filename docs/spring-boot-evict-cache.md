# Spring Boot 的缓存回收

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-evict-cache>

## 1。概述

在这个快速教程中，我们将学习如何使用 Spring 执行缓存回收。为了演示这一点，我们将创建一个小示例。

在继续之前，查看我们的文章，[Spring 缓存指南](/web/20221205110008/https://www.baeldung.com/spring-cache-tutorial)，熟悉 Spring 缓存是如何工作的。

## 2。如何驱逐缓存？

Spring 提供了两种方法来驱逐缓存，要么在方法上使用`@CacheEvict`注释，要么自动连接`CacheManger`并通过调用`clear()`方法来清除它。

下面是我们如何在代码中实现这两种缓存回收机制。

### 2.1。使用`@CacheEvict`

让我们用`@CacheEvict`注释创建一个空方法，并提供我们想要清除的缓存名称作为注释的参数。在这种情况下，我们希望清除名为“first”的缓存:

```java
@CacheEvict(value = "first", allEntries = true)
public void evictAllCacheValues() {}
```

**Spring 会拦截所有用`@CacheEvict`标注的方法，并根据`allEntries`标志**清除所有的值。

也可以基于特定的键来驱逐值。为此，我们所要做的就是将缓存键作为参数传递给注释，而不是`allEntries`标志:

```java
@CacheEvict(value = "first", key = "#cacheKey")
public void evictSingleCacheValue(String cacheKey) {}
```

由于`key`属性的值是动态的，我们可以使用 Spring 表达式语言，或者通过实现`KeyGenerator`来选择感兴趣的参数或嵌套属性，从而使用自定义的键生成器。

### 2.2。使用`CacheManager`

接下来，让我们看看如何使用 Spring Cache 模块提供的`CacheManager`来驱逐缓存。首先，我们必须自动连接已实现的`CacheManager` bean。

然后，我们可以根据自己的需要，用它来清除缓存:

```java
@Autowired
CacheManager cacheManager;

public void evictSingleCacheValue(String cacheName, String cacheKey) {
    cacheManager.getCache(cacheName).evict(cacheKey);
}

public void evictAllCacheValues(String cacheName) {
    cacheManager.getCache(cacheName).clear();
}
```

正如我们在代码中看到的，**`clear()`方法将清除所有的缓存条目，`evict()`方法将清除基于键**的值。

## 3。如何驱逐所有缓存？

Spring 没有提供开箱即用的功能来清除所有的缓存，但是我们可以通过使用缓存管理器的`getCacheNames()`方法很容易地实现这一点。

### 3.1。按需驱逐

现在让我们看看如何按需清除所有缓存。为了创建触发点，我们必须首先公开一个端点:

```java
@RestController
public class CachingController {

    @Autowired
    CachingService cachingService;

    @GetMapping("clearAllCaches")
    public void clearAllCaches() {
        cachingService.evictAllCaches();
    }
}
```

在`CachingService`中，我们可以**通过迭代从缓存管理器**获得的缓存名称来清除所有缓存:

```java
public void evictAllCaches() {
    cacheManager.getCacheNames().stream()
      .forEach(cacheName -> cacheManager.getCache(cacheName).clear());
}
```

### 3.2。自动驱逐

在某些用例中，应该以特定的时间间隔自动执行缓存回收。在这种情况下，**我们可以利用 Spring 的任务调度器**:

```java
@Scheduled(fixedRate = 6000)
public void evictAllcachesAtIntervals() {
    evictAllCaches();
}
```

## 4。结论

在本文中，我们学习了如何以不同的方式驱逐缓存。关于这些机制值得注意的一点是，它将与所有各种缓存实现一起工作，如 eh-cache、infini-span、apache-ignite 等。

和往常一样，本文提到的所有例子都可以在 Github 上找到[。](https://web.archive.org/web/20221205110008/https://github.com/eugenp/tutorials/tree/master/spring-caching)