# java.util.concurrent 概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-util-concurrent>

## 1。概述

`java.util.concurrent`包提供了创建并发应用程序的工具。

在本文中，我们将对整个包做一个概述。

## 2。主要部件

`java.util.concurrent`包含了太多的特性，无法在一篇文章中讨论。在本文中，我们将主要关注这个包中一些最有用的实用程序，比如:

*   `Executor`
*   `ExecutorService`
*   `ScheduledExecutorService`
*   `Future`
*   `CountDownLatch`
*   `CyclicBarrier`
*   `Semaphore`
*   `ThreadFactory`
*   `BlockingQueue`
*   `DelayQueue`
*   `Locks`
*   `Phaser`

你也可以在这里找到许多关于个别课程的文章。

### 2.1。`Executor`

**`[Executor](https://web.archive.org/web/20221209012414/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executor.html)`是表示执行规定任务的对象的接口。**

如果任务应该在新的或当前的线程上运行，这取决于特定的实现(从哪里发起调用)。因此，使用这个接口，我们可以将任务执行流从实际的任务执行机制中分离出来。

这里需要注意的一点是`Executor`并没有严格要求任务执行是异步的。在最简单的情况下，执行器可以在调用线程中立即调用提交的任务。

我们需要创建一个 invoker 来创建 executor 实例:

```
public class Invoker implements Executor {
    @Override
    public void execute(Runnable r) {
        r.run();
    }
}
```

现在，我们可以使用这个调用程序来执行任务。

```
public void execute() {
    Executor executor = new Invoker();
    executor.execute( () -> {
        // task to be performed
    });
}
```

这里要注意的一点是，如果执行者不能接受任务进行执行，就会抛出`[RejectedExecutionException](https://web.archive.org/web/20221209012414/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/RejectedExecutionException.html)`。

### 2.2。`ExecutorService`

`ExecutorService`是异步处理的完整解决方案。它管理内存中的队列，并根据线程可用性调度提交的任务。

为了使用`ExecutorService,` ，我们需要创建一个`Runnable`类。

```
public class Task implements Runnable {
    @Override
    public void run() {
        // task details
    }
}
```

现在我们可以创建`ExecutorService`实例并分配这个任务。在创建时，我们需要指定线程池的大小。

```
ExecutorService executor = Executors.newFixedThreadPool(10);
```

如果我们想创建一个单线程的`ExecutorService`实例，我们可以使用 **`newSingleThreadExecutor(ThreadFactory threadFactory)`** 来创建实例。

一旦创建了执行程序，我们就可以使用它来提交任务。

```
public void execute() { 
    executor.submit(new Task()); 
}
```

我们还可以在提交任务时创建`Runnable`实例。

```
executor.submit(() -> {
    new Task();
});
```

它还提供了两种现成的执行终止方法。第一个是`shutdown()`；它会一直等到所有提交的任务执行完毕。另一种方法是`shutdownNow()`，它试图终止所有正在执行的任务，并暂停正在等待的任务的处理。

还有另一种方法`awaitTermination(long timeout, TimeUnit unit)`，在触发关机事件或发生执行超时，或者执行线程本身中断后，强制阻塞，直到所有任务执行完毕。

```
try {
    executor.awaitTermination( 20l, TimeUnit.NANOSECONDS );
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

### 2.3。`ScheduledExecutorService`

`ScheduledExecutorService`是一个类似于`ExecutorService,`的接口，但是它可以周期性地执行任务。

**`Executor and ExecutorService`的方法是当场调度，不引入任何人为延迟。**零或任何负值表示请求需要立即执行。

我们可以同时使用`Runnable`和`Callable`接口来定义任务。

```
public void execute() {
    ScheduledExecutorService executorService
      = Executors.newSingleThreadScheduledExecutor();

    Future<String> future = executorService.schedule(() -> {
        // ...
        return "Hello world";
    }, 1, TimeUnit.SECONDS);

    ScheduledFuture<?> scheduledFuture = executorService.schedule(() -> {
        // ...
    }, 1, TimeUnit.SECONDS);

    executorService.shutdown();
}
```

`ScheduledExecutorService`也可以在给定的固定延迟后调度任务**:**

```
executorService.scheduleAtFixedRate(() -> {
    // ...
}, 1, 10, TimeUnit.SECONDS);

