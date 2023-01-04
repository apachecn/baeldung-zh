# Java cyclic barrier vs CountDownLatch

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cyclicbarrier-countdownlatch>

## 1.介绍

在本教程中，我们将比较`CyclicBarrier`和`CountDownLatch`，并尝试了解两者的异同。

## 2.这些是什么？

当谈到并发性时，概念化每一个的目的是什么是一个挑战。

首先， **`CountDownLatch`和`CyclicBarrier`都用于管理多线程应用**。

并且，**它们都旨在表达一个给定的线程或一组线程应该如何等待。**

### 2.1.`CountDownLatch`

一个`CountDownLatch`是一个构造，一个线程`wait`打开，而其他线程`count down`打开，直到它达到零。

我们可以把这想象成餐馆里正在准备的一道菜。不管哪个厨师准备了多少`n `道菜，服务员必须`wait`直到所有的菜都在盘子里。如果一个盘子里有`n `物品，任何厨师都会为她放在盘子里的每一件物品`count down `上插销。

### 2.2.`CyclicBarrier`

一个`CyclicBarrier `是一个可重用的构造，其中一组线程`waits`在一起，直到所有的线程`arrive`。在这一点上，障碍被打破，可以选择采取`action`。

我们可以把这个想象成一群朋友。每次他们计划在餐馆吃饭时，他们都决定一个他们可以见面的共同地点。他们在那里互相为对方`wait `，只有当大家`arrives`才能一起去餐厅吃饭。

### 2.3.进一步阅读

关于每一个的更多细节，请分别参考我们之前关于`[CountDownLatch](/web/20220929100636/https://www.baeldung.com/java-countdown-latch)`和`[CyclicBarrier](/web/20220929100636/https://www.baeldung.com/java-cyclic-barrier)`的教程。

## 3.任务与线程

让我们更深入地研究一下这两个类之间的一些语义差异。

如定义中所述，`CyclicBarrier`允许多个线程相互等待，而`CountDownLatch`允许一个或多个线程等待多个任务完成。

简而言之， **`CyclicBarrier`保持计数`threads`** ，而 **`CountDownLatch`保持计数`tasks`** 。

在下面的代码中，我们定义了一个计数为 2 的`CountDownLatch` 。接下来，我们从一个线程中调用`countDown()`两次:

```java
CountDownLatch countDownLatch = new CountDownLatch(2);
Thread t = new Thread(() -> {
    countDownLatch.countDown();
    countDownLatch.countDown();
});
t.start();
countDownLatch.await();

assertEquals(0, countDownLatch.getCount());
```

一旦锁存器达到零，对`await `的调用返回。

注意，在这种情况下，**我们能够让同一个线程减少两次计数。**

**`CyclicBarrier,` 虽然如此，在这一点上却有所不同。**

类似于上面的例子，我们再次创建一个计数为 2 的`CyclicBarrier,` ,并对其调用`await()`,这次是从同一个线程:

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(2);
Thread t = new Thread(() -> {
    try {
        cyclicBarrier.await();
        cyclicBarrier.await();    
    } catch (InterruptedException | BrokenBarrierException e) {
        // error handling
    }
});
t.start();

assertEquals(1, cyclicBarrier.getNumberWaiting());
assertFalse(cyclicBarrier.isBroken());
```

这里的第一个区别是等待的线程本身就是障碍。

第二，也是更重要的一点，**第二个`await()`是没用的**。**单线程不能`count down` 一个障碍两次。**

事实上，因为`t`必须`wait`让另一个线程调用`await()`——使计数变为 2——`t`对`await() `的第二次调用实际上不会被调用，直到障碍被打破！

在我们的测试中，**障碍没有被跨越，因为我们只有一个线程在等待，而不是需要两个线程来触发障碍。**这在返回`false`的`cyclicBarrier.isBroken()`方法中也很明显。

## 4.复用性

这两个类之间第二个最明显的区别是可重用性。详细来说，**当栅栏在`CyclicBarrier`跳闸时，计数重置为初始值。** **`CountDownLatch` 不同是因为计数从不重置。**

在给定的代码中，我们定义了一个计数为 7 的`CountDownLatch`,并通过 20 个不同的调用对其进行计数:

```java
CountDownLatch countDownLatch = new CountDownLatch(7);
ExecutorService es = Executors.newFixedThreadPool(20);
for (int i = 0; i < 20; i++) {
    es.execute(() -> {
        long prevValue = countDownLatch.getCount();
        countDownLatch.countDown();
        if (countDownLatch.getCount() != prevValue) {
            outputScraper.add("Count Updated");
        }
    }); 
} 
es.shutdown();

assertTrue(outputScraper.size() <= 7);
```

我们观察到，即使有 20 个不同的线程调用`countDown()`，计数一旦达到零就不会重置。

类似于上面的例子，我们定义了一个计数为 7 的`CyclicBarrier `,并从 20 个不同的线程中等待它:

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(7);

ExecutorService es = Executors.newFixedThreadPool(20);
for (int i = 0; i < 20; i++) {
    es.execute(() -> {
        try {
            if (cyclicBarrier.getNumberWaiting() <= 0) {
                outputScraper.add("Count Updated");
            }
            cyclicBarrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            // error handling
        }
    });
}
es.shutdown();

assertTrue(outputScraper.size() > 7);
```

在这种情况下，我们观察到，每次新线程运行时，该值都会减少，一旦它达到零，就会重置为原始值。

## 5.结论

总而言之，`CyclicBarrier`和`CountDownLatch`都是多线程间同步的有用工具。然而，就它们提供的功能而言，它们是根本不同的。在决定哪一个适合这项工作时，要仔细考虑每一个。

像往常一样，所有讨论过的例子都可以在 Github 上访问[。](https://web.archive.org/web/20220929100636/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)