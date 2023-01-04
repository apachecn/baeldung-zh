# Java 同步队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-synchronous-queue>

## 1。概述

在本文中，我们将关注来自`java.util.concurrent` 包的`[SynchronousQueue](https://web.archive.org/web/20220706104903/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/SynchronousQueue.html)`。

简单地说，这个实现允许我们以线程安全的方式在线程之间交换信息。

## 2。API 概述

`SynchronousQueue` 只有**两个支持的操作:`take()` 和`put(),` ，并且都在阻塞**。

例如，当我们想要向队列中添加一个元素时，我们需要调用`put()` 方法。该方法将被阻塞，直到其他线程调用`take()` 方法，发出准备好获取元素的信号。

虽然`SynchronousQueue`有一个队列接口，但是我们应该把它看作是两个线程之间单个元素的交换点，其中一个线程正在传递一个元素，而另一个线程正在获取该元素。

## 3。使用共享变量实现移交

为了了解为什么`SynchronousQueue` 如此有用，我们将使用两个线程之间的共享变量实现一个逻辑，接下来，我们将使用`SynchronousQueue`重写该逻辑，使我们的代码更简单、可读性更强。

假设我们有两个线程——一个生产者和一个消费者——当生产者设置一个共享变量的值时，我们希望将这个事实通知给消费者线程。接下来，消费者线程将从共享变量中获取一个值。

我们将使用`CountDownLatch` 来协调这两个线程，以防止消费者访问尚未设置的共享变量的值。

我们将定义一个`sharedState` 变量和一个`CountDownLatch` 变量，用于协调处理:

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
AtomicInteger sharedState = new AtomicInteger();
CountDownLatch countDownLatch = new CountDownLatch(1);
```

生产者将把一个随机整数保存到`sharedState` 变量中，并在`countDownLatch,` 上执行`countDown()` 方法，通知消费者它可以从`sharedState:`中获取一个值

```java
Runnable producer = () -> {
    Integer producedElement = ThreadLocalRandom
      .current()
      .nextInt();
    sharedState.set(producedElement);
    countDownLatch.countDown();
};
```

消费者将使用`await()` 方法等待`countDownLatch`。当生产者发出设置变量的信号时，消费者将从`sharedState:`中获取该变量

```java
Runnable consumer = () -> {
    try {
        countDownLatch.await();
        Integer consumedElement = sharedState.get();
    } catch (InterruptedException ex) {
        ex.printStackTrace();
    }
};
```

最后但同样重要的是，让我们开始我们的节目:

```java
executor.execute(producer);
executor.execute(consumer);

executor.awaitTermination(500, TimeUnit.MILLISECONDS);
executor.shutdown();
assertEquals(countDownLatch.getCount(), 0);
```

它将产生以下输出:

```java
Saving an element: -1507375353 to the exchange point
consumed an element: -1507375353 from the exchange point
```

我们可以看到，要实现像在两个线程之间交换元素这样简单的功能，需要很多代码。在下一节中，我们将努力使它变得更好。

## 4。使用`SynchronousQueue` 实现移交

现在让我们实现与上一节相同的功能，但是使用`SynchronousQueue.`它有双重效果，因为我们可以使用它在线程之间交换状态和协调动作，这样我们就不需要使用除了`SynchronousQueue.`之外的任何东西

首先，我们将定义一个队列:

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
SynchronousQueue<Integer> queue = new SynchronousQueue<>();
```

生成器将调用一个`put()` 方法，该方法将一直阻塞，直到其他线程从队列中取出一个元素:

```java
Runnable producer = () -> {
    Integer producedElement = ThreadLocalRandom
      .current()
      .nextInt();
    try {
        queue.put(producedElement);
    } catch (InterruptedException ex) {
        ex.printStackTrace();
    }
};
```

消费者将使用`take()` 方法简单地检索该元素:

```java
Runnable consumer = () -> {
    try {
        Integer consumedElement = queue.take();
    } catch (InterruptedException ex) {
        ex.printStackTrace();
    }
};
```

接下来，我们将开始我们的节目:

```java
executor.execute(producer);
executor.execute(consumer);

executor.awaitTermination(500, TimeUnit.MILLISECONDS);
executor.shutdown();
assertEquals(queue.size(), 0);
```

它将产生以下输出:

```java
Saving an element: 339626897 to the exchange point
consumed an element: 339626897 from the exchange point
```

我们可以看到,`SynchronousQueue` 被用作线程之间的交换点，这比前面的例子好得多，也更容易理解，前面的例子使用了共享状态和`CountDownLatch.`

## 5。结论

在这个快速教程中，我们看了一下`SynchronousQueue`构造。我们创建了一个使用共享状态在两个线程之间交换数据的程序，然后重写该程序以利用`SynchronousQueue` 结构。这充当协调生产者和消费者线程的交换点。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220706104903/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。