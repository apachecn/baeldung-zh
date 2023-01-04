# java.util.concurrent.Locks 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrent-locks>

## 1。概述

简单地说，锁是一种比标准的`synchronized`块更加灵活和复杂的线程同步机制。

从 Java 1.5 开始， `Lock` 接口就已经存在了。它被定义在`java.util.concurrent.lock` 包中，并为锁定提供广泛的操作。

在本教程中，我们将探索`Lock`接口的不同实现及其应用。

## 2。锁和同步块的区别

使用同步的`block`和使用`Lock`API 之间有一些区别:

*   **一个`synchronized` `block`完全包含在一个方法内。我们可以在不同的方法中拥有`Lock`API`lock()`和`unlock()`操作。**
*   A s `ynchronized block`不支持公平。一旦释放，任何线程都可以获得锁，并且不需要指定优先级。**我们可以通过指定`fairness`属性来实现`Lock`API 中的公平性。**它确保等待时间最长的线程可以访问锁。
*   如果一个线程不能访问同步的`block`，它就会被阻塞。**`Lock`API 提供了`tryLock()` 方法。只有当锁可用并且没有被任何其他线程持有时，线程才会获得锁。**这减少了线程等待锁的阻塞时间。
*   处于“等待”状态以获取对`synchronized block`的访问的线程不能被中断。**`Lock`API 提供了一个方法`lockInterruptibly()`,可以用来在线程等待锁的时候中断线程。**

## 3。`Lock` API

让我们来看看`Lock`界面中的方法:

*   `**void lock()**` –如果锁可用，获取锁。如果锁不可用，线程将被阻塞，直到锁被释放。
*   **`void lockInterruptibly()`**——这个类似于`lock()`，但是它允许被阻塞的线程通过抛出`java.lang.InterruptedException`来中断和恢复执行。
*   **`boolean tryLock()`**–这是`lock()` 方法的非阻塞版本。它试图立即获取锁，如果锁定成功，则返回 true。
*   `**boolean tryLock(long timeout, TimeUnit timeUnit)**` –这类似于`tryLock()`，除了它在放弃尝试获取`Lock`之前等待给定的超时。
*   **void `unlock()`** 解锁`Lock`实例。

锁定的实例应该总是解锁的，以避免死锁情况。

使用锁的推荐代码块应该包含一个`try/catch`和`finally`块:

```
Lock lock = ...; 
lock.lock();
try {
    // access to the shared resource
} finally {
    lock.unlock();
}
```

除了`Lock` 接口，我们还有一个`ReadWriteLock`接口维护一对锁，一个用于只读操作，一个用于写操作。只要没有写操作，读锁就可以被多个线程同时持有。

`ReadWriteLock` 声明获取读或写锁的方法:

*   返回用于读取的锁。
*   **`Lock writeLock()`** 返回用于书写的锁。

## 4。锁实现

### 4.1。`ReentrantLock`

`ReentrantLock` 类实现了`Lock` 接口。它提供了与使用`synchronized`方法和语句访问的隐式监控锁相同的并发性和内存语义，并具有扩展的功能。

让我们看看如何使用`ReentrantLock`进行同步:

```
public class SharedObject {
    //...
    ReentrantLock lock = new ReentrantLock();
    int counter = 0;

    public void perform() {
        lock.lock();
        try {
            // Critical section here
            count++;
        } finally {
            lock.unlock();
        }
    }
    //...
}
```

我们需要确保在`try-finally`块中包装`lock()`和`unlock()`调用，以避免死锁情况。

让我们看看`tryLock()` 是如何工作的:

```
public void performTryLock(){
    //...
    boolean isLockAcquired = lock.tryLock(1, TimeUnit.SECONDS);

    if(isLockAcquired) {
        try {
            //Critical section here
        } finally {
            lock.unlock();
        }
    }
    //...
} 
```

在这种情况下，调用`tryLock()` 的线程将等待一秒钟，如果锁不可用，它将放弃等待。

### 4.2。`ReentrantReadWriteLock`

