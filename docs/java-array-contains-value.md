# 检查 Java 数组是否包含值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-contains-value>

## 1。概述

在这篇文章中，我们将看看不同的方法来搜索一个指定值的数组。

我们还将使用[JMH](/web/20221007030332/https://www.baeldung.com/java-microbenchmark-harness)(Java 微基准测试工具)来比较这些方法的性能，以确定哪种方法效果最好。

## 2。设置

对于我们的例子，我们将使用一个数组，它包含为每个测试随机生成的`Strings`:

```java
String[] seedArray(int length) {
    String[] strings = new String[length];
    Random value = new Random();
    for (int i = 0; i < length; i++) {
        strings[i] = String.valueOf(value.nextInt());
    }
    return strings;
}
```

为了在每个基准测试中重用数组，我们将声明一个内部类来保存数组和计数，这样我们就可以声明它在 JMH 中的作用域:

```java
@State(Scope.Benchmark)
public static class SearchData {
    static int count = 1000;
    static String[] strings = seedArray(1000);
} 
```

## 3。基本搜索

**搜索数组的三种常用方法是作为一个`List,`一个`Set,`或者用一个循环**来检查每个成员直到找到一个匹配。

让我们从实现每种算法的三种方法开始:

```java
boolean searchList(String[] strings, String searchString) {
    return Arrays.asList(SearchData.strings)
      .contains(searchString);
}

boolean searchSet(String[] strings, String searchString) {
    Set<String> stringSet = new HashSet<>(Arrays.asList(SearchData.strings));

    return stringSet.contains(searchString);
}

boolean searchLoop(String[] strings, String searchString) {
    for (String string : SearchData.strings) {
        if (string.equals(searchString))
        return true;
    }

    return false;
}
```

我们将使用这些类注释告诉 JMH 以微秒为单位输出平均时间，并运行五次预热迭代，以确保我们的测试是可靠的:

```java
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 5)
@OutputTimeUnit(TimeUnit.MICROSECONDS) 
```

并循环运行每个测试:

```java
@Benchmark
public void searchArrayLoop() {
    for (int i = 0; i < SearchData.count; i++) {
        searchLoop(SearchData.strings, "T");
    }
}

@Benchmark
public void searchArrayAllocNewList() {
    for (int i = 0; i < SearchData.count; i++) {
        searchList(SearchData.strings, "T");
    }
}

@Benchmark
public void searchArrayAllocNewSet() {
    for (int i = 0; i < SearchData.count; i++) {
        searchSet(SearchData.strings, "S");
    }
} 
```

当我们对每种方法运行 1000 次搜索时，我们的结果如下所示:

```java
SearchArrayTest.searchArrayAllocNewList  avgt   20    937.851 ±  14.226  us/op
SearchArrayTest.searchArrayAllocNewSet   avgt   20  14309.122 ± 193.844  us/op
SearchArrayTest.searchArrayLoop          avgt   20    758.060 ±   9.433  us/op 
```

循环搜索比其他搜索更有效。但这至少部分是因为我们如何使用集合。

我们在每次调用`searchList()`时创建一个新的`List`实例，在每次调用`searchSet()`时创建一个新的`List` 和新的`HashSet`。创建这些对象会产生额外的开销，而循环遍历数组则不会。

## 4。更高效的搜索

当我们创建`List`和`Set`的单个实例，然后在每次搜索中重用它们时会发生什么？

让我们试一试:

```java
public void searchArrayReuseList() {
    List asList = Arrays.asList(SearchData.strings);
    for (int i = 0; i < SearchData.count; i++) {
        asList.contains("T");
    }
}

public void searchArrayReuseSet() {
    Set asSet = new HashSet<>(Arrays.asList(SearchData.strings));
    for (int i = 0; i < SearchData.count; i++) {
        asSet.contains("T");
    }
} 
```

我们将使用与上面相同的 JMH 注释运行这些方法，并包括简单循环的结果以供比较。

我们看到非常不同的结果:

```java
SearchArrayTest.searchArrayLoop          avgt   20    758.060 ±   9.433  us/op
SearchArrayTest.searchArrayReuseList     avgt   20    837.265 ±  11.283  us/op
SearchArrayTest.searchArrayReuseSet      avgt   20     14.030 ±   0.197  us/op 
```

**虽然搜索`List`比以前稍微快了一点，但是`Set`下降到不到循环所需时间的百分之一！**

既然我们已经从每次搜索中去掉了创建新集合所需的时间，这些结果就有意义了。

搜索哈希表，作为`HashSet`底层的结构，时间复杂度为 0(1)，而作为`ArrayList`底层的数组是 0(n)。

## 5。二分搜索法

另一种搜索数组的方法是[二分搜索法](/web/20221007030332/https://www.baeldung.com/java-binary-search)。虽然非常高效，但二分搜索法需要提前对数组进行排序。

让我们对数组进行排序，并尝试二分搜索法:

```java
@Benchmark
public void searchArrayBinarySearch() {
    Arrays.sort(SearchData.strings);
    for (int i = 0; i < SearchData.count; i++) {
        Arrays.binarySearch(SearchData.strings, "T");
    }
} 
```

```java
SearchArrayTest.searchArrayBinarySearch  avgt   20     26.527 ±   0.376  us/op 
```

二分搜索法非常快，尽管效率不如`HashSet:`。二分搜索法的最差性能是 0(log n ),其性能介于数组搜索和哈希表之间。

## 6。结论

我们已经看到了几种搜索数组的方法。

根据我们的结果，`HashSet`最适合在值列表中搜索。但是，我们需要提前创建它们，并将其存储在`Set.`中

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221007030332/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-basic)