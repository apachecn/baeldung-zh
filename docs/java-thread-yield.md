# Java Thread.yield()简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-yield>

## 1.概观

在本教程中，我们将探索`Thread`类中的方法`yield()`。

我们将把它与 Java 中可用的其他并发习惯用法进行比较，并最终探索它的实际应用。

## 2.`yield()`简介

正如官方文档所建议的，`yield()`提供了一种机制来通知“调度程序”**当前线程愿意放弃其当前对处理器的使用，但它希望尽快被调度回来。**

“调度程序”可以自由地坚持或忽略这些信息，事实上，根据操作系统的不同，它会有不同的行为。

下面的代码片段显示了在每个调度后产生的相同优先级的两个线程:

```java
public class ThreadYield {
    public static void main(String[] args) {
        Runnable r = () -> {
            int counter = 0;
            while (counter < 2) {
                System.out.println(Thread.currentThread()
                    .getName());
                counter++;
                Thread.yield();
            }
        };
        new Thread(r).start();
        new Thread(r).start();
    }
}
```

当我们尝试多次运行上述程序时，我们得到不同的结果；其中一些提到如下:

运行 1:

```java
Thread-0
Thread-1
Thread-1
Thread-0
```

运行 2:

```java
Thread-0
Thread-0
Thread-1
Thread-1
```

如你所见,`yield()`的行为是不确定的，并且依赖于平台。

## 3.与其他成语相比

还有其他影响线程相对进程的结构。它们包括作为`Object`类一部分的`wait()`、 `notify()`和`notifyAll()`，作为`Thread`类一部分的`join()`，以及作为`Thread`类一部分的`sleep()`。

让我们看看他们与`yield()`相比如何。

### 3.1.`yield()`对`wait()`

*   当`yield()`在当前线程的上下文中被调用时，`wait()`只能在同步块或方法中显式获取的锁上被调用
*   与`yield()`不同，wait `()`可以指定在再次尝试调度线程之前等待的最小时间段
*   使用`wait()`,还可以通过调用相关锁对象上的`notify()`或`notifyAll()`,随时唤醒线程

### 3.2.`yield()`对`sleep()`

*   虽然`yield()`只能试探性地尝试暂停当前线程的执行，而不能保证它何时被调度回来，但是`sleep()`可以强制调度器至少在它的参数中提到的时间段内暂停当前线程的执行。

### 3.3.`yield()`对`join()`

*   当前线程可以在任何其他线程上调用`join()`,这使得当前线程在继续之前等待其他线程死亡
*   可选地，它可以将时间段作为其参数，该参数指示当前线程在恢复之前应该等待的最大时间

## 4.`yield()`的用法

正如官方文档所建议的，很少需要使用`yield()`，因此应该避免使用，除非根据它的行为对目标非常清楚。

尽管如此，`yield()`的一些用途包括设计并发控制结构，提高计算密集型程序的系统响应能力等。

然而，这些用法必须伴随着详细的概要分析和基准测试，以确保预期的结果。

## 5.结论

在这篇简短的文章中，我们讨论了`Thread`类中的`yield()`方法，并通过一个代码片段看到了它的行为和局限性。

我们还探索了它与 Java 中其他可用的并发习惯用法的比较，最后查看了一些`yield()`可能有用的用例。

和往常一样，你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20221130215453/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)