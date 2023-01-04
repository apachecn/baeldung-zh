# threadpooltasktexecutor corePoolSize 与 maxPoolSize

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-threadpooltaskexecutor-core-vs-max-poolsize>

## 1.概观

Spring [`ThreadPoolTaskExecutor`](https://web.archive.org/web/20221127063856/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html) 是一个 JavaBean，它提供了一个围绕 [`java.util.concurrent.ThreadPoolExecutor`](/web/20221127063856/https://www.baeldung.com/java-executor-service-tutorial) 实例的抽象，并将其公开为 Spring [`org.springframework.core.task.TaskExecutor`](https://web.archive.org/web/20221127063856/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/TaskExecutor.html) 。此外，通过`corePoolSize, maxPoolSize, queueCapacity, allowCoreThreadTimeOut` 和`keepAliveSeconds.`的属性，它是高度可配置的。在本教程中，我们将查看`corePoolSize`和`maxPoolSize`属性。

## 2.`corePoolSize`对`maxPoolSize`

不熟悉这种抽象的用户可能很容易对这两种配置属性的区别感到困惑。所以，我们各自独立来看。

### 2.1.`corePoolSize`

**`corePoolSize`是保持**不超时的最小工作线程数。是`ThreadPoolTaskExecutor`的一个可配置属性。然而，`ThreadPoolTaskExecutor`抽象将这个值的设置委托给底层的`java.util.concurrent.ThreadPoolExecutor` `.`来澄清，所有线程都可能超时——如果我们已经将`allowCoreThreadTimeOut`设置为`true`，那么有效地将`corePoolSize`的值设置为零。

### 2.2.`maxPoolSize`

相比之下， **`maxPoolSize`定义了可以创建的最大线程数**。类似地，`ThreadPoolTaskExecutor` 的`maxPoolSize`属性也将其值委托给底层的`java.util.concurrent.ThreadPoolExecutor`。澄清一下， **`maxPoolSize`依赖于`queueCapacity`** ，因为`ThreadPoolTaskExecutor`只会在其队列中的项目数量超过`queueCapacity`时创建一个新线程。

## 3.那么有什么区别呢？

`corePoolSize`和`maxPoolSize`之间的区别似乎很明显。然而，他们的行为有一些微妙之处。

当我们向`ThreadPoolTaskExecutor,`提交一个新任务时，如果少于`corePoolSize`个线程正在运行，即使池中有空闲线程，或者如果少于`maxPoolSize`个线程正在运行并且由`queueCapacity` 定义的队列已满，它也会创建一个新线程。

接下来，让我们看一些代码，看看每个属性何时生效的例子。

## 4.例子

首先，假设我们有一个执行新线程的方法，从`ThreadPoolTaskExecutor`开始，命名为`startThreads`:

```java
public void startThreads(ThreadPoolTaskExecutor taskExecutor, CountDownLatch countDownLatch, 
  int numThreads) {
    for (int i = 0; i < numThreads; i++) {
        taskExecutor.execute(() -> {
            try {
                Thread.sleep(100L * ThreadLocalRandom.current().nextLong(1, 10));
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }
}
```

让我们测试一下`ThreadPoolTaskExecutor`的默认配置，它定义了一个线程的`corePoolSize`，一个无界的`maxPoolSize,`和一个无界的`queueCapacity`。因此，我们希望无论我们启动多少任务，都只有一个线程在运行:

```java
@Test
public void whenUsingDefaults_thenSingleThread() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
    taskExecutor.afterPropertiesSet();

    CountDownLatch countDownLatch = new CountDownLatch(10);
    this.startThreads(taskExecutor, countDownLatch, 10);

    while (countDownLatch.getCount() > 0) {
        Assert.assertEquals(1, taskExecutor.getPoolSize());
    }
}
```

现在，让我们将`corePoolSize`改为最多五个线程，并确保它的行为与宣传的一样。因此，无论提交给`ThreadPoolTaskExecutor`的任务数量有多少，我们都希望启动五个线程:

```java
@Test
public void whenCorePoolSizeFive_thenFiveThreads() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
    taskExecutor.setCorePoolSize(5);
    taskExecutor.afterPropertiesSet();

    CountDownLatch countDownLatch = new CountDownLatch(10);
    this.startThreads(taskExecutor, countDownLatch, 10);

    while (countDownLatch.getCount() > 0) {
        Assert.assertEquals(5, taskExecutor.getPoolSize());
    }
}
```

类似地，我们可以将`maxPoolSize`增加到 10，而将`corePoolSize`保持在 5。因此，我们预计只启动五个线程。澄清一下，只有五个线程启动，因为`queueCapacity`仍然是无界的:

```java
@Test
public void whenCorePoolSizeFiveAndMaxPoolSizeTen_thenFiveThreads() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
    taskExecutor.setCorePoolSize(5);
    taskExecutor.setMaxPoolSize(10);
    taskExecutor.afterPropertiesSet();

    CountDownLatch countDownLatch = new CountDownLatch(10);
    this.startThreads(taskExecutor, countDownLatch, 10);

    while (countDownLatch.getCount() > 0) {
        Assert.assertEquals(5, taskExecutor.getPoolSize());
    }
}
```

此外，我们现在将重复之前的测试，但是将`queueCapacity`增加到 10，并启动 20 个线程。因此，我们现在期望总共启动十个线程:

```java
@Test
public void whenCorePoolSizeFiveAndMaxPoolSizeTenAndQueueCapacityTen_thenTenThreads() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
    taskExecutor.setCorePoolSize(5);
    taskExecutor.setMaxPoolSize(10);
    taskExecutor.setQueueCapacity(10);
    taskExecutor.afterPropertiesSet();

    CountDownLatch countDownLatch = new CountDownLatch(20);
    this.startThreads(taskExecutor, countDownLatch, 20);

    while (countDownLatch.getCount() > 0) {
        Assert.assertEquals(10, taskExecutor.getPoolSize());
    }
}
```

同样，如果我们将`queueCapactity`设置为零，并且只开始十个任务，那么我们的`ThreadPoolTaskExecutor`中也会有十个线程。

## 5.结论

`ThreadPoolTaskExecutor`是围绕`java.util.concurrent.ThreadPoolExecutor`的强大抽象，提供了配置`corePoolSize`、`maxPoolSize`和`queueCapacity`的选项。在本教程中，我们查看了`corePoolSize` 和`maxPoolSize`属性，以及`maxPoolSize`如何与`queueCapacity`协同工作，这使得我们可以轻松地为任何用例创建线程池。

和往常一样，你可以在 Github 上找到可用的代码[。](https://web.archive.org/web/20221127063856/https://github.com/eugenp/tutorials/tree/master/spring-threads)