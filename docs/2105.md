# Java 中的可运行与可调用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-runnable-callable>

## 1。概述

自 Java 早期以来，多线程一直是该语言的一个主要方面。`Runnable` 是为表示多线程任务而提供的核心接口，Java 1.5 提供了`Callable`作为`Runnable`的改进版本。

在本教程中，我们将探索两种界面的区别和应用。

## 2。执行机构

这两个接口都被设计为表示可以由多个线程运行的任务。我们可以使用`Thread`类或`ExecutorService`来运行`Runnable`任务，然而我们只能使用后者来运行 `Callable` s。

## 3。返回值

让我们更深入地看看这些接口是如何处理返回值的。

### 3.1。`Runnable`同

`Runnable` 接口是一个函数接口，有一个单独的`run()` 方法，不接受任何参数也不返回任何值。

这适用于我们不需要寻找线程执行结果的情况，例如传入事件记录:

```
public interface Runnable {
    public void run();
}
```

让我们用一个例子来理解这一点:

```
public class EventLoggingTask implements  Runnable{
    private Logger logger
      = LoggerFactory.getLogger(EventLoggingTask.class);

    @Override
    public void run() {
        logger.info("Message");
    }
}
```

在本例中，线程将从队列中读取一条消息，并将其记录在日志文件中。任务没有返回值。

我们可以使用`ExecutorService`启动任务:

```
public void executeTask() {
    executorService = Executors.newSingleThreadExecutor();
    Future future = executorService.submit(new EventLoggingTask());
    executorService.shutdown();
}
```

在这种情况下， `Future` 对象将不保存任何值。

### 3.2。`Callable`同

`Callable`接口是一个通用接口，包含一个返回通用值`V`的`call()`方法:

```
public interface Callable<V> {
    V call() throws Exception;
}
```

让我们来看看计算一个数的阶乘:

```
public class FactorialTask implements Callable<Integer> {
    int number;

    // standard constructors

    public Integer call() throws InvalidParamaterException {
        int fact = 1;
        // ...
        for(int count = number; count > 1; count--) {
            fact = fact * count;
        }

        return fact;
    }
}
```

`call()` 方法的结果在`Future` 对象中返回:

```
@Test
public void whenTaskSubmitted_ThenFutureResultObtained(){
    FactorialTask task = new FactorialTask(5);
    Future<Integer> future = executorService.submit(task);

    assertEquals(120, future.get().intValue());
}
```

## 4。异常处理

让我们看看它们有多适合异常处理。

### 4.1。`Runnable`同

由于方法签名没有指定“throws”子句，我们没有办法传播进一步检查的异常。

### 4.2。`Callable`同

`Callable`的 `call()`方法包含“throws `Exception`”子句，因此我们可以轻松地进一步传播已检查的异常:

```
public class FactorialTask implements Callable<Integer> {
    // ...
    public Integer call() throws InvalidParamaterException {

        if(number < 0) {
            throw new InvalidParamaterException("Number should be positive");
        }
    // ...
    }
}
```

在使用 `ExecutorService`运行`Callable`的情况下，异常被收集在 `Future` 对象中。我们可以通过调用`Future.get()` 方法来检查这一点。

这将抛出一个`ExecutionException`，它包装了最初的异常:

```
@Test(expected = ExecutionException.class)
public void whenException_ThenCallableThrowsIt() {

    FactorialCallableTask task = new FactorialCallableTask(-5);
    Future<Integer> future = executorService.submit(task);
    Integer result = future.get().intValue();
}
```

在上面的测试中，因为我们传递了一个无效的数字，所以抛出了`ExecutionException` 。我们可以在这个异常对象上调用`getCause()` 方法来获得最初检查的异常。

如果我们不调用`Future` 类的`get()` 方法，那么`call()` 方法抛出的异常不会被报告回来，任务仍然会被标记为完成:

```
@Test
public void whenException_ThenCallableDoesntThrowsItIfGetIsNotCalled(){
    FactorialCallableTask task = new FactorialCallableTask(-5);
    Future<Integer> future = executorService.submit(task);

    assertEquals(false, future.isDone());
}
```

上面的测试将会成功通过，尽管我们已经为参数的负值向`FactorialCallableTask`抛出了一个异常。

## 5。结论

在本文中，我们探索了`Runnable` 和`Callable` 接口之间的差异。

和往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221122012856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)