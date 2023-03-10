# OpenJDK 项目织机

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/openjdk-project-loom>

## 1.概观

在本文中，我们将快速浏览一下 [Project Loom](https://web.archive.org/web/20220817024604/https://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html) 。本质上，**Project Loom 的主要目标是在 Java 中支持一个高吞吐量、轻量级的并发模型。**

## 2.项目织机

Project Loom 是 OpenJDK 社区将轻量级并发结构引入 Java 的一次尝试。到目前为止，Loom 的原型已经引入了 JVM 和 Java 库的变化。

虽然 Loom 还没有预定的发布，但我们可以在 [Project Loom 的 wiki](https://web.archive.org/web/20220817024604/https://wiki.openjdk.java.net/display/loom/Main) 上访问最近的原型。

在讨论 Loom 的各种概念之前，我们先讨论一下 Java 中当前的并发模型。

## 3.Java 的并发模型

目前，`Thread` 代表了 Java 中并发的核心抽象。这种抽象连同其他并发 API 使得编写并发应用程序变得容易。

然而，由于 Java 使用 [OS 内核](/web/20220817024604/https://www.baeldung.com/cs/os-kernel)线程来实现，它无法满足当今的并发需求。尤其有两个主要问题:

1.  `Threads `无法匹配域的并发单位的规模。例如，应用程序通常允许多达数百万的事务、用户或会话。但是，内核支持的线程数量要少得多。因此，对每个用户、事务或会话使用一个 **`T` `hread`通常是不可行的。**
2.  对于每个请求，大多数并发应用程序都需要线程间的同步。因此，操作系统线程之间会发生代价高昂的上下文切换。

这类问题的一个可能的解决方案是**使用异步并发 API**。常见的例子有`[CompletableFuture](/web/20220817024604/https://www.baeldung.com/java-completablefuture)`和 [RxJava](/web/20220817024604/https://www.baeldung.com/rx-java) 。假设这样的 API 不会阻塞内核线程，它会在 Java 线程之上给应用程序一个更细粒度的并发结构`.`

另一方面，**这样的 API 很难调试，也很难与传统 API**集成。因此，需要一种独立于内核线程的轻量级并发结构。

## 4.任务和调度程序

线程的任何实现，无论是轻量级的还是重量级的，都取决于两个构造:

1.  任务(也称为延续)–一个指令序列，它可以因某些阻塞操作而自行挂起
2.  调度程序–用于将延续分配给 CPU，以及从暂停的延续重新分配 CPU

目前， **Java 依赖于 OS 实现来继续和调度**。

现在，为了暂停 continuation，需要存储整个调用栈。类似地，在恢复时检索调用堆栈。由于 continuations 的操作系统实现包括本地调用栈和 Java 的调用栈，这导致了大量内存占用。

然而，一个更大的问题是操作系统调度程序的使用。由于调度程序在内核模式下运行，线程之间没有区别。它以同样的方式处理每个 CPU 请求。

这种类型的调度**对于 Java 应用程序来说不是最优的**。

例如，假设一个应用程序线程对请求执行一些操作，然后将数据传递给另一个线程进行进一步处理。这里，**最好将这两个线程安排在同一个 CPU** 上。但是由于调度程序对于请求 CPU 的线程是不可知的，这是不可能保证的。

Project Loom 提出通过**用户模式线程来解决这个问题，用户模式线程依赖于延续和调度器的 Java 运行时实现，而不是操作系统实现** `.`

## 5.纤维

在 OpenJDK 最近的原型中，一个名为`Fiber`的新类与`Thread`类一起被引入到库中。

由于`Fibers`的计划库与`Thread`相似，用户实现也应该保持相似。但是，有两个主要区别:

1.  `Fiber `会将任何任务包装在内部用户模式延续中吗？这将允许任务在 Java 运行时而不是内核中暂停和恢复
2.  将使用可插入的用户模式调度程序(例如`ForkJoinPool,`)

让我们详细检查一下这两项。

## 6.延续

延续(或协同例程)是一个指令序列，它可以被调用方在以后的阶段产生和恢复。

每个延续都有一个入口点和一个屈服点。屈服点是它被悬挂的地方。每当调用方恢复延续时，控件都返回到最后一个屈服点。

重要的是要认识到**这个挂起/恢复现在发生在语言运行时，而不是 OS** 。因此，它防止了内核线程之间代价高昂的上下文切换。

与线程类似，Project Loom 旨在支持嵌套纤维。因为纤程在内部依赖于延续，所以它也必须支持嵌套延续。为了更好地理解这一点，考虑一个允许嵌套的类`Continuation `:

```java
Continuation cont1 = new Continuation(() -> {
    Continuation cont2 = new Continuation(() -> {
        //do something
        suspend(SCOPE_CONT_2);
        suspend(SCOPE_CONT_1);
    });
});
```

如上所示，嵌套的延续可以通过传递一个作用域变量`. `来挂起自己或任何封闭的延续，因此，**它们被称为`scoped`延续。**

由于暂停延续还需要存储调用堆栈，所以 project Loom 的另一个目标是在恢复延续时添加轻量级堆栈检索。

## 7.调度程序

前面，我们讨论了 OS 调度程序在同一 CPU 上调度可相关线程的缺点。

虽然 Project Loom 的目标是允许使用纤程的可插拔调度器，但异步模式下的 **[`ForkJoinPool`](/web/20220817024604/https://www.baeldung.com/java-fork-join) 将被用作默认调度器。**

`ForkJoinPool `致力于**偷功算法**。因此，每个线程维护一个任务队列，并从它的头部执行任务。此外，任何空闲线程都不会阻塞，**等待任务，而是从另一个线程的队列尾部拉出它。**

异步模式中唯一的区别是**工作线程从另一个队列**的头部窃取任务。

`ForkJoinPool ` **将另一个正在运行的任务调度的任务添加到本地队列。因此，在同一个 CPU 上执行它。**

## 8.结论

在本文中，我们讨论了 Java 当前并发模型中的问题以及由 [Project Loom](https://web.archive.org/web/20220817024604/https://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html) 提出的改变。

在此过程中，我们还定义了任务和调度器，并研究了**纤程和 ForkJoinPool 如何使用内核线程提供 Java 的替代方案。**