# 在 Java 中同时启动两个线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-start-two-threads-at-same-time>

## 1.概观

多线程编程允许我们并发运行线程，每个线程可以处理不同的任务。因此，它可以最大限度地利用资源，尤其是当我们的计算机拥有多个多核 CPU 时。

有时，我们希望控制多个线程同时启动。

在本教程中，我们将首先理解需求，尤其是“完全相同的时间”的含义。此外，我们将讨论如何在 Java 中同时启动两个线程。

## 2.了解需求

我们的要求是:“同时启动两个线程。”

这个要求看起来很容易理解。然而，如果我们仔细想想，甚至有可能在`EXACT`同时启动两个线程吗？

首先，每个线程都会消耗 CPU 时间来工作。因此，**如果我们的应用运行在单核 CPU 的计算机上，就不可能同时启动两个线程`exact`。**

如果我们的计算机有一个多核 CPU 或多个 CPU，两个线程可能会在`exact`同时启动。但是，我们无法在 Java 端控制它。

这是因为当我们在 Java 中使用线程时，**Java 线程调度依赖于操作系统**的线程调度。因此，不同的操作系统可能会有不同的处理方式。

此外，如果我们以更严格的方式讨论“完全相同的时间”，根据爱因斯坦的[狭义相对论](https://web.archive.org/web/20220526054156/https://en.wikipedia.org/wiki/Special_relativity "Special relativity"):

> 如果两个截然不同的事件在空间上是分开的，就不可能绝对地说它们同时发生。

无论我们的 CPU 离主板或 CPU 中的内核有多近，都有空间。因此，我们不能确保两个线程同时在`EXACT`开始。

那么，是否意味着该要求无效呢？

不。这是一个有效的要求。即使我们不能让两个线程同时开始，我们也可以通过一些同步技术让它们非常接近。

在大多数实际情况下，当我们需要两个线程“同时”启动时，这些技术可能会有所帮助

在本教程中，我们将探索两种方法来解决这个问题:

*   使用`[CountDownLatch](/web/20220526054156/https://www.baeldung.com/java-countdown-latch)`类
*   使用`[CyclicBarrier](/web/20220526054156/https://www.baeldung.com/java-cyclic-barrier)`类
*   使用 [`Phaser`](/web/20220526054156/https://www.baeldung.com/java-phaser) 类

所有的方法都遵循同样的想法:我们不会真的同时启动两个线程。相反，我们在线程启动后立即阻塞线程，并试图同时恢复它们的执行。

因为我们的测试将与线程调度相关，所以在本教程中有必要提到运行测试的环境:

*   CPU:英特尔酷睿 i7-8850H CPU。处理器时钟在 2.6 到 4.3 GHz 之间(4 个内核时为 4.3 GHz，6 个内核时为 4 GHz)
*   操作系统:内核版本为 5.12.12 的 64 位 Linux
*   Java: Java 11

现在，让我们来看看`CountDonwLatch`和`CyclicBarrier`的动作。

## 3.使用`CountDownLatch`类

`CountDownLatch`是 Java 5 中引入的同步器，作为 `java.util.concurrent`包的一部分。通常，**我们使用一个`CountDownLatch`来阻塞线程，直到其他线程完成它们的任务。**

简单地说，我们在一个`latch`对象中设置一个`count`，并将`latch`对象关联到一些线程。当我们启动这些线程时，它们将被阻塞，直到闩锁的计数变为零。

另一方面，在其他线程中，我们可以控制在什么条件下减少`count`并让阻塞的线程恢复，例如，当主线程中的一些任务完成时。

### 3.1.工作线程

现在，让我们看看如何使用`CountDownLatch`类来解决我们的问题。

首先，我们将创建我们的`Thread`类。姑且称之为`WorkerWithCountDownLatch`:

```
public class WorkerWithCountDownLatch extends Thread {
    private CountDownLatch latch;

    public WorkerWithCountDownLatch(String name, CountDownLatch latch) {
        this.latch = latch;
        setName(name);
    }

    @Override public void run() {
        try {
            System.out.printf("[ %s ] created, blocked by the latch...\n", getName());
            latch.await();
            System.out.printf("[ %s ] starts at: %s\n", getName(), Instant.now());
            // do actual work here...
        } catch (InterruptedException e) {
            // handle exception
        }
    } 
```

我们已经在我们的`WorkerWithCountDownLatch `类中添加了一个`latch`对象。首先，让我们了解一下`latch`对象的功能。

在`run()`方法中，我们称之为`latch.await(). `方法，这意味着，如果我们启动了`worker`线程，它将检查`latch's count. `线程将被阻塞，直到`count`为零。

这样，我们可以在主线程中用`count=1` 创建一个`CountDownLatch(1)`锁，并将`latch`对象关联到我们想要同时启动的两个工作线程。

当我们希望两个线程继续做它们的实际工作时，我们通过调用主线程中的`latch.countDown()`来释放闩锁。

接下来，让我们看看主线程是如何控制两个工作线程的。

### 3.2.主线程

我们将在`usingCountDownLatch()`方法中实现主线程:

```
private static void usingCountDownLatch() throws InterruptedException {
    System.out.println("===============================================");
    System.out.println("        >>> Using CountDownLatch <<<<");
    System.out.println("===============================================");

    CountDownLatch latch = new CountDownLatch(1);

    WorkerWithCountDownLatch worker1 = new WorkerWithCountDownLatch("Worker with latch 1", latch);
    WorkerWithCountDownLatch worker2 = new WorkerWithCountDownLatch("Worker with latch 2", latch);

    worker1.start();
    worker2.start();

    Thread.sleep(10);//simulation of some actual work

    System.out.println("-----------------------------------------------");
    System.out.println(" Now release the latch:");
    System.out.println("-----------------------------------------------");
    latch.countDown();
} 
```

现在，让我们从我们的`main()`方法调用上面的`usingCountDownLatch()`方法。当我们运行`main()`方法时，我们将看到输出:

```
===============================================
        >>> Using CountDownLatch <<<<
===============================================
[ Worker with latch 1 ] created, blocked by the latch
[ Worker with latch 2 ] created, blocked by the latch
-----------------------------------------------
 Now release the latch:
-----------------------------------------------
[ Worker with latch 2 ] starts at: 2021-06-27T16:00:52.268532035Z
[ Worker with latch 1 ] starts at: 2021-06-27T16:00:52.268533787Z 
```

如上面的输出所示，**两个工作线程同时启动`almost`。两个开始时间之间的差异小于两微秒**。

## 4.使用`CyclicBarrier`类

`CyclicBarrier`类是 Java 5 中引入的另一个同步器。本质上， **`CyclicBarrier`允许固定数量的线程在继续执行**之前等待彼此到达一个公共点。

接下来，让我们看看如何使用`CyclicBarrier`类解决我们的问题。

### 4.1.工作线程

让我们先来看看我们的 worker 线程的实现:

```
public class WorkerWithCyclicBarrier extends Thread {
    private CyclicBarrier barrier;

    public WorkerWithCyclicBarrier(String name, CyclicBarrier barrier) {
        this.barrier = barrier;
        this.setName(name);
    }

    @Override public void run() {
        try {
            System.out.printf("[ %s ] created, blocked by the barrier\n", getName());
            barrier.await();
            System.out.printf("[ %s ] starts at: %s\n", getName(), Instant.now());
            // do actual work here...
        } catch (InterruptedException | BrokenBarrierException e) {
            // handle exception
        }
    }
} 
```

实现非常简单。我们将一个`barrier`对象与工作线程相关联。当线程启动时，我们立即调用`barrier.await() `方法。

这样，工作线程将被阻塞，并等待各方调用`barrier.await()`来恢复。

### 4.2.主线程

接下来，让我们看看如何控制两个工作线程在主线程中恢复:

```
private static void usingCyclicBarrier() throws BrokenBarrierException, InterruptedException {
    System.out.println("\n===============================================");
    System.out.println("        >>> Using CyclicBarrier <<<<");
    System.out.println("===============================================");

    CyclicBarrier barrier = new CyclicBarrier(3);

    WorkerWithCyclicBarrier worker1 = new WorkerWithCyclicBarrier("Worker with barrier 1", barrier);
    WorkerWithCyclicBarrier worker2 = new WorkerWithCyclicBarrier("Worker with barrier 2", barrier);

    worker1.start();
    worker2.start();

    Thread.sleep(10);//simulation of some actual work

    System.out.println("-----------------------------------------------");
    System.out.println(" Now open the barrier:");
    System.out.println("-----------------------------------------------");
    barrier.await();
} 
```

我们的目标是让两个工作线程同时恢复。所以，加上主线程，我们总共有三个线程。

如上面的方法所示，我们在主线程中创建了一个包含三方的`barrier`对象。接下来，我们创建并启动两个工作线程。

正如我们前面所讨论的，这两个工作线程被阻塞，并等待屏障的打开恢复。

在主线程中，我们可以做一些实际工作。当我们决定打开屏障时，我们调用方法`barrier.await() `让两个工人继续执行。

如果我们在`main()`方法中调用`usingCyclicBarrier()`，我们将得到输出:

```
===============================================
        >>> Using CyclicBarrier <<<<
===============================================
[ Worker with barrier 1 ] created, blocked by the barrier
[ Worker with barrier 2 ] created, blocked by the barrier
-----------------------------------------------
 Now open the barrier:
-----------------------------------------------
[ Worker with barrier 1 ] starts at: 2021-06-27T16:00:52.311346392Z
[ Worker with barrier 2 ] starts at: 2021-06-27T16:00:52.311348874Z
```

我们可以比较工人的两个开始时间。即使两个工人不是在完全相同的时间开始，我们也非常接近我们的目标:两个开始时间之间的差异小于 3 微秒。

## 5.使用`Phaser`类

`Phaser`类是 Java 7 中引入的同步器。类似于`CyclicBarrier`和`CountDownLatch`。不过`Phaser`类更灵活。

比如`CyclicBarrier`和`CountDownLatch`不同， **`Phaser`允许我们动态注册线程方。**

接下来，我们用`Phaser`来解题。

### 5.1.工作线程

像往常一样，我们先看一下实现，然后理解它是如何工作的:

```
public class WorkerWithPhaser extends Thread {
    private Phaser phaser;

    public WorkerWithPhaser(String name, Phaser phaser) {
        this.phaser = phaser;
        phaser.register();
        setName(name);
    }

    @Override public void run() {
        try {
            System.out.printf("[ %s ] created, blocked by the phaser\n", getName());
            phaser.arriveAndAwaitAdvance();
            System.out.printf("[ %s ] starts at: %s\n", getName(), Instant.now());
            // do actual work here...
        } catch (IllegalStateException e) {
            // handle exception
        }
    }
}
```

当一个工作线程被实例化时，我们通过调用`phaser.register()`将当前线程注册到给定的`Phaser`对象。这样，当前的工作就成了`phaser`屏障的一个线程方。

接下来，当工作线程启动时，我们立即调用`phaser.arriveAndAwaitAdvance()`。因此，我们告诉`phaser`当前线程已经到达，将等待其他线程方的到达来继续。当然，在其他线程方到达之前，当前线程被阻塞。

### 5.2.主线程

接下来，让我们继续，看看主线程的实现:

```
private static void usingPhaser() throws InterruptedException {
    System.out.println("\n===============================================");
    System.out.println("        >>> Using Phaser <<<");
    System.out.println("===============================================");

    Phaser phaser = new Phaser();
    phaser.register();

    WorkerWithPhaser worker1 = new WorkerWithPhaser("Worker with phaser 1", phaser);
    WorkerWithPhaser worker2 = new WorkerWithPhaser("Worker with phaser 2", phaser);

    worker1.start();
    worker2.start();

    Thread.sleep(10);//simulation of some actual work

    System.out.println("-----------------------------------------------");
    System.out.println(" Now open the phaser barrier:");
    System.out.println("-----------------------------------------------");
    phaser.arriveAndAwaitAdvance();
}
```

在上面的代码中，我们可以看到，**主线程将自己注册为`Phaser`对象**的线程方。

在我们创建并阻塞了两个`worker`线程之后，主线程也调用`phaser.arriveAndAwaitAdvance()`。这样，我们打开 phaser 屏障，这样两个`worker`线程可以同时恢复。

最后，我们调用`main()`方法中的`usingPhaser()`方法:

```
===============================================
        >>> Using Phaser <<<
===============================================
[ Worker with phaser 1 ] created, blocked by the phaser
[ Worker with phaser 2 ] created, blocked by the phaser
-----------------------------------------------
 Now open the phaser barrier:
-----------------------------------------------
[ Worker with phaser 2 ] starts at: 2021-07-18T17:39:27.063523636Z
[ Worker with phaser 1 ] starts at: 2021-07-18T17:39:27.063523827Z 
```

同样，**两个工作线程同时启动`almost`。两个开始时间之间的差异小于两微秒**。

## 6.结论

在本文中，我们首先讨论了需求:“同时启动两个线程。”

接下来，我们讨论了同时启动三个线程的两种方法:使用`CountDownLatch`、`CyclicBarrier`和`Phaser`。

**他们的想法很相似，阻塞两个线程，试图让它们同时恢复执行。**

尽管这些方法不能保证两个线程同时启动，但对于现实世界中的大多数情况来说，结果已经非常接近和足够了。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220526054156/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)