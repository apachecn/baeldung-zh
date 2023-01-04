# Java ExecutorService 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-executor-service-tutorial>

## 1。概述

`[ExecutorService](https://web.archive.org/web/20221012100323/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html)`是一个 JDK API，它简化了异步模式下的运行任务。一般来说，`ExecutorService`会自动提供一个线程池和一个为其分配任务的 API。

## 延伸阅读:

## [Java 中的 fork/join 框架指南](/web/20221012100323/https://www.baeldung.com/java-fork-join)

Java 7 中的 Fork/Join 框架介绍，以及通过尝试使用所有可用的处理器内核来帮助加速并行处理的工具。[阅读更多](/web/20221012100323/https://www.baeldung.com/java-fork-join)→

## [java.util.concurrent 概述](/web/20221012100323/https://www.baeldung.com/java-util-concurrent)

发现 Java . util . concurrent 包的内容。[阅读更多](/web/20221012100323/https://www.baeldung.com/java-util-concurrent)→

## [Java . util . concurrent . locks](/web/20221012100323/https://www.baeldung.com/java-concurrent-locks)

在本文中，我们探讨了锁接口的各种实现以及 Java 9 中新引入的 StampedLock 类。[阅读更多](/web/20221012100323/https://www.baeldung.com/java-concurrent-locks) →

## 2。实例化`ExecutorService`

### 2.1。`Executors`类的工厂方法

创建`ExecutorService`最简单的方法是使用`Executors`类的工厂方法之一。

例如，以下代码行将创建一个包含 10 个线程的线程池:

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
```

有几种其他的工厂方法来创建满足特定用例的预定义的 `ExecutorService`。要找到满足您需求的最佳方法，请查阅甲骨文的官方文档。

### 2.2。直接创建一个`ExecutorService`

因为`ExecutorService`是一个接口，所以可以使用它的任何实现的实例。在`[java.util.concurrent](https://web.archive.org/web/20221012100323/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html)`包中有几个实现可供选择，或者您可以创建自己的实现。

例如，`ThreadPoolExecutor`类有几个构造函数，我们可以用它们来配置 executor 服务及其内部池:

```java
ExecutorService executorService = 
  new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,   
  new LinkedBlockingQueue<Runnable>());
```

你可能会注意到上面的代码与工厂方法`newSingleThreadExecutor().` 的[源代码](https://web.archive.org/web/20221012100323/https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/classes/java/util/concurrent/Executors.java#L133)非常相似，在大多数情况下，详细的手动配置是不必要的。

## 3。`ExecutorService`给分配任务

`ExecutorService`可以执行`Runnable`和`Callable`任务。为了使本文简单，将使用两个基本任务。注意，我们在这里使用 lambda 表达式，而不是匿名内部类:

```java
Runnable runnableTask = () -> {
    try {
        TimeUnit.MILLISECONDS.sleep(300);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
};

Callable<String> callableTask = () -> {
    TimeUnit.MILLISECONDS.sleep(300);
    return "Task's execution";
};

List<Callable<String>> callableTasks = new ArrayList<>();
callableTasks.add(callableTask);
callableTasks.add(callableTask);
callableTasks.add(callableTask);
```

我们可以使用几种方法给`ExecutorService` 分配任务，包括从`Executor` 接口继承的`execute()`，以及`submit()`、 `invokeAny()`和`invokeAll()`。

**`execute()`** 方法是`void`，不提供任何获得任务执行结果或检查任务状态(是否正在运行)的可能性:

```java
executorService.execute(runnableTask);
```

**`submit()`** 向`ExecutorService`提交`Callable`或`Runnable`任务，并返回`Future`类型的结果:

```java
Future<String> future = 
  executorService.submit(callableTask);
```

**`invokeAny()`** 将一组任务分配给一个`ExecutorService`，使每个任务运行，并返回一个任务成功执行的结果(如果有成功执行的话):

```java
String result = executorService.invokeAny(callableTasks);
```

`**invokeAll()**`将一组任务分配给一个`ExecutorService`，使每个任务运行，并以`Future`类型对象列表的形式返回所有任务执行的结果:

```java
List<Future<String>> futures = executorService.invokeAll(callableTasks);
```

在继续之前，我们需要讨论另外两个项目:关闭一个`ExecutorService`和处理`Future`返回类型。

## 4。关闭一个`ExecutorService`

一般来说，当没有任务要处理时，`ExecutorService`不会被自动销毁。它将保持活力，等待新的工作来做。

在某些情况下，这非常有用，例如当应用程序需要处理不定期出现的任务，或者在编译时任务数量未知。

另一方面，一个应用程序可能会到达终点，但不会被停止，因为等待`ExecutorService`会导致 JVM 继续运行。

为了正确关闭一个`ExecutorService`，我们有了`shutdown()`和`shutdownNow()`API。

`**shutdown()**` 方法不会立即销毁`ExecutorService`。它将使`ExecutorService`停止接受新任务，并在所有正在运行的线程完成当前工作后关闭:

```java
executorService.shutdown();
```

**`shutdownNow()`** 方法试图立即销毁`ExecutorService`，但并不能保证所有正在运行的线程都会同时停止:

```java
List<Runnable> notExecutedTasks = executorService.shutDownNow();
```

此方法返回等待处理的任务列表。由开发人员决定如何处理这些任务。

关闭`ExecutorService` (这也是 Oracle 推荐的[)的一个好方法是将这两种方法与 **`awaitTermination()`** 方法结合使用:](https://web.archive.org/web/20221012100323/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html)

```java
executorService.shutdown();
try {
    if (!executorService.awaitTermination(800, TimeUnit.MILLISECONDS)) {
        executorService.shutdownNow();
    } 
} catch (InterruptedException e) {
    executorService.shutdownNow();
}
```

使用这种方法，`ExecutorService`将首先停止接受新任务，然后等待一段指定的时间来完成所有任务。如果计时器超时，执行将立即停止。

## 5。`Future`界面

`submit()` 和 `invokeAll()`方法返回一个对象或者一个`Future`类型的对象集合，这允许我们获得任务执行的结果或者检查任务的状态(它是否正在运行)。

`Future`接口提供了一个特殊的阻塞方法`get()`，返回`Callable` 任务执行的实际结果，或者在`Runnable`任务的情况下返回`null` :

```java
Future<String> future = executorService.submit(callableTask);
String result = null;
try {
    result = future.get();
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```

在任务仍在运行时调用`get()`方法将导致执行被阻塞，直到任务正确执行并且结果可用。

对于由`get()` 方法引起的非常长的阻塞，应用程序的性能会降低。如果结果数据不重要，可以通过使用超时来避免这种问题:

```java
String result = future.get(200, TimeUnit.MILLISECONDS);
```

如果执行时间长于指定时间(在本例中为 200 毫秒)，将抛出一个`TimeoutException`。

我们可以使用`isDone()`方法来检查分配的任务是否已经处理。

`Future`接口还提供了用`cancel()`方法取消任务执行和用`isCancelled()`方法检查取消的功能:

```java
boolean canceled = future.cancel(true);
boolean isCancelled = future.isCancelled();
```

## 6。`ScheduledExecutorService`界面

`ScheduledExecutorService`在一些预定义的延迟之后和/或周期性地运行任务。

同样，实例化一个`ScheduledExecutorService`的最好方法是使用`Executors` 类的工厂方法。

对于本节，我们使用一个带有一个线程的`ScheduledExecutorService`:

```java
ScheduledExecutorService executorService = Executors
  .newSingleThreadScheduledExecutor();
```

要在固定延迟后调度单个任务的执行，使用`ScheduledExecutorService`的`scheduled()`方法。

两种`scheduled()`方法允许您执行`Runnable`或`Callable` 任务:

```java
Future<String> resultFuture = 
  executorService.schedule(callableTask, 1, TimeUnit.SECONDS);
```

`scheduleAtFixedRate()`方法让我们在固定的延迟后定期运行一个任务。上面的代码在执行`callableTask`之前延迟了一秒钟。

下面的代码块将在初始延迟 100 毫秒后运行任务。之后，它会每隔 450 毫秒运行一次相同的任务:

```java
Future<String> resultFuture = service
  .scheduleAtFixedRate(runnableTask, 100, 450, TimeUnit.MILLISECONDS);
```

如果处理器需要比`scheduleAtFixedRate()`方法的`period`参数更多的时间来运行一个分配的任务，`ScheduledExecutorService`将等待直到当前任务完成，然后开始下一个任务。

如果有必要在任务的迭代之间有固定长度的延迟，应该使用`scheduleWithFixedDelay()`。

例如，以下代码将保证在当前执行结束和下一次执行开始之间有 150 毫秒的暂停:

```java
service.scheduleWithFixedDelay(task, 100, 150, TimeUnit.MILLISECONDS);
```

根据`scheduleAtFixedRate()`和`scheduleWithFixedDelay()`方法契约，任务的周期执行将在`ExecutorService`终止或任务执行过程中抛出异常`.`时结束

## 7。`ExecutorService` vs 分叉/加入

Java 7 发布后，很多开发者决定用 fork/join 框架替换`ExecutorService`框架。

然而，这并不总是正确的决定。尽管 fork/join 具有简单性和频繁的性能提升，但它减少了开发人员对并发执行的控制。

`ExecutorService`让开发人员能够控制生成线程的数量以及应该由单独线程运行的任务的粒度。`ExecutorService`的最佳用例是处理独立的任务，比如根据“一个任务一个线程”的方案处理事务或请求

相比之下，[根据甲骨文的文档](https://web.archive.org/web/20221012100323/https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html)，fork/join 的设计是为了加速那些可以递归分解成更小块的工作。

## 8。结论

尽管`ExecutorService`相对简单，但还是有一些常见的陷阱。

我们来总结一下:

**保持未使用的`ExecutorService`存活**:参见第 4 节关于如何关闭`ExecutorService`的详细解释。

**使用固定长度线程池时错误的线程池容量**:确定应用需要多少线程来高效运行任务非常重要。太大的线程池将导致不必要的开销，仅仅是为了创建大部分处于等待模式的线程。太少会使应用程序看起来没有响应，因为队列中的任务要等待很长时间。

**在任务取消**后调用`Future`的`get()`方法:试图获取一个已经取消的任务的结果触发了一个`CancellationException`。

**用`Future`的`get()`方法**意外长时间阻塞:我们应该使用超时来避免意外等待。

和往常一样，本文的代码可以在 GitHub 库中找到。