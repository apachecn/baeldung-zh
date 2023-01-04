# java.util.concurrent.Future 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-future>

## 1。概述

在本教程中，我们将学习`[Future](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html)`。这是一个从 Java 1.5 就存在的接口，在处理异步调用和并发处理时非常有用。

## 2。创造`Futures`

简单地说，`Future`类表示异步计算的未来结果。这个结果最终会在处理完成后出现在`Future`中。

让我们看看如何编写创建和返回一个`Future`实例的方法。

长时间运行的方法非常适合异步处理和`Future` 接口，因为我们可以在等待封装在`Future` 中的任务完成时执行其他流程。

利用`Future`异步特性的一些操作示例如下:

*   计算密集型过程(数学和科学计算)
*   操作大型数据结构(大数据)
*   远程方法调用(下载文件、HTML 废弃、web 服务)

### 2.1。用`FutureTask` 实现`Futures`

对于我们的例子，我们将创建一个非常简单的类来计算一个`Integer`的平方。这肯定不符合长时间运行的方法类别，但是我们将对它进行一个`Thread.sleep()`调用，以便它在完成之前持续 1 秒钟:

```java
public class SquareCalculator {    

    private ExecutorService executor 
      = Executors.newSingleThreadExecutor();

    public Future<Integer> calculate(Integer input) {        
        return executor.submit(() -> {
            Thread.sleep(1000);
            return input * input;
        });
    }
}
```

实际执行计算的代码包含在`call()`方法中，并作为 lambda 表达式提供。我们可以看到，除了前面提到的`sleep()` 调用之外，并没有什么特别之处。

当我们注意到`[Callable](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Callable.html)` 和`[ExecutorService](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html)`的用法时，事情就变得更有趣了。

`Callable` 是表示返回结果的任务的接口，有一个`call()`方法。在这里，我们使用 lambda 表达式创建了一个实例。

创建一个`Callable`的实例不会带我们去任何地方；我们仍然需要将这个实例传递给一个执行程序，它将负责在一个新线程中启动任务，并将有价值的`Future`对象返回给我们。这就是`ExecutorService`的用武之地。

有几种方法可以访问一个`ExecutorService`实例，其中大多数是由实用程序类`[Executors](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html)‘`静态工厂方法提供的。在这个例子中，我们使用了基本的`newSingleThreadExecutor()`，它给了我们一个能够一次处理一个线程的`ExecutorService`。

一旦我们有了一个`ExecutorService` 对象，我们只需要调用`submit(),`并传递我们的`Callable` 作为参数。然后`submit()` 将启动任务并返回一个`[FutureTask](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/FutureTask.html)` 对象，这是`Future`接口的一个实现。

## 3。消耗`Futures`

到目前为止，我们已经学习了如何创建一个`Future`的实例。

在这一节中，我们将通过探索属于`Future`的 API 的所有方法来学习如何使用这个实例。

### 3.1。使用`isDone()`和`get()` 获得结果

现在我们需要调用`calculate(),`并使用返回的`Future`来获得结果`Integer`。来自`Future` API 的两个方法将帮助我们完成这项任务。

告诉我们执行者是否已经完成了任务的处理。如果任务完成，它将返回`true;`，否则，它将返回`false`。

从计算返回实际结果的方法是`[Future.get()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html#get())`。我们可以看到，这个方法会阻塞执行，直到任务完成。然而，在我们的例子中这不是问题，因为我们将通过调用`isDone()`来检查任务是否完成。

通过使用这两种方法，我们可以在等待主任务完成的同时运行其他代码:

```java
Future<Integer> future = new SquareCalculator().calculate(10);

while(!future.isDone()) {
    System.out.println("Calculating...");
    Thread.sleep(300);
}

Integer result = future.get();
```

在这个例子中，我们将在输出上写一条简单的消息，让用户知道程序正在执行计算。