executorService.scheduleWithFixedDelay(() -> {
    // ...
}, 1, 10, TimeUnit.SECONDS);
```

这里，`**scheduleAtFixedRate( Runnable command, long initialDelay, long period, TimeUnit unit )**`方法创建并执行一个周期性动作，该动作首先在提供的初始延迟后被调用，然后在给定的时间段内被调用，直到服务实例关闭。

`**scheduleWithFixedDelay( Runnable command, long initialDelay, long delay, TimeUnit unit )**`方法创建并执行一个周期性动作，该动作在提供的初始延迟后首先被调用，并在执行动作终止和下一个动作调用之间的给定延迟内重复调用。

### 2.4。`Future`

**`Future`用来表示异步操作的结果。**它提供了检查异步操作是否完成、获取计算结果等方法。

更重要的是，`cancel(boolean mayInterruptIfRunning)` API 取消操作，释放正在执行的线程。如果`mayInterruptIfRunning`的值为真，执行任务的线程将立即终止。

否则，正在进行的任务将被允许完成。

我们可以使用下面的代码片段来创建一个未来实例:

```
public void invoke() {
    ExecutorService executorService = Executors.newFixedThreadPool(10);

    Future<String> future = executorService.submit(() -> {
        // ...
        Thread.sleep(10000l);
        return "Hello world";
    });
}
```

我们可以使用下面的代码片段来检查未来的结果是否准备好了，如果计算完成了，就获取数据:

```
if (future.isDone() && !future.isCancelled()) {
    try {
        str = future.get();
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

我们还可以为给定的操作指定超时。如果任务花费的时间超过这个时间，就会抛出一个`TimeoutException`:

```
try {
    future.get(10, TimeUnit.SECONDS);
} catch (InterruptedException | ExecutionException | TimeoutException e) {
    e.printStackTrace();
}
```

### 2.5。`CountDownLatch`

`CountDownLatch`(在`JDK 5`中引入)是一个实用程序类，它阻塞一组线程，直到某个操作完成。

用一个`counter(Integer`类型初始化一个`CountDownLatch`；当从属线程完成执行时，此计数器递减。但是一旦计数器达到零，其他线程就会被释放。

点击可以了解更多`CountDownLatch` [。](/web/20221209012414/https://www.baeldung.com/java-countdown-latch)

### 2.6。`CyclicBarrier`

`CyclicBarrier`的工作方式与`CountDownLatch`几乎相同，只是我们可以重用它。与`CountDownLatch`不同，它允许多个线程在调用最终任务之前使用`await()`方法(称为屏障条件)相互等待。

我们需要创建一个`Runnable`任务实例来启动障碍条件:

```
public class Task implements Runnable {

    private CyclicBarrier barrier;

    public Task(CyclicBarrier barrier) {
        this.barrier = barrier;
    }

    @Override
    public void run() {
        try {
            LOG.info(Thread.currentThread().getName() + 
              " is waiting");
            barrier.await();
            LOG.info(Thread.currentThread().getName() + 
              " is released");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }

}
```

现在我们可以调用一些线程来争用屏障条件:

```
public void start() {

    CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
        // ...
        LOG.info("All previous tasks are completed");
    });

    Thread t1 = new Thread(new Task(cyclicBarrier), "T1"); 
    Thread t2 = new Thread(new Task(cyclicBarrier), "T2"); 
    Thread t3 = new Thread(new Task(cyclicBarrier), "T3"); 

    if (!cyclicBarrier.isBroken()) { 
        t1.start(); 
        t2.start(); 
        t3.start(); 
    }
}
```

这里，`isBroken()`方法检查在执行期间是否有任何线程被中断。我们应该总是在执行实际流程之前执行这项检查。

### 2.7。`Semaphore`

`Semaphore`用于阻止线程级访问物理或逻辑资源的某个部分。一个[信号量](/web/20221209012414/https://www.baeldung.com/cs/semaphore)包含一组许可；每当一个线程试图进入临界区时，它需要检查信号量是否有许可。

**如果没有许可证(通过`tryAcquire()`)，不允许线程跳转到临界区；然而，如果许可可用，则准许访问，并且许可计数器减少。**

一旦执行线程释放了临界区，许可计数器再次增加(通过`release()`方法完成)。

我们可以通过使用`tryAcquire(long timeout, TimeUnit unit)`方法来指定获取访问的超时时间。

我们还可以检查可用许可的数量或者等待获取信号量的线程的数量。

以下代码片段可用于实现信号量:

```
static Semaphore semaphore = new Semaphore(10);

public void execute() throws InterruptedException {

    LOG.info("Available permit : " + semaphore.availablePermits());
    LOG.info("Number of threads waiting to acquire: " + 
      semaphore.getQueueLength());

    if (semaphore.tryAcquire()) {
        try {
            // ...
        }
        finally {
            semaphore.release();
        }
    }

}
```

我们可以使用`Semaphore`实现类似于`Mutex`的数据结构。更多关于这个[的细节可以在这里找到。](/web/20221209012414/https://www.baeldung.com/java-semaphore)

### 2.8。`ThreadFactory`

顾名思义，`ThreadFactory`充当一个线程(不存在的)池，按需创建一个新线程。它消除了实现高效线程创建机制所需的大量样板代码。

我们可以定义一个`ThreadFactory`:

```
public class BaeldungThreadFactory implements ThreadFactory {
    private int threadId;
    private String name;

    public BaeldungThreadFactory(String name) {
        threadId = 1;
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r, name + "-Thread_" + threadId);
        LOG.info("created new thread with id : " + threadId +
            " and name : " + t.getName());
        threadId++;
        return t;
    }
}
```

我们可以使用这个`newThread(Runnable r)`方法在运行时创建一个新线程:

```
BaeldungThreadFactory factory = new BaeldungThreadFactory( 
    "BaeldungThreadFactory");
for (int i = 0; i < 10; i++) { 
    Thread t = factory.newThread(new Task());
    t.start(); 
}
```

### 2.9。 `BlockingQueue`

在异步编程中，最常见的集成模式之一是[生产者-消费者模式](https://web.archive.org/web/20221209012414/https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)。`java.util.concurrent`包带有一个被称为`BlockingQueue`的数据结构，这在这些异步场景中非常有用。

更多信息和工作示例可在[这里](/web/20221209012414/https://www.baeldung.com/java-blocking-queue)获得。

### 2.10。`DelayQueue`

`DelayQueue`是一个无限大小的元素阻塞队列，其中一个元素只有在其到期时间(称为用户定义的延迟)结束时才能被取出。因此，最顶端的元素(`head`)将具有最大的延迟量，它将最后被轮询。

更多信息和工作示例可在[这里](/web/20221209012414/https://www.baeldung.com/java-delay-queue)获得。

### 2.11。`Locks`

毫不奇怪，`Lock`是一个阻止其他线程访问特定代码段的工具，除了当前正在执行它的线程。

锁和同步块的主要区别在于，同步块完全包含在方法中；但是，我们可以在不同的方法中使用 Lock API 的 Lock()和 unlock()操作。

更多信息和工作示例可在[这里](/web/20221209012414/https://www.baeldung.com/java-concurrent-locks)获得。

### 2.12。`Phaser`

`Phaser`是一个比`CyclicBarrier`和`CountDownLatch`更灵活的解决方案——用来作为一个可重用的屏障，在继续执行之前，动态数量的线程需要在其上等待。我们可以协调多个执行阶段，为每个程序阶段重用一个`Phaser`实例。

更多信息和工作示例可在[这里](/web/20221209012414/https://www.baeldung.com/java-phaser)获得。

## 3。结论

在这篇高层次的综述文章中，我们重点关注了`java.util.concurrent`包中不同的实用程序。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221209012414/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)