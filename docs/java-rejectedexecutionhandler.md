# RejectedExecutionHandler 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rejectedexecutionhandler>

## 1.概观

Java 中的 [Executor 框架](/web/20220523135802/https://www.baeldung.com/java-executor-service-tutorial)试图将任务提交与任务执行分离。虽然这种方法很好地抽象出了任务执行的细节，但有时，我们仍然需要对其进行配置，以实现更优化的执行。

在本教程中，我们将看到当一个线程池不能接受更多的任务时会发生什么。然后，我们将学习如何通过适当地应用饱和策略来控制这种极限情况。

## 2.重新审视线程池

下图显示了[执行器服务](/web/20220523135802/https://www.baeldung.com/thread-pool-java-and-guava)的内部工作方式:

[![](img/734a9ec9232de53d417c27bc56e9e9d3.png)](/web/20220523135802/https://www.baeldung.com/wp-content/uploads/2019/11/Untitled-Diagram-res.png)

下面是当我们向执行者提交一个新任务时发生的事情**:**

1.  如果其中一个线程可用，它将处理该任务。
2.  否则，执行器将新任务添加到它的队列中。
3.  当一个线程完成当前任务时，它从队列中选取另一个任务。

### 2.1.`ThreadPoolExecutor`

大多数执行器实现使用众所周知的`[ThreadPoolExecutor](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)` 作为它们的基本实现。因此，为了更好地理解任务排队是如何工作的，我们应该仔细看看它的构造函数:

```java
public ThreadPoolExecutor(
  int corePoolSize,
  int maximumPoolSize,
  long keepAliveTime,
  TimeUnit unit,
  BlockingQueue<Runnable> workQueue,
  RejectedExecutionHandler handler
)
```

### 2.2.核心池大小

`corePoolSize` 参数决定线程池的初始大小。**通常，执行器会确保线程池至少包含`corePoolSize` 个线程。**

然而，如果我们启用了`[allowCoreThreadTimeOut](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html#allowCoreThreadTimeOut(boolean))` 参数，就可能有更少的线程。

### 2.3.最大池大小

让我们假设所有的核心线程都忙于执行一些任务。因此，执行器将新任务排队，直到它们有机会在以后被处理。

当这个队列变满时，执行器可以向线程池添加更多的线程。**`maximumPoolSize`为线程池可能包含的线程数量设定了上限。**

当这些线程空闲一段时间后，执行器可以将它们从池中移除。因此，池大小可以收缩回其核心大小。

### 2.4.排队

正如我们前面看到的，当所有核心线程都忙时，执行器将新任务添加到队列中。有三种不同的方法对进行排队:

*   队列可以容纳无限数量的任务。因为这个队列永远不会填满，所以执行器会忽略最大大小。[固定大小](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newFixedThreadPool(int))和[单线程执行器](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newSingleThreadExecutor())都使用这种方法。
*   `Bounded Queue` **:** 顾名思义，队列只能容纳有限数量的任务。因此，当有界队列填满时，线程池会增长。
*   令人惊讶的是，这个队列容纳不下任何任务！使用这种方法，**我们可以将一个任务排队，当且仅当另一个线程同时在另一端选择相同的任务**。[缓存线程池执行器](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newCachedThreadPool())在内部使用这种方法。

当我们使用有界排队或同步切换时，让我们假设以下场景:

*   所有核心线程都很忙
*   内部队列已满
*   线程池增长到其最大可能大小，所有这些线程也很忙

当一个新任务进来时会发生什么？

## 3.饱和政策

当所有线程都很忙，内部队列填满时，执行器就会饱和。

一旦达到饱和，执行者可以执行预先定义的动作。这些行为被称为饱和策略。**我们可以通过将`RejectedExecutionHandler`的实例传递给它的构造函数来修改执行器的饱和策略。**

幸运的是，Java 为这个类提供了一些内置的实现，每个实现覆盖一个特定的用例。在接下来的部分中，我们将详细评估这些策略。

### 3.1.中止策略

默认策略是[中止策略](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.AbortPolicy.html)。**中止策略导致执行者抛出一个** `**RejectedExecutionException**`:

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS, 
  new SynchronousQueue<>(), 
  new ThreadPoolExecutor.AbortPolicy());

executor.execute(() -> waitFor(250));

assertThatThrownBy(() -> executor.execute(() -> System.out.println("Will be rejected")))
  .isInstanceOf(RejectedExecutionException.class);
```

因为第一个任务需要很长时间来执行，所以执行者拒绝了第二个任务。

### 3.2.呼叫者运行策略

这个策略**让调用者线程执行任务**，而不是在另一个线程中异步运行任务:

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS, 
  new SynchronousQueue<>(), 
  new ThreadPoolExecutor.CallerRunsPolicy());

executor.execute(() -> waitFor(250));

long startTime = System.currentTimeMillis();
executor.execute(() -> waitFor(500));
long blockedDuration = System.currentTimeMillis() - startTime;

assertThat(blockedDuration).isGreaterThanOrEqualTo(500);
```

提交第一个任务后，执行者不能再接受任何新的任务。因此，调用方线程会一直阻塞，直到第二个任务返回。

[呼叫者运行策略](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.CallerRunsPolicy.html) **使得实现简单形式的节流**变得容易。也就是说，慢速消费者可以减慢快速生产者的速度，以控制任务提交流程。

### 3.3.丢弃策略

[丢弃策略](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.DiscardPolicy.html) **在新任务未能提交**时静默丢弃该任务；

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS,
  new SynchronousQueue<>(), 
  new ThreadPoolExecutor.DiscardPolicy());

executor.execute(() -> waitFor(100));

BlockingQueue<String> queue = new LinkedBlockingDeque<>();
executor.execute(() -> queue.offer("Discarded Result"));

assertThat(queue.poll(200, MILLISECONDS)).isNull();
```

这里，第二个任务向队列发布一条简单的消息。因为它从来没有机会执行，所以队列保持为空，即使我们阻塞它一段时间。

### 3.4.丢弃-最旧的策略

[丢弃最旧的策略](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.DiscardOldestPolicy.html) **首先从队列的头部删除一个任务，然后重新提交新的任务**:

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS, 
  new ArrayBlockingQueue<>(2), 
  new ThreadPoolExecutor.DiscardOldestPolicy());

executor.execute(() -> waitFor(100));

BlockingQueue<String> queue = new LinkedBlockingDeque<>();
executor.execute(() -> queue.offer("First"));
executor.execute(() -> queue.offer("Second"));
executor.execute(() -> queue.offer("Third"));
waitFor(150);

List<String> results = new ArrayList<>();
queue.drainTo(results);

assertThat(results).containsExactlyInAnyOrder("Second", "Third");
```

这一次，我们使用一个只能容纳两个任务的有界队列。下面是我们提交这四个任务时发生的情况:

*   第一个任务占用单个线程 100 毫秒
*   执行器成功地将第二个和第三个任务排队
*   当第四个任务到达时，丢弃最旧的策略删除最旧的任务，为这个新任务腾出空间

**最旧丢弃策略和优先级队列不能很好地协同工作。**因为优先级队列的头具有最高优先级，**我们可能会丢失最重要的任务**。

### 3.5.自定义策略

也可以通过实现`[RejectedExecutionHandler](https://web.archive.org/web/20220523135802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/RejectedExecutionHandler.html)`接口来提供定制的饱和度策略:

```java
class GrowPolicy implements RejectedExecutionHandler {

    private final Lock lock = new ReentrantLock();

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        lock.lock();
        try {
            executor.setMaximumPoolSize(executor.getMaximumPoolSize() + 1);
        } finally {
            lock.unlock();
        }

        executor.submit(r);
    }
}
```

在本例中，当执行器饱和时，我们将最大池大小增加 1，然后重新提交相同的任务:

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS, 
  new ArrayBlockingQueue<>(2), 
  new GrowPolicy());

