# 如何杀死一个 Java 线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-stop>

## 1。简介

在这篇简短的文章中，**我们将介绍如何在 Java 中停止一个`Thread`——这并不简单，因为`Thread.stop()`方法已经被废弃了。**

正如在[中所解释的，来自 Oracle 的这个更新，](https://web.archive.org/web/20220929100635/https://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html) `stop()`会导致被监视的对象被破坏。

## 2。使用标志

让我们从创建和启动线程的类开始。这个任务不会自己结束，所以我们需要某种方法来停止这个线程。

我们将为此使用原子标记:

```java
public class ControlSubThread implements Runnable {

    private Thread worker;
    private final AtomicBoolean running = new AtomicBoolean(false);
    private int interval;

    public ControlSubThread(int sleepInterval) {
        interval = sleepInterval;
    }

    public void start() {
        worker = new Thread(this);
        worker.start();
    }

    public void stop() {
        running.set(false);
    }

    public void run() { 
        running.set(true);
        while (running.get()) {
            try { 
                Thread.sleep(interval); 
            } catch (InterruptedException e){ 
                Thread.currentThread().interrupt();
                System.out.println(
                  "Thread was interrupted, Failed to complete operation");
            }
            // do something here 
         } 
    } 
}
```

我们使用了一个`AtomicBoolean` ，而不是用一个`while`循环来计算一个常量`true`，现在我们可以通过将它设置为`true/false.`来开始/停止执行

正如我们在原子变量的[介绍中所解释的，使用`AtomicBoolean` 可以防止不同线程在设置和检查变量时发生冲突。](/web/20220929100635/https://www.baeldung.com/java-atomic-variables)

## 3。`Thread`打断一个

当`sleep()`被设置为长间隔时，或者如果我们在等待一个可能永远不会被释放的`[lock](/web/20220929100635/https://www.baeldung.com/java-util-concurrent)`会发生什么？

我们面临着长期阻塞或永远无法彻底终止的风险。

我们可以为这些情况创建`interrupt()`，让我们给这个类添加一些方法和一个新的标志:

```java
public class ControlSubThread implements Runnable {

    private Thread worker;
    private AtomicBoolean running = new AtomicBoolean(false);
    private int interval;

    // ...

    public void interrupt() {
        running.set(false);
        worker.interrupt();
    }

    boolean isRunning() {
        return running.get();
    }

    boolean isStopped() {
        return stopped.get();
    }

    public void run() {
        running.set(true);
        stopped.set(false);
        while (running.get()) {
            try {
                Thread.sleep(interval);
            } catch (InterruptedException e){
                Thread.currentThread().interrupt();
                System.out.println(
                  "Thread was interrupted, Failed to complete operation");
            }
            // do something
        }
        stopped.set(true);
    }
} 
```

我们添加了一个`interrupt()`方法，将我们的`running`标志设置为 false，并调用工作线程的`interrupt()`方法。

如果线程在被调用时处于休眠状态，`sleep()`将会退出，并带有一个`InterruptedException,` ，就像任何其他阻塞调用一样。

这将线程返回到循环，并且它将退出，因为`running`为假。

## 4。结论

在这个快速教程中，我们看了如何使用原子变量，可选地结合调用`interrupt(),` 来干净地关闭线程。这肯定比调用不赞成使用的`stop()` 方法更好，因为这样会冒永久锁定和内存损坏的风险。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220929100635/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)