`ReentrantReadWriteLock`类实现了`ReadWriteLock`接口。

让我们看看线程获取`ReadLock` 或 `WriteLock` 的规则:

*   **读锁**–如果没有线程获得写锁或请求写锁，多个线程可以获得读锁。
*   **写锁**–如果没有线程在读或写，只有一个线程可以获得写锁。

让我们看看如何利用`ReadWriteLock`:

```
public class SynchronizedHashMapWithReadWriteLock {

    Map<String,String> syncHashMap = new HashMap<>();
    ReadWriteLock lock = new ReentrantReadWriteLock();
    // ...
    Lock writeLock = lock.writeLock();

    public void put(String key, String value) {
        try {
            writeLock.lock();
            syncHashMap.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
    ...
    public String remove(String key){
        try {
            writeLock.lock();
            return syncHashMap.remove(key);
        } finally {
            writeLock.unlock();
        }
    }
    //...
}
```

对于这两种写方法，我们需要用写锁包围临界区—只有一个线程可以访问它:

```
Lock readLock = lock.readLock();
//...
public String get(String key){
    try {
        readLock.lock();
        return syncHashMap.get(key);
    } finally {
        readLock.unlock();
    }
}

public boolean containsKey(String key) {
    try {
        readLock.lock();
        return syncHashMap.containsKey(key);
    } finally {
        readLock.unlock();
    }
}
```

对于这两种读方法，我们需要用读锁包围临界区。如果没有正在进行的写操作，多个线程可以访问这个部分。

### 4.3。`StampedLock`

`StampedLock`是 Java 8 中引入的。它还支持读写锁。

但是，锁获取方法返回一个标记，用于释放锁或检查锁是否仍然有效:

```
public class StampedLockDemo {
    Map<String,String> map = new HashMap<>();
    private StampedLock lock = new StampedLock();

    public void put(String key, String value){
        long stamp = lock.writeLock();
        try {
            map.put(key, value);
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    public String get(String key) throws InterruptedException {
        long stamp = lock.readLock();
        try {
            return map.get(key);
        } finally {
            lock.unlockRead(stamp);
        }
    }
}
```

`StampedLock`提供的另一个特性是乐观锁定。大多数时候，读操作不需要等待写操作完成，因此，不需要完全成熟的读锁。

相反，我们可以升级到读锁:

```
public String readWithOptimisticLock(String key) {
    long stamp = lock.tryOptimisticRead();
    String value = map.get(key);

    if(!lock.validate(stamp)) {
        stamp = lock.readLock();
        try {
            return map.get(key);
        } finally {
            lock.unlock(stamp);               
        }
    }
    return value;
}
```

## 5。与`Condition` s 一起工作

`Condition`类为线程提供了在执行临界区时等待某些条件发生的能力。

当线程获得对临界区的访问权，但没有执行其操作的必要条件时，就会发生这种情况。例如，一个读线程可以访问一个共享队列的锁，而这个队列仍然没有任何数据可以使用。

传统上，Java 提供了`wait()`、`notify()`和`notifyAll()` 方法用于线程间的相互通信。

有相似的机制，但是我们也可以指定多个条件:

```
public class ReentrantLockWithCondition {

    Stack<String> stack = new Stack<>();
    int CAPACITY = 5;

    ReentrantLock lock = new ReentrantLock();
    Condition stackEmptyCondition = lock.newCondition();
    Condition stackFullCondition = lock.newCondition();

    public void pushToStack(String item){
        try {
            lock.lock();
            while(stack.size() == CAPACITY) {
                stackFullCondition.await();
            }
            stack.push(item);
            stackEmptyCondition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String popFromStack() {
        try {
            lock.lock();
            while(stack.size() == 0) {
                stackEmptyCondition.await();
            }
            return stack.pop();
        } finally {
            stackFullCondition.signalAll();
            lock.unlock();
        }
    }
}
```

## 6。结论

在本文中，我们看到了`Lock`接口和新引入的`StampedLock` 类的不同实现。

我们还探索了如何利用`Condition`类来处理多种条件。

这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220929100635/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced)