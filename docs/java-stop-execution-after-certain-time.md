# 在 Java 中如何在一定时间后停止执行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stop-execution-after-certain-time>

## 1.概观

在本文中，我们将学习如何在特定时间后结束长时间运行的执行。我们将探索这个问题的各种解决方案。此外，我们将涵盖他们的一些陷阱。

## 2.使用循环

假设我们正在一个循环中处理一堆项目，例如电子商务应用程序中产品项目的一些细节，但是可能没有必要完成所有项目。

事实上，我们希望只处理到某个时间，在那之后，我们希望停止执行并显示到那个时间为止列表已经处理的内容。

让我们看一个简单的例子:

```
long start = System.currentTimeMillis();
long end = start + 30 * 1000;
while (System.currentTimeMillis() < end) {
    // Some expensive operation on the item.
}
```

这里，如果时间超过了 30 秒的限制，循环将会中断。上述解决方案中有一些值得注意的地方:

*   低精度:**循环可以运行超过强制时间限制**。这将取决于每次迭代可能花费的时间。例如，如果每次迭代可能需要 7 秒，那么总时间可能会达到 35 秒，这比 30 秒的期望时间限制长了大约 17%
*   阻塞:**在主线程中进行这样的处理可能不是一个好主意，因为它会阻塞它很长时间**。相反，这些操作应该从主线程中分离出来

在下一节中，我们将讨论基于中断的方法如何消除这些限制。

## 3.使用中断机制

这里，我们将使用一个单独的线程来执行长时间运行的操作。主线程将在超时时向工作线程发送一个中断信号。

如果工作线程还活着，它会捕捉信号并停止执行。如果工作线程在超时前完成，对工作线程没有影响。

让我们来看看工作线程:

```
class LongRunningTask implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < Long.MAX_VALUE; i++) {
            if(Thread.interrupted()) {
                return;
            }
        }
    }
}
```

这里，通过`Long.MAX_VALUE` 的 for 循环模拟一个长时间运行的操作。除此之外，还可以有其他操作。**检查中断标志很重要，因为并非所有操作都是可中断的**。因此，在这些情况下，我们应该手动检查标志。

此外，我们应该在每次迭代中检查这个标志，以确保线程最多在一次迭代的延迟内停止执行自己。

接下来，我们将讨论发送中断信号的三种不同机制。

### 3.1.使用`Timer`

