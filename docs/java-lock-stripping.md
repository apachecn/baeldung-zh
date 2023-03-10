# 锁条带化简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lock-stripping>

## 1.介绍

在本教程中，我们将学习如何实现细粒度同步，也称为锁条带化，这是一种在保持良好性能的同时处理数据结构并发访问的模式。

## 2.问题是

[`HashMap`](/web/20220626205531/https://www.baeldung.com/java-hashmap) 由于其非同步的本质，不是线程安全的数据结构。这意味着来自多线程环境的命令可能会导致数据不一致。

为了克服这个问题，我们可以用`Collections#synchronizedMap` 方法转换原始地图，或者使用`HashTable` 数据结构。两者都将返回`Map`接口的线程安全实现，但是它们是以牺牲性能为代价的。

用单个锁对象定义对数据结构的独占**访问的方法被称为`coarse-grained synchronization`** 。

在粗粒度同步实现中，对对象的每次访问都必须由一个线程一次完成。我们结束了顺序访问。

我们的目标是允许并发线程处理数据结构，同时确保线程安全。

## 3.锁定条带化

为了达到我们的目标，我们将使用锁分条模式。锁条带化是一种技术，其中锁定发生在几个桶或条带上，这意味着访问一个桶仅锁定该桶，而不是整个数据结构。

有几种方法可以做到这一点:

*   首先，我们可以为每个任务使用一个锁，从而最大化任务间的并发性——尽管这样会占用更多内存
*   或者，我们可以为每个任务使用一个锁，这样可以使用更少的内存，但也会影响并发性能

**为了帮助我们管理这种性能-内存权衡，Guava 附带了一个名为`Striped.`** 的类，它类似于`[ConcurrentHashMap](/web/20220626205531/https://www.baeldung.com/java-concurrent-map)`中的逻辑，但是`Striped`类更进一步，它使用信号量或重入锁来减少不同任务的同步。

## 4.一个简单的例子

让我们做一个简单的例子来帮助我们理解这种模式的好处。

我们将在四个实验中比较`HashMap `与`ConcurrentHashMap`以及单锁与条带锁。

对于每个实验，我们将在底层`Map`上执行并发读写。不同的是我们访问每个存储桶的方式。

为此，我们将创建两个类——`SingleLock`和`StripedLock.` ,它们是完成这项工作的抽象类`ConcurrentAccessExperiment`的具体实现。

### 4.1.属国

由于我们将使用 Guava 的`Striped` 类，我们将添加`guava`依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

### 4.2.主流程

我们的`ConcurrentAccessExperiment`类实现了前面描述的行为:

```java
public abstract class ConcurrentAccessExperiment {

    public final Map<String,String> doWork(Map<String,String> map, int threads, int slots) {
        CompletableFuture<?>[] requests = new CompletableFuture<?>[threads * slots];

        for (int i = 0; i < threads; i++) {
            requests[slots * i + 0] = CompletableFuture.supplyAsync(putSupplier(map, i));
            requests[slots * i + 1] = CompletableFuture.supplyAsync(getSupplier(map, i));
            requests[slots * i + 2] = CompletableFuture.supplyAsync(getSupplier(map, i));
            requests[slots * i + 3] = CompletableFuture.supplyAsync(getSupplier(map, i));
        }
        CompletableFuture.allOf(requests).join();

        return map;
    }

    protected abstract Supplier<?> putSupplier(Map<String,String> map, int key);
    protected abstract Supplier<?> getSupplier(Map<String,String> map, int key);
}
```

值得注意的是，由于我们的测试是受 CPU 限制的，我们将存储桶的数量限制为可用处理器的倍数。

### 4.3.与`ReentrantLock`同时访问

现在我们将实现异步任务的方法。

我们的`SingleLock`类使用`ReentrantLock`为整个数据结构定义了一个锁:

```java
public class SingleLock extends ConcurrentAccessExperiment {
    ReentrantLock lock;

    public SingleLock() {
        lock = new ReentrantLock();
    }

    protected Supplier<?> putSupplier(Map<String,String> map, int key) {
        return (()-> {
            lock.lock();
            try {
                return map.put("key" + key, "value" + key);
            } finally {
                lock.unlock();
            }
        });
    }

    protected Supplier<?> getSupplier(Map<String,String> map, int key) {
        return (()-> {
            lock.lock();
            try {
                return map.get("key" + key);
            } finally {
                lock.unlock();
            }
        });
    }
}
```

### 4.4.与`Striped`同时访问

然后，`StripedLock`类为每个桶定义了一个条带锁:

```java
public class StripedLock extends ConcurrentAccessExperiment {
    Striped lock;

    public StripedLock(int buckets) {
        lock = Striped.lock(buckets);
    }

    protected Supplier<?> putSupplier(Map<String,String> map, int key) {
        return (()-> {
            int bucket = key % stripedLock.size();
            Lock lock = stripedLock.get(bucket);
            lock.lock();
            try {
                return map.put("key" + key, "value" + key);
            } finally {
                lock.unlock();
            }
        });
    }

    protected Supplier<?> getSupplier(Map<String,String> map, int key) {
        return (()-> {
            int bucket = key % stripedLock.size();
            Lock lock = stripedLock.get(bucket);
            lock.lock(); 
            try {
                return map.get("key" + key);
            } finally {
                lock.unlock();
            }
        });
    }
}
```

那么，哪种策略表现更好呢？

## 5.结果

让我们用[JMH](/web/20220626205531/https://www.baeldung.com/java-microbenchmark-harness)(Java 微基准测试工具)来找出答案。可以通过教程末尾的源代码链接找到这些基准。

运行我们的基准测试，我们能够看到类似下面的内容(注意，吞吐量越高越好):

```java
Benchmark                                                Mode  Cnt  Score   Error   Units
ConcurrentAccessBenchmark.singleLockConcurrentHashMap   thrpt   10  0,059 ± 0,006  ops/ms
ConcurrentAccessBenchmark.singleLockHashMap             thrpt   10  0,061 ± 0,005  ops/ms
ConcurrentAccessBenchmark.stripedLockConcurrentHashMap  thrpt   10  0,065 ± 0,009  ops/ms
ConcurrentAccessBenchmark.stripedLockHashMap            thrpt   10  0,068 ± 0,008  ops/ms 
```

## 6.结论

在本教程中，我们探索了在类似于`Map`的结构中使用锁分条来获得更好性能的不同方法。我们创建了一个基准来比较几个实现的结果。

从我们的基准测试结果中，我们可以了解不同的并发策略如何显著影响整个过程。条带锁模式获得了相当大的改进，因为它在使用`HashMap`和`ConcurrentHashMap`时获得了大约 10%的额外分数。

像往常一样，本教程的源代码可以在 [GitHub](https://web.archive.org/web/20220626205531/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections-2) 上获得。