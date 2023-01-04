# HashSet 中 removeAll()的性能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashset-removeall-performance>

## 1.概观

[`HashSet`](/web/20221208143859/https://www.baeldung.com/java-hashset) 是存储独特元素的集合。

在本教程中，我们将讨论`java.util.HashSet `类中`removeAll()`方法的性能。

## 2.`HashSet.removeAll()`

`removeAll`方法删除了包含在`collection`中的所有元素:

```
Set<Integer> set = new HashSet<Integer>();
set.add(1);
set.add(2);
set.add(3);
set.add(4);

Collection<Integer> collection = new ArrayList<Integer>();
collection.add(1);
collection.add(3);

set.removeAll(collection);

Integer[] actualElements = new Integer[set.size()];
Integer[] expectedElements = new Integer[] { 2, 4 };
assertArrayEquals(expectedElements, set.toArray(actualElements)); 
```

因此，元素 1 和 3 将从集合中移除。

## 3.内部实现和时间复杂性

方法决定了集合和集合哪个更小。这是通过在集合和集合上调用`size() `方法来完成的。

**如果集合的元素比集合**少，那么它会以[的时间复杂度](/web/20221208143859/https://www.baeldung.com/java-algorithm-complexity) O( `n`)迭代指定的集合。它还检查该元素是否存在于时间复杂度为 O(1)的集合中。如果该元素存在，则使用集合的`remove()`方法将其从集合中移除，这同样具有 O(1)的时间复杂度。所以**整体时间复杂度为 O( `n` )** 。

**如果集合的元素比集合**少，那么它使用 O( `n`)遍历这个集合。然后它通过调用它的`contains()`方法来检查集合中是否存在每个元素。并且如果这样的元素存在，则从集合中移除该元素。所以这取决于`contains()`方法的时间复杂度。

现在在这种情况下，如果集合是一个`ArrayList`，`contains()`方法的时间复杂度是 O( `m`)。因此**从集合中移除`ArrayList`中所有元素的总时间复杂度为 O( `n` * `m` )** 。

如果集合又是`HashSet`，那么`contains()`方法的时间复杂度是 O(1)。因此**从集合中移除`HashSet`中所有元素的总时间复杂度为 O( `n` )** 。

## 4.表演

要了解以上 3 种情况的性能差异，让我们编写一个简单的 [JMH 基准测试](/web/20221208143859/https://www.baeldung.com/java-microbenchmark-harness)测试。

对于第一种情况，我们将初始化集合和集合，其中集合中的元素比集合中的多。在第二种情况下，我们将初始化集合和集合，其中集合中的元素比集合中的多。在第三种情况下，我们将初始化 2 个集合，其中第二个集合的元素数量比第一个多:

```
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5)
public class HashSetBenchmark {

    @State(Scope.Thread)
    public static class MyState {
        private Set employeeSet1 = new HashSet<>();
        private List employeeList1 = new ArrayList<>();
        private Set employeeSet2 = new HashSet<>();
        private List employeeList2 = new ArrayList<>();
        private Set<Employee> employeeSet3 = new HashSet<>();
        private Set<Employee> employeeSet4 = new HashSet<>();

        private long set1Size = 60000;
        private long list1Size = 50000;
        private long set2Size = 50000;
        private long list2Size = 60000;
        private long set3Size = 50000;
        private long set4Size = 60000;

        @Setup(Level.Trial)
        public void setUp() {
            // populating sets
        }
    }
}
```

之后，我们添加我们的基准测试:

```
@Benchmark
public boolean given_SizeOfHashsetGreaterThanSizeOfCollection_whenRemoveAllFromHashSet_thenGoodPerformance(MyState state) {
    return state.employeeSet1.removeAll(state.employeeList1);
}

@Benchmark
public boolean given_SizeOfHashsetSmallerThanSizeOfCollection_whenRemoveAllFromHashSet_thenBadPerformance(MyState state) {
    return state.employeeSet2.removeAll(state.employeeList2);
}

@Benchmark
public boolean given_SizeOfHashsetSmallerThanSizeOfAnotherHashSet_whenRemoveAllFromHashSet_thenGoodPerformance(MyState state) {
    return state.employeeSet3.removeAll(state.employeeSet4);
}
```

结果如下:

```
Benchmark                                              Mode  Cnt            Score            Error  Units
HashSetBenchmark.testHashSetSizeGreaterThanCollection  avgt   20      2700457.099 ±     475673.379  ns/op
HashSetBenchmark.testHashSetSmallerThanCollection      avgt   20  31522676649.950 ± 3556834894.168  ns/op
HashSetBenchmark.testHashSetSmallerThanOtherHashset    avgt   20      2672757.784 ±     224505.866  ns/op
```

**我们可以看到，当`HashSet`的元素比`Collection`少的时候,`HashSet.removeAll()`的表现相当糟糕，后者被作为参数传递给`removeAll()`方法。但是当对方再次集合是`HashSet`时，那么表现就不错了。**

## 5.结论

在本文中，我们看到了`removeAll()`在 HashSet 中的表现。当集合的元素比集合少时，`removeAll()`的性能取决于集合的`contains()`方法的时间复杂度。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)