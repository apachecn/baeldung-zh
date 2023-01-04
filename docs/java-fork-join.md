# Java 中的 Fork/Join 框架指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-fork-join>

## 1。概述

Java 7 引入了 fork/join 框架。它提供了一些工具，通过尝试使用所有可用的处理器内核来帮助加速并行处理。它通过分而治之的方法来实现这个目标。

实际上，这意味着**框架首先“分叉”，**递归地将任务分解成更小的独立子任务，直到它们简单到足以异步运行。

之后，**“加入”部分开始。**所有子任务的结果被递归连接成一个结果。在任务返回 void 的情况下，程序只是等待，直到每个子任务都运行。

为了提供有效的并行执行，fork/join 框架使用了一个叫做`ForkJoinPool`的线程池。这个池管理类型为`ForkJoinWorkerThread`的工作线程。

## 2。 `ForkJoinPool`

`ForkJoinPool`是框架的核心。它是`[ExecutorService](/web/20221227031719/https://www.baeldung.com/java-executor-service-tutorial)`的一个实现，管理工作线程并为我们提供工具来获取关于线程池状态和性能的信息。

工作者线程一次只能执行一个任务，但是`ForkJoinPool`不会为每个单独的子任务创建单独的线程。相反，线程池中的每个线程都有自己的双端队列(或称[队列](https://web.archive.org/web/20221227031719/https://en.wikipedia.org/wiki/Double-ended_queue)，读作“deck”)来存储任务。

这种架构对于借助**工作窃取算法平衡线程的工作负载至关重要。**

### 2.1。偷工减料算法

简单地说，自由线程试图从繁忙线程的队列中“窃取”工作。

默认情况下，工作线程从自己的队列头获取任务。当它为空时，线程从另一个繁忙线程的队列尾部或全局入口队列中获取任务，因为这是最大的工作可能所在的位置。

这种方法最小化了线程竞争任务的可能性。它还减少了线程寻找工作的次数，因为它首先处理最大的可用工作块。

### 2.2。`ForkJoinPool` 实例化

在 Java 8 中，访问`ForkJoinPool` 实例的最便捷方式是使用它的静态方法`[commonPool](https://web.archive.org/web/20221227031719/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinPool.html#commonPool())()`。这将提供对公共池的引用，公共池是每个`ForkJoinTask`的默认线程池。

根据 [Oracle 的文档](https://web.archive.org/web/20221227031719/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)，使用预定义的公共池减少了资源消耗，因为这不鼓励为每个任务创建单独的线程池。

```
ForkJoinPool commonPool = ForkJoinPool.commonPool();
```

在 Java 7 中，我们可以通过创建一个`ForkJoinPool`并将其分配给一个实用程序类的`public static`字段来实现相同的行为:

```
public static ForkJoinPool forkJoinPool = new ForkJoinPool(2);
```

现在我们可以轻松访问它:

```
ForkJoinPool forkJoinPool = PoolUtil.forkJoinPool;
```

使用`ForkJoinPool’s` 构造函数，我们可以创建一个定制的线程池，具有特定级别的并行性、线程工厂和异常处理程序。这里，池的并行度为 2。这意味着池将使用两个处理器核心。

## 3。 `ForkJoinTask<V>`

`ForkJoinTask` 是在`ForkJoinPool`中执行的任务的基本类型。实际上，它的两个子类之一应该被扩展:用于`void` 任务的`RecursiveAction`和用于返回值的任务的`RecursiveTask<V>`。它们都有一个抽象方法`compute()` ，在其中定义了任务的逻辑。

### 3.1。 `RecursiveAction`

在下面的例子中，我们使用一个名为`workload`的`String`来表示要处理的工作单元。出于演示的目的，这个任务是无意义的:它只是大写输入并记录它。

为了演示框架的分叉行为，**如果`workload` `.length()`大于指定的阈值** ，那么这个例子使用`createSubtask()`方法分割任务。

该字符串被递归地分成子字符串，创建基于这些子字符串的`CustomRecursiveTask` 实例。

结果，该方法返回一个列表`<customrecursiveaction>`。

使用`invokeAll()`方法将列表提交给`ForkJoinPool` :

```
public class CustomRecursiveAction extends RecursiveAction {

    private String workload = "";
    private static final int THRESHOLD = 4;

    private static Logger logger = 
      Logger.getAnonymousLogger();

    public CustomRecursiveAction(String workload) {
        this.workload = workload;
    }

    @Override
    protected void compute() {
        if (workload.length() > THRESHOLD) {
            ForkJoinTask.invokeAll(createSubtasks());
        } else {
           processing(workload);
        }
    }

    private List<CustomRecursiveAction> createSubtasks() {
        List<CustomRecursiveAction> subtasks = new ArrayList<>();

        String partOne = workload.substring(0, workload.length() / 2);
        String partTwo = workload.substring(workload.length() / 2, workload.length());

        subtasks.add(new CustomRecursiveAction(partOne));
        subtasks.add(new CustomRecursiveAction(partTwo));

        return subtasks;
    }

    private void processing(String work) {
        String result = work.toUpperCase();
        logger.info("This result - (" + result + ") - was processed by " 
          + Thread.currentThread().getName());
    }
}
```

我们可以使用这个模式来开发我们自己的`RecursiveAction`类。为此，我们创建一个表示工作总量的对象，选择一个合适的阈值，定义一个划分工作的方法，并定义一个完成工作的方法。

### 3.2。 `RecursiveTask<V>`

对于返回值的任务，这里的逻辑是类似的。

不同之处在于，每个子任务的结果都统一在一个结果中:

```
public class CustomRecursiveTask extends RecursiveTask<Integer> {
    private int[] arr;

    private static final int THRESHOLD = 20;

    public CustomRecursiveTask(int[] arr) {
        this.arr = arr;
    }

    @Override
    protected Integer compute() {
        if (arr.length > THRESHOLD) {
            return ForkJoinTask.invokeAll(createSubtasks())
              .stream()
              .mapToInt(ForkJoinTask::join)
              .sum();
        } else {
            return processing(arr);
        }
    }

    private Collection<CustomRecursiveTask> createSubtasks() {
        List<CustomRecursiveTask> dividedTasks = new ArrayList<>();
        dividedTasks.add(new CustomRecursiveTask(
          Arrays.copyOfRange(arr, 0, arr.length / 2)));
        dividedTasks.add(new CustomRecursiveTask(
          Arrays.copyOfRange(arr, arr.length / 2, arr.length)));
        return dividedTasks;
    }

    private Integer processing(int[] arr) {
        return Arrays.stream(arr)
          .filter(a -> a > 10 && a < 27)
          .map(a -> a * 10)
          .sum();
    }
}
```

在这个例子中，我们使用存储在`CustomRecursiveTask` 类的`arr` 字段中的数组来表示工作。`createSubtasks()`方法递归地将任务分成更小的工作块，直到每一块都小于阈值。然后，`invokeAll()` 方法将子任务提交给公共池，并返回一个`[Future](https://web.archive.org/web/20221227031719/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html)`列表。

为了触发执行，每个子任务都要调用`join()`方法。

我们已经使用 Java 8 的`[Stream API](https://web.archive.org/web/20221227031719/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html)`实现了这一点。我们使用`sum()`方法来表示将子结果组合成最终结果。

## 4。向`ForkJoinPool`提交任务

我们可以使用一些方法向线程池提交任务。

让我们从 **`submit()`** 或`**execute()**` 方法开始(它们的用例是相同的):

```
forkJoinPool.execute(customRecursiveTask);
int result = customRecursiveTask.join();
```

`**invoke()**` 方法分叉任务并等待结果，不需要任何手动连接:

```
int result = forkJoinPool.invoke(customRecursiveTask);
```

**`invokeAll()`** 方法是向`ForkJoinPool`提交一系列`ForkJoinTasks`的最便捷方式。它将任务作为参数(两个任务，var args 或一个集合)，分叉，然后按照产生的顺序返回一个`Future`对象的集合。

或者，我们可以使用单独的 **`fork()` 和`join()`** 方法。`fork()`方法向池提交一个任务，但不触发它的执行。为此，我们必须使用`join()` 方法。

在`RecursiveAction`的情况下，`join()` 只返回 `null`；对于`RecursiveTask<V>`，返回任务执行的结果:

```
customRecursiveTaskFirst.fork();
result = customRecursiveTaskLast.join();
```

这里我们使用了`invokeAll()` 方法向池提交一系列子任务。我们可以用`fork()`和`join()`做同样的工作，尽管这会影响结果的排序。

为了避免混淆，使用`invokeAll()`方法向`ForkJoinPool`提交多个任务通常是个好主意。

## 5。结论

使用 fork/join 框架可以加快大型任务的处理速度，但是为了实现这一结果，我们应该遵循一些准则:

*   使用尽可能少的线程池。在大多数情况下，最好的决定是每个应用程序或系统使用一个线程池。
*   **如果不需要特定的调优，使用默认的公共线程池**。
*   **使用合理的阈值**将`ForkJoinTask`分割成子任务。
*   **避开** **`ForkJoinTasks`中的任何阻挡。**

本文中使用的例子可以在[链接的 GitHub 库](https://web.archive.org/web/20221227031719/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)中找到。