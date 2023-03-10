# Java 中的 CountDownLatch 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-countdown-latch>

## 1。简介

在本文中，我们将给出一个关于`[CountDownLatch](https://web.archive.org/web/20220802060310/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CountDownLatch.html)` 类的指南，并演示如何在几个实际例子中使用它。

本质上，通过使用`CountDownLatch`,我们可以使一个线程阻塞，直到其他线程完成给定的任务。

## 2。并发编程中的用法

简单地说，一个 `CountDownLatch`有一个`counter`字段，你可以根据我们的需要减少这个字段。然后我们可以用它来阻塞一个调用线程，直到它倒数到零。

如果我们正在做一些并行处理，我们可以用与我们想要处理的线程数量相同的计数器值来实例化`CountDownLatch` 。然后，我们可以在每个线程完成后调用`countdown()` ，保证调用`await()` 的依赖线程将阻塞，直到工作线程完成。

## 3。等待线程池完成

让我们通过创建一个`Worker` 并使用一个`CountDownLatch` 字段在它完成时发出信号来尝试这种模式:

```java
public class Worker implements Runnable {
    private List<String> outputScraper;
    private CountDownLatch countDownLatch;

    public Worker(List<String> outputScraper, CountDownLatch countDownLatch) {
        this.outputScraper = outputScraper;
        this.countDownLatch = countDownLatch;
    }

    @Override
    public void run() {
        doSomeWork();
        outputScraper.add("Counted down");
        countDownLatch.countDown();
    }
}
```

然后，让我们创建一个测试来证明我们可以让`CountDownLatch` 等待`Worker` 实例完成:

```java
@Test
public void whenParallelProcessing_thenMainThreadWillBlockUntilCompletion()
  throws InterruptedException {

    List<String> outputScraper = Collections.synchronizedList(new ArrayList<>());
    CountDownLatch countDownLatch = new CountDownLatch(5);
    List<Thread> workers = Stream
      .generate(() -> new Thread(new Worker(outputScraper, countDownLatch)))
      .limit(5)
      .collect(toList());

      workers.forEach(Thread::start);
      countDownLatch.await(); 
      outputScraper.add("Latch released");

      assertThat(outputScraper)
        .containsExactly(
          "Counted down",
          "Counted down",
          "Counted down",
          "Counted down",
          "Counted down",
          "Latch released"
        );
    }
```

自然地,“闩锁释放”将总是最后一个输出——因为它依赖于`CountDownLatch` 释放。

注意，如果我们不调用`await()`，我们将无法保证线程执行的顺序，因此测试将随机失败。

## 4.**等待开始的线程池**

如果我们以前面的例子为例，但是这次启动了数千个线程而不是五个，很可能在我们对后面的线程调用`start()` 之前，许多前面的线程就已经完成了处理。这可能会使尝试和重现并发问题变得困难，因为我们无法让所有线程并行运行。

为了解决这个问题，让我们让`CountdownLatch`以不同于前一个例子的方式工作。我们可以阻塞每个子线程，直到所有其他子线程都已经启动，而不是阻塞一个父线程，直到一些子线程已经完成。

让我们修改我们的`run()`方法，让它在处理之前阻塞:

```java
public class WaitingWorker implements Runnable {

    private List<String> outputScraper;
    private CountDownLatch readyThreadCounter;
    private CountDownLatch callingThreadBlocker;
    private CountDownLatch completedThreadCounter;

    public WaitingWorker(
      List<String> outputScraper,
      CountDownLatch readyThreadCounter,
      CountDownLatch callingThreadBlocker,
      CountDownLatch completedThreadCounter) {

        this.outputScraper = outputScraper;
        this.readyThreadCounter = readyThreadCounter;
        this.callingThreadBlocker = callingThreadBlocker;
        this.completedThreadCounter = completedThreadCounter;
    }

    @Override
    public void run() {
        readyThreadCounter.countDown();
        try {
            callingThreadBlocker.await();
            doSomeWork();
            outputScraper.add("Counted down");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            completedThreadCounter.countDown();
        }
    }
}
```

现在，让我们修改我们的测试，让它阻塞，直到所有的`Workers` 都已经启动，解除阻塞`Workers,` ，然后阻塞，直到`Workers`已经完成:

```java
@Test
public void whenDoingLotsOfThreadsInParallel_thenStartThemAtTheSameTime()
 throws InterruptedException {

    List<String> outputScraper = Collections.synchronizedList(new ArrayList<>());
    CountDownLatch readyThreadCounter = new CountDownLatch(5);
    CountDownLatch callingThreadBlocker = new CountDownLatch(1);
    CountDownLatch completedThreadCounter = new CountDownLatch(5);
    List<Thread> workers = Stream
      .generate(() -> new Thread(new WaitingWorker(
        outputScraper, readyThreadCounter, callingThreadBlocker, completedThreadCounter)))
      .limit(5)
      .collect(toList());

    workers.forEach(Thread::start);
    readyThreadCounter.await(); 
    outputScraper.add("Workers ready");
    callingThreadBlocker.countDown(); 
    completedThreadCounter.await(); 
    outputScraper.add("Workers complete");

    assertThat(outputScraper)
      .containsExactly(
        "Workers ready",
        "Counted down",
        "Counted down",
        "Counted down",
        "Counted down",
        "Counted down",
        "Workers complete"
      );
}
```

这种模式对于试图重现并发错误非常有用，因为它可以用来迫使数千个线程尝试并行执行某些逻辑。

## 5。心狠手辣一个`CountdownLatch`早

有时，我们可能会遇到这样的情况，在倒计数`CountDownLatch.` 之前`Workers`错误地终止，这可能导致它永远不会到达零，并且`await()`永远不会终止:

```java
@Override
public void run() {
    if (true) {
        throw new RuntimeException("Oh dear, I'm a BrokenWorker");
    }
    countDownLatch.countDown();
    outputScraper.add("Counted down");
}
```

让我们修改我们之前的测试，使用一个`BrokenWorker,` 来展示`await()` 将如何永远阻塞:

```java
@Test
public void whenFailingToParallelProcess_thenMainThreadShouldGetNotGetStuck()
  throws InterruptedException {

    List<String> outputScraper = Collections.synchronizedList(new ArrayList<>());
    CountDownLatch countDownLatch = new CountDownLatch(5);
    List<Thread> workers = Stream
      .generate(() -> new Thread(new BrokenWorker(outputScraper, countDownLatch)))
      .limit(5)
      .collect(toList());

    workers.forEach(Thread::start);
    countDownLatch.await();
}
```

显然，这不是我们想要的行为——对于应用程序来说，继续运行要比无限阻塞好得多。

为了解决这个问题，让我们在对`await().`的调用中添加一个超时参数

```java
boolean completed = countDownLatch.await(3L, TimeUnit.SECONDS);
assertThat(completed).isFalse();
```

正如我们所看到的，测试最终会超时，`await()` 将返回`false`。

## 6。结论

在这篇快速指南中，我们演示了如何使用`CountDownLatch` 来阻塞一个线程，直到其他线程完成一些处理。

我们还展示了如何通过确保线程并行运行来帮助调试并发性问题。

这些例子的实现可以在 GitHub 上找到[；这是一个基于 Maven 的项目，所以应该很容易运行。](https://web.archive.org/web/20220802060310/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)