# xJava 胡克斯

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-hooks>

## 1。概述

在本教程中，我们将学习 [RxJava](/web/20220627083908/https://www.baeldung.com/rx-java) 钩子。我们将创建一些简短的例子来演示钩子在不同情况下是如何工作的。

## 2。RxJava 钩子是什么？

顾名思义，RxJava 钩子允许我们将**钩入`Observable, `Completable`、`Maybe`、`Flowable,` 、`**、``**Single**.` 、`的生命周期中。此外，RxJava 允许我们将生命周期钩子添加到由``Schedulers.`、`返回的调度程序中。此外，我们还可以使用钩子指定一个全局错误处理程序。

在 RxJava 1 中，类`RxJavaHooks`用于定义钩子。但是，挂钩机制在 RxJava 2 中完全重写。**现在类`RxJavaHooks`不再能用来定义钩子。相反，我们应该使用`RxJavaPlugins`来实现生命周期挂钩**。

`RxJavaPlugins`类有许多设置钩子的 setter 方法。这些挂钩是全球性的。一旦它们被设置，那么我们要么调用`RxJavaPlugins`类的`reset()`方法，要么调用单个钩子的 setter 方法来移除它。

## 3。用于错误处理的钩子

我们可以使用`setErrorHandler( )`方法来处理由于下游的生命周期已经到达其终止状态而无法发出的错误。让我们看看如何实现错误处理程序并测试它:

```java
RxJavaPlugins.setErrorHandler(throwable -> {
    hookCalled = true;
});

Observable.error(new IllegalStateException()).subscribe();

assertTrue(hookCalled);
```

并不是所有的异常都按原样抛出。然而，RxJava 将检查抛出的错误是否是应该按原样传递的已经命名的 bug 案例之一，否则它将被包装到一个`UndeliverableException`中。被命名为 bug 案例的异常有:

*   `OnErrorNotImplementedException`–当用户忘记在`subscribe()`方法中添加`onError`处理程序时
*   `MissingBackpressureException –`由于操作员错误或并发`onNext`
*   发生一般协议违反时
*   `NullPointerException –`标准空指针异常
*   `IllegalArgumentException –`由于用户输入无效
*   `CompositeException –`由于处理异常时崩溃

## 4。`Completable`钩子为

RxJava [`Completable`](/web/20220627083908/https://www.baeldung.com/rxjava-completable) 有两个生命周期挂钩。现在让我们来看看它们。

### 4.1。`setOnCompletableAssembly`

RxJava 在`Completable`上实例化操作符和源时会调用这个钩子。我们可以使用当前的`Completable`对象，作为钩子函数的参数，对它进行任何操作:

```java
RxJavaPlugins.setOnCompletableAssembly(completable -> {
    hookCalled = true;
    return completable;
});

Completable.fromSingle(Single.just(1));

assertTrue(hookCalled);
```

### 4.2。`setOnCompletableSubscribe`

RxJava 在订阅者订阅一个`Completable`之前调用这个钩子:

```java
RxJavaPlugins.setOnCompletableSubscribe((completable, observer) -> {
    hookCalled = true;
    return observer;
});

Completable.fromSingle(Single.just(1)).test();

assertTrue(hookCalled);
```

## 5。`Observable`钩子为

接下来，让我们来看看 RxJava 对于`Observable`的三个生命周期挂钩。

### 5.1。`setOnObservableAssembly`

RxJava 在实例化`Observable`上的操作符和源时调用这个钩子:

```java
RxJavaPlugins.setOnObservableAssembly(observable -> {
    hookCalled = true;
    return observable;
});

Observable.range(1, 10);

assertTrue(hookCalled);
```

### 5.2。`setOnObservableSubscribe`

RxJava 在订阅者订阅一个`Observable`之前调用这个钩子:

```java
RxJavaPlugins.setOnObservableSubscribe((observable, observer) -> {
    hookCalled = true;
    return observer;
});

Observable.range(1, 10).test();

assertTrue(hookCalled);
```

### 5.3。`setOnConnectableObservableAssembly`

这个钩子是为`ConnectableObservable`准备的。一个`ConnectableObservable`是一个`Observable`本身的变体。唯一的区别是，当它被订阅时，它不开始发出项目，而是只有当它的`connect()`方法被调用时才开始发出项目:

```java
RxJavaPlugins.setOnConnectableObservableAssembly(connectableObservable -> {
    hookCalled = true;
    return connectableObservable;
});

ConnectableObservable.range(1, 10).publish().connect();

assertTrue(hookCalled);
```

## 6。`Flowable`钩子为

现在，让我们看看为 [`Flowable`](/web/20220627083908/https://www.baeldung.com/rxjava-2-flowable) 定义的生命周期挂钩。

### 6.1。`setOnFlowableAssembly`

RxJava 在实例化`Flowable`上的操作符和源时调用这个钩子:

```java
RxJavaPlugins.setOnFlowableAssembly(flowable -> {
    hookCalled = true;
    return flowable;
});

Flowable.range(1, 10);

assertTrue(hookCalled);
```

### 6.2。`setOnFlowableSubscribe`

RxJava 在用户订阅`Flowable`之前调用这个钩子:

```java
RxJavaPlugins.setOnFlowableSubscribe((flowable, observer) -> {
    hookCalled = true;
    return observer;
});

Flowable.range(1, 10).test();

assertTrue(hookCalled);
```

### 6.3。`setOnConnectableFlowableAssembly`

RxJava 在`ConnectableFlowable`上实例化操作符和源时调用这个钩子。和`ConnectableObservable`一样，`ConnectableFlowable`也只有在我们调用它的`connect()`方法时才开始发射物品:

```java
RxJavaPlugins.setOnConnectableFlowableAssembly(connectableFlowable -> {
    hookCalled = true;
    return connectableFlowable;
});

ConnectableFlowable.range(1, 10).publish().connect();

assertTrue(hookCalled);
```

### 6.4。`setOnParallelAssembly`

一个`ParallelFlowable`是为了在多个发布者之间实现并行。RxJava 在实例化`ParallelFlowable`上的操作符和源时调用`setOnParallelAssembly()`钩子:

```java
RxJavaPlugins.setOnParallelAssembly(parallelFlowable -> {
    hookCalled = true;
    return parallelFlowable;
});

Flowable.range(1, 10).parallel();

assertTrue(hookCalled);
```

## 7。`Maybe`钩子为

[`Maybe`](/web/20220627083908/https://www.baeldung.com/rxjava-maybe) 发射器定义了两个钩子来控制它的生命周期。

### 7.1。`setOnMaybeAssembly`

RxJava 在实例化`Maybe`上的操作符和源时调用这个钩子:

```java
RxJavaPlugins.setOnMaybeAssembly(maybe -> {
    hookCalled = true;
    return maybe;
});

Maybe.just(1);

assertTrue(hookCalled);
```

### 7.2。`setOnMaybeSubscribe`

RxJava 在用户订阅`Maybe`之前调用这个钩子:

```java
RxJavaPlugins.setOnMaybeSubscribe((maybe, observer) -> {
    hookCalled = true;
    return observer;
});

Maybe.just(1).test();

assertTrue(hookCalled);
```

## 8。`Single`钩子为

RxJava 也为`Single`发射器定义了两个基本的钩子。

### 8.1。`setOnSingleAssembly`

RxJava 在实例化`Single`上的操作符和源时调用这个钩子:

```java
RxJavaPlugins.setOnSingleAssembly(single -> {
    hookCalled = true;
    return single;
});

Single.just(1);

assertTrue(hookCalled);
```

### 8.2。`setOnSingleSubscribe`

RxJava 在用户订阅`Single`之前调用这个钩子:

```java
RxJavaPlugins.setOnSingleSubscribe((single, observer) -> {
    hookCalled = true;
    return observer;
});

Single.just(1).test();

assertTrue(hookCalled);
```

## 9。`Schedulers`钩子为

和 RxJava 发射器一样， [`Schedulers`](/web/20220627083908/https://www.baeldung.com/rxjava-schedulers) 也有一堆钩子来控制它们的生命周期。RxJava 定义了一个通用的钩子，当我们使用任何类型的`Schedulers.`时都会调用这个钩子。此外，还可以实现特定于各种`Schedulers`的钩子。

### 9.1。`setScheduleHandler`

当我们为一个操作使用任何调度程序时，RxJava 调用这个钩子:

```java
RxJavaPlugins.setScheduleHandler((runnable) -> {
    hookCalled = true;
    return runnable;
});

Observable.range(1, 10)
  .map(v -> v * 2)
  .subscribeOn(Schedulers.single())
  .test();

hookCalled = false;

Observable.range(1, 10)
  .map(v -> v * 2)
  .subscribeOn(Schedulers.computation())
  .test();

assertTrue(hookCalled);
```

由于我们已经用`single()`和`computation()`调度器重复了这个操作，当我们运行它时，测试用例将在控制台中打印消息两次。

### 9.2。调度程序的钩子

计算调度程序有两个钩子——即`setInitComputationSchedulerHandler`和`setComputationSchedulerHandler.`

当 RxJava 初始化一个计算调度器时，它调用我们使用`setInitComputationSchedulerHandler`函数设置的钩子。此外，当我们用`Schedulers.computation()`调度一个任务时，它调用我们用`setComputationSchedulerHandler function` 设置的钩子:

```java
RxJavaPlugins.setInitComputationSchedulerHandler((scheduler) -> {
    initHookCalled = true;
    return scheduler.call();
});

RxJavaPlugins.setComputationSchedulerHandler((scheduler) -> {
    hookCalled = true;
    return scheduler;
});

Observable.range(1, 10)
  .map(v -> v * 2)
  .subscribeOn(Schedulers.computation())
  .test();

assertTrue(hookCalled && initHookCalled);
```

### 9.3。调度程序的钩子

`IO`调度器也有两个钩子——即`setInitIoSchedulerHandler`和`setIoSchedulerHandler`:

```java
RxJavaPlugins.setInitIoSchedulerHandler((scheduler) -> {
    initHookCalled = true;
    return scheduler.call();
});

RxJavaPlugins.setIoSchedulerHandler((scheduler) -> {
    hookCalled = true;
    return scheduler;
});

Observable.range(1, 10)
  .map(v -> v * 2)
  .subscribeOn(Schedulers.io())
  .test();

assertTrue(hookCalled && initHookCalled);
```

### 9.4。调度程序的钩子

现在，让我们看看`Single`调度程序的钩子:

```java
RxJavaPlugins.setInitSingleSchedulerHandler((scheduler) -> {
    initHookCalled = true;
    return scheduler.call();
});

RxJavaPlugins.setSingleSchedulerHandler((scheduler) -> {
    hookCalled = true;
    return scheduler;
});

Observable.range(1, 10)
  .map(v -> v * 2)
  .subscribeOn(Schedulers.single())
  .test();

assertTrue(hookCalled && initHookCalled);
```

### 9.5。调度程序的钩子

和其他调度器一样，`NewThread`调度器也定义了两个钩子:

```java
RxJavaPlugins.setInitNewThreadSchedulerHandler((scheduler) -> {
    initHookCalled = true;
    return scheduler.call();
});

RxJavaPlugins.setNewThreadSchedulerHandler((scheduler) -> {
    hookCalled = true;
    return scheduler;
});

Observable.range(1, 15)
  .map(v -> v * 2)
  .subscribeOn(Schedulers.newThread())
  .test();

assertTrue(hookCalled && initHookCalled);
```

## 10。结论

在本教程中，我们已经了解了什么是各种 RxJava 生命周期挂钩，以及我们如何实现它们。在这些钩子中，错误处理钩子是最值得注意的一个。但是，我们可以使用其他方法进行审计，比如记录订户数量和其他特定用例。

和往常一样，我们在这里讨论的所有简短例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220627083908/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)