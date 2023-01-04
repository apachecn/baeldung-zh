# Java 中的守护线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-daemon-thread>

## 1。概述

在这篇短文中，我们将看看 Java 中的守护线程，看看它们能用来做什么。我们还将解释守护线程和用户线程之间的区别。

## 2。守护进程和用户线程的区别

Java 提供了两种类型的线程:用户线程和守护线程。

用户线程是高优先级线程。JVM 将等待任何用户线程完成它的任务，然后再终止它。

另一方面，**守护线程是低优先级线程，其唯一的作用是为用户线程提供服务。**

因为守护线程是为用户线程服务的，并且只在用户线程运行时才需要，所以一旦所有用户线程都完成了执行，它们不会阻止 JVM 退出。

这就是为什么通常存在于守护线程中的无限循环不会导致问题，因为一旦所有用户线程完成了它们的执行，任何代码，包括`finally`块，都不会被执行。因此，对于 I/O 任务，不推荐使用**守护线程。**

然而，这条规则也有例外。守护线程中设计不良的代码会阻止 JVM 退出。例如，在一个正在运行的守护线程上调用`Thread.join()`可以阻止应用程序的关闭。

## 3。守护线程的使用

守护线程对于后台支持任务非常有用，例如垃圾收集、释放未使用对象的内存以及从缓存中删除不需要的条目。大多数 JVM 线程都是守护线程。

## 4。创建守护线程

要将一个线程设置为守护线程，我们需要做的就是调用`Thread.setDaemon().` 在这个例子中，我们将使用`NewThread`类，它扩展了`Thread`类:

```
NewThread daemonThread = new NewThread();
daemonThread.setDaemon(true);
daemonThread.start();
```

任何线程都会继承创建它的线程的守护进程状态。因为主线程是用户线程，所以默认情况下，在主方法中创建的任何线程都是用户线程。

方法`setDaemon()`只能在`Thread`对象已经创建并且线程尚未启动之后调用。在线程运行时试图调用`setDaemon()`会抛出一个`IllegalThreadStateException`:

```
@Test(expected = IllegalThreadStateException.class)
public void whenSetDaemonWhileRunning_thenIllegalThreadStateException() {
    NewThread daemonThread = new NewThread();
    daemonThread.start();
    daemonThread.setDaemon(true);
}
```

## 5。检查线程是否是守护线程

最后，为了检查一个线程是否是守护线程，我们可以简单地调用方法`isDaemon()`:

```
@Test
public void whenCallIsDaemon_thenCorrect() {
    NewThread daemonThread = new NewThread();
    NewThread userThread = new NewThread();
    daemonThread.setDaemon(true);
    daemonThread.start();
    userThread.start();

    assertTrue(daemonThread.isDaemon());
    assertFalse(userThread.isDaemon());
}
```

## 6。结论

在这个快速教程中，我们看到了什么是守护线程，以及它们在一些实际场景中的用途。

和往常一样，GitHub 上有完整版本的代码[。](https://web.archive.org/web/20220525121750/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)