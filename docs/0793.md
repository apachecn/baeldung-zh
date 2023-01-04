# 如何检查所有的可运行程序是否都已完成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-runnables-check-status>

## 1.概观

在本文中，我们将获取一个`[Runnable](/web/20221107202803/https://www.baeldung.com/java-runnable-callable)`对象列表，并检查它们是否都已完成。我们知道，`Runnable`是一个接口，它的实例可以作为 [`Thread`](/web/20221107202803/https://www.baeldung.com/java-thread-lifecycle) 运行。我们将使用包装对象如`CompletableFuture`和`ThreadPoolExecutor`来运行这些线程。

## 2.示例设置

让我们创建一个基本的`Runnable`，它只会记录一条消息，然后暂停一毫秒:

```
static Runnable RUNNABLE = () -> {
    try {
        System.out.println("launching runnable");
        Thread.sleep(1000);
    } catch (InterruptedException e) {
    }
};
```

我们现在将创建`Runnable.`的`L` `ist`。在本例中，我们将重复添加相同的`Runnable.`。实现这一点的一种方法是使用`[IntStream](/web/20221107202803/https://www.baeldung.com/java-intstream-convert)`:

```
List<Runnable> runnables = IntStream.range(0, 5)
    .mapToObj(x -> RUNNABLE)
    .collect(Collectors.toList());
```

现在让我们看看如何运行这些`Runnable`对象，并了解它们是否都完成了。

## 3.使用`CompletableFuture`

**从 Java 8 开始，我们可以使用内置的 [`CompletableFuture`](/web/20221107202803/https://www.baeldung.com/java-completablefuture) 的`isDone()`方法来实现这个目的。**

对象使 Java 中的异步编程变得更容易。给定我们的`Runnable`列表，我们将使用`CompletableFuture`的`runAsync()`方法来异步运行相关任务。我们注意到，默认情况下，所有这些任务都将在`[ForkJoinPool](/web/20221107202803/https://www.baeldung.com/java-fork-join)`上运行。

出于进一步的目的，我们希望将所有的结果`CompletableFuture`包装在一个`[array](/web/20221107202803/https://www.baeldung.com/java-arrays-guide)`中:

```
CompletableFuture<?>[] completableFutures = runnables.stream()
    .map(CompletableFuture::runAsync)
    .toArray(CompletableFuture<?>[]::new);
```

现在，我们所有的`Runnable`任务都被打包到`CompletableFuture`执行中。这意味着当我们的程序继续运行时，这些任务将在后台异步运行。

为了查明是否所有的执行都在程序中的任意点完成，我们将从数组中创建一个新的包装`CompletableFuture`。`allOf()`方法将允许我们这样做。然后，我们将把`isDone()`方法直接应用于包装`CompletableFuture`:

```
boolean isEveryRunnableDone = CompletableFuture.allOf(completableFutures)
    .isDone();
```

如果任一`CompletableFuture`仍在运行，`isEveryRunnableDone`将为`false,`，否则将为`true.`

## 4.使用`ThreadPoolExecutor`

从 Java 5 开始，[线程池](/web/20221107202803/https://www.baeldung.com/thread-pool-java-and-guava)提供了额外的工具来帮助管理并发环境中的资源。特别是，他们维护一些统计数据，比如他们持有的已完成任务的数量。

### 4.1.计算剩余任务的数量

让我们创建一个有五个线程的`ThreadPoolExecutor`。然后我们将通过使用`execute()`方法提交每个`Runnable`来执行:

```
ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(5);
runnables.forEach(executor::execute);
```

**现在我们可以使用`getActiveCount()`方法来计算`ThreadPoolExecutor`中正在运行的任务的数量:**

```
int numberOfActiveThreads = executor.getActiveCount();
```

这里的问题是，我们能否将这个数字与`0`进行比较，以检查是否有任何`Runnable`仍在运行？事情实际上比这要复杂一点。问题是**`getActiveCount()`方法返回的数字是一个近似值，正如该类的文档所述。因此，我们不能依靠它来做任何决定。**

### 4.2.检查是否所有任务都已终止

`getActiveCount()`方法不会返回一个精确的值，因为这样做的计算量很大。因此，让我们马上放弃实现我们自己的计数器的选项。

**另一方面，`awaitTermination()`方法会让我们知道是否所有的任务都完成了。**但是首先，我们需要在执行程序上调用`shutdown()`方法。调用此方法将确保所有提交的任务都将完成。但是，它会阻止向执行器添加新任务:

```
executor.shutdown();
```

我们已经确保我们的`ThreadPoolExecutor`将正确关闭。我们现在可以通过调用 [`awaitTermination()`](/web/20221107202803/https://www.baeldung.com/java-executor-wait-for-threads#after-executors-shutdown) 随时检查这个池是否有任何正在运行的任务。此方法将一直阻塞，直到给定的超时或所有任务都完成为止。例如，为了我们的例子，让我们使用一秒钟的超时:

```
boolean isEveryRunnableDome = executor.awaitTermination(1000, TimeUnit.MILLISECONDS);
```

如果所有任务在一秒钟内完成，该方法立即返回`true.`，否则，程序将被阻塞一秒钟，然后返回`false.`

最后但同样重要的是，我们应该注意，如果任何底层线程被中断， **`awaitTermination()`就会抛出一个`[InterruptedException](/web/20221107202803/https://www.baeldung.com/java-interrupted-exception)`。**

## 5.结论

在本教程中，我们已经看到了如何检查是否所有的`Runnable`都完成了。对于高于 8 的 Java 版本，由于有了`CompletableFuture`类，这非常简单。对于旧版本，我们需要明智地选择我们的超时，因为程序可能会在我们设置的时间内被阻塞。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221107202803/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-2)