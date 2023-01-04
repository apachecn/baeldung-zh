# Java 中的 Thread.join()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-join>

## 1.概观

在本教程中，我们将讨论`Thread`类中不同的`join()`方法。我们将深入研究这些方法的细节和一些示例代码。

与`wait()`和`notify() methods`一样，`join()`是线程间同步的另一种机制。

你可以快速浏览一下[这篇教程](/web/20220524034555/https://www.baeldung.com/java-wait-notify)，了解更多关于`wait()`和`notify()`的内容。

## 2.`Thread.join()`法

join 方法在`[Thread](https://web.archive.org/web/20220524034555/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html#join())` 类中定义:

> `public final void join() throws InterruptedException`
> 

**当我们在一个线程上调用`join()`方法时，调用线程进入等待状态。它保持等待状态，直到被引用的线程终止。**

我们可以在下面的代码中看到这种行为:

```
class SampleThread extends Thread {
    public int processingCount = 0;

    SampleThread(int processingCount) {
        this.processingCount = processingCount;
        LOGGER.info("Thread Created");
    }

    @Override
    public void run() {
        LOGGER.info("Thread " + this.getName() + " started");
        while (processingCount > 0) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                LOGGER.info("Thread " + this.getName() + " interrupted");
            }
            processingCount--;
        }
        LOGGER.info("Thread " + this.getName() + " exiting");
    }
}

@Test
public void givenStartedThread_whenJoinCalled_waitsTillCompletion() 
  throws InterruptedException {
    Thread t2 = new SampleThread(1);
    t2.start();
    LOGGER.info("Invoking join");
    t2.join();
    LOGGER.info("Returned from join");
    assertFalse(t2.isAlive());
} 
```

在执行代码时，我们应该会看到类似以下的结果:

```
INFO: Thread Created
INFO: Invoking join
INFO: Thread Thread-1 started
INFO: Thread Thread-1 exiting
INFO: Returned from join
```

**如果引用的线程被中断**，`join()` 方法也可能返回。在这种情况下，该方法抛出一个`InterruptedException`。

最后，**如果被引用的线程已经终止或者还没有启动，对`join()`方法的调用会立即返回**。

```
Thread t1 = new SampleThread(0);
t1.join();  //returns immediately
```

## 3.`Thread.join()`带超时的方法

如果被引用的线程被阻塞或者处理时间太长，`join()`方法将继续等待。这可能会成为一个问题，因为调用线程将变得没有响应。为了处理这些情况，我们使用重载版本的`join()`方法，它允许我们指定一个超时周期。

**有两个[计时版本](https://web.archive.org/web/20220524034555/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html#join(long))，它们重载了`join()`方法:**

> `“public final void join(long `毫秒`) throws InterruptedException`
> `Waits at most` 毫秒` milliseconds for this thread to die. A timeout of 0 means to wait forever.”`
> 
> `“public final void join(long `毫里斯，int 纳诺`) throws InterruptedException`
> `Waits at most `毫里斯` milliseconds plus `纳诺` nanoseconds for this thread to die.”`

我们可以使用下面的定时`join()`:

```
@Test
public void givenStartedThread_whenTimedJoinCalled_waitsUntilTimedout()
  throws InterruptedException {
    Thread t3 = new SampleThread(10);
    t3.start();
    t3.join(1000);
    assertTrue(t3.isAlive());
} 
```

在这种情况下，调用线程等待大约 1 秒钟，等待线程 t3 完成。如果线程 t3 在这个时间段内没有完成，`join()`方法将控制权返回给调用方法。

**定时 `join()`取决于操作系统的定时。因此，我们不能假设`join()`会等待指定的时间。**

## 4.`Thread.join()`方法和同步

除了一直等到终止，调用`join()`方法还有一个同步效果。 **join()创建一个[发生前](https://web.archive.org/web/20220524034555/https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.5)关系:**

> `“All actions in a thread happen-before any other thread successfully returns from a join() on that thread.”`

这意味着当线程 t1 调用 t2.join()时，t2 所做的所有更改在返回时在 t1 中都是可见的。然而，如果我们不调用`join()`或使用其他同步机制，我们不能保证其他线程中的变化对当前线程是可见的，即使其他线程已经完成。

因此，即使对处于终止状态的线程的`join()`方法调用立即返回，我们仍然需要在某些情况下调用它。

我们可以在下面看到一个错误同步代码的例子:

```
SampleThread t4 = new SampleThread(10);
t4.start();
// not guaranteed to stop even if t4 finishes.
do {

} while (t4.processingCount > 0);
```

为了正确地同步上面的代码，我们可以在循环中添加定时的`t4.join()`或者使用一些其他的同步机制。

## 5.结论

`join()`方法对于线程间同步非常有用。在本文中，我们讨论了`join()`方法及其行为。我们也使用`join()`方法审查代码。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524034555/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)