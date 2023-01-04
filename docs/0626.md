# 线程与单线程执行器服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-single-thread-executor-service>

## 1。 **概述**

线程和执行器框架是 Java 中用来并行执行代码的两种机制。这提高了应用程序的性能。Executor 框架提供了不同种类的线程池。其中一个池只包含一个工作线程。

在本教程中，我们将了解一个线程和一个只有一个工作线程的执行器服务之间的区别。

## 2.线

线程是一个轻量级的进程，有独立的执行路径。它用于并行执行任务。因此，可以有多个线程同时运行，而不会相互干扰。

一个`Thread`对象执行`Runnable `任务。

让我们看看如何创建线程。**我们可以通过** **[扩展`Thread class `或者实现`Runnable` 接口](/web/20221116222756/https://www.baeldung.com/java-runnable-vs-extending-thread)来创建线程。**

让我们通过扩展`Thread`类来创建一个线程:

```
public class CustomThread extends Thread {
    // override the run() method to provide custom implementation

    public static void main(String[] args) { 
        CustomThread t1 = new CustomThread();
        t1.start(); 
    } 
}
```

在上面的例子中，`CustomThread`类扩展了`Thread` 类。在`main() `方法中，我们创建了`CustomThread` 类的对象，然后调用它的`start() `方法。它开始执行线程`.`

现在让我们看一个通过实现`Runnable `接口创建线程的例子:

```
public class TestClass implements Runnable {
    // implement the run() method of Runnable interface

    public static void main(String[] args) {
        TestClass testClassRef = new TestClass();
        Thread t1 = new Thread(testClassRef);
        t1.start();
    }
}
```

在上面的例子中，`TestClass` 实现了`Runnable `接口。我们在`Thread `类的构造函数中传递对`TestClass `对象的引用。然后，我们称之为`start() `法。这又调用了由`TestClass`实现的`run() `方法。

## 3.执行者框架

现在我们将学习[执行者框架](/web/20221116222756/https://www.baeldung.com/java-executor-service-tutorial)。它是在 JDK 1.5 `.` **中引入的，它是一个多线程框架，维护一个工作线程池并管理它们。**任务在队列中提交，然后由这些工作线程执行。

它消除了在代码中显式创建线程的开销。相反，它重用池中的线程来异步执行任务。

现在让我们看看由 Executor 框架维护的不同种类的[线程池](/web/20221116222756/https://www.baeldung.com/thread-pool-java-and-guava)。

### 3.1.固定线程池

这个池包含固定数量的线程。我们在创建池的过程中指定线程的数量。如果发生异常，线程被终止，就会创建一个新的线程。

让我们看看如何创建固定线程池:

```
ExecutorService executorService = Executors.newFixedThreadPool(5);
```

在上面的代码片段中，我们创建了一个包含五个工作线程的固定线程池。

### 3.2.缓存线程池

这个线程池在需要时创建新的线程。如果没有线程可以执行提交的任务，那么将创建一个新线程。

下面是我们创建缓存线程池的方法:

```
ExecutorService executorService = Executors.newCachedThreadPool();
```

在缓存线程池中，我们不提池的大小。这是因为当没有线程可用于执行提交的任务时，它会创建新的线程。当已经创建的线程可用时，它还会重用它们。

### 3.3.调度线程池

这个线程池在给定的延迟之后或者周期性地运行任务。

下面是我们如何创建一个调度线程池:

`ScheduledExecutorService executorService = Executors.newScheduledThreadPool(5);`

在上面的代码片段中，整数参数是核心池大小。它表示要保留在池中的线程数，即使它们是空闲的。

### 3.4.单线程池

这个池只包含一个线程。它按顺序执行提交的任务。如果发生异常，线程被终止，就会创建一个新线程。

下面的代码片段显示了如何创建单线程池:

`ExecutorService executorService = Executors.newSingleThreadExecutor();`

这里，`Executors `类的`static`方法`newSingleThreadExecutor()`创建了由单个工作线程组成的`ExecutorService `。

## 4.线程与单线程执行器服务

我们可能会想，如果一个线程池`ExecutorService`只包含一个线程，那么它与显式创建一个线程并使用它来执行任务有什么不同。

现在让我们来探讨一下线程和只有一个工作线程的 executor 服务之间的区别，以及何时使用哪一个。

### 4.1.任务处理

**线程只能处理`Runnable`任务，而单个线程执行器服务可以同时执行`Runnable`和`Callable`任务。**因此，使用这个，我们也可以运行能够返回一些值的任务。

`ExecutorService `接口中的`submit() `方法接受一个`Callable` 任务或一个`Runnable `任务并返回一个`Future o`对象。此对象表示异步任务的结果。

此外，一个线程只能处理一个任务并退出。但是单线程执行器服务可以处理一系列任务，并按顺序执行它们。

### 4.2.线程创建开销

创建线程会带来一些开销。例如，JVM 需要分配内存。当在代码中重复创建线程时，它会影响性能。但是在单线程执行器服务的情况下，相同的工作线程被重用。因此，**防止了创建多线程**的开销。

### 4.3.内存消耗

线程对象占用大量内存。因此，如果我们为每个异步任务创建线程，就可以导致 `OutOfMemoryError`。但是在单线程执行器服务中，相同的工作线程被重用，导致内存消耗更少。

### 4.4.资源的释放

一旦执行完成，线程就释放资源。但是对于 executor 服务，我们需要关闭服务，否则 JVM 将无法关闭。**像`shutdown() `和`shutdownNow()` 这样的方法关闭执行器服务。**

## 5.结论

在本文中，我们学习了线程、执行器框架和不同种类的线程池。我们还看到了线程和单线程执行器服务之间的差异。

我们了解到，如果有重复的工作或者有许多异步任务，那么 executor 服务是更好的选择。

像往常一样，这些例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221116222756/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-2)