# 线程转储中的“锁定的可拥有同步器”是什么？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/locked-ownable-synchronizers>

## 1.概观

在本教程中，我们将看看线程锁定的可拥有同步器的含义。我们将编写一个使用`Lock`进行同步的简单程序，看看这在[线程转储](/web/20220905041908/https://www.baeldung.com/java-thread-dump)中是什么样子。

## 2.什么是锁定的可拥有同步器？

每个线程都可能有一个同步器对象列表。该列表中的条目表示线程已经获得锁的可拥有同步器。

一个`AbstractOwnableSynchronizer`类的实例可以被用作同步器。它最常见的子类之一是`Sync`类，它是类似`ReentrantReadWriteLock`的`Lock`接口实现的一个字段。

当我们调用`ReentrantReadWriteLock.lock()`方法时，代码在内部将其委托给`Sync.lock()`方法。一旦我们获得了一个锁，`Lock`对象就被添加到线程的锁定的可拥有同步器列表中。

我们可以在典型的线程转储中查看该列表:

```java
"Thread-0" #1 prio=5 os_prio=0 tid=0x000002411a452800 nid=0x1c18 waiting on condition [0x00000051a2bff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at com.baeldung.ownablesynchronizers.Application.main(Application.java:25)

   Locked ownable synchronizers:
        - <0x000000076e185e68> (a java.util.concurrent.locks.ReentrantReadWriteLock$FairSync)
```

根据我们用来生成它的工具，我们可能需要提供一个特定的选项。例如，使用 [jstack](/web/20220905041908/https://www.baeldung.com/java-thread-dump#1-jstack) ，我们运行以下命令:

```java
jstack -l <pid>
```

使用`-l`选项，我们告诉 jstack 打印关于锁的附加信息。

## 3.锁定可拥有的同步器有什么帮助

**可拥有的同步器列表帮助我们识别可能的应用程序死锁**。例如，我们可以在线程转储中看到一个名为`Thread-1`的不同线程是否正在等待获取同一个`Lock`对象的锁:

```java
"Thread-1" #12 prio=5 os_prio=0 tid=0x00000241374d7000 nid=0x4da4 waiting on condition [0x00000051a42fe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076e185e68> (a java.util.concurrent.locks.ReentrantReadWriteLock$FairSync)
```

线程`Thread-1`处于状态`WAITING`。具体来说，它等待获取 id 为`<0x000000076e185e68>`的对象的锁。然而，相同的对象在线程`Thread-0`的锁定的可拥有同步器列表中。**我们现在知道，在线程`Thread-0`释放它自己的锁之前，线程`Thread-1`不能继续**。

如果在同一时间，同样的场景以相反的方式发生，例如，`Thread-1`获得了一个`Thread-0`等待的锁，我们就创建了一个死锁。

## 4.死锁诊断示例

让我们看一些简单的代码来说明以上所有内容。我们将用两个线程和两个`ReentrantLock `对象创建一个死锁场景:

```java
public class Application {

    public static void main(String[] args) throws Exception {
        ReentrantLock firstLock = new ReentrantLock(true);
        ReentrantLock secondLock = new ReentrantLock(true);
        Thread first = createThread("Thread-0", firstLock, secondLock);
        Thread second = createThread("Thread-1", secondLock, firstLock);

        first.start();
        second.start();

        first.join();
        second.join();
    }
}
```

我们的`main()`方法创建了两个`ReentrantLock`对象。第一个线程`Thread-0`，使用`firstLock`作为它的主锁，使用`secondLock`作为它的副锁。

我们将对`Thread-1`做同样的事情。具体来说，我们的目标是通过让每个线程获取其主锁并在试图获取其辅助锁时挂起来生成死锁。

`createThread()`方法用各自的锁生成我们的每个线程:

```java
public static Thread createThread(String threadName, ReentrantLock primaryLock, ReentrantLock secondaryLock) {
    return new Thread(() -> {
        primaryLock.lock();

        synchronized (Application.class) {
            Application.class.notify();
            if (!secondaryLock.isLocked()) {
                Application.class.wait();
            }
        }

        secondaryLock.lock();

        System.out.println(threadName + ": Finished");
        primaryLock.unlock();
        secondaryLock.unlock();
    });
}
```

为了确保每个线程在另一个线程试图锁定它的`primaryLock`，我们在`synchronized`块中使用`isLocked()`等待它。

运行这段代码将会挂起，并且永远不会打印出最终的控制台输出。如果我们运行 jstack，我们将看到以下内容:

```java
"Thread-0" #12 prio=5 os_prio=0 tid=0x0000027e1e31e800 nid=0x7d0 waiting on condition [0x000000a29acfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076e182558>

   Locked ownable synchronizers:
        - <0x000000076e182528>

"Thread-1" #13 prio=5 os_prio=0 tid=0x0000027e1e3ba000 nid=0x650 waiting on condition [0x000000a29adfe000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076e182528>

   Locked ownable synchronizers:
        - <0x000000076e182558>
```

我们可以看到`Thread-0`停在那里等待`0x000000076e182558`，而`Thread-1`在等待`0x000000076e182528`。同时，我们可以在它们各自线程的锁定的可拥有的同步器中找到这些句柄。**基本上，这意味着我们可以看到我们的线程正在等待哪些锁，以及哪个线程拥有这些锁**。这有助于我们解决包括死锁在内的并发问题。

需要注意的重要一点是，如果我们使用一个`ReentrantReadWriteLock.ReadLock`而不是一个`ReentrantLock`作为同步器，我们将不会在线程转储中获得相同的信息。只有`ReentrantReadWriteLock.WriteLock` 出现在同步器列表中。

## 5.结论

在本文中，我们讨论了线程转储中出现的锁定的可拥有同步器列表的含义，我们如何使用它来解决并发问题，还看到了一个示例场景。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220905041908/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)