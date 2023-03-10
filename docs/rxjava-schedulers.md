# RxJava 中的调度程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-schedulers>

## 1。概述

在本文中，我们将关注不同类型的`Schedulers` ，我们将使用它们来编写基于 RxJava `Observable's subscribeOn` 和`observeOn` 方法的多线程程序。

`Schedulers`提供机会指定在何处以及何时执行与`Observable`链的操作相关的任务。

我们可以从类`[Schedulers](https://web.archive.org/web/20221208143837/http://reactivex.io/RxJava/javadoc/rx/schedulers/Schedulers.html).`中描述的工厂方法中获得一个`Scheduler`

## 2。默认线程行为

**默认情况下，** **Rx 是单线程的**，这意味着一个`Observable`和我们可以应用到它的操作符链将通知它的观察者在同一个线程上调用它的`subscribe()`方法。

`observeOn` 和`subscribeOn`方法将一个`Scheduler,`作为参数，顾名思义，它是一个工具，我们可以用它来安排单个动作。

我们将通过使用`create` `Worker`方法创建一个`Scheduler` 的实现，该方法返回一个`[Scheduler.Worker](https://web.archive.org/web/20221208143837/http://reactivex.io/RxJava/javadoc/rx/Scheduler.Worker.html).``worker`接受动作并在单线程上顺序执行它们。

在某种程度上，`worker`本身就是一个`S`调度器，但是为了避免混淆，我们不会把它称为`Scheduler`。

### 2.1。安排行动

我们可以通过创建一个新的`worker`并调度一些动作来调度任何`Scheduler`上的作业:

```java
Scheduler scheduler = Schedulers.immediate();
Scheduler.Worker worker = scheduler.createWorker();
worker.schedule(() -> result += "action");

Assert.assertTrue(result.equals("action"));
```

然后，该操作在分配给该工作线程的线程上排队。

### 2.2。取消动作

`Scheduler.Worker` 延伸`Subscription`。在`worker`上调用`unsubscribe`方法将导致队列被清空，所有未决任务被取消。我们可以通过例子看出:

```java
Scheduler scheduler = Schedulers.newThread();
Scheduler.Worker worker = scheduler.createWorker();
worker.schedule(() -> {
    result += "First_Action";
    worker.unsubscribe();
});
worker.schedule(() -> result += "Second_Action");

Assert.assertTrue(result.equals("First_Action"));
```

第二个任务永远不会执行，因为前一个任务取消了整个操作。正在执行的操作将被中断。

## 3。`Schedulers.newThread`

每当通过`subscribeOn()` 或`observeOn()`请求一个新线程时，这个调度程序就启动一个新线程。

这几乎不是一个好的选择，不仅因为启动线程时的延迟，还因为这个线程没有被重用:

```java
Observable.just("Hello")
  .observeOn(Schedulers.newThread())
  .doOnNext(s ->
    result2 += Thread.currentThread().getName()
  )
  .observeOn(Schedulers.newThread())
  .subscribe(s ->
    result1 += Thread.currentThread().getName()
  );
Thread.sleep(500);
Assert.assertTrue(result1.equals("RxNewThreadScheduler-1"));
Assert.assertTrue(result2.equals("RxNewThreadScheduler-2"));
```

当`Worker`完成时，线程简单地终止。这个`Scheduler`只能在任务是粗粒度的时候使用:它需要很多时间来完成，但是它们很少，所以线程根本不可能被重用。

```java
Scheduler scheduler = Schedulers.newThread();
Scheduler.Worker worker = scheduler.createWorker();
worker.schedule(() -> {
    result += Thread.currentThread().getName() + "_Start";
    worker.schedule(() -> result += "_worker_");
    result += "_End";
});
Thread.sleep(3000);
Assert.assertTrue(result.equals(
  "RxNewThreadScheduler-1_Start_End_worker_"));
```

当我们在一个`NewThreadScheduler,` 上安排`worker`时，我们看到工人被绑定到一个特定的线程。

## 4。`Schedulers.immediate`

`Schedulers.immediate`是一个特殊的调度程序，它以阻塞的方式调用客户端线程中的任务，而不是异步调用，并在动作完成时返回:

```java
Scheduler scheduler = Schedulers.immediate();
Scheduler.Worker worker = scheduler.createWorker();
worker.schedule(() -> {
    result += Thread.currentThread().getName() + "_Start";
    worker.schedule(() -> result += "_worker_");
    result += "_End";
});
Thread.sleep(500);
Assert.assertTrue(result.equals(
  "main_Start_worker__End"));
```

