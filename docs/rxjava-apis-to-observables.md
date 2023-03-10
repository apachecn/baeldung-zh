# 使用 RxJava2 将同步和异步 API 转换为可观察对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-apis-to-observables>

## 1。概述

在本教程中，我们将学习**如何使用 RxJava2** 操作符将传统的同步和异步 API 转换成`Observables`。

我们将创建几个简单的函数来帮助我们详细讨论这些操作符。

## 2.Maven 依赖性

首先，我们必须添加 [RxJava2](https://web.archive.org/web/20220524031317/https://search.maven.org/search?q=g:io.reactivex.rxjava2%20AND%20a:rxjava&core=gav) 和 [RxJava2Extensions](https://web.archive.org/web/20220524031317/https://search.maven.org/search?q=g:com.github.akarnokd%20AND%20a:rxjava2-extensions&core=gav) 作为 Maven 依赖项:

```java
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>com.github.akarnokd</groupId>
    <artifactId>rxjava2-extensions</artifactId>
    <version>0.20.4</version>
</dependency>
```

## 3。操作员

RxJava2 为反应式编程的各种用例定义了大量的操作符。

但是我们将只讨论几个操作符，这些操作符通常用于根据它们的性质将同步或异步方法转换成`Observables`。**这些操作符将函数作为参数，并发出从函数**返回的值。

除了普通的操作符，RxJava2 还为扩展功能定义了一些操作符。

让我们探索如何利用这些操作符来转换同步和异步方法。

## 4。同步方法转换

### 4.1。使用`fromCallable()`

该操作符返回一个`Observable`,当订阅者订阅它时，调用作为参数传递的函数，然后发出从该函数返回的值。让我们创建一个返回整数并对其进行转换的函数:

```java
AtomicInteger counter = new AtomicInteger();
Callable<Integer> callable = () -> counter.incrementAndGet();
```

现在，让我们将它转换成一个`Observable`并通过订阅它来测试它:

```java
Observable<Integer> source = Observable.fromCallable(callable);

for (int i = 1; i < 5; i++) {
    source.test()
      .awaitDone(5, TimeUnit.SECONDS)
      .assertResult(i);
    assertEquals(i, counter.get());
}
```

**每次当被包装的`Observable`被订阅时，`fromCallable()`操作符就懒洋洋地执行指定的函数。**为了测试这一行为，我们使用一个循环创建了多个订户。

由于反应流在默认情况下是异步的，订阅者将立即返回。在大多数实际场景中，可调用函数在完成执行时会有某种延迟。因此，**在测试我们的可调用函数的结果之前，我们增加了五秒钟的最大等待时间**。

还要注意，我们使用了*可观测的*的`[test()](https://web.archive.org/web/20220524031317/http://reactivex.io/RxJava/javadoc/io/reactivex/Observable.html#test--)`方法。**这种方法在测试`Observables` 时[得心应手。](/web/20220524031317/https://www.baeldung.com/rxjava-testing)**它创建了一个`TestObserver `并订阅了我们的`Observable.`

### 4.2。使用`start()`

`start()`操作员是`RxJava2Extension`模块的一部分。它将异步调用指定的函数，并返回一个发出结果的`Observable`:

```java
Observable<Integer> source = AsyncObservable.start(callable);

for (int i = 1; i < 5; i++) {
    source.test()
      .awaitDone(5, TimeUnit.SECONDS)
      .assertResult(1);
    assertEquals(1, counter.get());
}
```

这个函数会被立即调用，而不是在订阅者订阅结果`Observable`的时候。**对这个可观察的多个订阅观察到相同的返回值。**

## 5。异步方法转换

### 5.1。使用`fromFuture()`

正如我们所知，在 Java 中创建异步方法最常见的方式是使用`Future`实现。`fromFuture`方法将一个`Future`作为其参数，并发出从`Future.get()`方法获得的值。

首先，让我们将之前创建的函数变成异步的:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(callable);
```

接下来，让我们通过转换它来进行测试:

```java
Observable<Integer> source = Observable.fromFuture(future);

for (int i = 1; i < 5; i++) {
    source.test()
      .awaitDone(5, TimeUnit.SECONDS)
      .assertResult(1);
    assertEquals(1, counter.get());
}
executor.shutdown();
```

**请再次注意，每个订阅都观察到相同的返回值。**

现在，`Observable`的`dispose()` 方法在防止内存泄漏方面真的很有帮助。但在这种情况下，它不会因为`Future.get()`的阻挡性质而取消未来。

所以，我们可以通过**结合`source`可观测值的`doOnDispose()`** 函数和`future`上的`cancel`方法来确定取消未来:

```java
source.doOnDispose(() -> future.cancel(true));
```

### 5.2。使用`startFuture()`

顾名思义，这个操作符将立即启动指定的`Future`,并在订户订阅时发出返回值。与缓存结果供下次使用的`fromFuture`操作符不同，**该操作符将在每次获得订阅时执行异步方法**:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Observable<Integer> source = AsyncObservable.startFuture(() -> executor.submit(callable));

for (int i = 1; i < 5; i++) {
    source.test()
      .awaitDone(5, TimeUnit.SECONDS)
      .assertResult(i);
    assertEquals(i, counter.get());
}
executor.shutdown();
```

### 5.3。使用`deferFuture()`

**该操作符聚集从`Future`方法返回的多个`Observables`，并返回从每个`Observable.`** 获得的返回值流。每当有新订户订阅时，这将启动传递的异步工厂函数。

因此，让我们首先创建异步工厂函数:

```java
List<Integer> list = Arrays.asList(new Integer[] { counter.incrementAndGet(), 
  counter.incrementAndGet(), counter.incrementAndGet() });
ExecutorService exec = Executors.newSingleThreadExecutor();
Callable<Observable<Integer>> callable = () -> Observable.fromIterable(list);
```

然后我们可以做一个快速测试:

```java
Observable<Integer> source = AsyncObservable.deferFuture(() -> exec.submit(callable));
for (int i = 1; i < 4; i++) {
    source.test()
      .awaitDone(5, TimeUnit.SECONDS)
      .assertResult(1,2,3);
}
exec.shutdown();
```

## 6。结论

在本教程中，我们学习了如何将同步和异步方法转换成 RxJava2 可观察对象。

当然，我们在这里展示的例子是基本的实现。但是我们可以将 RxJava2 用于更复杂的应用，比如视频流和需要分批发送大量数据的应用。

像往常一样，我们在这里讨论的所有简短例子都可以在 Github 项目中找到。