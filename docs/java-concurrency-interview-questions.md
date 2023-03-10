# Java 并发面试问题(+答案)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrency-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20220812064259/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20220812064259/https://www.baeldung.com/java-type-system-interview-questions)
• Java Concurrency Interview Questions (+ Answers) (current article)[• Java Class Structure and Initialization Interview Questions](/web/20220812064259/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20220812064259/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20220812064259/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20220812064259/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20220812064259/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20220812064259/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20220812064259/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20220812064259/https://www.baeldung.com/spring-interview-questions)

## 1.介绍

Java 中的并发性是技术访谈中提出的最复杂、最高级的话题之一。这篇文章为你可能遇到的一些面试问题提供了答案。

### Q1。进程和线程的区别是什么？

进程和线程都是并发的单元，但是它们有一个基本的区别:进程不共享一个公共内存，而线程共享。

从操作系统的角度来看，进程是一个独立的软件，它运行在自己的虚拟内存空间中。任何多任务操作系统(这意味着几乎任何现代操作系统)都必须在内存中分离进程，以便一个失败的进程不会通过争夺公共内存来拖累所有其他进程。

因此，这些进程通常是隔离的，它们通过进程间通信的方式进行协作，操作系统将进程间通信定义为一种中间 API。

相反，线程是应用程序的一部分，它与同一应用程序的其他线程共享公共内存。使用公共内存可以减少大量开销，设计线程来协作，并在它们之间更快地交换数据。

### Q2。如何创建一个线程实例并运行它？

要创建线程的实例，有两种选择。首先，将一个`Runnable`实例传递给它的构造函数，并调用`start()`。`Runnable`是一个函数接口，所以可以作为 lambda 表达式传递:

```java
Thread thread1 = new Thread(() ->
  System.out.println("Hello World from Runnable!"));
thread1.start();
```

Thread 也实现了`Runnable`，所以另一种启动线程的方式是创建一个匿名子类，覆盖它的`run()`方法，然后调用`start()`:

```java
Thread thread2 = new Thread() {
    @Override
    public void run() {
        System.out.println("Hello World from subclass!");
    }
};
thread2.start();
```

### Q3。描述线程的不同状态以及状态转换发生的时间。

使用`Thread.getState()`方法可以检查`Thread`的状态。在`Thread.State`枚举中描述了`Thread`的不同状态。它们是:

*   **`NEW`** —一个尚未通过`Thread.start()`启动的新`Thread`实例
*   **`RUNNABLE`** —一条运行中的线程。它之所以被称为 runnable，是因为在任何给定的时间，它要么正在运行，要么正在等待来自线程调度器的下一个时间段。当您在一个 `NEW`线程上调用`Thread.start()`时，它会进入`RUNNABLE`状态

***   **`WAITING`** —如果一个线程等待另一个线程执行特定的动作，它就进入这种状态。例如，一个线程在它持有的监视器上调用`Object.wait()`方法，或者在另一个线程上调用`Thread.join()`方法时，就会进入这种状态*   **`TIMED_WAITING`**——同上，但线程在调用定时版本的`Thread.sleep()`、`Object.wait()`、`Thread.join()`等一些方法后进入此状态*   **`TERMINATED`** —线程已经完成其`Runnable.run()`方法的执行并终止**

 **### Q4。Runnable 和 Callable 接口有什么区别？它们是如何使用的？

`Runnable`接口只有一个 `run`方法。它代表一个必须在单独的线程中运行的计算单元。`Runnable`接口不允许这个方法返回值或者抛出未检查的异常。

`Callable`接口只有一个`call`方法，代表一个有值的任务。这就是为什么`call`方法返回值的原因。它也可以抛出异常。`Callable`一般在`ExecutorService`实例中使用，用来启动一个异步任务，然后调用返回的`Future`实例来获取它的值。

### Q5。什么是守护线程，它的用例是什么？如何创建一个守护线程？

守护线程是不阻止 JVM 退出的线程。当所有非守护进程线程终止时，JVM 简单地放弃所有剩余的守护进程线程。守护线程通常用于为其他线程执行一些支持或服务任务，但是您应该考虑到它们可能会在任何时候被放弃。

要启动一个线程作为守护进程，您应该在调用`start()`之前使用`setDaemon()`方法:

```java
Thread daemon = new Thread(()
  -> System.out.println("Hello from daemon!"));
daemon.setDaemon(true);
daemon.start();
```

奇怪的是，如果将此作为`main()`方法的一部分运行，消息可能不会被打印出来。如果`main()`线程在守护进程打印消息之前终止，就会发生这种情况。你通常不应该在守护线程中做任何 I/O，因为如果被放弃，它们甚至不能执行它们的`finally`块和关闭资源。

### Q6。线程的中断标志是什么？你如何设置和检查它？它与 Interruptedexception 有什么关系？

中断标志，或中断状态，是一个内部`Thread`标志，当线程被中断时被置位。要设置它，只需在线程对象`.`上调用`thread.interrupt()`

如果线程当前在抛出`InterruptedException` ( `wait`、`join`、`sleep`等的方法之一中。)，那么这个方法立即抛出 InterruptedException。线程可以根据自己的逻辑自由处理这个异常。

如果一个线程不在这个方法中，并且调用了`thread.interrupt()`,没有什么特别的事情发生。使用`static Thread.interrupted()`或实例`isInterrupted()`方法定期检查中断状态是线程的责任。这些方法的区别在于`static Thread.interrupted()`清除中断标志，而`isInterrupted()`不清除。

### Q7。什么是遗嘱执行人和执行服务？这些接口有什么区别？

`Executor`和`ExecutorService`是`java.util.concurrent`框架的两个相关接口。`Executor`是一个非常简单的接口，只有一个`execute`方法接受`Runnable`实例来执行。在大多数情况下，这是任务执行代码应该依赖的接口。

`ExecutorService`用多种方法扩展了`Executor`接口，用于处理和检查并发任务执行服务的生命周期(在关闭的情况下终止任务),以及用于更复杂的异步任务处理的方法，包括`Futures`。

关于使用`Executor`和`ExecutorService`的更多信息，请参见文章[Java ExecutorService 的指南](/web/20220812064259/https://www.baeldung.com/java-executor-service-tutorial)。

### Q8。标准库中 Executorservice 有哪些可用的实现？

`ExecutorService`接口有三个标准实现:

*   **`ThreadPoolExecutor`** —用于使用线程池执行任务。一旦一个线程完成了任务的执行，它就回到池中。如果池中的所有线程都很忙，那么任务必须等待轮到它。
*   **`ScheduledThreadPoolExecutor`** 允许调度任务执行，而不是在线程可用时立即运行。它还可以调度固定速率或固定延迟的任务。
*   **`ForkJoinPool`** 是处理递归算法任务的专用`ExecutorService`。如果你对递归算法使用常规的`ThreadPoolExecutor`,你会很快发现所有的线程都在忙于等待底层递归的完成。`ForkJoinPool`实现了所谓的工作窃取算法，允许它更有效地使用可用线程。

### Q9。什么是 Java 内存模型(Jmm)？描述它的目的和基本思想。

Java 内存模型是在[第 17.4 章](https://web.archive.org/web/20220812064259/https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)中描述的 Java 语言规范的一部分。它指定了多线程如何在并发 Java 应用程序中访问公共内存，以及一个线程的数据更改如何对其他线程可见。虽然 JMM 非常简短，但如果没有很强的数学背景，可能很难理解。

对内存模型的需求源于这样一个事实，即 Java 代码访问数据的方式并不是底层实际发生的方式。Java 编译器、JIT 编译器甚至 CPU 都可以对内存读写进行重新排序或优化，只要这些读写的可观察结果是相同的。

当您的应用程序扩展到多线程时，这可能会导致与直觉相反的结果，因为大多数优化都考虑了单线程的执行(跨线程优化器仍然非常难以实现)。另一个巨大的问题是，现代系统中的内存是多层的:处理器的多个内核可能会在缓存或读/写缓冲区中保存一些未刷新的数据，这也会影响从其他内核观察到的内存状态。

更糟糕的是，不同内存访问架构的存在会打破 Java“写一次，到处运行”的承诺。令程序员高兴的是，JMM 规定了一些设计多线程应用程序时可以依赖的保证。坚持这些保证有助于程序员编写稳定的多线程代码，并且可以在各种架构之间移植。

JMM 的主要观点是:

*   **动作**，这些是线程间的动作，可以由一个线程执行，由另一个线程检测，如读取或写入变量，锁定/解锁监视器等等
*   **同步动作**，动作的某个子集，如读/写`volatile`变量，或锁定/解锁监视器
*   **程序顺序** (PO)，单线程内可观察到的动作总顺序
*   **同步顺序** (SO)，所有同步动作之间的总顺序——它必须与程序顺序一致，即如果两个同步动作在 PO 中先后出现，它们在 SO 中以相同的顺序出现
*   **synchronizes-with** (SW)某些同步动作之间的关系，如监视器的解锁和同一监视器的锁定(在另一个或同一线程中)
*   **发生在顺序**之前——将 PO 和 SW(在集合论中称为`transitive closure`)结合起来，创建线程间所有动作的部分顺序。如果一个动作`happens-before`是另一个动作，那么第一个动作的结果可以被第二个动作观察到(例如，在一个线程中写变量，在另一个线程中读变量)
*   **发生在一致性之前** —如果每次读取都观察到以发生在一致性之前的顺序对该位置的最后一次写入，或者通过数据竞争观察到一些其他写入，则一组操作是 HB 一致性的
*   **执行** —某一组有序的动作以及它们之间的一致性规则

对于一个给定的程序，我们可以观察到具有不同结果的多个不同的执行。但是如果一个程序被**正确同步**，那么它的所有执行看起来都是**顺序一致的**，这意味着你可以把多线程程序理解为一组以某种顺序发生的动作。这省去了您考虑底层重新排序、优化或数据缓存的麻烦。

### Q10。什么是易变字段，Jmm 对此类字段有什么保证？

根据 Java 内存模型,`volatile`字段具有特殊属性(参见 Q9)。对`volatile`变量的读取和写入是同步动作，这意味着它们有一个总的顺序(所有线程将遵守这些动作的一致顺序)。根据这个顺序，对易失性变量的读取保证观察到对该变量的最后一次写入。

如果您有一个被多个线程访问的字段，至少有一个线程向它写入数据，那么您应该考虑将它设为`volatile`，否则就很难保证某个线程会从这个字段中读取什么。

`volatile`的另一个保证是读写 64 位值的原子性(`long`和`double`)。如果没有 volatile 修饰符，读取此类字段可能会观察到部分由另一个线程写入的值。

### Q11。以下哪些操作是原子操作？

*   写一个非`volatile``int`；
*   写一个`volatile int`；
*   `volatile long`写给一个非；
*   写一个`volatile long`；
*   增加一个`volatile long`？

对一个`int` (32 位)变量的写操作保证是原子的，不管它是否是`volatile`。一个`long` (64 位)变量可以分两步编写，例如，在 32 位架构上，所以默认情况下，没有原子性保证。但是，如果您指定了`volatile`修饰符，那么`long`变量肯定会被自动访问。

增量操作通常在多个步骤中完成(检索一个值，改变它并写回)，所以它永远不能保证是原子的，不管变量是否是`volatile`。如果你需要实现一个值的原子增量，你应该使用类`AtomicInteger`、`AtomicLong`等。

### Q12。Jmm 对类的 Final 字段有什么特殊的保证？

JVM 基本上保证类的`final`字段将在任何线程获得对象之前被初始化。如果没有这种保证，由于重新排序或其他优化，对一个对象的引用可能会在该对象的所有字段被初始化之前被发布，即变得对另一个线程可见。这可能会导致对这些字段的恶意访问。

这就是为什么当创建一个不可变的对象时，你应该总是使它的所有字段`final`，即使它们不能通过 getter 方法访问。

### Q13。方法定义中的 Synchronized 关键字是什么意思？静态方法吗？在一个街区之前？

块前的`synchronized`关键字意味着任何进入该块的线程都必须获取监视器(括号中的对象)。如果监视器已经被另一个线程获取，前一个线程将进入`BLOCKED`状态，并等待直到监视器被释放。

```java
synchronized(object) {
    // ...
}
```

一个`synchronized`实例方法具有相同的语义，但是实例本身充当监视器。

```java
synchronized void instanceMethod() {
    // ...
}
```

对于一个`static synchronized`方法，监视器是代表声明类的`Class`对象。

```java
static synchronized void staticMethod() {
    // ...
}
```

### Q14。如果两个线程同时调用不同对象实例上的同步方法，其中一个线程会阻塞吗？如果方法是静态的呢？

如果该方法是实例方法，则该实例充当该方法的监视器。在不同实例上调用该方法的两个线程获取不同的监视器，因此它们都不会被阻塞。

如果方法是`static`，那么监视器就是`Class`对象。对于两个线程，监视器是相同的，因此其中一个线程可能会阻塞并等待另一个线程退出`synchronized`方法。

### Q15。Object 类的 Wait、Notify 和 Notifyall 方法的用途是什么？

拥有对象监视器的线程(例如，已经进入由对象保护的`synchronized`段的线程)可以调用`object.wait()`来临时释放监视器，并给其他线程一个获取监视器的机会。例如，这可以是为了等待某个条件。

当获得监视器的另一个线程满足条件时，它可以调用`object.notify()`或`object.notifyAll()`并释放监视器。`notify`方法唤醒处于等待状态的单个线程，`notifyAll`方法唤醒所有等待这个监视器的线程，它们都竞争重新获取锁。

下面的`BlockingQueue`实现展示了多线程如何通过`wait-notify`模式协同工作。如果我们将一个元素`put`到一个空队列中，所有在`take`方法中等待的线程都会醒来并尝试接收这个值。如果我们`put`一个元素进入一个满队列，那么`put`方法`wait` s 用于调用`get`方法。`get`方法删除一个元素，并通知在`put`方法中等待的线程，队列中有一个空位置用于存放新的项目。

```java
public class BlockingQueue<T> {

    private List<T> queue = new LinkedList<T>();

    private int limit = 10;

    public synchronized void put(T item) {
        while (queue.size() == limit) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        if (queue.isEmpty()) {
            notifyAll();
        }
        queue.add(item);
    }

    public synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        if (queue.size() == limit) {
            notifyAll();
        }
        return queue.remove(0);
    }

}
```

### Q16。描述死锁、活锁和饥饿的情况。描述这些情况的可能原因。

**死锁**是一组线程中无法取得进展的情况，因为该组中的每个线程都必须获取该组中另一个线程已经获取的一些资源。最简单的情况是，当两个线程需要同时锁定两个资源才能继续运行时，第一个资源已经被一个线程锁定，第二个资源被另一个线程锁定。这些线程将永远不会获得两个资源的锁，因此永远不会取得进展。

**活锁**是一种多线程对自身产生的条件或事件做出反应的情况。一个事件发生在一个线程中，必须由另一个线程处理。在这个处理过程中，一个新的事件发生，它必须在第一个线程中处理，依此类推。这样的线程是活跃的，没有被阻塞，但是仍然没有取得任何进展，因为它们用无用的工作压倒了彼此。

**饥饿**是一个线程不能获得资源的情况，因为其他线程(或多个线程)占用它太长时间或具有更高的优先级。线程无法取得进展，因此无法完成有用的工作。

### Q17。描述 Fork/Join 框架的用途和用例。

fork/join 框架允许并行化递归算法。使用类似于`ThreadPoolExecutor`的东西来并行化递归的主要问题是，您可能会很快耗尽线程，因为每个递归步骤都需要自己的线程，而堆栈中的线程将处于空闲和等待状态。

fork/join 框架入口点是`ForkJoinPool`类，它是`ExecutorService`的一个实现。它实现了工作窃取算法，空闲线程试图从繁忙线程中“窃取”工作。这允许在不同的线程之间进行计算，并在使用比普通线程池更少的线程的情况下取得进展。

关于 fork/join 框架的更多信息和代码示例可以在文章[“Java 中的 Fork/Join 框架指南”](/web/20220812064259/https://www.baeldung.com/java-fork-join)中找到。

Next **»**[Java Class Structure and Initialization Interview Questions](/web/20220812064259/https://www.baeldung.com/java-classes-initialization-questions)**«** Previous[Java Type System Interview Questions](/web/20220812064259/https://www.baeldung.com/java-type-system-interview-questions)**