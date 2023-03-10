# Java 中原子变量的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-atomic-variables>

## 1。简介

简单地说，当涉及到并发性时，共享的可变状态很容易导致问题。如果对共享可变对象的访问管理不当，应用程序很快就会出现一些难以检测的并发错误。

在本文中，我们将再次讨论使用锁来处理并发访问，探索与锁相关的一些缺点，最后引入原子变量作为替代。

## 2。锁

让我们来看看这个班级:

```java
public class Counter {
    int counter; 

    public void increment() {
        counter++;
    }
}
```

在单线程环境的情况下，这可以完美地工作；然而，一旦我们允许多个线程写入，我们就开始得到不一致的结果。

这是因为简单的增量操作(`counter++`)，它看起来像一个原子操作，但实际上是三个操作的组合:获取值、增量和写回更新的值。

如果两个线程试图同时获取和更新值，可能会导致更新丢失。

管理对象访问的方法之一是使用锁。这可以通过在`increment`方法签名中使用`synchronized`关键字来实现。`synchronized`关键字确保一次只有一个线程可以进入该方法(要了解更多关于锁定和同步的信息，请参考–[Java 同步关键字指南](/web/20221120182145/https://www.baeldung.com/java-synchronized)):

```java
public class SafeCounterWithLock {
    private volatile int counter;

    public synchronized void increment() {
        counter++;
    }
}
```

此外，我们需要添加关键字`volatile`来确保线程间适当的引用可见性。

使用锁解决了这个问题。然而，性能受到了影响。

当多个线程试图获取一个锁时，其中一个线程获胜，而其余的线程要么被阻塞，要么被挂起。

**挂起然后恢复线程的过程非常昂贵**并且会影响系统的整体效率。

在一个小程序中，比如`counter`，花费在上下文切换上的时间可能会比实际的代码执行多得多，从而大大降低整体效率。

## 3。原子操作

有一个研究分支致力于为并发环境创建非阻塞算法。这些算法利用低级原子机器指令，如比较和交换(CAS)，以确保数据完整性。

典型的 CAS 操作对三个操作数起作用:

1.  要操作的内存位置(M)
2.  变量的现有期望值(A)
3.  需要设置的新值(B)

CAS 操作自动地将 M 中的值更新到 B，但仅当 M 中的现有值与 A 匹配时，否则不采取任何行动。

在这两种情况下，都会返回 M 中的现有值。这将三个步骤——获取值、比较值和更新值——结合到一个机器级别的操作中。

当多个线程试图通过 CAS 更新同一个值时，其中一个会胜出并更新该值。然而，与锁的情况不同，没有其他线程被挂起；相反，他们只是被告知他们没有更新这个值。然后线程可以继续做进一步的工作，完全避免了上下文切换。

另一个后果是核心程序逻辑变得更加复杂。这是因为我们必须处理 CAS 操作失败的情况。我们可以一次又一次地重试，直到成功，或者我们可以什么都不做，继续前进，这取决于用例。

## 4。Java 中的原子变量

Java 中最常用的原子变量类有 [AtomicInteger](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html) 、[atomic loning](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicLong.html)、 [AtomicBoolean](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicBoolean.html) 和 [AtomicReference](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html) 。这些类分别代表一个`int`、`long`、`boolean, `和对象引用，它们可以被自动更新。这些类公开的主要方法有:

*   `get()`–从内存中获取值，以便其他线程所做的更改可见；相当于读取一个`volatile`变量
*   `set()`–将值写入内存，以便其他线程可以看到该变化；相当于写一个`volatile`变量
*   `lazySet()`–最终将值写入存储器，可能会随后的相关存储器操作重新排序。一个用例是为了垃圾收集而使引用无效，这些引用将不再被访问。在这种情况下，通过延迟空`volatile`写操作可以获得更好的性能
*   `compareAndSet()`–同第 3 节所述，成功时返回真，否则返回假
*   `weakCompareAndSet()`–与第 3 节所述相同，但在某种意义上更弱，即它不创建发生前排序。这意味着它不一定会看到对其他变量的更新。[从 Java 9](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html#weakCompareAndSet(int,int)) 开始，这个方法在所有原子实现中都被弃用，取而代之的是 [`weakCompareAndSetPlain()`](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html#weakCompareAndSetPlain(int,int)) 。`weakCompareAndSet() `的记忆效应是显而易见的，但它的名字暗示了易变的记忆效应。为了避免这种混乱，他们弃用了这种方法，并添加了四种具有不同记忆效果的方法，如`weakCompareAndSetPlain() `或 [`weakCompareAndSetVolatile()`](https://web.archive.org/web/20221120182145/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html#weakCompareAndSetVolatile(int,int))

用`AtomicInteger`实现的线程安全计数器如下例所示:

```java
public class SafeCounterWithoutLock {
    private final AtomicInteger counter = new AtomicInteger(0);

    public int getValue() {
        return counter.get();
    }
    public void increment() {
        while(true) {
            int existingValue = getValue();
            int newValue = existingValue + 1;
            if(counter.compareAndSet(existingValue, newValue)) {
                return;
            }
        }
    }
}
```

如您所见，我们重试了`compareAndSet`操作，并在失败时再次尝试，因为我们希望保证对`increment`方法的调用总是将值增加 1。

## 5。结论

在这篇快速教程中，我们描述了一种处理并发的替代方法，这种方法可以避免与锁定相关的缺点。我们还看了 Java 中原子变量类公开的主要方法。

和往常一样，这些例子都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221120182145/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)

要了解更多内部使用非阻塞算法的类，请参考[concurrent map 指南](/web/20221120182145/https://www.baeldung.com/java-concurrent-map)。