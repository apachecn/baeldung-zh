# Java 中原语链表的性能比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-primitive-performance>

## 1。概述

在本教程中，我们将**比较 Java** 中一些流行的[原语链表库](/web/20220626082909/https://www.baeldung.com/java-list-primitive-int)的性能。

为此，我们将测试每个库的`add(), get(),` 和 `contains()`方法。

## 2。性能比较

现在，**让我们看看哪个库提供了快速工作的原语集合 API** 。

为此，让我们比较一下来自`Trove, Fastutil`和`Colt`的`[List](/web/20220626082909/https://www.baeldung.com/java-arraylist)` 类似物。我们将使用 [JMH](/web/20220626082909/https://www.baeldung.com/java-microbenchmark-harness) (Java 微基准测试工具)工具来编写我们的性能测试。

### 2.1。JMH 参数

我们将使用以下参数运行基准测试:

```
@BenchmarkMode(Mode.SingleShotTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Measurement(batchSize = 100000, iterations = 10)
@Warmup(batchSize = 100000, iterations = 10)
@State(Scope.Thread)
public class PrimitivesListPerformance {
}
```

这里，**我们想要测量每个基准方法的执行时间。**此外，我们希望以毫秒为单位显示我们的结果。

`@State`注释表明类中声明的变量不会成为运行基准测试的一部分。然而，我们可以在我们的基准方法中使用它们。

此外，让我们定义并初始化我们的原语列表:

```
public static class PrimitivesListPerformance {
    private List<Integer> arrayList = new ArrayList<>(Arrays.asList(0, 1, 2, 3, 4, 5, 6, 7, 8, 9));
    private TIntArrayList tList = new TIntArrayList(new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9});
    private cern.colt.list.IntArrayList coltList = new cern.colt.list.IntArrayList(new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9});
    private IntArrayList fastUtilList = new IntArrayList(new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9});

    private int getValue = 4;
}
```

现在，我们准备好编写我们的基准了。

## 3。`add()`

首先，让我们测试将元素添加到原始列表中。我们还将为`ArrayList `添加一个作为我们的控件。

### 3.1。基准测试

第一个微基准是针对`[ArrayList](/web/20220626082909/https://www.baeldung.com/java-arraylist)‘` s `add()`方法的:

```
@Benchmark
public boolean addArrayList() {
    return arrayList.add(getValue);
}
```

同样，对于宝藏的`TIntArrayList.add()`:

```
@Benchmark
public boolean addTroveIntList() {
    return tList.add(getValue);
}
```

同样，柯尔特的`IntArrayList.add() `看起来像:

```
@Benchmark
public void addColtIntList() {
    coltList.add(getValue);
}
```

并且，对于 Fastutil 库，`IntArrayList.add()`方法基准将是:

```
@Benchmark
public boolean addFastUtilIntList() {
    return fastUtilList.add(getValue);
}
```

### 3.2。测试结果

现在，我们运行并比较结果:

```
Benchmark           Mode  Cnt  Score   Error  Units
addArrayList          ss   10  4.527 ± 4.866  ms/op
addColtIntList        ss   10  1.823 ± 4.360  ms/op
addFastUtilIntList    ss   10  2.097 ± 2.329  ms/op
addTroveIntList       ss   10  3.069 ± 4.026  ms/op
```

从结果中我们可以清楚的看到`ArrayList's add()`是最慢的选项。

这是合乎逻辑的，正如我们在 **[原语列表库](/web/20220626082909/https://www.baeldung.com/java-list-primitive-int)** 一文中解释的，`ArrayList`将使用装箱/自动装箱来存储集合中的 int 值。因此，我们这里有明显的减速。

另一方面，用于 Colt 和 Fastutil 的`add()`方法是最快的。

在幕后，这三个库都将值存储在一个`int[]`中。那么为什么他们的`add()`方法有不同的运行时间呢？

答案是当默认容量已满时，他们如何增加`int[]`:

*   **小马只有吃饱了才会长出内脏`int[]`**
*   相比之下， **Trove 和 Fastutil 将在扩展`int[]`容器时使用一些额外的计算**

这就是柯尔特在我们的测试结果中胜出的原因。

## 4。`get()`

现在，我们来添加`get()`操作微基准。

### 4.1。基准测试

首先，对于`ArrayList'` s `get()` 操作:

```
@Benchmark
public int getArrayList() {
    return arrayList.get(getValue);
}
```

同样，对于宝藏的`TIntArrayList `,我们将拥有:

```
@Benchmark
public int getTroveIntList() {
    return tList.get(getValue);
}
```

对于 Colt 的`cern.colt.list.IntArrayList, `，`get()`方法将是:

```
@Benchmark
public int getColtIntList() {
    return coltList.get(getValue);
}
```

最后，对于 Fastutil 的`IntArrayList`,我们将测试`getInt()`操作:

```
@Benchmark
public int getFastUtilIntList() {
    return fastUtilList.getInt(getValue);
}
```

### 4.2。测试结果

之后，我们运行基准测试并查看结果:

```
Benchmark           Mode  Cnt  Score   Error  Units
getArrayList        ss     20  5.539 ± 0.552  ms/op
getColtIntList      ss     20  4.598 ± 0.825  ms/op
getFastUtilIntList  ss     20  4.585 ± 0.489  ms/op
getTroveIntList     ss     20  4.715 ± 0.751  ms/op
```

虽然分数相差不多，但我们可以注意到`getArrayList()`运行得更慢。

对于其余的库，我们有几乎相同的`get()`方法实现。它们将**立即从`int[]`中检索值，无需任何进一步的工作。**这就是为什么柯尔特、法斯托夫和特罗弗在`get()`行动中有相似的表现。

## 5。`contains()`

最后，让我们为列表的每种类型测试一下`contains()`方法。

### 5.1。基准测试

让我们为`ArrayList'`的`contains()` 方法添加第一个微基准:

```
@Benchmark
public boolean containsArrayList() {
    return arrayList.contains(getValue);
}
```

同样，对于宝藏的`TIntArrayList `，`contains()`基准将是:

```
@Benchmark
public boolean containsTroveIntList() {
    return tList.contains(getValue);
}
```

同样，对柯尔特`cern.colt.list.IntArrayList.contains()` 的测试是:

```
@Benchmark
public boolean containsColtIntList() {
    return coltList.contains(getValue);
}
```

并且，对于 Fastutil 的`IntArrayList, `, contains()方法测试看起来像:

```
@Benchmark
public boolean containsFastUtilIntList() {
    return fastUtilList.contains(getValue);
}
```

### 5.2。测试结果

最后，我们运行测试并比较结果:

```
Benchmark                  Mode  Cnt   Score    Error  Units
containsArrayList          ss     20   2.083  ± 1.585  ms/op
containsColtIntList        ss     20   1.623  ± 0.960  ms/op
containsFastUtilIntList    ss     20   1.406  ± 0.400  ms/op
containsTroveIntList       ss     20   1.512  ± 0.307  ms/op
```

**和往常一样，`containsArrayList`方法的表现最差**。相比之下，Trove、Colt、Fastutil 相比 Java 的核心解决方案性能更好。

这一次，由开发人员决定选择哪个库。三个库的结果非常接近，可以认为它们是相同的。

## 6。结论

在本文中，我们通过 JVM 基准测试研究了原语列表的实际运行时性能。此外，我们将测试结果与 JDK 的`ArrayList`进行了比较。

另外，**请记住，我们在这里给出的数字只是 JMH 基准测试结果**——总是在给定的系统和运行时间范围内进行测试。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626082909/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)