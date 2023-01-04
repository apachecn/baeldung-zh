# Arrays.sort(Object[])和 Arrays.sort(int[])的时间比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/arrays-sortobject-vs-sortint>

## 1。概述

在这个快速教程中，**我们将比较两个`Arrays.sort(Object[])`和`Arrays.sort(int[])` [排序](/web/20220901123627/https://www.baeldung.com/java-sorting)操作**。

首先，我们将分别描述每种方法。之后，我们将编写性能测试来测量它们的运行时间。

## 2。`Arrays.sort(Object[])`

在我们继续之前，重要的是要记住 **`[Arrays.sort()](/web/20220901123627/https://www.baeldung.com/java-util-arrays) `** 对原始和引用类型数组都有效。

**`Arrays.sort(Object[])` 接受引用类型**。

例如，我们有一个由*整数*对象组成的数组:

```java
Integer[] numbers = {5, 22, 10, 0};
```

要对数组进行排序，我们可以简单地使用:

```java
Arrays.sort(numbers);
```

现在，numbers 数组的所有元素都按升序排列:

```java
[0, 5, 10, 22]
```

`Arrays.sort(Object[]) `基于 TimSort 算法，**给出了时间复杂度`O(n log(n))`。**简而言之，TimSort 利用了[插入排序](/web/20220901123627/https://www.baeldung.com/java-insertion-sort)和[合并排序](/web/20220901123627/https://www.baeldung.com/java-merge-sort)算法。然而，与其他排序算法(如一些快速排序实现)相比，它仍然较慢。

## 3。`Arrays.sort(int[])`

另一方面， **`Arrays.sort(int[])`使用原始的`int`数组。**

类似地，我们可以定义一个原语的`int[]`数组:

```java
int[] primitives = {5, 22, 10, 0};
```

并用`Arrays.sort(int[])`的另一个实现进行排序。这一次，接受一个原语数组:

```java
Arrays.sort(primitives);
```

此操作的结果与前面的示例没有什么不同。并且`primitives`数组中的项目将看起来像:

```java
[0, 5, 10, 22]
```

在引擎盖下，它使用了双支点[快速排序算法](/web/20220901123627/https://www.baeldung.com/java-quicksort)。JDK 10 的内部实现通常比传统的单支点快速排序更快。

**该算法提供了`O(n log(n))`平均值** [**时间复杂度**](/web/20220901123627/https://www.baeldung.com/java-algorithm-complexity) 。对于许多收藏来说，这是一个很好的平均排序时间。而且它的优点是完全到位，所以不需要任何额外的存储。

虽然，**在最坏的情况下，其时间复杂度是`O(n²)`。**

## 4。时间比较

那么，哪种算法更快，为什么？让我们先做一些理论，然后我们将与 JMH 运行一些具体的测试。

### 4.1.定性分析

由于一些不同的原因， **`Arrays.sort(Object[])`通常比`Arrays.sort(int[])`** 慢。

首先是算法不同。快速排序通常比时间排序快。

其次是每个方法如何比较这些值。

请看，**因为`Arrays.sort(Object[])`需要将一个对象与另一个进行比较，所以它需要调用每个元素的`compareTo`方法。**至少，除了实际的比较操作之外，这需要一个方法查找和将一个调用推送到堆栈上。

另一方面， **`Arrays.sort(int[])`可以简单地使用像`<``>`**这样的原始关系运算符，它们是单字节码指令。

### 4.2。JMH 参数

最后，我们用实际数据来看看**哪种排序方法运行速度更快。为此，我们将使用 [JMH](/web/20220901123627/https://www.baeldung.com/java-microbenchmark-harness) (Java 微基准测试工具)来编写我们的基准测试。**

因此，我们将在这里做一个非常简单的基准测试。这并不全面，但是**将会给我们一个如何接近**的想法，比较`Arrays.sort(int[])`和`Arrays.sort(` `Integer[]` `)` 的排序方法。

在我们的基准测试类中，我们将使用配置注释:

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Measurement(batchSize = 100000, iterations = 10)
@Warmup(batchSize = 100000, iterations = 10)
public class ArraySortBenchmark {
}
```

这里，我们想要测量单个操作的平均时间(`Mode.AverageTime)` )并以毫秒(`TimeUnit.MILLISECONDS)`)显示我们的结果。此外，通过`batchSize`参数，我们告诉 JMH 执行 100，000 次迭代，以确保我们的结果具有高精度。

### 4.3。基准测试

在运行测试之前，我们需要定义我们想要排序的数据容器:

```java
@State(Scope.Thread)
public static class Initialize {
    Integer[] numbers = {-769214442, -1283881723, 1504158300, -1260321086, -1800976432, 1278262737, 
      1863224321, 1895424914, 2062768552, -1051922993, 751605209, -1500919212, 2094856518, 
      -1014488489, -931226326, -1677121986, -2080561705, 562424208, -1233745158, 41308167 };
    int[] primitives = {-769214442, -1283881723, 1504158300, -1260321086, -1800976432, 1278262737, 
      1863224321, 1895424914, 2062768552, -1051922993, 751605209, -1500919212, 2094856518, 
      -1014488489, -931226326, -1677121986, -2080561705, 562424208, -1233745158, 41308167};
}
```

让我们选择图元元素的`Integer[] numbers `和`int[]` *图元*数组。`@State`注释表明类中声明的变量不会成为运行基准测试的一部分。然而，我们可以在我们的基准方法中使用它们。

现在，我们准备为`Arrays.sort(Integer[])`添加第一个微基准:

```java
@Benchmark
public Integer[] benchmarkArraysIntegerSort(ArraySortBenchmark.Initialize state) {
    Arrays.sort(state.numbers);
    return state.numbers;
}
```

接下来，对于`Arrays.sort(int[])`:

```java
@Benchmark
public int[] benchmarkArraysIntSort(ArraySortBenchmark.Initialize state) {
    Arrays.sort(state.primitives);
    return state.primitives;
}
```

### 4.4。测试结果

最后，我们运行测试并比较结果:

```java
Benchmark                   Mode  Cnt  Score   Error  Units
benchmarkArraysIntSort      avgt   10  1.095 ± 0.022  ms/op
benchmarkArraysIntegerSort  avgt   10  3.858 ± 0.060  ms/op
```

从结果中，我们可以看到，在我们的测试中， **`Arrays.sort(int[])` 方法比`Arrays.sort(Object[])`** 方法表现得更好，这可能是我们之前确定的原因。

尽管这些数字似乎支持我们的理论，尽管我们需要用更多种类的输入来做测试以得到更好的想法。

另外，**请记住，我们在这里给出的数字只是 JMH 基准测试的结果**——所以我们应该总是在我们自己的系统和运行时范围内进行测试。

### 4.5.那为什么是 Timsort？

那么，我们也许应该问自己一个问题。如果快速排序更快，为什么不在两种实现中都使用它呢？

看到了，**快速排序不是`stable`** ，所以我们不能用它来排序`Objects`。基本上，如果两个`int`相等，它们的相对顺序保持不变并不重要，因为一个`2 `和另一个`2. `在对象上没有什么不同，我们可以先按一个属性排序，然后再按另一个属性排序，使得开始顺序很重要。

## 5。结论

在本文中，**我们比较了 Java 中可用的两种排序方法:`Arrays.sort(int[])`和** `**Arrays.sort(**` **`Integer[]`** `**)**.` 此外，我们还讨论了它们实现中使用的排序算法。

最后，借助基准性能测试，我们展示了每个排序选项的运行时间示例。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220901123627/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)