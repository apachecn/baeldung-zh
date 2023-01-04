# 使用并发哈希表读写

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/concurrenthashmap-reading-and-writing>

## 1.介绍

在本教程中，我们将学习如何使用`ConcurrentHashMap`类以线程安全的方式读写哈希表数据结构。

## 2.概观

一个`ConcurrentHashMap`是 [`ConcurrentMap` 接口](/web/20221221184035/https://www.baeldung.com/java-concurrent-map)的一个实现，它是 Java 提供的线程安全集合之一。它以常规地图为后盾，工作方式类似于 [`Hashtable`](/web/20221221184035/https://www.baeldung.com/java-hash-table) ，有一些细微差别我们将在下面的章节中涉及。

### 2.2.有用的方法

在本教程中，`ConcurrentHashMap` API 规范提供了使用集合`.` 的实用方法，我们将主要关注其中的两个:

*   `get(K key)`:在给定的`key`检索元素。这是我们的阅读方法。
*   `computeIfPresent(K key, BiFunction<K, V, V> remappingFunction)`:如果`key `存在，将`remappingFunction`应用于给定`key`的值。

我们将在第 3 节中看到这些方法的实践。

### 2.2.为什么使用`ConcurrentHashMap`

`ConcurrentHashMap`和普通`HashMap`的主要区别在于，前者实现了读的完全并发和写的高并发。

**保证读操作不被阻塞或阻塞一个键。写操作被阻止，并在映射`Entry`级别阻止其他写操作。**在我们希望实现高吞吐量和最终一致性的环境中，这两个想法非常重要。

`HashTable` 和`Collections.synchronizedMap`集合还实现了读写的并发性。然而，它们的效率较低，因为它们锁定了整个集合，而不是锁定线程正在写入的`Entry` 。

另一方面，**`ConcurrentHashMap`级锁定在地图*入口*级别**。因此，其他线程不会被阻止写入其他映射键。因此，为了实现高吞吐量，与`HashTable` 和`synchronizedMap` 集合相比，多线程环境中的`ConcurrentHashMap`是更好的选择。

## 3.线程安全操作

实现了代码被认为是线程安全的大部分保证。这有助于避免 Java 中一些常见的并发陷阱。

为了说明`ConcurrentHashMap`如何在多线程环境中工作，我们将使用一个 Java 测试来检索和更新给定数字的频率。让我们首先定义测试的基本结构:

```
public class ConcurrentHashMapUnitTest {

    private Map<Integer, Integer> frequencyMap;

    @BeforeEach
    public void setup() {
        frequencyMap = new ConcurrentHashMap<>();
        frequencyMap.put(0, 0);
        frequencyMap.put(1, 0);
        frequencyMap.put(2, 0);
    }

    @AfterEach
    public void teardown() {
        frequencyMap.clear();
    }

    private static void sleep(int timeout) {
        try {
            TimeUnit.SECONDS.sleep(timeout);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

上面的类定义了数字的频率映射，一个用初始值填充它的`setup`方法，一个清除它的内容的`teardown`方法，以及一个处理`InterruptedException`的助手方法`sleep`。

### 3.1.阅读

`ConcurrentHashMap`允许完全并发读取，这意味着**任何给定数量的线程可以同时读取同一个键**。这也意味着读操作不会阻塞，也不会被写操作阻塞。因此，从地图中读取可能会得到“旧的”或不一致的值。

让我们来看一个例子，一个线程写入一个键，第二个线程在写入完成前读取，第三个线程在写入完成后读取:

```
@Test
public void givenOneThreadIsWriting_whenAnotherThreadReads_thenGetCorrectValue() throws Exception {
    ExecutorService threadExecutor = Executors.newFixedThreadPool(3);

    Runnable writeAfter1Sec = () -> frequencyMap.computeIfPresent(1, (k, v) -> {
        sleep(1);
        return frequencyMap.get(k) + 1;
    });

    Callable<Integer> readNow = () -> frequencyMap.get(1);
    Callable<Integer> readAfter1001Ms = () -> {
        TimeUnit.MILLISECONDS.sleep(1001);
        return frequencyMap.get(1);
    };

    threadExecutor.submit(writeAfter1Sec);
    List<Future<Integer>> results = threadExecutor.invokeAll(asList(readNow, readAfter1001Ms));

    assertEquals(0, results.get(0).get());
    assertEquals(1, results.get(1).get());

    if (threadExecutor.awaitTermination(2, TimeUnit.SECONDS)) {
        threadExecutor.shutdown();
    }
}
```

让我们仔细看看上面的代码中发生了什么:

1.  我们首先用一个写线程和两个读线程定义一个`ExecutorService `。写操作需要一秒钟完成。因此，在此之前的任何读取都应该得到旧的结果。之后的任何读取(在本例中，精确地说是一毫秒之后)都应该得到更新后的值。
2.  然后，我们使用`invokeAll `调用所有的读线程，并将结果收集到一个列表中。因此，列表的位置 0 指的是第一次读取，位置 1 指的是第二次读取。
3.  最后，我们使用`assertEquals`验证已完成任务的结果，并关闭`ExecutorService`。

从这段代码中，我们得出结论，即使其他线程同时对同一资源进行写操作，读操作也不会被阻塞。如果我们把读和写想象成事务，**`ConcurrentHashMap`实现了读**的最终一致性。这意味着我们不会总是读取一致的值(最新的值)，但是一旦映射停止接收写入，读取就会再次变得一致。查看这个[对交易](/web/20221221184035/https://www.baeldung.com/cs/transactions-intro)的介绍，以获得关于最终一致性的更多细节。

提示:如果您还想让读操作阻塞并被其他读操作阻塞，请不要使用`get()`方法。相反，您可以实现一个 identity `BiFunction` 来返回给定键上未修改的值，并将该函数传递给`computeIfPresent`方法。使用它，我们将牺牲读取速度，以防止读取旧的或不一致的值的问题。

### 3.2.写作

如前所述，`ConcurrentHashMap`实现了写操作的部分并发，这阻止了相同映射键上的其他写操作，并允许对不同键的写操作。**这对于在多线程环境中实现高吞吐量和一致的写入至关重要。**为了说明一致性，让我们定义一个测试，让两个线程在相同的资源上写，并检查 map 如何处理:

```
@Test
public void givenOneThreadIsWriting_whenAnotherThreadWritesAtSameKey_thenWaitAndGetCorrectValue() throws Exception {
    ExecutorService threadExecutor = Executors.newFixedThreadPool(2);

    Callable<Integer> writeAfter5Sec = () -> frequencyMap.computeIfPresent(1, (k, v) -> {
        sleep(5);
        return frequencyMap.get(k) + 1;
    });

    Callable<Integer> writeAfter1Sec = () -> frequencyMap.computeIfPresent(1, (k, v) -> {
        sleep(1);
        return frequencyMap.get(k) + 1;
    });

    List<Future<Integer>> results = threadExecutor.invokeAll(asList(writeAfter5Sec, writeAfter1Sec));

    assertEquals(1, results.get(0).get());
    assertEquals(2, results.get(1).get());

    if (threadExecutor.awaitTermination(2, TimeUnit.SECONDS)) {
        threadExecutor.shutdown();
    }
}
```

上面的测试显示两个写线程被提交到`ExecutorService. `中，第一个线程花了 5 秒来写，第二个线程花了 1 秒来写。第一个线程获得锁，并阻塞 map key 1 处的任何其他写活动。因此，第二个线程必须等待五秒钟，直到第一个线程释放锁。第一次写入完成后，第二个线程获取最新的值，并在一秒钟内更新它。

来自`ExecutorService`的结果列表按照任务提交的顺序出现，所以第一个元素应该返回 1，第二个元素应该返回 2。

`ConcurrentHashMap `的另一个用例是在不同的映射键中实现高吞吐量的写入。让我们用另一个使用两个写线程来更新映射中不同键的单元测试来说明这一点:

```
@Test
public void givenOneThreadIsWriting_whenAnotherThreadWritesAtDifferentKey_thenNotWaitAndGetCorrectValue() throws Exception {
    ExecutorService threadExecutor = Executors.newFixedThreadPool(2);

    Callable<Integer> writeAfter5Sec = () -> frequencyMap.computeIfPresent(1, (k, v) -> {
        sleep(5);
        return frequencyMap.get(k) + 1;
    });

    AtomicLong time = new AtomicLong(System.currentTimeMillis());
    Callable<Integer> writeAfter1Sec = () -> frequencyMap.computeIfPresent(2, (k, v) -> {
        sleep(1);
        time.set((System.currentTimeMillis() - time.get()) / 1000);
        return frequencyMap.get(k) + 1;
    });

    threadExecutor.invokeAll(asList(writeAfter5Sec, writeAfter1Sec));

    assertEquals(1, time.get());

    if (threadExecutor.awaitTermination(2, TimeUnit.SECONDS)) {
        threadExecutor.shutdown();
    }
}
```

该测试验证了第二个线程不需要等待第一个线程完成，因为写操作发生在不同的映射键上。因此，第二次写入只需一秒钟即可完成。**在`ConcurrentHashMap,` 中，线程可以在不同的映射条目中同时工作，与其他线程安全结构**相比，并发写操作更快。

## 4.结论

在本文中，我们已经看到了如何对一个`ConcurrentHashMap` 进行读写，以实现读写的高吞吐量和最终的读一致性。和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221221184035/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections-2)