事实上，通过`immediate Scheduler`订阅`Observable`通常与根本不订阅任何特定的`S`日程具有相同的效果:

```java
Observable.just("Hello")
  .subscribeOn(Schedulers.immediate())
  .subscribe(s ->
    result += Thread.currentThread().getName()
  );
Thread.sleep(500);
Assert.assertTrue(result.equals("main"));
```

## 5。`Schedulers.trampoline`

`trampoline` `Scheduler`与`immediate`非常相似，因为它也在同一个线程中调度任务，有效地阻塞。

然而，当所有先前计划的任务完成时，即将到来的任务被执行:

```java
Observable.just(2, 4, 6, 8)
  .subscribeOn(Schedulers.trampoline())
  .subscribe(i -> result += "" + i);
Observable.just(1, 3, 5, 7, 9)
  .subscribeOn(Schedulers.trampoline())
  .subscribe(i -> result += "" + i);
Thread.sleep(500);
Assert.assertTrue(result.equals("246813579"));
```

`Immediate`立即调用给定的任务，而`trampoline` 等待当前任务完成。

`trampoline`的`worker`在调度第一个任务的线程上执行每个任务。对`schedule`的第一个调用被阻塞，直到队列被清空:

```java
Scheduler scheduler = Schedulers.trampoline();
Scheduler.Worker worker = scheduler.createWorker();
worker.schedule(() -> {
    result += Thread.currentThread().getName() + "Start";
    worker.schedule(() -> {
        result += "_middleStart";
        worker.schedule(() ->
            result += "_worker_"
        );
        result += "_middleEnd";
    });
    result += "_mainEnd";
});
Thread.sleep(500);
Assert.assertTrue(result
  .equals("mainStart_mainEnd_middleStart_middleEnd_worker_"));
```

## 6。`Schedulers.from`

`Schedulers` 在内部比`java.util.concurrent` 的`Executors`更复杂——所以需要一个单独的抽象。

但是因为它们在概念上非常相似，所以毫不奇怪有一个包装器可以使用`from`工厂方法将`Executor`转换成`Scheduler`:

```java
private ThreadFactory threadFactory(String pattern) {
    return new ThreadFactoryBuilder()
      .setNameFormat(pattern)
      .build();
}

@Test
public void givenExecutors_whenSchedulerFrom_thenReturnElements() 
 throws InterruptedException {

    ExecutorService poolA = newFixedThreadPool(
      10, threadFactory("Sched-A-%d"));
    Scheduler schedulerA = Schedulers.from(poolA);
    ExecutorService poolB = newFixedThreadPool(
      10, threadFactory("Sched-B-%d"));
    Scheduler schedulerB = Schedulers.from(poolB);

    Observable<String> observable = Observable.create(subscriber -> {
      subscriber.onNext("Alfa");
      subscriber.onNext("Beta");
      subscriber.onCompleted();
    });;

    observable
      .subscribeOn(schedulerA)
      .subscribeOn(schedulerB)
      .subscribe(
        x -> result += Thread.currentThread().getName() + x + "_",
        Throwable::printStackTrace,
        () -> result += "_Completed"
      );
    Thread.sleep(2000);
    Assert.assertTrue(result.equals(
      "Sched-A-0Alfa_Sched-A-0Beta__Completed"));
}
```

`SchedulerB`被使用了一小段时间，但是它几乎没有在`schedulerA`上安排新的动作，它完成了所有的工作。因此，多个`subscribeOn methods`不仅会被忽略，还会引入少量开销。

## 7。`Schedulers.io`

这个`Scheduler`类似于`newThread`,除了已经启动的线程被回收并可能处理未来的请求。

这个实现的工作方式类似于来自`java.util.concurrent`的`ThreadPoolExecutor` ,具有一个无限的线程池。每次请求一个新的`worker`时，要么启动一个新线程(稍后保持空闲一段时间)，要么重用空闲的线程:

```java
Observable.just("io")
  .subscribeOn(Schedulers.io())
  .subscribe(i -> result += Thread.currentThread().getName());

Assert.assertTrue(result.equals("RxIoScheduler-2"));
```

