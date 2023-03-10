# 用 Java 中的一个键获取一个锁

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-acquire-lock-by-key>

## 1.概观

在本文中，我们将了解如何锁定一个特定的键，以防止对该键的并发操作，而不妨碍对其他键的操作。

一般来说，我们希望实现两种方法，并了解如何操作它们:

*   `void lock(String key)`
*   `void unlock(String key)`

为了教程的简单，我们总是假设我们的键是`Strings`。你可以用你需要的对象类型替换它们，唯一的条件是`equals`和`hashCode`方法被正确定义，因为我们将把它们用作 [`HashMap`](/web/20220524061956/https://www.baeldung.com/java-hashmap) 键。

## 2.一个简单的互斥锁

首先，假设我们想要阻止任何请求的动作，如果相应的键已经被使用。这里，我们宁愿定义一个`boolean tryLock(String key)`方法，而不是我们想象的`lock`方法。

具体来说，我们的目标是维护一个`Set`键，我们将随时用使用中的键来填充它。因此，当在一个键上请求一个新的动作时，如果我们发现这个键已经被另一个线程使用了，我们只能拒绝它。

这里我们面临的问题是`Set`没有线程安全的实现。于是，我们就用一个`Set`撑腰一个 [`ConcurrentHashMap`](/web/20220524061956/https://www.baeldung.com/java-concurrent-map) 。使用`ConcurrentHashMap` 保证了多线程环境中的数据一致性。

让我们来看看实际情况:

```java
public class SimpleExclusiveLockByKey {

    private static Set<String> usedKeys= ConcurrentHashMap.newKeySet();

    public boolean tryLock(String key) {
        return usedKeys.add(key);
    }

    public void unlock(String key) {
        usedKeys.remove(key);
    }

}
```

下面是我们如何使用这个类:

```java
String key = "key";
SimpleExclusiveLockByKey lockByKey = new SimpleExclusiveLockByKey();
try {
    lockByKey.tryLock(key);
    // insert the code that needs to be executed only if the key lock is available
} finally { // CRUCIAL
    lockByKey.unlock(key);
}
```

**让我们坚持 [`finally`](/web/20220524061956/https://www.baeldung.com/java-finally-keyword) 块的存在:调用其中的`unlock` 方法是至关重要的。**这样，即使我们的代码在`try` 括号内抛出了一个`Exception`，我们也会解锁密钥。

## 3.通过钥匙获取和释放锁

现在，让我们更深入地研究这个问题，并说我们不想简单地拒绝在同一个键上的同时动作，但我们宁愿让新的输入动作等到该键上的当前动作完成。

申请流程如下:

*   第一个线程请求一个键上的锁:它获得键上的锁
*   第二个线程请求同一把钥匙上的锁:线程 2 被告知等待
*   第一个线程释放键上的锁
*   第二个线程获得键上的锁，并可以执行它的动作

### 3.1.用线程计数器定义锁

在这种情况下，用一个 [`Lock`](/web/20220524061956/https://www.baeldung.com/java-concurrent-locks) 听起来很自然。简而言之，`Lock `是一个用于线程同步的对象，它允许阻塞线程直到它被获取。`Lock`是一个接口——我们将使用一个`ReentrantLock`，它的基本实现。

让我们从在内部类中包装我们的`Lock`开始。这个类将能够跟踪当前等待锁定该项的线程数。它将公开两个方法，一个用于递增线程计数器，另一个用于递减它:

```java
private static class LockWrapper {
    private final Lock lock = new ReentrantLock();
    private final AtomicInteger numberOfThreadsInQueue = new AtomicInteger(1);

    private LockWrapper addThreadInQueue() {
        numberOfThreadsInQueue.incrementAndGet(); 
        return this;
    }

    private int removeThreadFromQueue() {
        return numberOfThreadsInQueue.decrementAndGet(); 
    }

}
```

### 3.2.让锁处理排队线程

此外，我们将继续使用一个`ConcurrentHashMap`。但是，我们将使用`LockWrapper `对象作为值，而不是像以前那样简单地提取`Map` 的键:

```java
private static ConcurrentHashMap<String, LockWrapper> locks = new ConcurrentHashMap<String, LockWrapper>(); 
```

当一个线程想要获得一个键的锁时，我们需要查看这个键是否已经存在一个`LockWrapper` :

*   如果没有，我们将为给定的键实例化一个`new LockWrapper` ,计数器设置为 1
*   如果是这样，我们将返回现有的`LockWrapper` 并增加其相关的计数器

让我们看看这是如何做到的:

```java
public void lock(String key) {
    LockWrapper lockWrapper = locks.compute(key, (k, v) -> v == null ? new LockWrapper() : v.addThreadInQueue());
    lockWrapper.lock.lock();
}
```

由于使用了`HashMap`的`compute`方法，代码非常简洁。让我们详细介绍一下这种方法的功能:

*   将`compute`方法应用于以`key`作为第一个参数的对象`locks` :获取对应于`locks`中`key` 的初始值
*   作为`compute`的第二个参数给出的`[BiFunction](/web/20220524061956/https://www.baeldung.com/java-bifunction-interface)`应用于`key`和初始值:结果给出一个新值
*   新值替换了`locks`中键`key`的初始值

### 3.3.解锁并选择性地移除地图条目

此外，当一个线程释放一个锁时，我们将减少与`LockWrapper`相关的线程数量。如果计数下降到零，那么我们将从`ConcurrentHashMap`中移除密钥:

```java
public void unlock(String key) {
    LockWrapper lockWrapper = locks.get(key);
    lockWrapper.lock.unlock();
    if (lockWrapper.removeThreadFromQueue() == 0) { 
        // NB : We pass in the specific value to remove to handle the case where another thread would queue right before the removal
        locks.remove(key, lockWrapper);
    }
}
```

### 3.4.摘要

一言以蔽之，让我们看看我们整个班级最后是什么样子的:

```java
public class LockByKey {

    private static class LockWrapper {
        private final Lock lock = new ReentrantLock();
        private final AtomicInteger numberOfThreadsInQueue = new AtomicInteger(1);

        private LockWrapper addThreadInQueue() {
            numberOfThreadsInQueue.incrementAndGet(); 
            return this;
        }

        private int removeThreadFromQueue() {
            return numberOfThreadsInQueue.decrementAndGet(); 
        }

    }

    private static ConcurrentHashMap<String, LockWrapper> locks = new ConcurrentHashMap<String, LockWrapper>();

    public void lock(String key) {
        LockWrapper lockWrapper = locks.compute(key, (k, v) -> v == null ? new LockWrapper() : v.addThreadInQueue());
        lockWrapper.lock.lock();
    }

    public void unlock(String key) {
        LockWrapper lockWrapper = locks.get(key);
        lockWrapper.lock.unlock();
        if (lockWrapper.removeThreadFromQueue() == 0) { 
            // NB : We pass in the specific value to remove to handle the case where another thread would queue right before the removal
            locks.remove(key, lockWrapper);
        }
    }

}
```

这种用法与我们之前的用法非常相似:

```java
String key = "key"; 
LockByKey lockByKey = new LockByKey(); 
try { 
    lockByKey.lock(key);
    // insert your code here 
} finally { // CRUCIAL 
    lockByKey.unlock(key); 
}
```

## 4.允许同时进行多项操作

最后但同样重要的是，让我们考虑另一种情况:不是一次只允许一个线程对一个给定的键执行一个动作，我们希望将允许同时对同一个键执行动作的线程数量限制在某个整数`n`。为了简单起见，我们将设置`n` =2。

让我们详细描述一下我们的用例:

*   第一个线程想要获得键上的锁:它将被允许这样做
*   第二个线程想要获得同一个锁:它也将被允许
*   第三个线程请求同一键上的锁:它必须排队，直到前两个线程中的一个释放它的锁

[信号量](/web/20220524061956/https://www.baeldung.com/java-semaphore)就是为此而生的。`Semaphore`是一个用来限制同时访问一个资源的线程数量的对象。

全局功能和代码看起来与我们的锁非常相似:

```java
public class SimultaneousEntriesLockByKey {

    private static final int ALLOWED_THREADS = 2;

    private static ConcurrentHashMap<String, Semaphore> semaphores = new ConcurrentHashMap<String, Semaphore>();

    public void lock(String key) {
        Semaphore semaphore = semaphores.compute(key, (k, v) -> v == null ? new Semaphore(ALLOWED_THREADS) : v);
        semaphore.acquireUninterruptibly();
    }

    public void unlock(String key) {
        Semaphore semaphore = semaphores.get(key);
        semaphore.release();
        if (semaphore.availablePermits() == ALLOWED_THREADS) { 
            semaphores.remove(key, semaphore);
        }  
    }

}
```

用法是相同的:

```java
String key = "key"; 
SimultaneousEntriesLockByKey lockByKey = new SimultaneousEntriesLockByKey(); 
try { 
    lockByKey.lock(key); 
    // insert your code here 
} finally { // CRUCIAL 
    lockByKey.unlock(key); 
}
```

## 5.结论

在本文中，我们已经看到了如何在键上加锁来完全阻止并发操作，或者将并发操作的数量限制为一个(使用锁)或多个(使用信号量)。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220524061956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)