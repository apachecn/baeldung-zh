# 在 Java 中调试反应流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-debugging-reactive-streams>

## 1.概观

一旦我们开始使用这些数据结构，调试[反应流](/web/20221126215822/https://www.baeldung.com/reactor-core)可能是我们必须面对的主要挑战之一。

记住，反应流在过去几年里越来越受欢迎，了解我们如何有效地执行这项任务是一个好主意。

让我们从使用反应式堆栈建立一个项目开始，看看为什么这经常是麻烦的。

## 2.有 bug 的场景

我们想要模拟一个真实的场景，其中几个异步进程正在运行，我们在代码中引入了一些最终会触发异常的缺陷。

为了理解大图，我们将提到我们的应用程序将消费和处理简单的`Foo`对象流，这些对象只包含一个`id`、`formattedName`和一个`quantity`字段。

### 2.1.分析日志输出

现在，让我们检查一个代码片段以及当出现未处理的错误时它生成的输出:

```
public void processFoo(Flux<Foo> flux) {
    flux.map(FooNameHelper::concatFooName)
      .map(FooNameHelper::substringFooName)
      .map(FooReporter::reportResult)
      .subscribe();
}

public void processFooInAnotherScenario(Flux<Foo> flux) {
    flux.map(FooNameHelper::substringFooName)
      .map(FooQuantityHelper::divideFooQuantity)
      .subscribe();
}
```

在运行我们的应用程序几秒钟后，我们会意识到它会不时地记录异常。

仔细查看其中一个错误，我们会发现类似这样的内容:

```
Caused by: java.lang.StringIndexOutOfBoundsException: String index out of range: 15
    at j.l.String.substring(String.java:1963)
    at com.baeldung.debugging.consumer.service.FooNameHelper
      .lambda$1(FooNameHelper.java:38)
    at r.c.p.FluxMap$MapSubscriber.onNext(FluxMap.java:100)
    at r.c.p.FluxMap$MapSubscriber.onNext(FluxMap.java:114)
    at r.c.p.FluxConcatMap$ConcatMapImmediate.innerNext(FluxConcatMap.java:275)
    at r.c.p.FluxConcatMap$ConcatMapInner.onNext(FluxConcatMap.java:849)
    at r.c.p.Operators$MonoSubscriber.complete(Operators.java:1476)
    at r.c.p.MonoDelayUntil$DelayUntilCoordinator.signal(MonoDelayUntil.java:211)
    at r.c.p.MonoDelayUntil$DelayUntilTrigger.onComplete(MonoDelayUntil.java:290)
    at r.c.p.MonoDelay$MonoDelayRunnable.run(MonoDelay.java:118)
    at r.c.s.SchedulerTask.call(SchedulerTask.java:50)
    at r.c.s.SchedulerTask.call(SchedulerTask.java:27)
    at j.u.c.FutureTask.run(FutureTask.java:266)
    at j.u.c.ScheduledThreadPoolExecutor$ScheduledFutureTask
      .access$201(ScheduledThreadPoolExecutor.java:180)
    at j.u.c.ScheduledThreadPoolExecutor$ScheduledFutureTask
      .run(ScheduledThreadPoolExecutor.java:293)
    at j.u.c.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at j.u.c.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at j.l.Thread.run(Thread.java:748)
```

根据根本原因，并注意堆栈跟踪中提到的`FooNameHelper`类，我们可以想象在某些情况下，我们的`Foo` 对象正在用比预期更短的`formattedName` 值进行处理。

当然，这只是一个简化的情况，解决方案似乎相当明显。

但是让我们想象这是一个真实的场景，在没有一些上下文信息的情况下，异常本身并不能帮助我们解决问题。

异常是作为`processFoo,`还是`processFooInAnotherScenario`方法的一部分被触发的？

在到达此阶段之前，之前的其他步骤是否影响了`formattedName`字段？

日志条目不会帮助我们解决这些问题。

更糟糕的是，有时异常甚至不是从我们的功能内部抛出的。

例如，假设我们依赖一个反应式存储库来保存我们的`Foo`对象。如果此时出现错误，我们可能甚至不知道从哪里开始调试代码。

我们需要工具来有效地调试反应流。

## 3.使用调试会话

了解我们的应用程序发生了什么的一个选项是使用我们最喜欢的 IDE 启动一个调试会话。

我们必须设置几个条件断点，并在执行数据流中的每一步时分析数据流。

事实上，这可能是一项繁琐的任务，尤其是当我们有许多反应式进程在运行和共享资源时。

此外，在许多情况下，出于安全原因，我们无法启动调试会话。

## 4.用`doOnErrorMethod`或使用 Subscribe 参数记录信息

有时，**我们可以添加有用的上下文信息，通过提供一个`Consumer`作为`subscribe`方法**的第二个参数:

```
public void processFoo(Flux<Foo> flux) {

    // ...

    flux.subscribe(foo -> {
        logger.debug("Finished processing Foo with Id {}", foo.getId());
    }, error -> {
        logger.error(
          "The following error happened on processFoo method!",
           error);
    });
}
```

**注:值得一提的是，如果我们不需要对`subscribe`方法进行进一步的处理，我们可以在我们的发布者:**上链接`doOnError`函数

```
flux.doOnError(error -> {
    logger.error("The following error happened on processFoo method!", error);
}).subscribe();
```

现在我们对错误可能来自哪里有了一些指导，尽管我们仍然没有太多关于产生异常的实际元素的信息。

## 5.激活反应器的全局调试配置

[反应器](https://web.archive.org/web/20221126215822/https://projectreactor.io/)库提供了一个 [`Hooks`](https://web.archive.org/web/20221126215822/https://projectreactor.io/docs/core/release/reference/#hooks) 类，让我们配置`Flux`和`Mono`操作符的行为。

**通过添加以下语句，我们的应用程序将检测对发布者方法的调用，包装操作符的构造，并捕获堆栈跟踪**:

```
Hooks.onOperatorDebug();
```

调试模式激活后，我们的异常日志将包含一些有用的信息:

```
16:06:35.334 [parallel-1] ERROR c.b.d.consumer.service.FooService
  - The following error happened on processFoo method!
java.lang.StringIndexOutOfBoundsException: String index out of range: 15
    at j.l.String.substring(String.java:1963)
    at c.d.b.c.s.FooNameHelper.lambda$1(FooNameHelper.java:38)
    ...
    at j.l.Thread.run(Thread.java:748)
    Suppressed: r.c.p.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.FluxMapFuseable] :
    reactor.core.publisher.Flux.map(Flux.java:5653)
    c.d.b.c.s.FooNameHelper.substringFooName(FooNameHelper.java:32)
    c.d.b.c.s.FooService.processFoo(FooService.java:24)
    c.d.b.c.c.ChronJobs.consumeInfiniteFlux(ChronJobs.java:46)
    o.s.s.s.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:84)
    o.s.s.s.DelegatingErrorHandlingRunnable
      .run(DelegatingErrorHandlingRunnable.java:54)
    o.u.c.Executors$RunnableAdapter.call(Executors.java:511)
    o.u.c.FutureTask.runAndReset(FutureTask.java:308)
Error has been observed by the following operator(s):
    |_    Flux.map ⇢ c.d.b.c.s.FooNameHelper
            .substringFooName(FooNameHelper.java:32)
    |_    Flux.map ⇢ c.d.b.c.s.FooReporter.reportResult(FooReporter.java:15)
```

如我们所见，第一部分相对保持不变，但以下部分提供了有关以下内容的信息:

1.  发布者的汇编跟踪——这里我们可以确认错误首先是在`processFoo`方法中生成的。
2.  第一次触发错误后观察到错误的操作员，以及链接他们的用户类。

注意:在这个例子中，主要是为了清楚地看到这一点，我们在不同的类上添加了操作。

我们可以随时打开或关闭调试模式，但它不会影响已经实例化的`Flux`和`Mono`对象。

### 5.1.在不同的线程上执行运算符

要记住的另一个方面是，即使有不同的线程在流上操作，也能正确生成程序集跟踪。

让我们看看下面的例子:

```
public void processFoo(Flux<Foo> flux) {
    flux.publishOn(Schedulers.newSingle("foo-thread"))
       // ...
      .publishOn(Schedulers.newSingle("bar-thread"))
      .map(FooReporter::reportResult)
      .subscribeOn(Schedulers.newSingle("starter-thread"))
      .subscribe();
}
```

现在，如果我们检查日志，我们会意识到，在这种情况下，第一部分可能会有一点点变化，但最后两部分仍然相当相同。

第一部分是线程堆栈跟踪，因此它将只显示特定线程执行的操作。

正如我们所看到的，当我们调试应用程序时，这不是最重要的部分，所以这种变化是可以接受的。

## 6.激活单个进程的调试输出

在每个单独的反应性进程中检测和生成堆栈跟踪是非常昂贵的。

因此，**我们应该只在关键情况下实施前一种方法**。

总之， **Reactor 提供了一种在单个关键进程上启用调试模式的方法，这种方法消耗的内存更少**。

我们指的是`checkpoint`操作符:

```
public void processFoo(Flux<Foo> flux) {

    // ...

    flux.checkpoint("Observed error on processFoo", true)
      .subscribe();
}
```

请注意，通过这种方式，将在检查点阶段记录程序集跟踪:

```
Caused by: java.lang.StringIndexOutOfBoundsException: String index out of range: 15
	...
Assembly trace from producer [reactor.core.publisher.FluxMap],
  described as [Observed error on processFoo] :
    r.c.p.Flux.checkpoint(Flux.java:3096)
    c.b.d.c.s.FooService.processFoo(FooService.java:26)
    c.b.d.c.c.ChronJobs.consumeInfiniteFlux(ChronJobs.java:46)
    o.s.s.s.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:84)
    o.s.s.s.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)
    j.u.c.Executors$RunnableAdapter.call(Executors.java:511)
    j.u.c.FutureTask.runAndReset(FutureTask.java:308)
Error has been observed by the following operator(s):
    |_    Flux.checkpoint ⇢ c.b.d.c.s.FooService.processFoo(FooService.java:26)
```

**我们应该在反应链的末端实施`checkpoint`方法。**

否则，操作员将无法观察到下游发生的错误。

另外，让我们注意这个库提供了一个重载的方法。我们可以避免:

*   如果使用无参数选项，则为观察到的错误指定描述
*   通过只提供自定义描述来生成填充的堆栈跟踪(这是开销最大的操作)

## 7.记录一系列元素

最后，Reactor 出版商提供了一种在某些情况下可能会派上用场的方法。

**通过调用我们的反应链中的 `log`方法，应用程序将记录流中的每个元素在那个阶段**的状态。

让我们在我们的例子中尝试一下:

```
public void processFoo(Flux<Foo> flux) {
    flux.map(FooNameHelper::concatFooName)
      .map(FooNameHelper::substringFooName)
      .log();
      .map(FooReporter::reportResult)
      .doOnError(error -> {
        logger.error("The following error happened on processFoo method!", error);
      })
      .subscribe();
}
```

并检查日志:

```
INFO  reactor.Flux.OnAssembly.1 - onSubscribe(FluxMap.MapSubscriber)
INFO  reactor.Flux.OnAssembly.1 - request(unbounded)
INFO  reactor.Flux.OnAssembly.1 - onNext(Foo(id=0, formattedName=theFo, quantity=8))
INFO  reactor.Flux.OnAssembly.1 - onNext(Foo(id=1, formattedName=theFo, quantity=3))
INFO  reactor.Flux.OnAssembly.1 - onNext(Foo(id=2, formattedName=theFo, quantity=5))
INFO  reactor.Flux.OnAssembly.1 - onNext(Foo(id=3, formattedName=theFo, quantity=6))
INFO  reactor.Flux.OnAssembly.1 - onNext(Foo(id=4, formattedName=theFo, quantity=6))
INFO  reactor.Flux.OnAssembly.1 - cancel()
ERROR c.b.d.consumer.service.FooService 
  - The following error happened on processFoo method!
...
```

我们很容易看到这个阶段每个`Foo`对象的状态，以及当异常发生时框架是如何取消流程的。

当然，这种方法也很昂贵，我们必须适度使用。

## 8.结论

如果我们不知道正确调试应用程序的工具和机制，我们可能会花费大量的时间和精力来解决问题。

如果我们不习惯处理反应式和异步数据结构，并且我们需要额外的帮助来弄清楚事情是如何工作的，这一点尤其正确。

和往常一样，完整的例子可以在 [GitHub repo](https://web.archive.org/web/20221126215822/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-reactive) 上找到。