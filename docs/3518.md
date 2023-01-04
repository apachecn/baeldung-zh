# 如何在 Java 中延迟代码执行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-delay-code-execution>

## 1.介绍

Java 程序在其操作中添加延迟或暂停是相对常见的。这对于任务调整或暂停执行直到另一个任务完成非常有用。

本教程将描述在 Java 中实现延迟的两种方法。

## 2.基于`Thread`的方法

当一个 Java 程序运行时，它产生一个在主机上运行的进程。**这个进程包含至少一个线程——主线程**——程序在其中运行。此外，Java 支持[多线程](/web/20221126232147/https://www.baeldung.com/java-thread-lifecycle)，这使得应用程序能够创建与主线程并行或异步运行的新线程。

### 2.1.使用`Thread.sleep`

在 Java 中，暂停的一种快速而肮脏的方法是告诉当前线程休眠一段指定的时间。这可以通过使用`Thread.sleep(milliseconds)`来完成:

```
try {
    Thread.sleep(secondsToSleep * 1000);
} catch (InterruptedException ie) {
    Thread.currentThread().interrupt();
}
```

**将`sleep`方法包装在一个 try/catch 块中是一个好习惯，以防另一个线程中断了睡眠线程。**在这种情况下，我们捕获`InterruptedException` 并显式中断当前线程，因此它可以在以后被捕获并处理。这在多线程程序中更重要，但在单线程程序中仍然是很好的实践，以防我们以后添加其他线程。

### 2.2.使用`TimeUnit.sleep`

**为了更好的可读性，我们可以使用`TimeUnit.XXX.sleep(y)`，其中`XXX`是睡眠的时间单位(`SECONDS`、`MINUTES`等)。)，而`y`是该单元的睡眠数。**这个在幕后使用`Thread.sleep`。下面是一个`TimeUnit`语法的例子:

```
try {
    TimeUnit.SECONDS.sleep(secondsToSleep);
} catch (InterruptedException ie) {
    Thread.currentThread().interrupt();
}
```

然而，**使用这些基于线程的方法有一些缺点**:

*   睡眠时间并不精确，尤其是在使用毫秒和纳秒这样的较小时间增量时
*   当在循环内部使用时，由于其他代码的执行，sleep 会在循环迭代之间轻微漂移，因此在多次迭代后，执行时间可能会变得不精确

## 3.基于`ExecutorService`的方法

Java 提供了 [`ScheduledExecutorService`](/web/20221126232147/https://www.baeldung.com/java-executor-service-tutorial) 接口，这是一个更加健壮和精确的解决方案。该接口可以安排代码在指定的延迟后或以固定的时间间隔运行一次。

要在延迟后运行一段代码，我们可以使用`schedule` 方法:

```
ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();

executorService.schedule(Classname::someTask, delayInSeconds, TimeUnit.SECONDS);
```

`Classname::someTask`部分是我们指定延迟后运行的方法:

*   `someTask`是我们要执行的方法的名称
*   `Classname`是包含`someTask`方法的类的名称

要以固定的时间间隔运行任务，我们可以使用`scheduleAtFixedRate`方法:

```
ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();

executorService.scheduleAtFixedRate(Classname::someTask, 0, delayInSeconds, TimeUnit.SECONDS);
```

这将重复调用`someTask`方法，在每次调用之间暂停`delayInSeconds`。

除了允许更多的定时选项，`ScheduledExecutorService`方法还能产生更精确的时间间隔，因为它防止了漂移问题。

## 4.结论

在本文中，我们讨论了在 Java 程序中产生延迟的两种方法。

这篇文章的完整代码可以在 Github 上找到[。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221126232147/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-2)