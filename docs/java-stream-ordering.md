# Java 中的流排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-ordering>

## 1。概述

在本教程中，我们将深入探讨 [Java 流 API](/web/20221005184735/https://www.baeldung.com/java-8-streams-introduction) 的**不同用法如何影响流生成、处理和收集数据**的顺序。

我们还将看看**排序如何影响性能**。

## 2。遭遇顺序

简单来说，[遭遇顺序](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html#Ordering)就是**a`Stream`遭遇数据**的顺序。

### 2.1。`Collection`来源的遭遇顺序

我们选择的`Collection`会影响`Stream.`的遭遇顺序

为了测试这一点，让我们简单地创建两个流。

我们的第一个是从一个`List`创建的，它有一个内在的顺序。

我们的第二个是从一个没有。

然后我们将每个`Stream`的输出收集到一个`Array`中来比较结果。

```
@Test
public void givenTwoCollections_whenStreamedSequentially_thenCheckOutputDifferent() {
    List<String> list = Arrays.asList("B", "A", "C", "D", "F");
    Set<String> set = new TreeSet<>(list);

    Object[] listOutput = list.stream().toArray();
    Object[] setOutput = set.stream().toArray();

    assertEquals("[B, A, C, D, F]", Arrays.toString(listOutput));
    assertEquals("[A, B, C, D, F]", Arrays.toString(setOutput)); 
}
```

从我们的例子可以看出，`TreeSet `没有保持我们输入序列的顺序，因此打乱了`Stream`的相遇顺序。

如果我们的`Stream`是有序的，那么**不管我们的数据是顺序处理还是并行处理；**实现将保持`Stream`的相遇顺序。

当我们使用并行流重复我们的测试时，我们得到相同的结果:

```
@Test
public void givenTwoCollections_whenStreamedInParallel_thenCheckOutputDifferent() {
    List<String> list = Arrays.asList("B", "A", "C", "D", "F");
    Set<String> set = new TreeSet<>(list);

    Object[] listOutput = list.stream().parallel().toArray();
    Object[] setOutput = set.stream().parallel().toArray();

    assertEquals("[B, A, C, D, F]", Arrays.toString(listOutput));
    assertEquals("[A, B, C, D, F]", Arrays.toString(setOutput));
}
```

### 2.2。移除订单

在任何时候，我们都可以使用`unordered `方法来**显式地移除顺序约束。**

例如，让我们声明一个`TreeSet`:

```
Set<Integer> set = new TreeSet<>(
  Arrays.asList(-9, -5, -4, -2, 1, 2, 4, 5, 7, 9, 12, 13, 16, 29, 23, 34, 57, 102, 230));
```

如果我们流而不调用`unordered`:

```
set.stream().parallel().limit(5).toArray();
```

然后`TreeSet`的自然顺序被保留下来:

```
[-9, -5, -4, -2, 1]
```

但是，如果我们明确删除排序:

```
set.stream().unordered().parallel().limit(5).toArray();
```

那么输出是不同的:

```
[1, 4, 7, 9, 23]
```

原因有两个:首先，由于顺序流一次处理一个元素的数据，`unordered `本身没有什么影响。然而，当我们调用`parallel`时，我们也影响了输出。

## 3。中间操作

我们还可以通过中间操作来影响流的排序。

虽然大多数中间操作将保持`Stream,`的顺序，但有些操作本质上会改变它。

例如，我们可以通过排序来影响流的排序:

```
@Test
public void givenUnsortedStreamInput_whenStreamSorted_thenCheckOrderChanged() {
    List<Integer> list = Arrays.asList(-3, 10, -4, 1, 3);

    Object[] listOutput = list.stream().toArray();
    Object[] listOutputSorted = list.stream().sorted().toArray();

    assertEquals("[-3, 10, -4, 1, 3]", Arrays.toString(listOutput));
    assertEquals("[-4, -3, 1, 3, 10]", Arrays.toString(listOutputSorted));
}
```

`[unordered](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/BaseStream.html#unordered()) `和 [`empty`](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/IntStream.html#empty()) 是另外两个中间操作的例子，它们将最终改变`Stream.`的排序

## 4。终端操作

最后，我们可以根据我们使用的终端操作来影响顺序**。**

### 4.1。`ForEach `vs`ForEachOrdered`

`ForEach `和`ForEachOrdered`似乎提供了相同的功能，但是它们有一个关键的区别: **`ForEachOrdered `保证维护`Stream`的顺序。**

如果我们声明一个列表:

```
List<String> list = Arrays.asList("B", "A", "C", "D", "F");
```

并在并行化后使用`forEachOrdered`:

```
list.stream().parallel().forEachOrdered(e -> logger.log(Level.INFO, e));
```

那么输出是有序的:

```
INFO: B
INFO: A
INFO: C
INFO: D
INFO: F
```

但是，如果我们使用`forEach:`

```
list.stream().parallel().forEach(e -> logger.log(Level.INFO, e));
```

那么输出是**无序的**:

```
INFO: C
INFO: F
INFO: B
INFO: D
INFO: A
```

`ForEach`按照元素从每个线程到达的顺序记录元素。第二个`Stream `及其 **`ForEachOrdered `方法在调用`log `方法之前等待每个先前的线程完成**。

### 4.2。`Collect`

当我们使用`collect `方法聚合`Stream `输出时，需要注意的是我们选择的`Collection`将影响顺序。

例如，**天生无序`Collections`如`TreeSet`不会服从`Stream`** 输出的顺序:

```
@Test
public void givenSameCollection_whenStreamCollected_checkOutput() {
    List<String> list = Arrays.asList("B", "A", "C", "D", "F");

    List<String> collectionList = list.stream().parallel().collect(Collectors.toList());
    Set<String> collectionSet = list.stream().parallel()
      .collect(Collectors.toCollection(TreeSet::new)); 

    assertEquals("[B, A, C, D, F]", collectionList.toString()); 
    assertEquals("[A, B, C, D, F]", collectionSet.toString()); 
}
```

当运行我们的代码时，我们看到`Stream`的顺序通过收集到`Set.`中而改变

### 4.3。指定`Collection` s

在我们使用`Collectors.toMap`收集到一个无序集合的情况下，我们仍然可以通过**将我们的`Collectors `方法的实现改为使用链接的实现**来强制排序。

首先，我们将初始化我们的列表，以及通常的`toMap`方法的[双参数版本](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function)):

```
@Test
public void givenList_whenStreamCollectedToHashMap_thenCheckOrderChanged() {
  List<String> list = Arrays.asList("A", "BB", "CCC");

  Map<String, Integer> hashMap = list.stream().collect(Collectors
    .toMap(Function.identity(), String::length));

  Object[] keySet = hashMap.keySet().toArray();

  assertEquals("[BB, A, CCC]", Arrays.toString(keySet));
}
```

正如所料，我们的新`H` `ashMap`没有保持输入列表的原始顺序，但是让我们改变一下。

对于我们的第二个`Stream`，我们将使用`toMap `方法的 [4 参数版本](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function,java.util.function.BinaryOperator,java.util.function.Supplier))来告诉我们的`supplier `提供一个新的`LinkedHashMap`:

```
@Test
public void givenList_whenCollectedtoLinkedHashMap_thenCheckOrderMaintained(){
    List<String> list = Arrays.asList("A", "BB", "CCC");

    Map<String, Integer> linkedHashMap = list.stream().collect(Collectors.toMap(
      Function.identity(),
      String::length,
      (u, v) -> u,
      LinkedHashMap::new
    ));

    Object[] keySet = linkedHashMap.keySet().toArray();

    assertEquals("[A, BB, CCC]", Arrays.toString(keySet));
}
```

嘿，这样好多了！

通过将我们的数据收集到一个`LinkedHashMap`中，我们已经设法保持了列表的原始顺序。

## 5。性能

如果我们使用顺序流，顺序的存在与否对程序的性能影响不大。**然而，并行流会受到有序`Stream`的严重影响。**

原因是每个线程必须等待`Stream`的前一个元素的计算。

让我们使用 JMH 的 [Java 微基准测试工具](/web/20221005184735/https://www.baeldung.com/java-microbenchmark-harness)来测试性能。

在下面的例子中，我们将用一些常见的中间操作来衡量处理有序和无序并行流的性能成本。

### 5.1。`Distinct`

让我们使用`[distinct](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/IntStream.html#distinct()) `函数对有序流和无序流进行测试。

```
@Benchmark 
public void givenOrderedStreamInput_whenStreamDistinct_thenShowOpsPerMS() { 
    IntStream.range(1, 1_000_000).parallel().distinct().toArray(); 
}

@Benchmark
public void givenUnorderedStreamInput_whenStreamDistinct_thenShowOpsPerMS() {
    IntStream.range(1, 1_000_000).unordered().parallel().distinct().toArray();
}
```

当我们点击 run 时，我们可以看到每次操作所用时间的差异:

```
Benchmark                        Mode  Cnt       Score   Error  Units
TestBenchmark.givenOrdered...    avgt    2  222252.283          us/op
TestBenchmark.givenUnordered...  avgt    2   78221.357          us/op 
```

### 5.2。`Filter` 

接下来，我们将使用一个并行的`Stream`和一个简单的`[filter](https://web.archive.org/web/20221005184735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/IntStream.html#filter(java.util.function.IntPredicate)) `方法来返回每第 10 个整数`:`

```
@Benchmark
public void givenOrderedStreamInput_whenStreamFiltered_thenShowOpsPerMS() {
    IntStream.range(1, 100_000_000).parallel().filter(i -> i % 10 == 0).toArray();
}

@Benchmark
public void givenUnorderedStreamInput_whenStreamFiltered_thenShowOpsPerMS(){
    IntStream.range(1,100_000_000).unordered().parallel().filter(i -> i % 10 == 0).toArray();
}
```

有趣的是，我们的两个流之间的差异比使用`distinct `方法时要小得多。

```
Benchmark                        Mode  Cnt       Score   Error  Units
TestBenchmark.givenOrdered...    avgt    2  116333.431          us/op
TestBenchmark.givenUnordered...  avgt    2  111471.676          us/op
```

## 6。结论

在本文中，我们查看了流的排序，重点是`Stream `过程的不同阶段的**以及每个阶段如何有自己的效果**。

最后，我们看到了放置在`Stream`上的**订单契约如何影响并行流的性能。**

像往常一样，在 GitHub 上查看完整的样本集[。](https://web.archive.org/web/20221005184735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)