# Collections.synchronizedMap 与 ConcurrentHashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-synchronizedmap-vs-concurrenthashmap>

## 1.概观

在本教程中，我们将讨论`[Collections.synchronizedMap()](https://web.archive.org/web/20220701005015/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#synchronizedMap(java.util.Map))`和`[ConcurrentHashMap](/web/20220701005015/https://www.baeldung.com/java-concurrent-map)` `.`之间的区别

此外，我们将查看每种操作的读写操作的性能输出。

## 2.差异

`Collections.synchronizedMap()`和`ConcurrentHashMap`都提供了对数据集合的线程安全操作。

[`Collections`](https://web.archive.org/web/20220701005015/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html) 实用程序类提供了对集合进行操作的**多态算法，并返回包装的集合**。它的 `synchronizedMap()`方法提供了线程安全的功能。

顾名思义，`synchronizedMap()`返回一个由我们在参数中提供的`Map`支持的同步`Map`。为了提供线程安全，`synchronizedMap()`允许所有通过返回的`Map`对后台`Map`的访问。

`ConcurrentHashMap`是在 JDK 1.5 中作为`HashMap`的**增强版引入的，它支持检索和更新**的高并发性。`HashMap`不是线程安全的，所以在线程争用期间可能会导致不正确的结果。

类是线程安全的。因此，多线程可以对单个对象进行操作，而不会带来任何麻烦。

**在`ConcurrentHashMap,` 中，读操作是非阻塞的，而写操作锁定特定的段或桶。**默认的桶或并发级别是 16，这意味着在锁定一个段或桶后，16 个线程可以在任何时刻写入。

### 2.1.`ConcurrentModificationException`

对于`HashMap`这样的对象，不允许执行并发操作。因此，如果我们试图在迭代时更新一个`HashMap`，我们将会收到一个`ConcurrentModificationException`。使用`synchronizedMap()`时也会出现这种情况:

```
@Test(expected = ConcurrentModificationException.class)
public void whenRemoveAndAddOnHashMap_thenConcurrentModificationError() {
    Map<Integer, String> map = new HashMap<>();
    map.put(1, "baeldung");
    map.put(2, "HashMap");
    Map<Integer, String> synchronizedMap = Collections.synchronizedMap(map);
    Iterator<Entry<Integer, String>> iterator = synchronizedMap.entrySet().iterator();
    while (iterator.hasNext()) {
        synchronizedMap.put(3, "Modification");
        iterator.next();
    }
}
```

然而，`ConcurrentHashMap`却不是这样:

```
Map<Integer, String> map = new ConcurrentHashMap<>();
map.put(1, "baeldung");
map.put(2, "HashMap");

Iterator<Entry<Integer, String>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    map.put(3, "Modification");
    iterator.next()
}

Assert.assertEquals(3, map.size());
```

### 2.2.`null`支持

`Collections.synchronizedMap()`和`ConcurrentHashMap` **处理`null`键和值的方式不同**。

`ConcurrentHashMap`不允许`null`出现在键或值中:

```
@Test(expected = NullPointerException.class)
public void allowNullKey_In_ConcurrentHasMap() {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    map.put(null, 1);
}
```

然而，**使用`Collections.synchronizedMap()`，`null`支持依赖于输入`Map`** `.` 我们可以有一个`null`作为键，当`Collections.synchronizedMap()`由`HashMap`或`LinkedHashMap,` 支持时，可以有任意数量的`null`值，而如果我们使用`TreeMap`，我们可以有`null`值，但没有`null`键。

让我们断言，我们可以使用一个由`HashMap`支持的`Collections.synchronizedMap()`的`null`密钥:

```
Map<String, Integer> map = Collections
  .synchronizedMap(new HashMap<String, Integer>());
map.put(null, 1);
Assert.assertTrue(map.get(null).equals(1));
```

类似地，我们可以验证`Collections.synchronizedMap()`和`ConcurrentHashMap`的值中的`null`支持。

## 3.性能比较

让我们比较一下`ConcurrentHashMap`和`Collections.synchronizedMap().`的性能。在这种情况下，我们使用开源框架 [Java 微基准管理](/web/20220701005015/https://www.baeldung.com/java-microbenchmark-harness) (JMH)和**来比较这些方法在纳秒**内的性能。

我们对这些地图上的随机读写操作进行了比较。让我们快速看一下我们的 JMH 基准代码:

```
@Benchmark
public void randomReadAndWriteSynchronizedMap() {
    Map<String, Integer> map = Collections.synchronizedMap(new HashMap<String, Integer>());
    performReadAndWriteTest(map);
}

@Benchmark
public void randomReadAndWriteConcurrentHashMap() {
    Map<String, Integer> map = new ConcurrentHashMap<>();
    performReadAndWriteTest(map);
}

private void performReadAndWriteTest(final Map<String, Integer> map) {
    for (int i = 0; i < TEST_NO_ITEMS; i++) {
        Integer randNumber = (int) Math.ceil(Math.random() * TEST_NO_ITEMS);
        map.get(String.valueOf(randNumber));
        map.put(String.valueOf(randNumber), randNumber);
    }
} 
```

我们使用 10 个线程对 1，000 个项目进行了 5 次迭代，运行了我们的性能基准。让我们看看基准测试结果:

```
Benchmark                                                     Mode  Cnt        Score        Error  Units
MapPerformanceComparison.randomReadAndWriteConcurrentHashMap  avgt  100  3061555.822 ±  84058.268  ns/op
MapPerformanceComparison.randomReadAndWriteSynchronizedMap    avgt  100  3234465.857 ±  60884.889  ns/op
MapPerformanceComparison.randomReadConcurrentHashMap          avgt  100  2728614.243 ± 148477.676  ns/op
MapPerformanceComparison.randomReadSynchronizedMap            avgt  100  3471147.160 ± 174361.431  ns/op
MapPerformanceComparison.randomWriteConcurrentHashMap         avgt  100  3081447.009 ±  69533.465  ns/op
MapPerformanceComparison.randomWriteSynchronizedMap           avgt  100  3385768.422 ± 141412.744  ns/op
```

以上结果表明， **`ConcurrentHashMap`比** `**Collections.synchronizedMap()**`表现更好。

## 4.何时使用

当数据一致性至关重要时，我们应该选择`Collections.synchronizedMap()`,对于性能关键型应用程序，我们应该选择`ConcurrentHashMap`,在这些应用程序中，写操作远远多于读操作。

这是因为`Collections.synchronizedMap()`要求每个线程在读/写操作中获取整个对象的锁。相比之下，`ConcurrentHashMap` 允许线程在集合的不同段上获取锁，并同时进行修改。

## 5。结论

在本文中，我们已经展示了`ConcurrentHashMap`和`Collections.synchronizedMap()`之间的区别。我们还用一个简单的 JMH 基准测试展示了它们的性能。

与往常一样，代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220701005015/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-3)