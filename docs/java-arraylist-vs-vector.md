# Java 数组列表与向量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraylist-vs-vector>

## 1。概述

在本教程中，**我们将关注`ArrayList`和`Vector`类**的区别。它们都属于 Java 集合框架，并实现了`java.util.List`接口。

然而，**这些类在实现上有着显著的差异**。

## 2。有什么不同？

作为一个快速的开始，让我们展示一下 **`ArrayList`和`Vector.`** 的主要区别，然后，我们将更详细地讨论一些要点:

*   同步——这两者之间的第一个主要区别。`Vector`是同步的而`ArrayList `不是。
*   大小增长——两者之间的另一个区别是它们在达到最大容量时调整大小的方式。**`Vector`是它的两倍大。相比之下，`ArrayList `仅增加其长度**的一半
*   iteration–And**`Vector`可以使用`Iterator`和`Enumeration`遍历元素。**另一方面，`ArrayList`只能用`Iterator`。
*   性能–主要由于同步，`Vector`操作比`ArrayList`慢
*   框架–另外，`ArrayList`是集合框架的一部分，在 JDK 1.2 中引入。同时，`Vector`作为遗留类出现在 Java 的早期版本中。

## 3。`Vector`

因为我们已经有了一个关于 ****`[ArrayList,](/web/20220524112326/https://www.baeldung.com/java-arraylist)`**** 的扩展指南，所以这里就不讨论它的 API 和功能了。另一方面，我们将呈现一些关于`Vector`的核心细节。

**简单地说`,`一个 `Vector`是一个可调整大小的数组**。它可以随着我们添加或删除元素而增长或收缩。

我们可以用典型的方式创建一个向量:

```java
Vector<String> vector = new Vector<>();
```

默认构造函数创建一个空的`Vector`，初始容量为 10。

让我们添加几个值:

```java
vector.add("baeldung");
vector.add("Vector");
vector.add("example");
```

最后，让我们通过使用 [`Iterator`](/web/20220524112326/https://www.baeldung.com/java-iterator) 接口来迭代这些值:

```java
Iterator<String> iterator = vector.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
    // ...
}
```

或者，我们可以使用`Enumeration`遍历`Vector`:

```java
Enumeration e = vector.elements();
while(e.hasMoreElements()) {
    String element = e.nextElement();
    // ... 
}
```

现在，让我们更深入地探索它们的一些独特功能。

## 4。并发性

我们已经提到过`ArrayList`和`Vector`在并发策略上是不同的，但是让我们仔细看看。如果我们深入研究`Vector's`方法签名，我们会看到每个都有 synchronized 关键字:

```java
public synchronized E get(int index)
```

简单地说，**这意味着一次只有一个线程可以访问一个给定的向量**。

实际上，这种操作级的同步无论如何都需要与我们自己的复合操作同步重叠。

所以相比之下，`ArrayList`采取了不同的方法。它的方法是`not`同步的，这种关注被分离到专门用于并发的类中。

例如，我们可以使用 [`CopyOnWriteArrayList`](/web/20220524112326/https://www.baeldung.com/java-copy-on-write-arraylist) 或 [`Collections.synchronizedList`](/web/20220524112326/https://www.baeldung.com/java-synchronized-collections) 来获得类似于`Vector`的效果:

```java
vector.get(1); // synchronized
Collections.synchronizedList(arrayList).get(1); // also synchronized
```

## 5。性能

正如我们上面已经讨论过的， **`Vector`是同步的，这直接影响了性能**。

为了查看`Vector`与`ArrayList `操作之间的性能差异，让我们编写一个简单的 [JMH 基准](/web/20220524112326/https://www.baeldung.com/java-microbenchmark-harness)测试。

过去，我们已经研究过[的`ArrayList`的操作](/web/20220524112326/https://www.baeldung.com/java-collections-complexity)的时间复杂度，所以让我们添加`Vector.` 的测试用例

`First`，我们来测试一下`get()`的方法:

```java
@Benchmark
public Employee testGet(ArrayListBenchmark.MyState state) {
    return state.employeeList.get(state.employeeIndex);
}

@Benchmark
public Employee testVectorGet(ArrayListBenchmark.MyState state) {
    return state.employeeVector.get(state.employeeIndex);
}
```

我们将配置 JMH 使用三个线程和 10 次预热迭代。

让我们报告一下纳秒级的每个操作的平均时间:

```java
Benchmark                         Mode  Cnt   Score   Error  Units
ArrayListBenchmark.testGet        avgt   20   9.786 ± 1.358  ns/op
ArrayListBenchmark.testVectorGet  avgt   20  37.074 ± 3.469  ns/op
```

我们可以看到 **`ArrayList#get`比`Vector#get`快三倍左右。**

现在，让我们比较一下`contains()`操作的结果:

```java
@Benchmark
public boolean testContains(ArrayListBenchmark.MyState state) {
    return state.employeeList.contains(state.employee);
}

@Benchmark
public boolean testContainsVector(ArrayListBenchmark.MyState state) {
    return state.employeeVector.contains(state.employee);
}
```

并将结果打印出来:

```java
Benchmark                              Mode  Cnt  Score   Error  Units
ArrayListBenchmark.testContains        avgt   20  8.665 ± 1.159  ns/op
ArrayListBenchmark.testContainsVector  avgt   20  36.513 ± 1.266  ns/op
```

我们可以看到，对于`contains()`操作，`Vector`的执行时间比`ArrayList`长得多。

## 6。总结

在本文中，我们看了一下 Java 中的`Vector`和`ArrayList`类之间的区别。此外，我们还详细介绍了`Vector`的功能。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524112326/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)