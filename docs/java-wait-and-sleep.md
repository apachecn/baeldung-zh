# Java 中等待和睡眠的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-wait-and-sleep>

## 1。概述

在这篇短文中，我们将看看 core Java 中的标准`sleep()`和`wait()`方法，并理解它们之间的区别和相似之处。

## 2。`Wait`与`Sleep`和的一般区别

简单来说， **`wait()`是一个用于线程同步的实例方法。**

它可以在任何对象上调用，因为它是在 `java.lang.Object,` 上定义的，但是**只能从同步块**中调用。它释放对象上的锁，以便另一个线程可以加入并获得锁。

另一方面，`Thread.sleep()`是一个静态方法，可以从任何上下文中调用。 **`Thread.sleep()`暂停当前线程，不释放任何锁。**

下面是对这两个核心 API 的简单介绍:

```java
private static Object LOCK = new Object();

private static void sleepWaitExamples() 
  throws InterruptedException {

    Thread.sleep(1000);
    System.out.println(
      "Thread '" + Thread.currentThread().getName() +
      "' is woken after sleeping for 1 second");

    synchronized (LOCK) {
        LOCK.wait(1000);
        System.out.println("Object '" + LOCK + "' is woken after" +
          " waiting for 1 second");
    }
} 
```

运行此示例将产生以下输出:

`Thread ‘main' is woken after sleeping for 1 second`


## 3。唤醒`Wait`和`Sleep`

当我们使用`sleep()`方法时，一个线程在指定的时间间隔后启动，除非它被中断。

对于`wait()`，唤醒过程稍微复杂一点。我们可以通过调用被等待的监视器上的`notify()` 或`notifyAll()` 方法来唤醒线程。

当您想要唤醒所有处于等待状态的线程时，请使用`notifyAll()`而不是`notify()`。类似于`wait()`方法本身，`notify()`和 `notifyAll()`必须从同步的上下文中调用。

例如，你可以这样做:

```java
synchronized (b) {
    while (b.sum == 0) {
        System.out.println("Waiting for ThreadB to complete...");
        b.wait();
    }

    System.out.println("ThreadB has completed. " + 
      "Sum from that thread is: " + b.sum);
}
```

然后，下面是另一个线程如何通过调用监视器上的`notify()`来**唤醒等待的线程:**

```java
int sum;

@Override 
public void run() {
    synchronized (this) {
        int i = 0;
        while (i < 100000) {
            sum += i;
            i++; 
        }
        notify(); 
    } 
}
```

运行此示例将产生以下输出:

`Waiting for ThreadB to complete…`
`ThreadB has completed.` 总和 `from that thread is: 704982704`

## 4。结论

这是 Java 中`wait`和`sleep`语义的快速入门。

一般来说，我们应该用`sleep()`来控制一个线程的执行时间，用`wait()`来控制多线程的同步。当然，在很好地理解了基础知识之后，还有很多东西需要探索。

和往常一样，你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20221006231509/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-2)