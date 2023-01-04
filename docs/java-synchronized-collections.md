# 同步 Java 集合简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-synchronized-collections>

## 1。概述

[集合框架](https://web.archive.org/web/20221121030850/https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)是 Java 的关键组件。它提供了大量的接口和实现，允许我们以简单的方式创建和操作不同类型的集合。

尽管使用普通的非同步集合总体上很简单，但在多线程环境中工作时(也称为并发编程)，这也可能成为一个令人望而生畏且容易出错的过程。

因此，Java 平台通过在`[Collections](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#synchronizedCollection(java.util.Collection))`类中实现不同的同步 `wrappers`为这个场景提供了强大的支持。

这些包装器使得通过几个静态工厂方法创建所提供集合的同步视图变得容易。

在本教程中，**我们将深入研究这些**静态同步包装器。此外，我们将强调同步收集和并发收集的区别**。**

## 2。`synchronizedCollection()`法

我们将在本文中介绍的第一个同步包装器是`synchronizedCollection()`方法。顾名思义，**它返回一个由指定的 [`Collection`](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)** 备份的线程安全集合。

现在，为了更清楚地理解如何使用这个方法，让我们创建一个基本的单元测试:

```java
Collection<Integer> syncCollection = Collections.synchronizedCollection(new ArrayList<>());
    Runnable listOperations = () -> {
        syncCollection.addAll(Arrays.asList(1, 2, 3, 4, 5, 6));
    };

    Thread thread1 = new Thread(listOperations);
    Thread thread2 = new Thread(listOperations);
    thread1.start();
    thread2.start();
    thread1.join();
    thread2.join();

    assertThat(syncCollection.size()).isEqualTo(12);
} 
```

如上所示，用这种方法创建所提供集合的同步视图非常简单。

为了演示该方法实际上返回了一个线程安全的集合，我们首先创建了两个线程。

之后，我们将一个 [`Runnable`](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runnable.html) 实例以 lambda 表达式的形式注入到它们的构造函数中。让我们记住`Runnable`是一个函数接口，所以我们可以用一个 lambda 表达式来代替它。

最后，我们只是检查每个线程有效地向同步集合添加了六个元素，所以它的最终大小是十二。

## 3。`synchronizedList()`法

同样，类似于`synchronizedCollection()`方法，我们可以使用`synchronizedList()`包装器创建一个同步的 [`List`](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html) 。

正如我们所料，**方法返回指定的`List` :** 的线程安全视图

```java
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
```

毫不奇怪，`synchronizedList()`方法的使用看起来与它的更高级对应方法`synchronizedCollection()`几乎相同。

因此，正如我们在前面的单元测试中所做的，一旦我们创建了一个同步的`List`，我们就可以产生几个线程。之后，我们将使用它们以线程安全的方式访问/操作目标`List`。

此外，如果我们想要迭代一个同步的集合并防止意外的结果，我们应该显式地提供我们自己的线程安全的循环实现。因此，我们可以使用一个`synchronized`块来实现:

```java
List<String> syncCollection = Collections.synchronizedList(Arrays.asList("a", "b", "c"));
List<String> uppercasedCollection = new ArrayList<>();

Runnable listOperations = () -> {
    synchronized (syncCollection) {
        syncCollection.forEach((e) -> {
            uppercasedCollection.add(e.toUpperCase());
        });
    }
}; 
```

在所有需要迭代同步集合的情况下，我们都应该实现这个习语。这是因为同步集合上的迭代是通过多次调用该集合来执行的。因此，它们需要作为单个原子操作来执行。

**块`synchronized`的使用保证了**操作的原子性。

## 4。`synchronizedMap()`法

`Collections`类实现了另一个简洁的同步包装器，叫做`synchronizedMap().`，我们可以用它轻松地创建一个同步的 [`Map`](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) 。

**该方法返回所提供的`Map`实现**的线程安全视图:

```java
Map<Integer, String> syncMap = Collections.synchronizedMap(new HashMap<>()); 
```

## 5。`synchronizedSortedMap()`法

还有一个对应的`synchronizedMap()`方法的实现。它叫做`synchronizedSortedMap()`，我们可以用它来创建一个同步的`[SortedMap](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/SortedMap.html)`实例:

```java
Map<Integer, String> syncSortedMap = Collections.synchronizedSortedMap(new TreeMap<>()); 
```

## 6。`synchronizedSet()`法

接下来，我们继续讨论`synchronizedSet()`方法。顾名思义，它允许我们以最小的代价创建同步的`[Sets](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html)`。

**包装器返回由指定的`Set`** 支持的线程安全集合:

```java
Set<Integer> syncSet = Collections.synchronizedSet(new HashSet<>()); 
```

## 7。`synchronizedSortedSet()`法

最后，我们在这里展示的最后一个同步包装器是 `synchronizedSortedSet()`。

类似于我们到目前为止已经讨论过的其他包装器实现，**方法返回给定的`[SortedSet](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/SortedSet.html)`** 的线程安全版本:

```java
SortedSet<Integer> syncSortedSet = Collections.synchronizedSortedSet(new TreeSet<>()); 
```

## 8。同步与并发收集

到目前为止，我们仔细研究了集合框架的同步包装器。

现在，让我们来关注一下**同步集合和并发集合**的区别，比如 [`ConcurrentHashMap`](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html) 和`[BlockingQueue](https://web.archive.org/web/20221121030850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/BlockingQueue.html)`的实现。

### 8.1。同步收藏

**同步的集合通过[intrinsi](https://web.archive.org/web/20221121030850/https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html)c 锁实现线程安全，整个集合被锁定**。内在锁定是通过包装集合的方法中的同步块实现的。

正如我们所料，同步收集确保了多线程环境中的数据一致性/完整性。但是，它们可能会带来性能损失，因为一次只有一个线程可以访问集合(也称为同步访问)。

关于如何使用`synchronized`方法和程序块的详细指南，请查看[我们关于该主题的文章](/web/20221121030850/https://www.baeldung.com/java-synchronized)。

### 8.2。并发收款

**并发收集(例如`ConcurrentHashMap),`通过将它们的数据分成段来实现线程安全**。例如，在一个`ConcurrentHashMap`中，不同的线程可以获取每个段上的锁，因此多个线程可以同时访问`Map`(也称为并发访问)。

由于并发线程访问的内在优势，并发集合**比同步集合**的性能高得多。

因此，选择使用哪种类型的线程安全集合取决于每个用例的需求，并且应该相应地进行评估。

## 9。结论

**在本文中，我们深入研究了在`Collections`类**中实现的同步包装器。

此外，我们强调了同步收集和并发收集之间的区别，并研究了它们实现线程安全的方法。

像往常一样，本文中展示的所有代码示例都可以在 [GitHub](https://web.archive.org/web/20221121030850/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections) 上获得。