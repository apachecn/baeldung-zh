# Java 中的信号量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-semaphore>

## 1。概述

在这个快速教程中，我们将探索 Java 中的[信号量](/web/20220923173814/https://www.baeldung.com/cs/semaphore)和互斥体的基础知识。

## 2。`Semaphore`

我们从`java.util.concurrent.Semaphore.` 开始，我们可以使用信号量来限制访问特定资源的并发线程的数量。

在下面的示例中，我们将实现一个简单的登录队列来限制系统中的用户数量:

```
class LoginQueueUsingSemaphore {

    private Semaphore semaphore;

    public LoginQueueUsingSemaphore(int slotLimit) {
        semaphore = new Semaphore(slotLimit);
    }

    boolean tryLogin() {
        return semaphore.tryAcquire();
    }

    void logout() {
        semaphore.release();
    }

    int availableSlots() {
        return semaphore.availablePermits();
    }

}
```

注意我们是如何使用以下方法的:

*   `tryAcquire()`–如果许可立即可用，则返回真，否则获取许可，否则返回假，但是`acquire()` 获取许可并阻塞，直到一个许可可用
*   `release() – release a permit`
*   `availablePermits() –` 返回当前可用许可证的数量

为了测试我们的登录队列，我们将首先尝试达到限制，并检查下一次登录尝试是否会被阻止:

```
@Test
public void givenLoginQueue_whenReachLimit_thenBlocked() {
    int slots = 10;
    ExecutorService executorService = Executors.newFixedThreadPool(slots);
    LoginQueueUsingSemaphore loginQueue = new LoginQueueUsingSemaphore(slots);
    IntStream.range(0, slots)
      .forEach(user -> executorService.execute(loginQueue::tryLogin));
    executorService.shutdown();

    assertEquals(0, loginQueue.availableSlots());
    assertFalse(loginQueue.tryLogin());
}
```

接下来，我们将查看注销后是否有可用的插槽:

```
@Test
public void givenLoginQueue_whenLogout_thenSlotsAvailable() {
    int slots = 10;
    ExecutorService executorService = Executors.newFixedThreadPool(slots);
    LoginQueueUsingSemaphore loginQueue = new LoginQueueUsingSemaphore(slots);
    IntStream.range(0, slots)
      .forEach(user -> executorService.execute(loginQueue::tryLogin));
    executorService.shutdown();
    assertEquals(0, loginQueue.availableSlots());
    loginQueue.logout();

    assertTrue(loginQueue.availableSlots() > 0);
    assertTrue(loginQueue.tryLogin());
}
```

## 3。定时`Semaphore`

接下来，我们将讨论 Apache Commons `TimedSemaphore.` `TimedSemaphore`允许一些许可作为简单的信号量，但是在给定的时间段内，在该时间段之后，时间重置并且所有许可被释放。

我们可以使用`TimedSemaphore`建立一个简单的延迟队列，如下所示:

```
class DelayQueueUsingTimedSemaphore {

    private TimedSemaphore semaphore;

    DelayQueueUsingTimedSemaphore(long period, int slotLimit) {
        semaphore = new TimedSemaphore(period, TimeUnit.SECONDS, slotLimit);
    }

    boolean tryAdd() {
        return semaphore.tryAcquire();
    }

    int availableSlots() {
        return semaphore.getAvailablePermits();
    }

}
```

当我们使用以一秒钟为时间段的延迟队列，并且在一秒钟内使用了所有时隙之后，应该没有可用的时隙:

```
public void givenDelayQueue_whenReachLimit_thenBlocked() {
    int slots = 50;
    ExecutorService executorService = Executors.newFixedThreadPool(slots);
    DelayQueueUsingTimedSemaphore delayQueue 
      = new DelayQueueUsingTimedSemaphore(1, slots);

    IntStream.range(0, slots)
      .forEach(user -> executorService.execute(delayQueue::tryAdd));
    executorService.shutdown();

    assertEquals(0, delayQueue.availableSlots());
    assertFalse(delayQueue.tryAdd());
}
```

但是在休眠一段时间后，**信号量应该复位并释放许可**:

```
@Test
public void givenDelayQueue_whenTimePass_thenSlotsAvailable() throws InterruptedException {
    int slots = 50;
    ExecutorService executorService = Executors.newFixedThreadPool(slots);
    DelayQueueUsingTimedSemaphore delayQueue = new DelayQueueUsingTimedSemaphore(1, slots);
    IntStream.range(0, slots)
      .forEach(user -> executorService.execute(delayQueue::tryAdd));
    executorService.shutdown();

    assertEquals(0, delayQueue.availableSlots());
    Thread.sleep(1000);
    assertTrue(delayQueue.availableSlots() > 0);
    assertTrue(delayQueue.tryAdd());
}
```

## 4。信号量与互斥量

互斥的行为类似于二元信号量，我们可以用它来实现互斥。

在下面的例子中，我们将使用一个简单的二进制信号量来构建一个计数器:

```
class CounterUsingMutex {

    private Semaphore mutex;
    private int count;

    CounterUsingMutex() {
        mutex = new Semaphore(1);
        count = 0;
    }

    void increase() throws InterruptedException {
        mutex.acquire();
        this.count = this.count + 1;
        Thread.sleep(1000);
        mutex.release();

    }

    int getCount() {
        return this.count;
    }

    boolean hasQueuedThreads() {
        return mutex.hasQueuedThreads();
    }
}
```

当许多线程试图同时访问计数器时，**它们会被简单地阻塞在一个队列中**:

```
@Test
public void whenMutexAndMultipleThreads_thenBlocked()
 throws InterruptedException {
    int count = 5;
    ExecutorService executorService
     = Executors.newFixedThreadPool(count);
    CounterUsingMutex counter = new CounterUsingMutex();
    IntStream.range(0, count)
      .forEach(user -> executorService.execute(() -> {
          try {
              counter.increase();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }));
    executorService.shutdown();

    assertTrue(counter.hasQueuedThreads());
}
```

当我们等待时，所有线程都将访问计数器，队列中没有线程剩余:

```
@Test
public void givenMutexAndMultipleThreads_ThenDelay_thenCorrectCount()
 throws InterruptedException {
    int count = 5;
    ExecutorService executorService
     = Executors.newFixedThreadPool(count);
    CounterUsingMutex counter = new CounterUsingMutex();
    IntStream.range(0, count)
      .forEach(user -> executorService.execute(() -> {
          try {
              counter.increase();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }));
    executorService.shutdown();

    assertTrue(counter.hasQueuedThreads());
    Thread.sleep(5000);
    assertFalse(counter.hasQueuedThreads());
    assertEquals(count, counter.getCount());
}
```

## 5。结论

在本文中，我们探索了 Java 中信号量的基础知识。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220923173814/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)