executor.execute(() -> waitFor(100));

BlockingQueue<String> queue = new LinkedBlockingDeque<>();
executor.execute(() -> queue.offer("First"));
executor.execute(() -> queue.offer("Second"));
executor.execute(() -> queue.offer("Third"));
waitFor(150);

List<String> results = new ArrayList<>();
queue.drainTo(results);

assertThat(results).contains("First", "Second", "Third");
```

正如预期的那样，所有四个任务都被执行。

### 3.6.关机

除了过载的执行器，**饱和策略也适用于所有已经关闭的执行器**:

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS, new LinkedBlockingQueue<>());
executor.shutdownNow();

assertThatThrownBy(() -> executor.execute(() -> {}))
  .isInstanceOf(RejectedExecutionException.class);
```

**这同样适用于所有处于关闭状态的执行程序:**

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1, 0, MILLISECONDS, new LinkedBlockingQueue<>());
executor.execute(() -> waitFor(100));
executor.shutdown();

assertThatThrownBy(() -> executor.execute(() -> {}))
  .isInstanceOf(RejectedExecutionException.class);
```

## 4.结论

在本教程中，首先，我们快速回顾了一下 Java 中的线程池。然后，在介绍了饱和执行者之后，我们学习了如何以及何时应用不同的饱和策略。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220523135802/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)