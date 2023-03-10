# 在 Java 中使用番石榴进行 Bloom 过滤

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-bloom-filter>

## 1。概述

在本文中，我们将会看到来自`Guava`库的[布鲁姆过滤器](/web/20220628235924/https://www.baeldung.com/cs/bloom-filter)构造。一个[布鲁姆过滤器](/web/20220628235924/https://www.baeldung.com/cs/bloom-filter)是一个内存高效的概率数据结构，我们可以用它来回答**的问题，一个给定的元素是否在集合**中。

**布隆过滤器没有假阴性**，所以当它返回`false`时，我们可以 100%确定该元素不在集合中。

然而，Bloom filter **可能会返回误报**，因此当它返回`true`时，该元素很有可能在集合中，但我们不能 100%确定。

为了更深入地分析一个 Bloom filter 是如何工作的，你可以浏览这个教程。

## 2。Maven 依赖关系

我们将使用 Bloom filter 的 Guava 实现，所以让我们添加`guava`依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220628235924/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 上找到。

## 3。为什么要用布鲁姆滤镜？

布隆过滤器**被设计成节省空间且快速**。当使用它时，我们可以**指定我们可以接受的假阳性响应的概率**，根据该配置，布隆过滤器将尽可能少地占用内存。

由于这种空间效率，**Bloom filter 将很容易适应内存**,即使对于大量的元素。一些数据库，包括 Cassandra 和 Oracle，在转到磁盘或缓存之前使用这个过滤器作为第一个检查，例如，当一个特定 ID 的请求进来时。

如果过滤器返回 ID 不存在，数据库可以停止进一步处理请求并返回给客户端。否则，它将转到磁盘，如果在磁盘上找到该元素，则返回该元素。

## 4。创建布隆过滤器

假设我们想要为多达 500 个`Integers`创建一个布隆过滤器，并且我们可以容忍百分之一(0.01)的误报概率。

我们可以使用`Guava` 库中的`BloomFilter` 类来实现这一点。我们需要传递我们期望插入到过滤器中的元素数量和期望的误报概率:

```java
BloomFilter<Integer> filter = BloomFilter.create(
  Funnels.integerFunnel(),
  500,
  0.01);
```

现在让我们给过滤器添加一些数字:

```java
filter.put(1);
filter.put(2);
filter.put(3);
```

我们只添加了三个元素，并且我们定义了插入的最大数量是 500，所以我们的 Bloom filter **应该产生非常精确的结果**。让我们使用`mightContain()`方法来测试它:

```java
assertThat(filter.mightContain(1)).isTrue();
assertThat(filter.mightContain(2)).isTrue();
assertThat(filter.mightContain(3)).isTrue();

assertThat(filter.mightContain(100)).isFalse();
```

顾名思义，当方法返回`true`时，我们不能 100%确定给定的元素确实在过滤器中。

在我们的示例中，当`mightContain()`返回`true`时，我们可以 99%确定元素在过滤器中，并且有百分之一的概率结果是误报。当过滤器返回`false`时，我们可以 100%确定该元素不存在。

## 5。过饱和布隆过滤器

当我们设计布隆过滤器时，**重要的是，我们要为元素的预期数量**提供一个合理准确的值。否则，我们的过滤器将以比预期高得多的比率返回误报。让我们看一个例子。

假设我们创建了一个过滤器，其所需的误报概率为百分之一，预期的一些元素等于五，但是我们插入了 100，000 个元素:

```java
BloomFilter<Integer> filter = BloomFilter.create(
  Funnels.integerFunnel(),
  5,
  0.01);

IntStream.range(0, 100_000).forEach(filter::put); 
```

因为预期的元素数量很少，所以过滤器将占用很少的内存。

然而，随着我们添加的项目比预期的多，**过滤器变得过饱和，返回假阳性结果**的概率比预期的 1%高得多:

```java
assertThat(filter.mightContain(1)).isTrue();
assertThat(filter.mightContain(2)).isTrue();
assertThat(filter.mightContain(3)).isTrue();
assertThat(filter.mightContain(1_000_000)).isTrue();
```

注意，`mightContatin()` 方法**返回了`true`，即使是我们之前没有在过滤器中插入**的值。

## 6。结论

在这个快速教程中，我们研究了 Bloom filter 数据结构的概率性质——利用了`Guava`实现。

你可以在 [GitHub 项目](https://web.archive.org/web/20220628235924/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)中找到所有这些例子和代码片段的实现。

这是一个 Maven 项目，因此应该很容易导入和运行。