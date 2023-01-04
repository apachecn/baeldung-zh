# Executors newCachedThreadPool() vs newFixedThreadPool()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-executors-cached-fixed-threadpool>

## 1.概观

当谈到[线程池](/web/20221127171007/https://www.baeldung.com/java-executor-service-tutorial)实现时，Java 标准库提供了大量选项供选择。在这些实现中，固定线程池和缓存线程池非常普遍。

在本教程中，我们将了解线程池是如何工作的，然后比较这些实现及其用例。

## 2.缓存线程池

我们来看看 Java 在调用 [`Executors.newCachedThreadPool()`](https://web.archive.org/web/20221127171007/https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/util/concurrent/Executors.java#L217) 时是如何创建缓存线程池的:

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, 
      new SynchronousQueue<Runnable>());
}
```

缓存线程池使用“同步切换”来排队新任务。同步切换的基本思想很简单，但却是反直觉的:当且仅当另一个线程同时获取一个项目时，才可以将该项目排队。换句话说，**`SynchronousQueue `不能持有任何任务。**

假设有一个新任务进来。如果有一个空闲线程在队列中等待，那么任务生产者将任务移交给那个线程。否则，由于队列总是满的，执行器创建一个新线程来处理任务。

缓存池从零个线程开始，可能会增长到有`Integer.MAX_VALUE `个线程。实际上，缓存线程池的唯一限制是可用的系统资源。

为了更好地管理系统资源，缓存线程池将删除空闲一分钟的线程。

### 2.1.用例

缓存线程池配置将线程(因此得名)缓存一小段时间，以便在其他任务中重用它们。因此，当我们在处理一定数量的短期任务时，这种方法最有效。

这里的关键是“合理”和“短暂”。为了澄清这一点，让我们评估一个场景，其中缓存池不是一个很好的选择。这里我们将提交 100 万个任务，每个任务需要 100 微秒来完成:

```java
Callable<String> task = () -> {
    long oneHundredMicroSeconds = 100_000;
    long startedAt = System.nanoTime();
    while (System.nanoTime() - startedAt <= oneHundredMicroSeconds);

    return "Done";
};

var cachedPool = Executors.newCachedThreadPool();
var tasks = IntStream.rangeClosed(1, 1_000_000).mapToObj(i -> task).collect(toList());
var result = cachedPool.invokeAll(tasks);
```

这将产生大量线程，转化为不合理的内存使用，甚至更糟，大量的 CPU 上下文切换。这两种异常都会严重影响整体性能。

因此，当执行时间不可预测时，比如 IO 绑定任务，我们应该避免使用这个线程池。

## 3.固定线程池

让我们看看[固定线程](https://web.archive.org/web/20221127171007/https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/util/concurrent/Executors.java#L91)池是如何工作的:

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, 
      new LinkedBlockingQueue<Runnable>());
}
```

与缓存线程池相反，这个线程池使用一个无限队列，其中有固定数量的永不过期的线程**。因此，固定线程池试图用固定数量的线程**来执行传入的任务，而不是不断增加线程数量。当所有线程都忙的时候，执行器会把新的任务排队。这样，我们可以更好地控制程序的资源消耗。

因此，固定线程池更适合执行时间不可预测的任务。

## 4.不幸的相似之处

到目前为止，我们只列举了缓存线程池和固定线程池之间的区别。

抛开所有这些差异，他们都使用`[AbortPolicy](/web/20221127171007/https://www.baeldung.com/java-rejectedexecutionhandler#1-abort-policy)` 作为他们的[饱和策略](/web/20221127171007/https://www.baeldung.com/java-rejectedexecutionhandler) `.` 因此，我们期望这些执行者在他们不能接受甚至排队更多任务时抛出异常。

让我们看看现实世界会发生什么。

在极端情况下，缓存线程池将继续创建越来越多的线程，因此，实际上，**它们永远不会达到饱和点**。类似地，固定线程池将继续在其队列中添加越来越多的任务。**因此，固定池也永远不会达到饱和点**。

由于两个池都不会饱和，当负载异常高时，它们将消耗大量内存来创建线程或排队任务。雪上加霜的是，缓存线程池还会导致大量处理器上下文切换。

无论如何，为了让**更好地控制资源消耗，强烈建议创建一个自定义的** `**[ThreadPoolExecutor](https://web.archive.org/web/20221127171007/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)**`:

```java
var boundedQueue = new ArrayBlockingQueue<Runnable>(1000);
new ThreadPoolExecutor(10, 20, 60, SECONDS, boundedQueue, new AbortPolicy()); 
```

在这里，我们的线程池最多可以有 20 个线程，最多只能排队 1000 个任务。此外，当它不能接受更多的负载时，它将简单地抛出一个异常。

## 5.结论

在本教程中，我们偷看了一下 JDK 的源代码，看看在引擎盖下的工作有多么不同。然后，我们比较了固定线程池和缓存线程池及其用例。

最后，我们尝试用自定义线程池来解决这些池的失控资源消耗。