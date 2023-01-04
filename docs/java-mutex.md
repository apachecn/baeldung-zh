# 在 Java 中使用互斥对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mutex>

## 1.概观

在本教程中，我们将看到用 Java 实现[互斥](/web/20221206143356/https://www.baeldung.com/cs/what-is-mutex)的**种不同方法。**

## 2.互斥（体）…

在多线程应用程序中，两个或多个线程可能需要同时访问共享资源，从而导致意外行为。这种共享资源的例子有数据结构、输入输出设备、文件和网络连接。

我们称这种情况为`race condition`。并且，访问共享资源的程序部分被称为`critical section`。**因此，为了避免竞争情况，我们需要同步对临界区的访问。**

互斥体(或互斥体)是最简单的类型`synchronizer –`，它**确保一次只有一个线程可以执行计算机程序的关键部分**。

为了访问临界区，一个线程获取互斥体，然后访问临界区，最后释放互斥体。与此同时，**所有其他线程都会阻塞，直到互斥体释放。**一个线程一退出临界区，另一个线程就可以进入临界区。

## 3.为什么是互斥？

首先，让我们以一个`SequenceGeneraror`类为例，它通过每次将`currentValue`递增 1 来生成下一个序列:

```java
public class SequenceGenerator {

    private int currentValue = 0;

    public int getNextSequence() {
        currentValue = currentValue + 1;
        return currentValue;
    }

}
```

现在，让我们创建一个测试用例，看看当多个线程试图同时访问该方法时，该方法的行为如何:

```java
@Test
public void givenUnsafeSequenceGenerator_whenRaceCondition_thenUnexpectedBehavior() throws Exception {
    int count = 1000;
    Set<Integer> uniqueSequences = getUniqueSequences(new SequenceGenerator(), count);
    Assert.assertEquals(count, uniqueSequences.size());
}

private Set<Integer> getUniqueSequences(SequenceGenerator generator, int count) throws Exception {
    ExecutorService executor = Executors.newFixedThreadPool(3);
    Set<Integer> uniqueSequences = new LinkedHashSet<>();
    List<Future<Integer>> futures = new ArrayList<>();

    for (int i = 0; i < count; i++) {
        futures.add(executor.submit(generator::getNextSequence));
    }

    for (Future<Integer> future : futures) {
        uniqueSequences.add(future.get());
    }

    executor.awaitTermination(1, TimeUnit.SECONDS);
    executor.shutdown();

    return uniqueSequences;
}
```

一旦我们执行了这个测试用例，我们可以看到它大部分时间都失败了，原因类似于:

```java
java.lang.AssertionError: expected:<1000> but was:<989>
  at org.junit.Assert.fail(Assert.java:88)
  at org.junit.Assert.failNotEquals(Assert.java:834)
  at org.junit.Assert.assertEquals(Assert.java:645)
```

`uniqueSequences`的大小应该等于我们在测试用例中执行`getNextSequence`方法的次数。然而，由于竞争条件，情况并非如此。显然，我们不希望出现这种行为。

所以，为了避免这样的竞争情况，我们需要**确保一次只有一个线程可以执行`getNextSequence`方法**。在这种情况下，我们可以使用互斥来同步线程。

有很多种方法，我们可以在 Java 中实现一个互斥体。因此，接下来，我们将看到为我们的`SequenceGenerator`类实现互斥体的不同方法。

## 4.使用`synchronized` 关键字

首先，我们将讨论 [`synchronized`关键字](/web/20221206143356/https://www.baeldung.com/java-synchronized)，这是在 Java 中实现互斥的最简单的方法。

Java 中的每个对象都有一个关联的固有锁。****`synchronized` 方法和** **`synchronized` 块使用这个内在锁**来限制临界区一次只能被一个线程访问。**

 **因此，当线程调用`synchronized`方法或进入`synchronized`块时，它会自动获取锁。当方法或块完成或者从它们抛出异常时，锁被释放。

让我们简单地通过添加关键字`synchronized` 将`getNextSequence`改为互斥:

```java
public class SequenceGeneratorUsingSynchronizedMethod extends SequenceGenerator {

    @Override
    public synchronized int getNextSequence() {
        return super.getNextSequence();
    }

}
```

`synchronized`块类似于`synchronized`方法，对临界区和我们可以用来锁定的对象有更多的控制。

所以，现在让我们看看如何使用**块来同步定制互斥对象**:

```java
public class SequenceGeneratorUsingSynchronizedBlock extends SequenceGenerator {

    private Object mutex = new Object();

    @Override
    public int getNextSequence() {
        synchronized (mutex) {
            return super.getNextSequence();
        }
    }

}
```

## 5.使用`ReentrantLock`

Java 1.5 中引入了`[ReentrantLock](/web/20221206143356/https://www.baeldung.com/java-concurrent-locks)` 类。它比`synchronized`关键字方法提供了更多的灵活性和控制。

让我们看看如何使用`ReentrantLock`来实现互斥:

```java
public class SequenceGeneratorUsingReentrantLock extends SequenceGenerator {

    private ReentrantLock mutex = new ReentrantLock();

    @Override
    public int getNextSequence() {
        try {
            mutex.lock();
            return super.getNextSequence();
        } finally {
            mutex.unlock();
        }
    }
}
```

## 6.使用`Semaphore`

和`ReentrantLock`一样，`[Semaphore](/web/20221206143356/https://www.baeldung.com/java-semaphore)` 类也是在 Java 1.5 中引入的。

在互斥的情况下，只有一个线程可以访问临界区，`Semaphore`允许**固定数量的线程访问临界区**。因此，**我们也可以通过将一个`Semaphore`中允许的线程数设置为一个**来实现互斥。

现在让我们使用`Semaphore`创建另一个线程安全版本的`SequenceGenerator`:

```java
public class SequenceGeneratorUsingSemaphore extends SequenceGenerator {

    private Semaphore mutex = new Semaphore(1);

    @Override
    public int getNextSequence() {
        try {
            mutex.acquire();
            return super.getNextSequence();
        } catch (InterruptedException e) {
            // exception handling code
        } finally {
            mutex.release();
        }
    }
}
```

## 7.使用番石榴的`Monitor`类

到目前为止，我们已经看到了使用 Java 提供的特性来实现互斥体的选项。

然而，谷歌番石榴库的`Monitor`类是`ReentrantLock` 类的更好替代。根据它的[文档](https://web.archive.org/web/20221206143356/https://guava.dev/releases/19.0/api/docs/com/google/common/util/concurrent/Monitor.html)，使用`Monitor`的代码比使用`ReentrantLock`的代码可读性更好，更不容易出错。

首先，我们将为[番石榴](https://web.archive.org/web/20221206143356/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava)添加 Maven 依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在，我们将使用`Monitor`类编写`SequenceGenerator`的另一个子类:

```java
public class SequenceGeneratorUsingMonitor extends SequenceGenerator {

    private Monitor mutex = new Monitor();

    @Override
    public int getNextSequence() {
        mutex.enter();
        try {
            return super.getNextSequence();
        } finally {
            mutex.leave();
        }
    }

}
```

## 8.结论

在本教程中，我们研究了互斥的概念。此外，我们已经看到了在 Java 中实现它的不同方法。

与往常一样，本教程中使用的代码示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221206143356/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-2)**