方法`get()` 将阻塞执行，直到任务完成。同样，这不会是一个问题，因为在我们的例子中，`get()`只会在确保任务完成后被调用。所以在这种情况下，`future.get()`总是会立即返回。

值得一提的是，`get()`有一个重载版本，以超时和一个 [`TimeUnit`](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/TimeUnit.html) 作为参数:

```java
Integer result = future.get(500, TimeUnit.MILLISECONDS);
```

`[get(long, TimeUnit)](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html#get(long,java.util.concurrent.TimeUnit))`和`[get()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html#get())`的区别在于，如果任务在指定的超时期限内没有返回，前者将抛出一个`[TimeoutException](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/TimeoutException.html)`。

### 3.2。用`cancel()` 取消`Future`

假设我们触发了一个任务，但是由于某种原因，我们不再关心结果了。我们可以使用`[Future.cancel(boolean)](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html#cancel(boolean))`来告诉执行器停止操作并中断其底层线程:

```java
Future<Integer> future = new SquareCalculator().calculate(4);

boolean canceled = future.cancel(true);
```

上面代码中的`Future,` 实例将永远不会完成它的操作。事实上，如果我们试图从那个实例调用`get()`，在调用`cancel()`之后，结果将是一个`[CancellationException](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CancellationException.html)`。`[Future.isCancelled()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html#isCancelled())`会告诉我们`Future`是否已经被取消。这对于避免出现`CancellationException`非常有用。

对`cancel()`的调用也有可能失败。在这种情况下，返回值将是`false`。需要注意的是，`cancel()`接受一个`boolean`值作为参数。这控制了执行任务的线程是否应该被中断。

## 4。更多多线程与`Thread`池

我们当前的`ExecutorService`是单线程的，因为它是通过[executors . newsinglethreadexecutor](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newSingleThreadExecutor())获得的。为了突出显示这个单线程，让我们同时触发两个计算:

```java
SquareCalculator squareCalculator = new SquareCalculator();

Future<Integer> future1 = squareCalculator.calculate(10);
Future<Integer> future2 = squareCalculator.calculate(100);

while (!(future1.isDone() && future2.isDone())) {
    System.out.println(
      String.format(
        "future1 is %s and future2 is %s", 
        future1.isDone() ? "done" : "not done", 
        future2.isDone() ? "done" : "not done"
      )
    );
    Thread.sleep(300);
}

Integer result1 = future1.get();
Integer result2 = future2.get();

System.out.println(result1 + " and " + result2);

squareCalculator.shutdown();
```

现在让我们分析这段代码的输出:

```java
calculating square for: 10
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
calculating square for: 100
future1 is done and future2 is not done
future1 is done and future2 is not done
future1 is done and future2 is not done
100 and 10000
```

很明显，这个过程不是平行的。我们可以看到，只有在第一个任务完成后，第二个任务才开始，这使得整个过程大约需要 2 秒钟才能完成。

为了让我们的程序真正多线程化，我们应该使用另一种风格的`ExecutorService`。让我们看看，如果我们使用工厂方法`[Executors.newFixedThreadPool()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newFixedThreadPool(int))`提供的线程池，我们的示例的行为会如何变化:

```java
public class SquareCalculator {

    private ExecutorService executor = Executors.newFixedThreadPool(2);

    //...
}
```

通过对我们的`SquareCalculator`类做一个简单的改变，我们现在有了一个能够同时使用两个线程的执行器。

如果我们再次运行完全相同的客户端代码，我们将得到以下输出:

```java
calculating square for: 10
calculating square for: 100
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
100 and 10000
```

现在看起来好多了。我们可以看到，这两个任务同时开始和结束运行，整个过程大约需要 1 秒钟完成。

还有其他工厂方法可以用来创建线程池，比如`[Executors.newCachedThreadPool()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newCachedThreadPool()),` 可以重用之前使用的可用的`Thread`，以及`[Executors.newScheduledThreadPool()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html#newScheduledThreadPool(int)),` 可以调度命令在给定的延迟`.` 后运行

