# 向 Java 线程传递参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-parameters>

## 1。概述

在本教程中，我们将介绍向 Java [线程](/web/20220524030648/https://www.baeldung.com/java-thread-lifecycle)传递参数的不同选项。

## 2。螺纹基础知识

快速提醒一下，我们可以通过[实现`Runnable`](/web/20220524030648/https://www.baeldung.com/java-runnable-vs-extending-thread) 或 [`Callable`](/web/20220524030648/https://www.baeldung.com/java-runnable-callable) 在 Java 中创建一个线程。

要运行一个线程，我们可以调用`Thread#start`(通过传递一个 `Runnable`的实例)或者[使用一个线程池](/web/20220524030648/https://www.baeldung.com/thread-pool-java-and-guava)，通过提交给一个`[ExecutorService](/web/20220524030648/https://www.baeldung.com/java-executor-service-tutorial).`

然而，这两种方法都不接受任何额外的参数。

让我们看看如何将参数传递给线程。

## 3。在构造函数中发送参数

我们可以向线程发送参数的第一种方式是简单地在它们的构造函数中向我们的`Runnable `或`Callable`提供参数。

让我们创建一个接受一组数字并返回其平均值的`AverageCalculator`:

```
public class AverageCalculator implements Callable<Double> {

    int[] numbers;

    public AverageCalculator(int... numbers) {
        this.numbers = numbers == null ? new int[0] : numbers;
    }

    @Override
    public Double call() throws Exception {
        return IntStream.of(numbers).average().orElse(0d);
    }
}
```

接下来，我们将向我们的平均计算器线程提供一些数字，并验证输出:

```
@Test
public void whenSendingParameterToCallable_thenSuccessful() throws Exception {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    Future<Double> result = executorService.submit(new AverageCalculator(1, 2, 3));
    try {
        assertEquals(2.0, result.get().doubleValue());
    } finally {
        executorService.shutdown();
    }
}
```

注意，这样做的原因是在启动线程之前，我们已经将类的状态交给了它。

## 4。通过闭包发送参数

向线程传递参数的另一种方式是通过[创建闭包](/web/20220524030648/https://www.baeldung.com/cs/closure) `.`

`closure` 是一个可以继承其父作用域的作用域–**我们看到它有 lambdas 和匿名内部类。**

让我们扩展前面的例子，创建两个线程。

第一个将计算平均值:

```
executorService.submit(() -> IntStream.of(numbers).average().orElse(0d));
```

并且，第二个将做总和:

```
executorService.submit(() -> IntStream.of(numbers).sum());
```

让我们看看如何将同一个参数传递给两个线程并得到结果:

```
@Test
public void whenParametersToThreadWithLamda_thenParametersPassedCorrectly()
  throws Exception {
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    int[] numbers = new int[] { 4, 5, 6 };

    try {
        Future<Integer> sumResult = 
          executorService.submit(() -> IntStream.of(numbers).sum()); 
        Future<Double> averageResult = 
          executorService.submit(() -> IntStream.of(numbers).average().orElse(0d));
        assertEquals(Integer.valueOf(15), sumResult.get());
        assertEquals(Double.valueOf(5.0), averageResult.get());
    } finally {
        executorService.shutdown();
    }
}
```

需要记住的一件重要事情是 [**保持参数有效最终**](/web/20220524030648/https://www.baeldung.com/java-8-lambda-expressions-tips) 否则我们无法将它们交给闭包。

**同样，同样的并发规则在这里和其他地方都适用。**如果我们在线程运行时改变了`numbers`数组中的一个值，不保证在[没有引入](/web/20220524030648/https://www.baeldung.com/java-synchronized-collections) [某种同步](/web/20220524030648/https://www.baeldung.com/java-synchronized)的情况下它们会看到。

总结一下，如果我们使用的是旧版本的 Java，匿名内部类也是可行的:

```
final int[] numbers = { 1, 2, 3 };
Thread parameterizedThread = new Thread(new Callable<Double>() {
    @Override
    public Double call() {
        return calculateTheAverage(numbers);
    }
});
parameterizedThread.start();
```

## 5。结论

在本文中，我们发现了向 Java 线程传递参数的不同选项。

与往常一样，代码示例可以在 Github 上的[处获得。](https://web.archive.org/web/20220524030648/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)