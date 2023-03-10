# 实现可运行线程与扩展线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-runnable-vs-extending-thread>

## 1。简介

“我应该实现一个`Runnable`还是扩展`Thread`类”？是一个很常见的问题。

在本文中，我们将看到哪种方法在实践中更有意义，以及为什么。

## 2。使用`Thread`

让我们首先定义一个扩展了`Thread`的`SimpleThread`类:

```java
public class SimpleThread extends Thread {

    private String message;

    // standard logger, constructor

    @Override
    public void run() {
        log.info(message);
    }
}
```

让我们看看如何运行这种类型的线程:

```java
@Test
public void givenAThread_whenRunIt_thenResult()
  throws Exception {

    Thread thread = new SimpleThread(
      "SimpleThread executed using Thread");
    thread.start();
    thread.join();
}
```

我们也可以使用一个`ExecutorService`来执行线程:

```java
@Test
public void givenAThread_whenSubmitToES_thenResult()
  throws Exception {

    executorService.submit(new SimpleThread(
      "SimpleThread executed using ExecutorService")).get();
}
```

在一个单独的线程中运行一个日志操作需要很多代码。

另外，注意 **`SimpleThread`不能扩展任何其他类**，因为 Java 不支持多重继承。

## 3。实施一个`Runnable`

现在，让我们创建一个实现`java.lang.Runnable`接口的简单任务:

```java
class SimpleRunnable implements Runnable {

    private String message;

    // standard logger, constructor

    @Override
    public void run() {
        log.info(message);
    }
}
```

上面的`SimpleRunnable`只是一个我们想要在一个单独的线程中运行的任务。

我们可以用各种方法来运行它。其中之一是使用`Thread`类:

```java
@Test
public void givenRunnable_whenRunIt_thenResult()
 throws Exception {
    Thread thread = new Thread(new SimpleRunnable(
      "SimpleRunnable executed using Thread"));
    thread.start();
    thread.join();
}
```

我们甚至可以使用一个`ExecutorService`:

```java
@Test
public void givenARunnable_whenSubmitToES_thenResult()
 throws Exception {

    executorService.submit(new SimpleRunnable(
      "SimpleRunnable executed using ExecutorService")).get();
}
```

我们可以在的[中阅读更多关于`ExecutorService`的内容。](/web/20220525125149/https://www.baeldung.com/java-executor-service-tutorial)

因为我们现在实现了一个接口，所以如果需要的话，我们可以自由地扩展另一个基类。

从 Java 8 开始，任何公开单一抽象方法的接口都被视为函数接口，这使得它成为有效的 lambda 表达式目标。

**我们可以使用λ表达式**重写上面的`Runnable`代码:

```java
@Test
public void givenARunnableLambda_whenSubmitToES_thenResult() 
  throws Exception {

    executorService.submit(
      () -> log.info("Lambda runnable executed!"));
}
```

## 4。 `Runnable`还是`Thread`？

简单地说，我们通常鼓励使用`Runnable`而不是`Thread`:

*   当扩展`Thread`类时，我们没有覆盖它的任何方法。相反，我们覆盖了`Runnable (`的方法，而`Thread`恰好实现了`)`。这明显违反了 IS-A `Thread`原则
*   创建`Runnable`的实现并将其传递给`Thread`类利用了组合而不是继承——这更灵活
*   在扩展了`Thread`类之后，我们不能扩展任何其他类
*   从 Java 8 开始，`Runnables`可以用 lambda 表达式来表示

## 5。结论

在这个快速教程中，我们看到了实现`Runnable`通常是比扩展`Thread`类更好的方法。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525125149/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)