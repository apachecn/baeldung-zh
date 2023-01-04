# 二元信号量与可重入锁

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-binary-semaphore-vs-reentrant-lock>

## 1.概观

在本教程中，我们将探索二进制信号量和可重入锁。此外，我们将对它们进行相互比较，看看哪一种最适合常见情况。

## 2.什么是二元信号量？

二进制信号量提供了访问单个资源的信号机制。换句话说，一个二进制信号量**提供了一个互斥机制，一次只允许一个线程访问一个临界区**。

为此，它只保留一个访问许可证。因此，**一个二元信号量只有两种状态:一个许可可用或零个许可可用**。

让我们讨论一个简单的二进制信号量的[实现，使用 Java 中可用的](/web/20220627182628/https://www.baeldung.com/java-semaphore#mutex) [`Semaphore`](https://web.archive.org/web/20220627182628/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Semaphore.html) 类:

```java
Semaphore binarySemaphore = new Semaphore(1);
try {
    binarySemaphore.acquire();
    assertEquals(0, binarySemaphore.availablePermits());
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    binarySemaphore.release();
    assertEquals(1, binarySemaphore.availablePermits());
}
```

在这里，我们可以观察到`acquire`方法将可用的许可减少了一个。类似地，`release`方法将可用许可增加一个。

另外，`Semaphore`类提供了`fairness`参数。当设置为`true`时，`fairness`参数确保请求线程获取许可的顺序(基于它们的等待时间):

```java
Semaphore binarySemaphore = new Semaphore(1, true);
```

## 3.什么是可重入锁？

[重入锁是一种互斥机制](/web/20220627182628/https://www.baeldung.com/java-concurrent-locks#lock-implementations)，**允许线程(多次)重新进入一个资源的锁，而不会出现死锁情况**。

进入锁的线程每增加一次持有计数。类似地，当请求解锁时，保持计数减少。因此，**资源被锁定，直到计数器归零**。

例如，让我们看一个使用 Java 中可用的 [`ReentrantLock`](https://web.archive.org/web/20220627182628/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html) 类的简单实现:

```java
ReentrantLock reentrantLock = new ReentrantLock();
try {
    reentrantLock.lock();
    assertEquals(1, reentrantLock.getHoldCount());
    assertEquals(true, reentrantLock.isLocked());
} finally {
    reentrantLock.unlock();
    assertEquals(0, reentrantLock.getHoldCount());
    assertEquals(false, reentrantLock.isLocked());
}
```

在这里， `lock` 方法将持有计数加 1 并锁定资源。类似地， `unlock` 方法减少保持计数，如果保持计数为零，则解锁资源。

当线程重新进入锁时，它必须请求解锁相同的次数来释放资源:

```java
reentrantLock.lock();
reentrantLock.lock();
assertEquals(2, reentrantLock.getHoldCount());
assertEquals(true, reentrantLock.isLocked());

reentrantLock.unlock();
assertEquals(1, reentrantLock.getHoldCount());
assertEquals(true, reentrantLock.isLocked());

reentrantLock.unlock();
assertEquals(0, reentrantLock.getHoldCount());
assertEquals(false, reentrantLock.isLocked());
```

类似于`Semaphore`类，`ReentrantLock`类也支持`fairness`参数:

```java
ReentrantLock reentrantLock = new ReentrantLock(true);
```

## 4.二元信号量与可重入锁

### 4.1.机制

**二元信号量是一种信号机制**，而可重入锁是一种锁定机制。

### 4.2.所有权

没有线程是二元信号量的所有者。然而，**最后一个成功锁定资源的线程是一个可重入锁**的所有者。

### 4.3.自然

二进制信号量本质上是不可重入的，这意味着同一个线程不能重新获取临界区，否则将导致死锁情况。

另一方面，可重入锁本质上允许同一线程多次重入一个锁。

### 4.4.灵活性

**二进制信号量通过允许锁定机制和死锁恢复的定制实现，提供了更高级别的同步机制**。因此，它给了开发者更多的控制权。

然而，**重入锁是带有固定锁机制**的低级同步。

### 4.5.修正

二进制信号量支持 wait 和 signal 之类的操作(在 Java 的`Semaphore`类中是获取和释放),允许任何进程修改可用的许可。

另一方面，只有锁定/解锁资源的同一个线程才能修改可重入锁。

### 4.6.死锁恢复

**二进制信号量提供非所有权释放机制**。因此，任何线程都可以释放对二进制信号量的死锁恢复的许可。

相反，在可重入锁的情况下，死锁恢复很难实现。例如，如果一个可重入锁的所有者线程进入睡眠或无限期等待，就不可能释放资源，就会导致死锁。

## 5.结论

在这篇短文中，我们探讨了二元信号量和可重入锁。

首先，我们讨论了二元信号量和可重入锁的基本定义，以及 Java 中的基本实现。然后，我们基于一些参数(如机制、所有权和灵活性)对它们进行了比较。

我们当然可以断定**二元信号量为互斥**提供了一个非基于所有权的信号机制。同时，它还可以进一步扩展，以提供锁定功能和简单的死锁恢复。

另一方面，**可重入锁提供了具有基于所有者的锁定能力的可重入互斥**，并且作为一个简单的互斥体是有用的。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627182628/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)