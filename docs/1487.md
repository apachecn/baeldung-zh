# 如何在 Java 中启动线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-start-thread>

## 1.介绍

在本教程中，我们将探索启动线程和执行并行任务的不同方式。

这非常有用，尤其是在处理不能在主线程上运行的长时间或循环操作时，或者在等待操作结果时不能暂停 UI 交互的情况下。

要了解更多关于线程的细节，一定要阅读我们关于 Java 线程生命周期的教程。

## 2.运行线程的基础

通过使用`Thread`框架，我们可以很容易地编写一些在并行线程中运行的逻辑。

让我们通过扩展`Thread`类来尝试一个基本的例子:

```
public class NewThread extends Thread {
    public void run() {
        long startTime = System.currentTimeMillis();
        int i = 0;
        while (true) {
            System.out.println(this.getName() + ": New Thread is running..." + i++);
            try {
                //Wait for one sec so it doesn't print too fast
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            ...
        }
    }
}
```

现在我们编写第二个类来初始化和启动我们的线程:

```
public class SingleThreadExample {
    public static void main(String[] args) {
        NewThread t = new NewThread();
        t.start();
    }
}
```

我们应该在处于`[NEW](https://web.archive.org/web/20221026205735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.State.html#NEW) `状态(相当于未启动)的线程上调用`start() `方法。否则，Java 会抛出一个 [`IllegalThreadStateException`](https://web.archive.org/web/20221026205735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IllegalThreadStateException.html) 异常的实例。

现在让我们假设我们需要启动多个线程:

```
public class MultipleThreadsExample {
    public static void main(String[] args) {
        NewThread t1 = new NewThread();
        t1.setName("MyThread-1");
        NewThread t2 = new NewThread();
        t2.setName("MyThread-2");
        t1.start();
        t2.start();
    }
}
```

我们的代码看起来仍然非常简单，与我们在网上找到的例子非常相似。

当然，这远远不是生产就绪代码，在生产就绪代码中，以正确的方式管理资源以避免过多的上下文切换或过多的内存使用是至关重要的。

**因此，为了做好生产准备，我们现在需要编写额外的样板文件**来处理:

*   新线程的持续创建
*   并发活动线程的数量
*   线程释放:为了避免泄漏，这对守护线程非常重要

如果我们愿意，我们可以为所有这些情况甚至更多的情况编写自己的代码，但是我们为什么要重新发明轮子呢？

## 3.`ExecutorService`框架

`ExecutorService`实现了线程池设计模式(也称为复制工人或工人团队模型),并负责我们上面提到的线程管理，此外它还添加了一些非常有用的功能，如线程可重用性和任务队列。

线程的可重用性尤其重要:在一个大规模的应用程序中，分配和释放许多线程对象会产生巨大的内存管理开销。

有了工作线程，我们可以最小化线程创建带来的开销。

为了简化池配置，`ExecutorService`提供了一个简单的构造函数和一些定制选项，比如队列类型、最小和最大线程数以及它们的命名约定。

关于`ExecutorService,`的更多细节，请阅读我们的[Java ExecutorService](/web/20221026205735/https://www.baeldung.com/java-executor-service-tutorial)指南。

## 4.使用执行者启动任务

多亏了这个强大的框架，我们可以从开始线程转换到提交任务。

让我们看看如何向我们的执行器提交一个异步任务:

```
ExecutorService executor = Executors.newFixedThreadPool(10);
...
executor.submit(() -> {
    new Task();
});
```

我们可以使用两种方法:`execute`，它不返回任何内容，以及`submit`，它返回一个封装了计算结果的`Future`。

关于`Futures,`的更多信息，请阅读我们的[Java . util . concurrent . future](/web/20221026205735/https://www.baeldung.com/java-future)指南。

## 5.使用`CompletableFutures`启动任务

为了从一个`Future`对象中获取最终结果，我们可以使用该对象中可用的`get`方法，但是这会阻塞父线程，直到计算结束。

或者，我们可以通过向任务中添加更多的逻辑来避免阻塞，但是我们必须增加代码的复杂性。

**Java 1.8 在`Future`构造的基础上引入了一个新的框架，以更好地处理计算结果:T1。**

**`CompletableFuture`实现了`CompletableStage`，它增加了大量的方法选择来附加回调，避免了在结果准备好之后运行操作所需的所有管道工作。**

提交任务的实现要简单得多:

```
CompletableFuture.supplyAsync(() -> "Hello");
```

`supplyAsync`接受一个包含我们想要异步执行的代码的`Supplier`——在我们的例子中是 lambda 参数。

**任务现在被隐式地提交给`ForkJoinPool.commonPool()`，或者我们可以指定我们喜欢的`Executor`作为第二个参数。**

要了解更多关于`CompletableFuture,`的信息，请阅读我们的[指南，以完成](/web/20221026205735/https://www.baeldung.com/java-completablefuture)的未来。

## 6.运行延迟或周期性任务

当使用复杂的 web 应用程序时，我们可能需要在特定的时间运行任务，也许是定期运行。

Java 几乎没有什么工具可以帮助我们运行延迟或重复的操作:

*   `java.util.Timer`
*   `java.util.concurrent.ScheduledThreadPoolExecutor`

### 6.1.` Timer`

`Timer`是一个在后台线程中调度未来执行任务的工具。

任务可以被安排为一次性执行，或者定期重复执行。

如果我们想在延迟一秒钟后运行一个任务，让我们看看代码是什么样子的:

```
TimerTask task = new TimerTask() {
    public void run() {
        System.out.println("Task performed on: " + new Date() + "n" 
          + "Thread's name: " + Thread.currentThread().getName());
    }
};
Timer timer = new Timer("Timer");
long delay = 1000L;
timer.schedule(task, delay);
```

现在让我们添加一个重复计划:

```
timer.scheduleAtFixedRate(repeatedTask, delay, period);
```

这一次，任务将在指定的延迟后运行，并在一段时间后重复运行。

更多信息，请阅读我们的 [Java 定时器](/web/20221026205735/https://www.baeldung.com/java-timer-and-timertask)指南。

### 6.2.`ScheduledThreadPoolExecutor`

`ScheduledThreadPoolExecutor`有类似于`Timer`类的方法:

```
ScheduledExecutorService executorService = Executors.newScheduledThreadPool(2);
ScheduledFuture<Object> resultFuture
  = executorService.schedule(callableTask, 1, TimeUnit.SECONDS);
```

为了结束我们的例子，我们使用`scheduleAtFixedRate()`来表示周期性任务:

```
ScheduledFuture<Object> resultFuture
 = executorService.scheduleAtFixedRate(runnableTask, 100, 450, TimeUnit.MILLISECONDS);
```

上面的代码将在初始延迟 100 毫秒后执行一个任务，之后，它将每隔 450 毫秒执行一次相同的任务。

**如果处理器不能在下一次发生之前及时完成任务处理，`ScheduledExecutorService`将等待当前任务完成，然后开始下一个任务。**

为了避免这种等待时间，我们可以使用`scheduleWithFixedDelay()`，正如它的名字所描述的，它保证了任务迭代之间的固定长度延迟。

For more details about `ScheduledExecutorService,` please read our [Guide to the Java ExecutorService](/web/20221026205735/https://www.baeldung.com/java-executor-service-tutorial).

### 6.3.哪个工具比较好？

如果我们运行上面的例子，计算的结果看起来是一样的。

那么，**我们如何选择合适的工具**？

当一个框架提供多种选择时，理解底层技术以做出明智的决定是很重要的。

让我们试着深入一点。

**:**

*   不提供实时保证:它使用`Object.wait(long) `方法调度任务
*   只有一个后台线程，所以任务按顺序运行，长时间运行的任务会延迟其他任务
*   在`TimerTask`中抛出的运行时异常会杀死唯一可用的线程，从而杀死`Timer`

**:**

*   可以配置任意数量的线程
*   可以利用所有可用的 CPU 内核
*   捕捉运行时异常，并让我们在需要时处理它们(通过覆盖`ThreadPoolExecutor`中的`afterExecute`方法)
*   取消引发异常的任务，同时让其他任务继续运行
*   依靠操作系统调度系统跟踪时区、延迟、太阳时等。
*   如果我们需要多个任务之间的协调，比如等待提交的所有任务完成，则提供协作 API
*   为线程生命周期的管理提供了更好的 API

现在的选择显而易见，对吧？

## 7.`Future`和`ScheduledFuture`的区别

在我们的代码示例中，我们可以看到`ScheduledThreadPoolExecutor`返回了特定类型的`Future` : `ScheduledFuture`。

*ScheduledFuture* 扩展了`Future`和`Delayed`接口，从而继承了返回与当前任务相关的剩余延迟的附加方法`getDelay`。它由`RunnableScheduledFuture`扩展，增加了检查任务是否是周期性的方法。

**`ScheduledThreadPoolExecutor`通过内部类`ScheduledFutureTask`实现所有这些构造，并使用它们来控制任务生命周期。**

## 8.结论

在本教程中，我们尝试了不同的框架来并行启动线程和运行任务。

然后，我们深入探讨了`Timer`和`ScheduledThreadPoolExecutor.`之间的差异

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221026205735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-simple)