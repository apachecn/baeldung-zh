# Java 中线程和虚拟线程的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-virtual-thread-vs-thread>

## 1.介绍

在本教程中，我们将展示 Java 中传统线程和 [Project Loom](/web/20220719110719/https://www.baeldung.com/openjdk-project-loom) 中引入的虚拟线程之间的区别。

接下来，我们将分享虚拟线程和该项目引入的 API 的几个用例。

在我们开始之前，我们需要注意这个项目正在积极开发中。我们将在早期访问的 loom VM 上运行我们的示例:open JDK-15-loom+4-55 _ windows-x64 _ bin。

较新版本的构建可以自由更改和破坏当前的 API。也就是说，API 中已经有了一个重大的变化，因为以前使用的`java.lang.Fiber`类已经被移除，并被新的`java.lang.VirtualThread`类所取代。

## 2.线程与虚拟线程的高级概述

在高层，**线程由操作系统管理和调度，而虚拟线程由虚拟机**管理和调度。现在，**要创建一个新的内核线程，我们必须做一个系统调用，这是一个开销很大的操作**。

这就是为什么我们使用线程池，而不是根据需要重新分配和释放线程。接下来，如果我们想通过添加更多线程来扩展我们的应用程序，由于上下文切换及其内存占用，维护这些线程的成本可能会很高，并影响处理时间。

然后，通常，我们不想阻塞那些线程，这导致了非阻塞 I/O API 和异步 API 的使用，这可能会使我们的代码混乱。

相反，**虚拟线程由 JVM** 管理。因此，它们的**分配不需要系统调用**，它们的**不受操作系统上下文切换**的影响。此外，虚拟线程在载体线程上运行，载体线程是实际使用的内核线程。结果，由于我们摆脱了系统的上下文切换，我们可以产生更多这样的虚拟线程。

接下来，虚拟线程的一个关键属性是它们不会阻塞我们的载体线程。这样，阻塞一个虚拟线程就变成了一个更便宜的操作，因为 JVM 会调度另一个虚拟线程，让载体线程保持畅通。

最终，我们不需要寻求 NIO 或异步 API。这应该会产生可读性更强、更易于理解和调试的代码。然而，**continuation 可能会阻塞一个载体线程**——特别是，当一个线程调用一个本地方法并从那里执行阻塞操作时。

## 3.新线程生成器 API

在 Loom 中，我们在`Thread`类中获得了新的构建器 API，以及几个工厂方法。让我们看看如何创建标准和虚拟工厂，并利用它们来执行线程:

```java
Runnable printThread = () -> System.out.println(Thread.currentThread());

ThreadFactory virtualThreadFactory = Thread.builder().virtual().factory();
ThreadFactory kernelThreadFactory = Thread.builder().factory();

Thread virtualThread = virtualThreadFactory.newThread(printThread);
Thread kernelThread = kernelThreadFactory.newThread(printThread);

virtualThread.start();
kernelThread.start();
```

下面是上面运行的输出:

```java
Thread[Thread-0,5,main]
VirtualThread[<unnamed>,ForkJoinPool-1-worker-3,CarrierThreads]
```

这里，第一个条目是内核线程的标准`toString` 输出。

现在，我们在输出中看到虚拟线程没有名字，它在来自`CarrierThreads`线程组的 Fork-Join 池的一个工作线程上执行。

正如我们所见，无论底层实现如何，**API 都是相同的，这意味着我们可以轻松地在虚拟线程**上运行现有代码。

此外，我们不需要学习一个新的 API 来使用它们。

## 4.虚拟线程组成

它是一个**延续和一个调度器**，共同组成了一个虚拟线程。现在，我们的用户模式调度程序可能是`Executor`接口的任何实现。上面的例子告诉我们，默认情况下，我们在`ForkJoinPool`上运行。

现在，类似于内核线程——可以在 CPU 上执行，然后暂停，重新调度，然后恢复其执行——延续是一个执行单元，可以启动，然后暂停(让出)，重新调度，并以相同的方式从停止的地方恢复其执行，并且仍然由 JVM 管理，而不是依赖于操作系统。

注意，continuation 是一个低级 API，程序员应该使用更高级的 API，比如 builder API 来运行虚拟线程。

然而，为了展示它是如何在引擎盖下工作的，现在我们将继续进行我们的实验:

```java
var scope = new ContinuationScope("C1");
var c = new Continuation(scope, () -> {
    System.out.println("Start C1");
    Continuation.yield(scope);
    System.out.println("End C1");
});

while (!c.isDone()) {
    System.out.println("Start run()");
    c.run();
    System.out.println("End run()");
}
```

下面是上面运行的输出:

```java
Start run()
Start C1
End run()
Start run()
End C1
End run()
```

在这个例子中，我们运行了 continuation，并在某个时候决定停止处理。然后，一旦我们重新运行它，我们的延续从它停止的地方继续。根据输出，我们看到`run()`方法被调用了两次，但是 continuation 被启动了一次，然后在第二次运行时从停止的地方继续执行。

这就是 JVM 处理阻塞操作的方式。**一旦阻塞操作发生，延续将放弃，保持载体线程畅通。**

因此，我们的主线程为`run()`方法的调用堆栈创建了一个新的堆栈框架，并继续执行。然后，在延续产生之后，JVM 保存了其执行的当前状态。

接下来，主线程继续执行，就像`run()`方法返回并继续执行`while`循环一样。在第二次调用 continuation 的`run`方法后，JVM 将主线程的状态恢复到 continuation 已经放弃并完成执行的位置。

## 5.结论

在本文中，我们讨论了内核线程和虚拟线程之间的区别。接下来，我们展示了如何使用 Project Loom 的新线程构建器 API 来运行虚拟线程。

最后，我们展示了什么是延续以及它是如何工作的。我们可以通过检查[早期访问](https://web.archive.org/web/20220719110719/https://jdk.java.net/loom/) VM 来进一步探索 Project Loom 的状态。或者，我们可以探索更多已经标准化的 [Java 并发](/web/20220719110719/https://www.baeldung.com/java-concurrency)API。