# 番石榴缓存加载器简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-cacheloader>

## 1。简介

在这篇文章中，我们将介绍番石榴

在进一步阅读之前，建议先对`[LoadingCache](https://web.archive.org/web/20220524023625/https://google.github.io/guava/releases/21.0/api/docs/com/google/common/cache/LoadingCache.html)`类有个基本的了解。这是因为`CacheLoader` 专门与它一起工作。

本质上，`CacheLoader`是一个函数，用于计算在番石榴`[LoadingCache](https://web.archive.org/web/20220524023625/https://google.github.io/guava/releases/21.0/api/docs/com/google/common/cache/LoadingCache.html).`中找不到的值

## 2。使用一个`CacheLoader`和一个`LoadingCache`

当使用`LoadingCache,` 出现缓存未命中或缓存需要刷新时，`CacheLoader` 将用于计算值。这有助于将我们的缓存逻辑封装在一个地方，使我们的代码更具凝聚力。

### 2.1。Maven 依赖关系

首先，让我们添加 Maven 依赖项:

```java
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>31.0.1-jre</version>
</dependency>
```

你可以在 [Maven 资源库](https://web.archive.org/web/20220524023625/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)中找到最新版本。

### 2.2。计算和缓存值

现在，让我们看看如何用一个`CacheLoader`实例化一个`LoadingCache` :

```java
LoadingCache<String, String> loadingCache = CacheBuilder.newBuilder()
  .build(new CacheLoader<String, String>() {
    @Override
    public String load(final String s) throws Exception {
      return slowMethod(s);
    }
});
```

本质上，`LoadingCache` 将调用我们的内联`CacheLoader` 来计算一个没有被缓存的值。让我们试着计算一下当我们多次从缓存中检索某个东西时，我们的`slowMethod()` 被调用了多少次:

```java
String value = loadingCache.get("key");
value = loadingCache.get("key");

assertThat(callCount).isEqualTo(1);
assertThat(value).isEqualTo("expectedValue"); 
```

我们可以看到，它只被调用了一次。第一次值没有被缓存，因为它还没有被计算。第二次，它是从上一次调用中缓存的，所以我们可以避免再次调用我们的`slowMethod()` 的开销。

### 2.3。刷新缓存

缓存的另一个常见问题是刷新缓存。尽管最困难的方面是知道`when`来刷新缓存，但另一个方面是知道`how.`

使用`CacheLoader.` 时，求解`how` 很简单。`LoadingCache` 将简单地为每个需要刷新的值调用它。让我们用一个测试来试试这个:

```java
String value = loadingCache.get("key");
loadingCache.refresh("key");

assertThat(callCount).isEqualTo(2);
assertThat(value).isEqualTo("key");
```

不像我们随后对`get(), refresh()` 的调用会强制再次调用`CacheLoader` ，确保我们的值是最新的。

## 3。结论

在本文中，我们已经解释了`CacheLoader` 如何使用`LoadingCache` 来计算缓存未命中以及缓存刷新的值。这篇关于[番石榴贮藏](/web/20220524023625/https://www.baeldung.com/guava-cache)的更深入的文章也值得一读。

这些例子的实现可以在 GitHub 的[中找到。这是一个 Maven 项目，所以应该很容易运行。](https://web.archive.org/web/20220524023625/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)