或者，我们可以创建一个 [`TimerTask`](/web/20220929100635/https://www.baeldung.com/java-timer-and-timertask) 来在超时时中断工作线程:

```
class TimeOutTask extends TimerTask {
    private Thread thread;
    private Timer timer;

    public TimeOutTask(Thread thread, Timer timer) {
        this.thread = thread;
        this.timer = timer;
    }

    @Override
    public void run() {
        if(thread != null && thread.isAlive()) {
            thread.interrupt();
            timer.cancel();
        }
    }
}
```

这里，我们定义了一个`TimerTask` ,它在创建时获取一个工作线程。它将在调用其`run`方法时**中断工作线程。 [`Timer`](/web/20220929100635/https://www.baeldung.com/java-timer-and-timertask) 会在三秒延迟后触发`TimerTask `:**

```
Thread thread = new Thread(new LongRunningTask());
thread.start();

Timer timer = new Timer();
TimeOutTask timeOutTask = new TimeOutTask(thread, timer);
timer.schedule(timeOutTask, 3000);
```

### 3.2.使用方法`Future#get`

我们也可以使用 [`Future`](/web/20220929100635/https://www.baeldung.com/java-future) 的`get`方法来代替使用`Timer`:

```
ExecutorService executor = Executors.newSingleThreadExecutor();
Future future = executor.submit(new LongRunningTask());
try {
    future.get(7, TimeUnit.SECONDS);
} catch (TimeoutException e) {
    future.cancel(true);
} catch (Exception e) {
    // handle other exceptions
} finally {
    executor.shutdownNow();
}
```

这里，我们使用了`ExecutorService`来提交返回`Future`实例的工作线程，其`get`方法将阻塞主线程直到指定的时间。它会在指定的超时后引发一个`TimeoutException` 。在`catch`块中，我们通过调用`F` `uture`对象上的`cancel`方法来中断工作线程。

与前一种方法相比，这种方法的主要优点是它**使用一个池来管理线程，而`Timer`只使用一个线程(没有池)**。

### 3.3.使用`ScheduledExcecutorSercvice`

我们也可以使用 [`ScheduledExecutorService`](/web/20220929100635/https://www.baeldung.com/java-executor-service-tutorial#ScheduledExecutorService) 来中断任务。这个类是一个`ExecutorService`的扩展，提供相同的功能，并增加了几个处理执行调度的方法。这可以在设定的时间单位的一定延迟之后执行给定的任务:

```
ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
Future future = executor.submit(new LongRunningTask());
Runnable cancelTask = () -> future.cancel(true);

executor.schedule(cancelTask, 3000, TimeUnit.MILLISECONDS);
executor.shutdown();
```

这里，我们用方法`newScheduledThreadPool`创建了一个大小为 2 的调度线程池。`ScheduledExecutorService#` `schedule`法取一个 [`Runnable`](/web/20220929100635/https://www.baeldung.com/java-runnable-vs-extending-thread) ，一个延时值，以及延时的单位。

上面的程序将任务安排在提交后三秒钟执行。该任务将取消原来的长时间运行的任务。

注意，与前面的方法不同，我们没有通过调用`Future#get`方法来阻塞主线程。因此，**是上述所有方法**中最优选的方法。

## 4.有保证书吗？

**不能保证一定时间**后执行停止。主要原因是不是所有的阻塞方法都是可中断的。事实上，只有少数定义明确的方法是可中断的。因此，**如果一个线程被中断，并且设置了一个标志，那么在它到达这些可中断方法之一之前，不会发生任何事情**。

例如，`read`和`write`方法只有在使用 [`InterruptibleChannel`](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/InterruptibleChannel.html) 创建的流上被调用时才是可中断的。 [`BufferedReader`](/web/20220929100635/https://www.baeldung.com/java-buffered-reader) 不是一个`InterruptibleChannel`。因此，如果线程使用它来读取文件，那么在这个线程上调用在`read`方法中被阻塞的`interrupt()`没有任何效果。

然而，我们可以在循环中的每次读取后明确检查中断标志。这将为延迟一段时间停止线程提供一个合理的保证。但是，这并不能保证在一段严格的时间后停止线程，因为我们不知道一个读操作会花费多少时间。

另一方面，`Object` 类的`wait`方法是可中断的。因此，`wait`方法中阻塞的线程将在设置中断标志后立即抛出一个`InterruptedException`。

我们可以通过在方法签名中寻找`throws` `InterruptedException`来识别阻塞方法。

一个重要的建议是**避免使用被弃用的 [`Thread.stop()`](https://web.archive.org/web/20220929100635/https://docs.oracle.com/javase/7/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html) 方法。**停止线程会使其解锁所有已锁定的监视器。发生这种情况是因为 [`ThreadDeath`](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ThreadDeath.html) 异常在堆栈中向上传播。

如果先前受这些监视器保护的任何对象处于不一致状态，则不一致的对象对其他线程变得可见。这可能导致难以检测和推理的任意行为。

## 5.中断设计

在上一节中，我们强调了使用可中断方法来尽快停止执行的重要性。因此，我们的代码需要从设计的角度考虑这种期望。

假设我们有一个长时间运行的任务要执行，我们需要确保它不会花费超过指定的时间。此外，假设任务可以分解成单独的步骤。

让我们为任务步骤创建一个类:

```
class Step {
    private static int MAX = Integer.MAX_VALUE/2;
    int number;

    public Step(int number) {
        this.number = number;
    }

    public void perform() throws InterruptedException {
        Random rnd = new Random();
        int target = rnd.nextInt(MAX);
        while (rnd.nextInt(MAX) != target) {
            if (Thread.interrupted()) {
                throw new InterruptedException();
            }
        }
    }
}
```

这里，`Step#perform`方法试图在每次迭代中寻找一个目标随机整数，同时请求标记。当标志被激活时，该方法抛出一个`InterruptedException`。

现在，让我们定义将执行所有步骤的任务:

```
public class SteppedTask implements Runnable {
    private List<Step> steps;

    public SteppedTask(List<Step> steps) {
        this.steps = steps;
    }

    @Override
    public void run() {
        for (Step step : steps) {
            try {
                step.perform();
            } catch (InterruptedException e) {
                // handle interruption exception
                return;
            }
        }
    }
}
```

这里，`SteppedTask`有一个要执行的步骤列表。for 循环执行每一步，并在任务发生时处理用于停止任务的`InterruptedException`。

最后，让我们看一个使用可中断任务的例子:

```
List<Step> steps = Stream.of(
  new Step(1),
  new Step(2),
  new Step(3),
  new Step(4))
.collect(Collectors.toList());

Thread thread = new Thread(new SteppedTask(steps));
thread.start();

Timer timer = new Timer();
TimeOutTask timeOutTask = new TimeOutTask(thread, timer);
timer.schedule(timeOutTask, 10000);
```

首先，我们用四个步骤创建一个`SteppedTask`。其次，我们使用线程运行任务。最后，我们使用定时器和超时任务在十秒钟后中断线程。

通过这种设计，我们可以确保我们的长时间运行的任务可以在执行任何步骤时被中断。正如我们以前看到的，缺点是不能保证它会在指定的确切时间停止，但肯定比不可中断的任务好。

## 6.结论

在本教程中，我们学习了在给定时间后停止执行的各种技术，以及每种技术的优缺点。完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220929100635/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-2)