# Java 中的长加法器和长累加器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-longadder-and-longaccumulator>

## 1。概述

在本文中，我们将关注来自`java.util.concurrent`包的两个构造: [`LongAdder`](https://web.archive.org/web/20221105024754/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/LongAdder.html) 和`[LongAccumulator](https://web.archive.org/web/20221105024754/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/LongAccumulator.html).`

两者都是为了在多线程环境中非常高效而创建的，并且都利用非常聪明的策略来实现**无锁并且仍然保持线程安全。**

## 2。`LongAdder`

让我们考虑一些经常增加一些值的逻辑，其中使用`AtomicLong`可能是一个瓶颈。这使用了比较和交换操作，在激烈的争用情况下，会导致大量 CPU 周期的浪费。

另一方面，当线程增加时，使用一个非常聪明的技巧来减少线程间的争用。

当我们想要增加`LongAdder,` 的实例时，我们需要调用`increment()` 方法。这个实现**保留了一个可以按需增长的计数器数组**。

因此，当更多的线程调用`increment()`时，数组会更长。数组中的每条记录都可以单独更新，从而减少了争用。由于这个事实，`LongAdder` 是一种非常有效的从多线程中递增计数器的方法。

让我们创建一个`LongAdder` 类的实例，并从多线程中更新它:

```java
LongAdder counter = new LongAdder();
ExecutorService executorService = Executors.newFixedThreadPool(8);

int numberOfThreads = 4;
int numberOfIncrements = 100;

Runnable incrementAction = () -> IntStream
  .range(0, numberOfIncrements)
  .forEach(i -> counter.increment());

for (int i = 0; i < numberOfThreads; i++) {
    executorService.execute(incrementAction);
}
```

在我们调用`sum()` 方法之前，`LongAdder` 中计数器的结果是不可用的。该方法将迭代底层数组的所有值，并对这些值求和，返回正确的值。但是我们需要小心，因为调用`sum()` 方法的代价可能非常高:

```java
assertEquals(counter.sum(), numberOfIncrements * numberOfThreads);
```

有时，在我们调用了`sum()`之后，我们想要清除与`LongAdder` 的实例相关的所有状态，并从头开始计数。我们可以使用`sumThenReset()` 方法来实现:

```java
assertEquals(counter.sumThenReset(), numberOfIncrements * numberOfThreads);
assertEquals(counter.sum(), 0);
```

注意，对`sum()` 方法的后续调用返回零，这意味着状态被成功重置。

此外，Java 还提供了 [`DoubleAdder`](https://web.archive.org/web/20221105024754/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/DoubleAdder.html) ，用类似于`LongAdder.`的 API 来维护`double `值的总和

## 3。长累加器

`LongAccumulator`也是一个非常有趣的类——它允许我们在许多场景中实现无锁算法。例如，它可以用来根据提供的`LongBinaryOperator`累积结果——这类似于来自流 API 的`reduce()`操作。

可以通过向构造函数提供`LongBinaryOperator` 和初始值来创建`LongAccumulator` 的实例。重要的是要记住，如果我们给 **`LongAccumulator`提供一个累加顺序无关紧要的交换函数，它就会正确工作。**

```java
LongAccumulator accumulator = new LongAccumulator(Long::sum, 0L);
```

我们正在创建一个`LongAccumulator`，它将向累加器中已经存在的值添加一个新值。我们将`LongAccumulator` 的初始值设置为零，因此在第一次调用`accumulate()` 方法时，`previousValue`的值将为零。

让我们从多个线程调用`accumulate()` 方法:

```java
int numberOfThreads = 4;
int numberOfIncrements = 100;

Runnable accumulateAction = () -> IntStream
  .rangeClosed(0, numberOfIncrements)
  .forEach(accumulator::accumulate);

for (int i = 0; i < numberOfThreads; i++) {
    executorService.execute(accumulateAction);
}
```

注意我们是如何将一个数字作为参数传递给`accumulate()`方法的。该方法将调用我们的`sum()` 函数。

`LongAccumulator`正在使用比较和交换实现——这导致了这些有趣的语义。

首先，它执行一个定义为`LongBinaryOperator,` 的动作，然后检查`previousValue` 是否改变。如果它被改变了，则用新值再次执行该动作。如果不是，它成功地改变了存储在累加器中的值。

我们现在可以断言所有迭代的所有值的总和是`20200`:

```java
assertEquals(accumulator.get(), 20200);
```

有趣的是，Java 也为 [`DoubleAccumulator`](https://web.archive.org/web/20221105024754/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/DoubleAccumulator.html) 提供了相同的用途和 API，但针对的是`double `值。

## 4.动态条带化

Java 中所有加法器和累加器的实现都是从一个有趣的基类`[Striped64](https://web.archive.org/web/20221105024754/https://github.com/openjdk/jdk14u/blob/master/src/java.base/share/classes/java/util/concurrent/atomic/Striped64.java). ` **继承而来，这个类不是只使用一个值来保持当前状态，而是使用一个状态数组来将争用分配到不同的内存位置。**

下面是对`Striped64 `功能的简单描述:

[![Dynamic Striping](img/ed3984f984db3890cce2f7dc4bd2ae5f.png)](/web/20221105024754/https://www.baeldung.com/wp-content/uploads/2017/04/Untitled-2020-05-22-0432.svg)

不同的线程正在更新不同的内存位置。由于我们使用的是状态数组(也就是条带),这种想法被称为动态条带化。有趣的是，`Striped64 `是以这种思想和它对 64 位数据类型有效的事实命名的。

我们希望动态条带化能够提高整体性能。然而，JVM 分配这些状态的方式可能会产生反效果。

更具体地说，JVM 可以在堆中把这些状态分配在彼此附近。这意味着几个状态可以驻留在同一个 CPU 缓存行中。因此，**更新一个存储器位置可能导致对其附近状态**的高速缓存未命中。**这种现象，被称为[假分享](https://web.archive.org/web/20221105024754/https://alidg.me/blog/2020/4/24/thread-local-random#false-sharing)，会伤害到业绩**。

防止虚假分享。`Striped64 `实现在每个状态周围添加了足够的填充，以确保每个状态驻留在自己的缓存行中:

[![False Sharing](img/a734c3ad1f334a616ec95d2f4655af47.png)](/web/20221105024754/https://www.baeldung.com/wp-content/uploads/2017/04/Untitled-2020-05-22-0432.png)

`[@Contended](https://web.archive.org/web/20221105024754/https://github.com/openjdk/jdk14u/blob/master/src/java.base/share/classes/jdk/internal/vm/annotation/Contended.java) `注释负责添加这个填充。填充以消耗更多内存为代价来提高性能。

## 5。结论

在这个快速教程中，我们看了一下`LongAdder`和`LongAccumulator` ，我们已经展示了如何使用这两个构造来实现非常高效的无锁解决方案。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221105024754/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。