想了解更多关于`ExecutorService`的信息，请阅读我们的[文章](/web/20221007090011/https://www.baeldung.com/java-executor-service-tutorial)专门讨论这个主题。

## 5。`ForkJoinTask`概述

`[ForkJoinTask](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinTask.html)`是实现`Future,`的抽象类，能够运行由`[ForkJoinPool](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)`中少量实际线程托管的大量任务。

在这一节中，我们将快速介绍一下`ForkJoinPool`的主要特征。关于这个主题的全面指南，请查看我们的[Java](/web/20221007090011/https://www.baeldung.com/java-fork-join)中的 Fork/Join 框架指南。

一个`ForkJoinTask`的主要特征是它通常会产生新的子任务，作为完成其主要任务所需工作的一部分。它通过调用`[fork()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinTask.html#fork()),`生成新的任务，并收集所有带有`[join()](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinTask.html#join()),` 的结果，即类名。

有两个抽象类实现了`ForkJoinTask` : `[RecursiveTask](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/RecursiveTask.html),`完成后返回值，还有`[RecursiveAction](https://web.archive.org/web/20221007090011/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/RecursiveAction.html),`不返回任何东西。顾名思义，这些类将用于递归任务，如文件系统导航或复杂的数学计算。

让我们扩展前面的例子来创建一个类，给定一个`Integer`，它将计算所有阶乘元素的平方和。例如，如果我们将数字 4 传递给计算器，我们应该得到 4 + 3 + 2 + 1 的和，也就是 30。

首先，我们需要创建一个`RecursiveTask`的具体实现，并实现它的`compute()` 方法。这是我们编写业务逻辑的地方:

```java
public class FactorialSquareCalculator extends RecursiveTask<Integer> {

    private Integer n;

    public FactorialSquareCalculator(Integer n) {
        this.n = n;
    }

    @Override
    protected Integer compute() {
        if (n <= 1) {
            return n;
        }

        FactorialSquareCalculator calculator 
          = new FactorialSquareCalculator(n - 1);

        calculator.fork();

        return n * n + calculator.join();
    }
}
```

注意我们是如何通过在`compute()`中创建一个新的`FactorialSquareCalculator`实例来实现递归的。通过调用非阻塞方法`fork()`，我们要求 *ForkJoinPool* 启动这个子任务的执行。

`join()`方法将返回计算结果，我们将把当前访问的数字的平方加到计算结果中。

现在我们只需要创建一个`ForkJoinPool` 来处理执行和线程管理:

```java
ForkJoinPool forkJoinPool = new ForkJoinPool();

FactorialSquareCalculator calculator = new FactorialSquareCalculator(10);

forkJoinPool.execute(calculator);
```

## 6。结论

在本文中，我们全面探索了`Future` 接口，触及了它的所有方法。我们还了解了如何利用线程池来触发多个并行操作。还简要介绍了来自`ForkJoinTask`类、`fork()`和`join(),`的主要方法。

我们还有许多关于 Java 中并行和异步操作的其他优秀文章。这里有三个与`Future`接口密切相关的接口，其中一些已经在文章中提到:

*   [`CompletableFuture`](/web/20221007090011/https://www.baeldung.com/java-completablefuture)指南 Java 8 中引入了许多额外特性的`Future` 的实现
*   [Java 中的 Fork/Join 框架指南](/web/20221007090011/https://www.baeldung.com/java-fork-join)——更多关于我们在第 5 节中提到的`ForkJoinTask`
*   [Java`ExecutorService`](/web/20221007090011/https://www.baeldung.com/java-executor-service-tutorial)指南——专用于`ExecutorService`接口

和往常一样，本文中使用的源代码可以在我们的 [GitHub 资源库](https://web.archive.org/web/20221007090011/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)中找到。