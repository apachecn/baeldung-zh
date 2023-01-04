# 并发地图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrent-map>

## 1。概述

`Maps`自然是 Java 收藏最广泛的风格之一。

重要的是， [`HashMap`](/web/20220626084203/https://www.baeldung.com/java-hashmap) 不是线程安全的实现，而`Hashtable`通过同步操作来提供线程安全。

即使`Hashtable`是线程安全的，它也不是很高效。另一个完全同步的`Map,` `Collections.synchronizedMap,` 也没有表现出很大的效率。如果我们想要在高并发下具有高吞吐量的线程安全，这些实现不是我们要走的路。

为了解决这个问题，`Java Collections Framework` **在`Java 1.5`中引入了`ConcurrentMap`。**

以下讨论基于`Java 1.8`。

## 2。`ConcurrentMap`

`ConcurrentMap`是`Map`接口的扩展。它旨在提供一种结构和指导来解决协调吞吐量和线程安全性的问题。

通过覆盖几个接口默认方法，`ConcurrentMap`给出了有效实现的指导方针，以提供线程安全和内存一致的原子操作。

几个默认实现被覆盖，禁用了`null`键/值支持:

*   `getOrDefault`
*   `forEach`
*   `replaceAll`
*   `computeIfAbsent`
*   `computeIfPresent`
*   `compute`
*   `merge`

下面的`APIs`也被覆盖以支持原子性，没有默认的接口实现:

*   `putIfAbsent`
*   `remove`
*   `replace(key, oldValue, newValue)`
*   `replace(key, value)`

其余动作直接继承，与`Map`基本一致。

## 3。`ConcurrentHashMap`

`ConcurrentHashMap`是现成的`ConcurrentMap`实施。

为了获得更好的性能，它由一个作为表桶的节点数组组成(在`Java 8`之前是表段)，在更新期间主要使用 [CAS](https://web.archive.org/web/20220626084203/https://en.wikipedia.org/wiki/Compare-and-swap) 操作。

在第一次插入时，表桶被延迟初始化。通过锁定存储桶中的第一个节点，可以单独锁定每个存储桶。读取操作不会阻塞，并且更新争用被最小化。

所需的段数与访问表的线程数有关，因此每个段中正在进行的更新在大多数情况下不会超过一次。

**在`Java 8`之前，所需的“段”数量与访问表的线程数量相关，因此每个段中正在进行的更新在大多数情况下不超过一次。**

这就是为什么与`HashMap`相比，构造函数提供了额外的`concurrencyLevel`参数来控制估计要使用的线程数量:

```
public ConcurrentHashMap(
```

```
public ConcurrentHashMap(
 int initialCapacity, float loadFactor, int concurrencyLevel)
```

另外两个参数:`initialCapacity`和`loadFactor`的工作原理与`HashMap` 的[完全相同。](/web/20220626084203/https://www.baeldung.com/java-hashmap)

**然而，从`Java 8`开始，构造函数只是为了向后兼容而存在:参数只能影响地图**的初始大小。

### 3.1。线程安全

`ConcurrentMap`保证多线程环境中键/值操作的内存一致性。

将对象作为键或值放入`ConcurrentMap`之前线程中的动作`happen-before`在另一个线程中访问或移除该对象之后的动作。

为了证实，我们来看一个记忆不一致的案例:

```
@Test
public void givenHashMap_whenSumParallel_thenError() throws Exception {
    Map<String, Integer> map = new HashMap<>();
    List<Integer> sumList = parallelSum100(map, 100);

    assertNotEquals(1, sumList
      .stream()
      .distinct()
      .count());
    long wrongResultCount = sumList
      .stream()
      .filter(num -> num != 100)
      .count();

    assertTrue(wrongResultCount > 0);
}

private List<Integer> parallelSum100(Map<String, Integer> map, 
  int executionTimes) throws InterruptedException {
    List<Integer> sumList = new ArrayList<>(1000);
    for (int i = 0; i < executionTimes; i++) {
        map.put("test", 0);
        ExecutorService executorService = 
          Executors.newFixedThreadPool(4);
        for (int j = 0; j < 10; j++) {
            executorService.execute(() -> {
                for (int k = 0; k < 10; k++)
                    map.computeIfPresent(
                      "test", 
                      (key, value) -> value + 1
                    );
            });
        }
        executorService.shutdown();
        executorService.awaitTermination(5, TimeUnit.SECONDS);
        sumList.add(map.get("test"));
    }
    return sumList;
}
```

对于并行的每个`map.computeIfPresent`动作，`HashMap`不提供当前整数值的一致视图，导致不一致和不期望的结果。

至于`ConcurrentHashMap`，我们可以得到一致正确的结果:

```
@Test
public void givenConcurrentMap_whenSumParallel_thenCorrect() 
  throws Exception {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    List<Integer> sumList = parallelSum100(map, 1000);

    assertEquals(1, sumList
      .stream()
      .distinct()
      .count());
    long wrongResultCount = sumList
      .stream()
      .filter(num -> num != 100)
      .count();

    assertEquals(0, wrongResultCount);
}
```

### 3.2。`Null`键/值

`ConcurrentMap`提供的大多数`API`不允许`null`键或值，例如:

```
@Test(expected = NullPointerException.class)
public void givenConcurrentHashMap_whenPutWithNullKey_thenThrowsNPE() {
    concurrentMap.put(null, new Object());
}

@Test(expected = NullPointerException.class)
public void givenConcurrentHashMap_whenPutNullValue_thenThrowsNPE() {
    concurrentMap.put("test", null);
}
```

然而，**对于`compute*`和`merge`动作，计算出的值可以是`null`，这表示键-值映射如果存在则被移除，或者如果先前不存在则保持不存在**。

```
@Test
public void givenKeyPresent_whenComputeRemappingNull_thenMappingRemoved() {
    Object oldValue = new Object();
    concurrentMap.put("test", oldValue);
    concurrentMap.compute("test", (s, o) -> null);

    assertNull(concurrentMap.get("test"));
}
```

### 3.3。流支持

`Java 8`也在`ConcurrentHashMap`中提供`Stream`支持。

与大多数流方法不同，批量(顺序和并行)操作允许安全地进行并发修改。`ConcurrentModificationException`不会被抛出，这也适用于它的迭代器。与流相关，还添加了几个`forEach*`、`search`和`reduce*`方法，以支持更丰富的遍历和 map-reduce 操作。

### 3.4。性能

**本质上，`ConcurrentHashMap`有点类似于`HashMap`** ，基于散列表进行数据访问和更新(尽管更复杂)。

当然，`ConcurrentHashMap`应该在大多数并发情况下为数据检索和更新提供更好的性能。

让我们为`get`和`put`的性能编写一个快速微基准，并将其与`Hashtable`和`Collections.synchronizedMap`进行比较，在 4 个线程中运行两个操作 500，000 次。

```
@Test
public void givenMaps_whenGetPut500KTimes_thenConcurrentMapFaster() 
  throws Exception {
    Map<String, Object> hashtable = new Hashtable<>();
    Map<String, Object> synchronizedHashMap = 
      Collections.synchronizedMap(new HashMap<>());
    Map<String, Object> concurrentHashMap = new ConcurrentHashMap<>();

    long hashtableAvgRuntime = timeElapseForGetPut(hashtable);
    long syncHashMapAvgRuntime = 
      timeElapseForGetPut(synchronizedHashMap);
    long concurrentHashMapAvgRuntime = 
      timeElapseForGetPut(concurrentHashMap);

    assertTrue(hashtableAvgRuntime > concurrentHashMapAvgRuntime);
    assertTrue(syncHashMapAvgRuntime > concurrentHashMapAvgRuntime);
}

private long timeElapseForGetPut(Map<String, Object> map) 
  throws InterruptedException {
    ExecutorService executorService = 
      Executors.newFixedThreadPool(4);
    long startTime = System.nanoTime();
    for (int i = 0; i < 4; i++) {
        executorService.execute(() -> {
            for (int j = 0; j < 500_000; j++) {
                int value = ThreadLocalRandom
                  .current()
                  .nextInt(10000);
                String key = String.valueOf(value);
                map.put(key, value);
                map.get(key);
            }
        });
    }
    executorService.shutdown();
    executorService.awaitTermination(1, TimeUnit.MINUTES);
    return (System.nanoTime() - startTime) / 500_000;
}
```

请记住，微基准测试只关注单一场景，并不总是真实世界性能的良好反映。

也就是说，在具有平均开发系统的 OS X 系统上，我们看到了连续运行 100 次的平均样本结果(以纳秒为单位):

```
Hashtable: 1142.45
SynchronizedHashMap: 1273.89
ConcurrentHashMap: 230.2
```

在多线程环境中，预期多个线程访问一个公共的`Map`，显然`ConcurrentHashMap`更好。

然而，当`Map`只能被单线程访问时，`HashMap`可能是更好的选择，因为它简单且性能可靠。

### 3.5。陷阱

检索操作通常不会阻塞在`ConcurrentHashMap`中，可能会与更新操作重叠。因此为了获得更好的性能，它们只反映最近完成的更新操作的结果，正如在[官方 Javadoc](https://web.archive.org/web/20220626084203/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html) 中所述。

还有其他几个事实需要记住:

*   包括`size`、`isEmpty`和`containsValue`在内的聚集状态方法的结果通常仅在地图没有在其他线程中进行并发更新时有用:

```
@Test
public void givenConcurrentMap_whenUpdatingAndGetSize_thenError() 
  throws InterruptedException {
    Runnable collectMapSizes = () -> {
        for (int i = 0; i < MAX_SIZE; i++) {
            mapSizes.add(concurrentMap.size());
        }
    };
    Runnable updateMapData = () -> {
        for (int i = 0; i < MAX_SIZE; i++) {
            concurrentMap.put(String.valueOf(i), i);
        }
    };
    executorService.execute(updateMapData);
    executorService.execute(collectMapSizes);
    executorService.shutdown();
    executorService.awaitTermination(1, TimeUnit.MINUTES);

    assertNotEquals(MAX_SIZE, mapSizes.get(MAX_SIZE - 1).intValue());
    assertEquals(MAX_SIZE, concurrentMap.size());
}
```

如果并发更新受到严格控制，聚集状态仍然是可靠的。

虽然这些**汇总状态方法不能保证实时准确性，但它们可能足以用于监控或评估目的**。

注意，`ConcurrentHashMap` 的`size()`的用法应该用`mappingCount()`代替，因为后一种方法返回一个`long`计数，尽管本质上它们是基于相同的估计。

*   **`hashCode`重要**:注意使用许多具有完全相同`hashCode()`的键肯定会降低任何哈希表的性能。

为了改善键为`Comparable`时的影响，`ConcurrentHashMap`可以使用键之间的比较顺序来帮助打破平局。尽管如此，我们应该尽可能避免使用同一个`hashCode()`。

*   迭代器只被设计用于单线程，因为它们提供弱一致性而不是快速失败遍历，并且它们永远不会抛出`ConcurrentModificationException.`
*   默认的初始表容量是 16，并根据指定的并发级别进行调整:

```
public ConcurrentHashMap(
  int initialCapacity, float loadFactor, int concurrencyLevel) {

    //...
    if (initialCapacity < concurrencyLevel) {
        initialCapacity = concurrencyLevel;
    }
    //...
}
```

*   关于重映射函数的警告:尽管我们可以使用提供的`compute`和`merge*` 方法进行重映射操作，但是我们应该保持它们快速、简短和简单，并且关注当前映射以避免意外阻塞。
*   `ConcurrentHashMap`中的键没有排序，所以对于需要排序的情况，`ConcurrentSkipListMap` 是一个合适的选择。

## 4。`ConcurrentNavigableMap`

如果需要对键进行排序，我们可以使用`ConcurrentSkipListMap`，它是`TreeMap`的并发版本。

作为对`ConcurrentMap`的补充，`ConcurrentNavigableMap`支持其键的全排序(默认为升序),并且可以并发导航。为了并发兼容性，返回地图视图的方法被重写:

*   `subMap`
*   `headMap`
*   `tailMap`
*   `subMap`
*   `headMap`
*   `tailMap`
*   `descendingMap`

视图的迭代器和拆分器增强了弱内存一致性:

*   `navigableKeySet`
*   `keySet`
*   `descendingKeySet`

## 5。`ConcurrentSkipListMap`

之前，我们已经介绍过`NavigableMap`接口及其实现 [`TreeMap`](/web/20220626084203/https://www.baeldung.com/java-treemap) 。`ConcurrentSkipListMap`可以看作是`TreeMap`的一个可扩展的并发版本。

实际上，Java 中没有红黑树的并发实现。在`ConcurrentSkipListMap`中实现了 [`SkipLists`](https://web.archive.org/web/20220626084203/https://en.wikipedia.org/wiki/Skip_list) 的并发变体，为`containsKey`、`get`、`put`和`remove`操作及其变体提供了预期的平均 log(n)时间成本。

除了`TreeMap`的特性，密钥插入、移除、更新和访问操作都有线程安全保证。以下是并发导航时与`TreeMap` 的比较:

```
@Test
public void givenSkipListMap_whenNavConcurrently_thenCountCorrect() 
  throws InterruptedException {
    NavigableMap<Integer, Integer> skipListMap
      = new ConcurrentSkipListMap<>();
    int count = countMapElementByPollingFirstEntry(skipListMap, 10000, 4);

    assertEquals(10000 * 4, count);
}

@Test
public void givenTreeMap_whenNavConcurrently_thenCountError() 
  throws InterruptedException {
    NavigableMap<Integer, Integer> treeMap = new TreeMap<>();
    int count = countMapElementByPollingFirstEntry(treeMap, 10000, 4);

    assertNotEquals(10000 * 4, count);
}

private int countMapElementByPollingFirstEntry(
  NavigableMap<Integer, Integer> navigableMap, 
  int elementCount, 
  int concurrencyLevel) throws InterruptedException {

    for (int i = 0; i < elementCount * concurrencyLevel; i++) {
        navigableMap.put(i, i);
    }

    AtomicInteger counter = new AtomicInteger(0);
    ExecutorService executorService
      = Executors.newFixedThreadPool(concurrencyLevel);
    for (int j = 0; j < concurrencyLevel; j++) {
        executorService.execute(() -> {
            for (int i = 0; i < elementCount; i++) {
                if (navigableMap.pollFirstEntry() != null) {
                    counter.incrementAndGet();
                }
            }
        });
    }
    executorService.shutdown();
    executorService.awaitTermination(1, TimeUnit.MINUTES);
    return counter.get();
}
```

对幕后性能问题的全面解释超出了本文的范围。细节可以在`ConcurrentSkipListMap's` Javadoc 中找到，它位于`src.zip`文件中的`java/util/concurrent`下。

## 6。结论

在本文中，我们主要介绍了`ConcurrentMap`接口和`ConcurrentHashMap` 的特性，并介绍了需要按键排序的`ConcurrentNavigableMap`。

本文中使用的所有示例的完整源代码可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20220626084203/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)