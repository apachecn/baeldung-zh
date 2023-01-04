# 什么原因导致 java.lang.OutOfMemoryError:无法创建新的本机线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-outofmemoryerror-unable-to-create-new-native-thread>

## 1.介绍

在本教程中，我们将讨论`java.lang.OutOfMemoryError: unable to create new native thread`错误的原因和可能的补救措施。

## 2.理解问题

### 2.1.问题的原因

**大多数 Java 应用程序本质上都是多线程的**，由多个组件组成，执行特定的任务，并在不同的线程中执行。然而，底层的**操作系统(OS)对 Java 应用程序可以创建的最大线程数**设置了上限。

当 **JVM 向底层操作系统请求新线程时，JVM 抛出一个`unable to create new native thread `错误，而操作系统无法创建新的[内核](/web/20220627170439/https://www.baeldung.com/cs/os-kernel)线程，也称为操作系统或系统线程**。事件的顺序如下:

1.  在 Java 虚拟机(JVM)中运行的应用程序请求一个新线程
2.  JVM 本地代码向操作系统发送一个创建新内核线程的请求
3.  操作系统试图创建一个需要内存分配的新内核线程
4.  操作系统拒绝本机内存分配，因为
    *   发出请求的 **Java 进程已经耗尽了它的内存地址空间**
    *   **操作系统已经耗尽了它的[虚拟内存](/web/20220627170439/https://www.baeldung.com/cs/virtual-memory)**
5.  然后 Java 进程返回`java.lang.OutOfMemoryError: unable to create new native thread` 错误

### 2.2.线程分配模型

一个操作系统通常有**两种类型的线程——用户线程(由 Java 应用程序创建的线程)和内核线程**。内核线程之上支持用户线程，内核线程由操作系统管理。

在它们之间，有三种常见的关系:

1.  `Many-To-One` –许多用户线程映射到一个内核线程
2.  `One-To-One` –一个用户线程映射到一个内核线程
3.  `Many-To-Many` –许多用户线程复用到更少或相等数量的内核线程

## 3.重现错误

通过在连续循环中创建线程，然后让线程等待，我们可以很容易地重现这个问题:

```java
while (true) {
  new Thread(() -> {
    try {
        TimeUnit.HOURS.sleep(1);     
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
  }).start();
}
```

由于我们在一个小时内保持每个线程，同时不断创建新的线程，我们将很快达到操作系统的最大线程数。

## 4.解决方法

解决此错误的一种方法是在操作系统级别增加线程限制配置。

然而，这不是一个理想的解决方案，因为`OutOfMemoryError`很可能表示编程错误。让我们看看解决这个问题的其他方法。

### 4.1。利用执行者服务框架

利用 Java 的 [executor 服务框架](/web/20220627170439/https://www.baeldung.com/java-executor-service-tutorial)进行线程管理可以在一定程度上解决这个问题。默认的执行器服务框架或定制的执行器配置可以控制线程的创建。

我们可以使用`Executors#newFixedThreadPool`方法来设置一次可以使用的最大线程数:

```java
ExecutorService executorService = Executors.newFixedThreadPool(5);

Runnable runnableTask = () -> {
  try {
    TimeUnit.HOURS.sleep(1);
  } catch (InterruptedException e) {
      // Handle Exception
  }
};

IntStream.rangeClosed(1, 10)
  .forEach(i -> executorService.submit(runnableTask));

assertThat(((ThreadPoolExecutor) executorService).getQueue().size(), is(equalTo(5)));
```

在上面的例子中，我们首先创建了一个有五个线程的固定线程池和一个 runnable 任务，它让线程等待一个小时。然后，我们向线程池提交十个这样的任务，并断言有五个任务正在 executor 服务队列中等待。

由于线程池有五个线程，它在任何时候最多可以处理五个任务。

### 4.2。捕获和分析线程转储

[捕获和分析线程转储](/web/20220627170439/https://www.baeldung.com/java-thread-dump)有助于理解线程的状态。

让我们看一个示例线程转储，看看我们能学到什么:

[![](img/6b5aaad01967dd8987a17b2960ab4c96.png)](/web/20220627170439/https://www.baeldung.com/wp-content/uploads/2020/06/VisualVMThreadDump.png)

对于前面给出的例子，上面的线程快照来自 Java VisualVM。这张快照清楚地展示了连续的线程创建。

一旦我们确定存在连续的线程创建，我们就可以捕获应用程序的线程转储，以确定创建线程的源代码:

[![](img/8fe039e16a1a1957e20fbe99e50dae11.png)](/web/20220627170439/https://www.baeldung.com/wp-content/uploads/2020/06/ThreadDumpSourceCode.png)

在上面的快照中，我们可以确定负责线程创建的代码。这为采取适当的措施提供了有用的见解。

## 5.结论

在本文中，我们了解了`java.lang.OutOfMemoryError: unable to create new native thread`错误，我们看到**是由 Java 应用程序中过多的线程创建**引起的。

通过查看`ExecutorService `框架和线程转储分析，我们探索了一些解决和分析错误的解决方案，作为解决这个问题的两个有用的措施。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627170439/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)