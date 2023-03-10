# Java 8 并行流中的自定义线程池

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-parallel-streams-custom-threadpool>

## 1。概述

Java 8 引入了 S `treams`的概念，作为对数据执行批量操作的有效方式。而并行`Streams`可以在支持并发的环境中获得。

这些流可以提高性能，但代价是多线程开销。

在这个快速教程中，我们将看看**对`Stream` API** 的最大限制之一，并看看如何让一个并行流与一个定制的`ThreadPool` 实例一起工作，或者——[有一个库处理这个](https://web.archive.org/web/20220630140730/https://github.com/pivovarit/parallel-collectors)。

## 2。平行`Stream`

让我们从一个简单的例子开始——在任何一个`Collection`类型上调用`parallelStream`方法——这将返回一个可能并行的`Stream`:

```java
@Test
public void givenList_whenCallingParallelStream_shouldBeParallelStream(){
    List<Long> aList = new ArrayList<>();
    Stream<Long> parallelStream = aList.parallelStream();

    assertTrue(parallelStream.isParallel());
}
```

在这样一个`Stream`中发生的默认处理使用`ForkJoinPool.commonPool(),` **一个由整个应用程序共享的线程池。**

## 3。自定义线程池

**我们实际上可以在处理`stream`时传递一个自定义`ThreadPool` 。**

以下示例让 parallel `Stream`使用自定义`ThreadPool`来计算从 1 到 1，000，000(包括 1 和 1，000，000)的长值之和:

```java
@Test
public void giveRangeOfLongs_whenSummedInParallel_shouldBeEqualToExpectedTotal() 
  throws InterruptedException, ExecutionException {

    long firstNum = 1;
    long lastNum = 1_000_000;

    List<Long> aList = LongStream.rangeClosed(firstNum, lastNum).boxed()
      .collect(Collectors.toList());

    ForkJoinPool customThreadPool = new ForkJoinPool(4);
    long actualTotal = customThreadPool.submit(
      () -> aList.parallelStream().reduce(0L, Long::sum)).get();

    assertEquals((lastNum + firstNum) * lastNum / 2, actualTotal);
}
```

我们使用了并行度为 4 的`ForkJoinPool` 构造函数。需要进行一些实验来确定不同环境的最佳值，但是一个好的经验法则是根据您的 CPU 有多少个内核来选择数量。

接下来，我们处理并行`Stream`的内容，在`reduce` 调用中对它们进行汇总。

这个简单的例子可能没有展示使用定制线程池的全部用处，但是在我们不希望将公共线程池与长时间运行的任务捆绑在一起的情况下——比如处理来自网络源的数据——或者公共线程池被应用程序中的其他组件使用的情况下，它的好处是显而易见的。

如果我们运行上面的测试方法，它会通过。到目前为止，一切顺利。

但是，如果我们用普通方法实例化`ForkJoinPool`类，就像在测试方法中一样，可能会导致`OutOfMemoryError`。

接下来，让我们仔细看看内存泄漏的原因。

## 4.小心内存泄漏

正如我们前面谈到的，默认情况下，整个应用程序都使用公共线程池。**公共线程池是一个静态的`ThreadPool` 实例。**

因此，如果我们使用默认的线程池，就不会发生内存泄漏。

现在，让我们回顾一下我们的测试方法。在测试方法中，我们创建了一个对象`ForkJoinPool. `。当测试方法完成后，**这个`customThreadPool`对象将不会被解引用和被垃圾收集——相反，它将等待分配新的任务**。

也就是说，我们每次调用测试方法，都会创建一个新的`customThreadPool`对象，不会释放。

这个问题的解决非常简单:在我们执行方法之后的`shutdown`对象:

```java
try {
    long actualTotal = customThreadPool.submit(
      () -> aList.parallelStream().reduce(0L, Long::sum)).get();
    assertEquals((lastNum + firstNum) * lastNum / 2, actualTotal);
} finally {
    customThreadPool.shutdown();
} 
```

## 5。结论

我们简要地看了如何使用定制的`ThreadPool`来运行并行`Stream`。在正确的环境中，通过正确使用并行级别，在某些情况下可以获得性能提升。

如果我们创建一个自定义的 `ThreadPool`，我们应该记住调用它的`shutdown()`方法以避免内存泄漏。

本文中引用的完整代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220630140730/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)