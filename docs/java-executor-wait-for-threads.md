# ExecutorService–等待线程完成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-executor-wait-for-threads>

## 1。概述

[`ExecutorService`](/web/20220625231716/https://www.baeldung.com/java-executor-service-tutorial) 框架让多线程处理任务变得简单。我们将举例说明一些等待线程完成执行的场景。

此外，我们将展示如何优雅地关闭一个`ExecutorService`并等待已经运行的线程完成它们的执行。

## 2。在`Executor's`关机后

当使用`Executor,`时，我们可以通过调用`shutdown()`或`shutdownNow()`方法来关闭它。**虽然，不会等到所有线程都停止执行。**

**等待现有线程完成它们的执行可以通过使用`awaitTermination()` 方法来实现。**

这将阻塞线程，直到所有任务完成执行或达到指定的超时时间:

```java
public void awaitTerminationAfterShutdown(ExecutorService threadPool) {
    threadPool.shutdown();
    try {
        if (!threadPool.awaitTermination(60, TimeUnit.SECONDS)) {
            threadPool.shutdownNow();
        }
    } catch (InterruptedException ex) {
        threadPool.shutdownNow();
        Thread.currentThread().interrupt();
    }
}
```

## 3。使用`CountDownLatch`

接下来，让我们看看解决这个问题的另一种方法——使用`CountDownLatch`来表示任务完成。

我们可以用一个值来初始化它，该值表示在通知所有调用了`await()`方法的线程之前它可以递减的次数。

例如，如果我们需要当前线程等待另一个`N`线程完成它们的执行，我们可以使用`N`初始化锁存器:

```java
ExecutorService WORKER_THREAD_POOL 
  = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(2);
for (int i = 0; i < 2; i++) {
    WORKER_THREAD_POOL.submit(() -> {
        try {
            // ...
            latch.countDown();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });
}

// wait for the latch to be decremented by the two remaining threads
latch.await();
```

## 4。使用 `invokeAll()`

我们可以用来运行线程的第一种方法是`invokeAll()`方法。**该方法在所有任务完成或超时**后返回一个`Future`对象列表。

此外，我们必须注意返回的`Future`对象的顺序与提供的`Callable`对象的列表相同:

```java
ExecutorService WORKER_THREAD_POOL = Executors.newFixedThreadPool(10);

List<Callable<String>> callables = Arrays.asList(
  new DelayedCallable("fast thread", 100), 
  new DelayedCallable("slow thread", 3000));

long startProcessingTime = System.currentTimeMillis();
List<Future<String>> futures = WORKER_THREAD_POOL.invokeAll(callables);

awaitTerminationAfterShutdown(WORKER_THREAD_POOL);

long totalProcessingTime = System.currentTimeMillis() - startProcessingTime;

assertTrue(totalProcessingTime >= 3000);

String firstThreadResponse = futures.get(0).get();

assertTrue("fast thread".equals(firstThreadResponse));

String secondThreadResponse = futures.get(1).get();
assertTrue("slow thread".equals(secondThreadResponse));
```

## 5。使用`ExecutorCompletionService`

运行多线程的另一种方法是使用 `ExecutorCompletionService.`它使用提供的`ExecutorService`来执行任务。

与`invokeAll()`的一个区别是代表已执行任务的`Futures,`的返回顺序。 **`ExecutorCompletionService`使用队列按照结果完成的顺序存储结果**，而`invokeAll()`返回一个列表，其顺序与迭代器为给定任务列表生成的顺序相同:

```java
CompletionService<String> service
  = new ExecutorCompletionService<>(WORKER_THREAD_POOL);

List<Callable<String>> callables = Arrays.asList(
  new DelayedCallable("fast thread", 100), 
  new DelayedCallable("slow thread", 3000));

for (Callable<String> callable : callables) {
    service.submit(callable);
} 
```

可以使用`take()`方法访问结果:

```java
long startProcessingTime = System.currentTimeMillis();

Future<String> future = service.take();
String firstThreadResponse = future.get();
long totalProcessingTime
  = System.currentTimeMillis() - startProcessingTime;

assertTrue("First response should be from the fast thread", 
  "fast thread".equals(firstThreadResponse));
assertTrue(totalProcessingTime >= 100
  && totalProcessingTime < 1000);
LOG.debug("Thread finished after: " + totalProcessingTime
  + " milliseconds");

future = service.take();
String secondThreadResponse = future.get();
totalProcessingTime
  = System.currentTimeMillis() - startProcessingTime;

assertTrue(
  "Last response should be from the slow thread", 
  "slow thread".equals(secondThreadResponse));
assertTrue(
  totalProcessingTime >= 3000
  && totalProcessingTime < 4000);
LOG.debug("Thread finished after: " + totalProcessingTime
  + " milliseconds");

awaitTerminationAfterShutdown(WORKER_THREAD_POOL);
```

## 6。结论

根据不同的用例，我们有不同的选择来等待线程完成它们的执行。

当我们需要一种机制来通知一个或多个线程由其他线程执行的一组操作已经完成时，一个`CountDownLatch`是有用的。

**`ExecutorCompletionService`在我们需要尽快访问任务结果时很有用，在我们希望等待所有正在运行的任务完成时也很有用。**

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625231716/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)