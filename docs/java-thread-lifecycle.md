# Java 中线程的生命周期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-lifecycle>

## 1。简介

在本文中，我们将详细讨论 Java 中的一个核心概念——线程的生命周期。

我们将使用一个快速图解，当然，还有实用的代码片段来更好地理解线程执行期间的这些状态。

要开始理解 Java 中的线程，这篇关于创建线程的文章是一个很好的起点。

## 2。Java 中的多线程

在 Java 语言中，多线程是由线程的核心概念驱动的。在线程的生命周期中，线程会经历不同的状态:

[![Life Cycle of a Thread](img/538c5640b9def664c5ac1ecefb14e88a.png)](/web/20221206113224/https://www.baeldung.com/wp-content/uploads/2018/02/Life_cycle_of_a_Thread_in_Java.jpg)

## 3。Java 中线程的生命周期

`java.lang.Thread`类包含一个定义其潜在状态的`static State enum –`。在任何给定的时间点，线程只能处于以下状态之一:

1.  **`NEW –`** 一个新创建的尚未开始执行的线程
2.  **`RUNNABLE –`** 要么正在运行，要么准备执行，但它在等待资源分配
3.  **`BLOCKED –`** 等待获取监控锁以进入或重新进入同步块/方法
4.  **`WAITING –`** 等待某个其他线程执行某个特定的动作而没有任何时间限制
5.  **`TIMED_WAITING –`** 等待某个其他线程执行某个特定动作一段指定的时间
6.  **`TERMINATED –`** 已经完成了它的执行

上图涵盖了所有这些状态；现在让我们详细讨论一下其中的每一个。

### 3.1。新

**`NEW``Thread`(或天生的`Thread`)是已经被创建但还没有开始的线程。**它一直保持这种状态，直到我们使用`start()` 方法启动它。

下面的代码片段显示了一个新创建的处于`NEW`状态的线程:

```java
Runnable runnable = new NewState();
Thread t = new Thread(runnable);
System.out.println(t.getState());
```

因为我们还没有开始提到的线程，方法`t.getState()` 打印:

```java
NEW
```

### 3.2。可运行的

当我们创建了一个新线程并在其上调用了`start()` 方法时，它就从`NEW`状态转移到了`RUNNABLE`状态。**处于这种状态的线程要么正在运行，要么准备运行，但它们正在等待系统分配资源。**

在多线程环境中，线程调度器(JVM 的一部分)为每个线程分配固定的时间。所以它运行一段特定的时间，然后将控制权交给其他`RUNNABLE`线程。

例如，让我们将`t.start()`方法添加到前面的代码中，并尝试访问它的当前状态:

```java
Runnable runnable = new NewState();
Thread t = new Thread(runnable);
t.start();
System.out.println(t.getState());
```

此代码**最有可能**返回如下输出:

```java
RUNNABLE
```

注意，在这个例子中，并不总是保证当我们的控件到达`t.getState()`时，它仍然处于`RUNNABLE`状态。

可能发生的情况是，它立即被`Thread-Scheduler`调度，并可能完成执行。在这种情况下，我们可能会得到不同的输出。

### 3.3。受阻

当线程当前不符合运行条件时，它处于`BLOCKED`状态。**当它在等待一个监视器锁，并试图访问一段被其他线程锁定的代码时，它就进入这种状态。**

让我们试着重现这种状态:

```java
public class BlockedState {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new DemoBlockedRunnable());
        Thread t2 = new Thread(new DemoBlockedRunnable());

        t1.start();
        t2.start();

        Thread.sleep(1000);

        System.out.println(t2.getState());
        System.exit(0);
    }
}

class DemoBlockedRunnable implements Runnable {
    @Override
    public void run() {
        commonResource();
    }

    public static synchronized void commonResource() {
        while(true) {
            // Infinite loop to mimic heavy processing
            // 't1' won't leave this method
            // when 't2' try to enter this
        }
    }
}
```

在这段代码中:

1.  我们创建了两个不同的线程—`t1`和`t2`
2.  `t1` 启动并进入同步`commonResource()` 方式；这意味着只有一个线程可以访问它；尝试访问此方法的所有其他后续线程将被阻止进一步执行，直到当前线程完成处理
3.  当`t1` 进入这个方法时，它被保持在一个无限 while 循环中；这只是为了模仿繁重的处理，以便所有其他线程都不能进入该方法
4.  现在当我们启动`t2`时，它试图进入`commonResource()` 方法，该方法已经被`t1,`访问，因此`t2` 将保持在`BLOCKED`状态

在这种状态下，我们调用`t2.getState()` 并得到如下输出:

```java
BLOCKED
```

### 3.4。等待

**当一个线程在等待其他线程执行某个特定的动作时，它就处于`WAITING`状态。** [根据 JavaDocs](https://web.archive.org/web/20221206113224/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Thread.State.html#WAITING) 的说法，任何线程都可以通过调用以下三种方法中的任意一种来进入这种状态:

1.  `object.wait()`
2.  `thread.join()`或者
3.  `LockSupport.park()`

请注意，在`wait()` 和`join()`中，我们没有定义任何超时时间，因为该场景将在下一节中介绍。

我们有[单独的教程](/web/20221206113224/https://www.baeldung.com/java-wait-notify)，详细讨论了`wait()`、`notify()`和`notifyAll()`的用法。

现在，让我们试着重现这种状态:

```java
public class WaitingState implements Runnable {
    public static Thread t1;

    public static void main(String[] args) {
        t1 = new Thread(new WaitingState());
        t1.start();
    }

    public void run() {
        Thread t2 = new Thread(new DemoWaitingStateRunnable());
        t2.start();

        try {
            t2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }
    }
}

class DemoWaitingStateRunnable implements Runnable {
    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }

        System.out.println(WaitingState.t1.getState());
    }
}
```

让我们讨论一下我们在做什么:

1.  我们已经创建并启动了`t1`
2.  `t1`创建一个`t2` 并启动它
3.  当`t2` 的处理继续时，我们调用`t2.join()`，这将`t1` 置于`WAITING`状态，直到`t2` 完成执行
4.  由于`t1` 正在等待`t2` 完成，我们从`t2`呼叫`t1.getState()`

如您所料，这里的输出是:

```java
WAITING
```

### 3.5。定时等待

**当一个线程在规定的时间内等待另一个线程执行特定的动作时，它就处于`TIMED_WAITING`状态。**

[根据 JavaDocs](https://web.archive.org/web/20221206113224/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Thread.State.html#TIMED_WAITING) 的说法，有五种方法可以将线程置于`TIMED_WAITING`状态:

1.  `thread.sleep(long millis)`
2.  `wait(int timeout)`或`wait(int timeout, int nanos)`
3.  `thread.join(long` 毫秒`)`
4.  `LockSupport.parkNanos`
5.  `LockSupport.parkUntil`

要阅读更多关于 Java 中`wait()`和`sleep()`的区别，请看[这篇专门的文章](/web/20221206113224/https://www.baeldung.com/java-wait-and-sleep)。

现在，让我们试着快速重现这种状态:

```java
public class TimedWaitingState {
    public static void main(String[] args) throws InterruptedException {
        DemoTimeWaitingRunnable runnable= new DemoTimeWaitingRunnable();
        Thread t1 = new Thread(runnable);
        t1.start();

        // The following sleep will give enough time for ThreadScheduler
        // to start processing of thread t1
        Thread.sleep(1000);
        System.out.println(t1.getState());
    }
}

class DemoTimeWaitingRunnable implements Runnable {
    @Override
    public void run() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }
    }
}
```

这里，我们创建并启动了一个线程`t1`，它进入休眠状态，超时时间为 5 秒；输出将是:

```java
TIMED_WAITING
```

### 3.6。已终止

这是死线的状态。**当它完成执行或异常终止时，它处于`TERMINATED`状态。**

我们有一篇专门的文章讨论停止线程的不同方法。

让我们试着在下面的例子中达到这种状态:

```java
public class TerminatedState implements Runnable {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new TerminatedState());
        t1.start();
        // The following sleep method will give enough time for 
        // thread t1 to complete
        Thread.sleep(1000);
        System.out.println(t1.getState());
    }

    @Override
    public void run() {
        // No processing in this block
    }
}
```

这里，虽然我们已经启动了线程`t1`，但是下一条语句`Thread.sleep(1000)` 给了`t1`足够的时间来完成，所以这个程序给我们的输出是:

```java
TERMINATED
```

除了线程状态之外，我们还可以检查`[isAlive()](https://web.archive.org/web/20221206113224/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html#isAlive()) `方法来确定线程是否处于活动状态。例如，如果我们在这个线程上调用`isAlive() `方法:

```java
Assert.assertFalse(t1.isAlive());
```

简单地说，它返回`false. `，**一个线程是活的当且仅当它已经** **被启动并且还没有死亡。**

## 4。结论

在本教程中，我们学习了 Java 中线程的生命周期。我们查看了由`Thread.State` enum 定义的所有六种状态，并用快速示例再现了它们。

尽管代码片段在几乎每台机器上都给出相同的输出，但在某些例外情况下，我们可能会得到一些不同的输出，因为无法确定线程调度器的确切行为。

和往常一样，这里使用的代码片段可以从 GitHub 上获得。