我们需要小心任何种类的无限资源——在像 web 服务这样缓慢或无响应的外部依赖的情况下，`io` `scheduler`可能会启动大量的线程，导致我们自己的应用程序变得无响应。

实际上，跟随`Schedulers.io`几乎总是更好的选择。

## 8。`Schedulers.computation`

默认情况下，`Computation` S `cheduler`将并行运行的线程数量限制为`availableProcessors()`的值，如`Runtime.getRuntime()`实用程序类中所示。

因此，当任务完全受限于 CPU 时，我们应该使用`computation scheduler`；也就是说，它们需要计算能力，并且没有阻塞代码。

它在每个线程前面使用一个无限队列，所以如果任务被调度，但所有内核都被占用，它将被排队。但是，就在每个线程之前的队列会不断增长:

```java
Observable.just("computation")
  .subscribeOn(Schedulers.computation())
  .subscribe(i -> result += Thread.currentThread().getName());
Assert.assertTrue(result.equals("RxComputationScheduler-1"));
```

如果出于某种原因，我们需要不同于缺省数量的线程，我们总是可以使用`rx.scheduler.max-computation-threads`系统属性。

通过使用更少的线程，我们可以确保始终有一个或多个 CPU 内核空闲，即使在重负载下，`computation`线程池也不会使服务器饱和。计算线程不可能比内核多。

## 9。`Schedulers.test`

这个`Scheduler`仅用于测试目的，我们永远不会在产品代码中看到它。它的主要优点是能够提前时钟，模拟时间任意流逝:

```java
List<String> letters = Arrays.asList("A", "B", "C");
TestScheduler scheduler = Schedulers.test();
TestSubscriber<String> subscriber = new TestSubscriber<>();

Observable<Long> tick = Observable
  .interval(1, TimeUnit.SECONDS, scheduler);

Observable.from(letters)
  .zipWith(tick, (string, index) -> index + "-" + string)
  .subscribeOn(scheduler)
  .subscribe(subscriber);

subscriber.assertNoValues();
subscriber.assertNotCompleted();

scheduler.advanceTimeBy(1, TimeUnit.SECONDS);
subscriber.assertNoErrors();
subscriber.assertValueCount(1);
subscriber.assertValues("0-A");

scheduler.advanceTimeTo(3, TimeUnit.SECONDS);
subscriber.assertCompleted();
subscriber.assertNoErrors();
subscriber.assertValueCount(3);
assertThat(
  subscriber.getOnNextEvents(), 
  hasItems("0-A", "1-B", "2-C"));
```

## 10。默认调度程序

RxJava 中的一些`Observable`操作符有替代形式，允许我们设置操作符将使用哪个`Scheduler`进行操作。其他人不操作任何特定的`Scheduler`或操作特定的默认`Scheduler`。

例如，`delay`操作符获取上游事件，并在给定时间后将它们推到下游。显然，它不能在此期间保持原来的线程，所以它必须使用不同的`Scheduler`:

```java
ExecutorService poolA = newFixedThreadPool(
  10, threadFactory("Sched1-"));
Scheduler schedulerA = Schedulers.from(poolA);
Observable.just('A', 'B')
  .delay(1, TimeUnit.SECONDS, schedulerA)
  .subscribe(i -> result+= Thread.currentThread().getName() + i + " ");

Thread.sleep(2000);
Assert.assertTrue(result.equals("Sched1-A Sched1-B "));
```

如果不提供自定义的`schedulerA`，那么`delay`以下的所有操作员都将使用`computation Scheduler`。

支持自定义`Schedulers`的其他重要运算符有`buffer,` `interval``range``timer``skip``take``timeout`等。如果我们不为这样的操作符提供`Scheduler`，那么就使用`computation` 调度器，这在大多数情况下是一个安全的缺省值。

## 11。结论

在真正的反应式应用中，所有长时间运行的操作都是异步的，只需要很少的线程，因此`Schedulers`。

掌握调度程序对于使用 RxJava 编写可伸缩且安全的代码至关重要。`subscribeOn`和`observeOn`之间的差异在高负载下尤其重要，在高负载下，每个任务都必须在我们期望的时间精确执行。

最后但同样重要的是，我们必须确保下游使用的`Schedulers`能跟上`Schedulers` upstrea m 产生的 lo ad，更多信息，有这篇关于[背压](/web/20221208143837/https://www.baeldung.com/rxjava-backpressure)的文章。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。