# Java 中的可变变量与原子变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-volatile-vs-atomic>

## 1.概观

在本教程中，我们将学习关键字和原子类的区别以及它们解决的问题。首先，有必要了解 Java 如何处理线程间的通信，以及会出现哪些[意料之外的问题](/web/20221005030132/https://www.baeldung.com/java-common-concurrency-pitfalls)。

[线程安全](/web/20221005030132/https://www.baeldung.com/java-thread-safety)是一个至关重要的话题，它让我们深入了解多线程应用的内部工作。我们还将讨论[的比赛条件](/web/20221005030132/https://www.baeldung.com/cs/race-conditions)，但我们不会深入这个话题。

## 2.并发问题

让我们举一个简单的例子来看看原子类和`volatile`关键字之间的区别。假设我们试图创建一个将在多线程环境中工作的计数器。

理论上，任何应用程序线程都可以增加这个计数器的值。让我们从一个简单的方法开始实现它，并检查会出现什么问题:

```java
public class UnsafeCounter {

    private int counter;

    int getValue() {
        return counter;
    }

    void increment() {
        counter++;
    }
}
```

这是一个非常好的计数器，但不幸的是，它只适用于单线程应用程序。这种方法在多线程环境中会遇到可见性和同步问题。在大型应用程序中，跟踪漏洞甚至破坏用户数据可能会造成困难。

## 3.能见度问题

[可见性问题](/web/20221005030132/https://www.baeldung.com/java-volatile)是在多线程应用中工作时的问题之一。可见性问题与 Java [内存模型](/web/20221005030132/https://www.baeldung.com/java-volatile#shared-multiprocessor-architecture)紧密相关。

在多线程应用程序中，每个线程都有其共享资源的缓存版本，并根据事件或时间表更新主内存中或主内存中的值。

**线程缓存和主内存值可能不同。**因此，即使一个线程更新了主存中的值，其他线程也不会立即看到这些变化。这被称为可见性问题。

**`volatile`关键字[帮助](/web/20221005030132/https://www.baeldung.com/java-volatile-variables-thread-safety)通过绕过本地线程中的缓存来解决这个问题。**因此，`volatile`变量对所有线程都是可见的，并且所有这些线程都将看到相同的值。因此，当一个线程更新该值时，所有线程都会看到新值。我们可以把它看作一个低级的观察者模式，并且可以重写前面的实现:

```java
public class UnsafeVolatileCounter {

    private volatile int counter;

    public int getValue() {
        return counter;
    }

    public void increment() {
        counter++;
    }
}
```

上面的例子改进了计数器，解决了可见性的问题。然而，我们仍然有一个同步问题，我们的计数器在多线程环境中不能正常工作。

## 4.同步问题

**虽然`volatile`关键词有助于我们提高可见性，但我们还有另一个问题。**在我们的增量示例中，我们用变量`count.`执行两个操作。首先，我们读取这个变量，然后给它赋一个新值。**这意味着增量操作不是原子的。**

**我们在这里面对的是一个[的竞争条件](/web/20221005030132/https://www.baeldung.com/cs/race-conditions#read-modify-write)T3。每个线程应该首先读取值，递增它，然后写回它。当几个线程开始处理这个值，并在另一个线程写入之前读取它时，就会发生[问题](/web/20221005030132/https://www.baeldung.com/java-testing-multithreaded#3-anatomy-of-thread-interleaving)。**

这样，一个线程可以覆盖另一个线程写的结果。关键字`[synchronized](/web/20221005030132/https://www.baeldung.com/java-synchronized)`可以解决这个问题。然而，这种方法可能会产生一个瓶颈，它不是解决这个问题的最佳方案。

## 5.原子值

原子值提供了一种更好更直观的方式来处理这个问题。它们的接口允许我们在没有同步问题的情况下交互和更新值。

在内部，原子类确保在这种情况下，增量将是原子操作。因此，我们可以用它来创建一个线程安全的实现:

```java
public class SafeAtomicCounter {
    private final AtomicInteger counter = new AtomicInteger(0);

    public int getValue() {
        return counter.get();
    }

    public void increment() {
        counter.incrementAndGet();
    }
}
```

我们最终的实现是线程安全的，可以在多线程应用中使用。这与我们的第一个例子没有太大的不同，只有通过使用原子类，我们才能解决多线程代码中的可见性和同步问题。

## 6.结论

在这篇文章中，我们了解到当我们在多线程环境中工作时应该非常小心。bug 和问题可能很难追踪，在调试时可能不会出现。这就是为什么了解 Java 如何处理这些情况是至关重要的。

**`volatile `关键字可以帮助解决可见性问题，并通过本质上的原子操作解决问题。**设置标志是`volatile` 关键字可能有用的例子之一。

原子变量有助于处理非原子操作，如递增-递减或任何需要在赋值前读取值的操作。原子值是解决我们代码中同步问题的一种简单方便的方法。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221005030132/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)