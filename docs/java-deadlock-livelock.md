# Java 线程死锁和活锁

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-deadlock-livelock>

## 1.概观

虽然多线程有助于提高应用程序的性能，但它也会带来一些问题。在本教程中，我们将借助 Java 示例来研究两个这样的问题，[死锁和活锁](/web/20220627093650/https://www.baeldung.com/cs/deadlock-livelock-starvation)。

## 2.僵局

### 2.1.什么是死锁？

当**两个或多个线程永远等待另一个线程**持有的锁或资源时，就会发生[死锁](/web/20220627093650/https://www.baeldung.com/cs/os-deadlock)。因此，应用程序可能会因为死锁的线程无法继续运行而停滞或失败。

经典的[哲学家进餐问题](/web/20220627093650/https://www.baeldung.com/java-dining-philoshophers)很好地展示了多线程环境中的同步问题，并且经常被用作死锁的例子。

### 2.2.死锁示例

首先，让我们看一个简单的 Java 例子来理解死锁。

在这个例子中，我们将创建两个线程，`T1`和`T2`。线程`T1`调用`operation1`，线程`T2`调用`operations`。

为了完成它们的操作，线程`T1`需要先获取`lock1`，然后获取`lock2`，而线程`T2`需要先获取`lock2`，然后获取`lock1`。因此，基本上，两个线程都试图以相反的顺序获取锁。

现在，让我们编写`DeadlockExample`类:

```java
public class DeadlockExample {

    private Lock lock1 = new ReentrantLock(true);
    private Lock lock2 = new ReentrantLock(true);

    public static void main(String[] args) {
        DeadlockExample deadlock = new DeadlockExample();
        new Thread(deadlock::operation1, "T1").start();
        new Thread(deadlock::operation2, "T2").start();
    }

    public void operation1() {
        lock1.lock();
        print("lock1 acquired, waiting to acquire lock2.");
        sleep(50);

        lock2.lock();
        print("lock2 acquired");

        print("executing first operation.");

        lock2.unlock();
        lock1.unlock();
    }

    public void operation2() {
        lock2.lock();
        print("lock2 acquired, waiting to acquire lock1.");
        sleep(50);

        lock1.lock();
        print("lock1 acquired");

        print("executing second operation.");

        lock1.unlock();
        lock2.unlock();
    }

    // helper methods

}
```

现在让我们运行这个死锁示例，并注意输出:

```java
Thread T1: lock1 acquired, waiting to acquire lock2.
Thread T2: lock2 acquired, waiting to acquire lock1.
```

一旦我们运行程序，我们可以看到程序导致了一个死锁，并且永远不会退出。日志显示线程`T1`正在等待被线程`T2`持有的`lock2`。同样，线程`T2`也在等待`lock1`，它被线程`T1`持有。

### 2.3.避免死锁

死锁是 Java 中常见的并发问题。因此，我们应该设计一个 Java 应用程序来避免任何潜在的死锁情况。

首先，我们应该避免为一个线程获取多个锁的需要。然而，如果一个线程确实需要多个锁，我们应该确保每个线程以相同的顺序获取锁，以**避免锁获取**中的任何循环依赖。

我们还可以使用**定时锁尝试**，就像 [`Lock`](/web/20220627093650/https://www.baeldung.com/java-concurrent-locks) 接口中的`tryLock`方法，来确保线程在无法获得锁的情况下不会无限阻塞。

## 3.Livelock(锁定)

### 3.1.什么是活锁

活锁是另一个并发问题，类似于死锁。在活锁中，**两个或多个线程继续在彼此之间转移状态**，而不是像我们在死锁示例中看到的那样无限等待。因此，线程不能执行它们各自的任务。

活锁的一个很好的例子是消息传递系统，在该系统中，当发生异常时，消息使用者回滚事务并将消息放回队列的头部。然后从队列中重复读取相同的消息，结果导致另一个异常并被放回队列中。消费者将永远不会从队列中获得任何其他消息。

### 3.2.活锁示例

现在，为了演示活锁情况，我们将采用前面讨论过的同一个死锁示例。同样在这个例子中，线程`T1`调用`operation1`，线程`T2`调用`operation2`。然而，我们将稍微改变这些操作的逻辑。

两个线程都需要两个锁来完成它们的工作。每个线程都获得了它的第一个锁，但是发现第二个锁不可用。因此，为了让另一个线程先完成，每个线程都释放它的第一个锁，并尝试再次获取两个锁。

让我们用一个`LivelockExample`类来演示活锁:

```java
public class LivelockExample {

    private Lock lock1 = new ReentrantLock(true);
    private Lock lock2 = new ReentrantLock(true);

    public static void main(String[] args) {
        LivelockExample livelock = new LivelockExample();
        new Thread(livelock::operation1, "T1").start();
        new Thread(livelock::operation2, "T2").start();
    }

    public void operation1() {
        while (true) {
            tryLock(lock1, 50);
            print("lock1 acquired, trying to acquire lock2.");
            sleep(50);

            if (tryLock(lock2)) {
                print("lock2 acquired.");
            } else {
                print("cannot acquire lock2, releasing lock1.");
                lock1.unlock();
                continue;
            }

            print("executing first operation.");
            break;
        }
        lock2.unlock();
        lock1.unlock();
    }

    public void operation2() {
        while (true) {
            tryLock(lock2, 50);
            print("lock2 acquired, trying to acquire lock1.");
            sleep(50);

            if (tryLock(lock1)) {
                print("lock1 acquired.");
            } else {
                print("cannot acquire lock1, releasing lock2.");
                lock2.unlock();
                continue;
            }

            print("executing second operation.");
            break;
        }
        lock1.unlock();
        lock2.unlock();
    }

    // helper methods

}
```

现在，让我们运行这个例子:

```java
Thread T1: lock1 acquired, trying to acquire lock2.
Thread T2: lock2 acquired, trying to acquire lock1.
Thread T1: cannot acquire lock2, releasing lock1.
Thread T2: cannot acquire lock1, releasing lock2.
Thread T2: lock2 acquired, trying to acquire lock1.
Thread T1: lock1 acquired, trying to acquire lock2.
Thread T1: cannot acquire lock2, releasing lock1.
Thread T1: lock1 acquired, trying to acquire lock2.
Thread T2: cannot acquire lock1, releasing lock2.
..
```

正如我们在日志中看到的，两个线程都在重复获取和释放锁。因此，没有一个线程能够完成操作。

### 3.3.避免活锁

为了避免活锁，我们需要研究导致活锁的条件，然后提出相应的解决方案。

例如，如果我们有两个线程重复获取和释放锁，从而导致活锁，我们可以设计代码，以便线程以随机的时间间隔重试获取锁。这将给线程一个公平的机会来获得它们需要的锁。

在我们之前讨论的消息传递系统示例中，处理活跃度问题的另一种方法是将失败的消息放在一个单独的队列中，而不是再次放回同一个队列中。

## 4.结论

在本教程中，我们已经讨论了死锁和活锁。此外，我们还研究了 Java 示例来演示这些问题，并简要介绍了如何避免这些问题。

和往常一样，本例中使用的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627093650/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)