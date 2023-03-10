# 在 RxJava 中组合观察值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-combine-observables>

## 1。简介

在这个快速教程中，我们将讨论在 RxJava 中组合`Observables`的不同方式。

如果你是 RxJava 的新手，一定要先看看这个[介绍教程](/web/20221205171539/http://www.baeldung.com/rxjava-tutorial)。

现在，让我们开始吧。

## 2。`Observables`

`Observable`序列，或简称为`Observables`，是异步数据流的表示。

这些基于[观察者模式](https://web.archive.org/web/20221205171539/https://en.wikipedia.org/wiki/Observer_pattern)，其中一个名为`Observer`的对象订阅由`Observable`发出的项目。

订阅是非阻塞的，因为`Observer`会对`Observable`将来发出的任何消息做出反应。这反过来又促进了并发性。

下面是 RxJava 中的一个简单演示:

```java
Observable
  .from(new String[] { "John", "Doe" })
  .subscribe(name -> System.out.println("Hello " + name))
```

## 3。组合可观测量

当使用反应式框架编程时，组合各种`Observables`是一个常见的用例。

例如，在一个 web 应用程序中，我们可能需要获得两组相互独立的异步数据流。

我们可以同时调用这两个流并订阅合并后的流，而不是等待前一个流完成后再请求下一个流。

在这一节中，我们将讨论在 RxJava 中组合多个`Observables`的一些不同方法，以及每种方法适用的不同用例。

### 3.1。`Merge`

我们可以使用`merge`操作符来组合多个`Observables`的输出，这样它们就像一个:

```java
@Test
public void givenTwoObservables_whenMerged_shouldEmitCombinedResults() {
    TestSubscriber<String> testSubscriber = new TestSubscriber<>();

    Observable.merge(
      Observable.from(new String[] {"Hello", "World"}),
      Observable.from(new String[] {"I love", "RxJava"})
    ).subscribe(testSubscriber);

    testSubscriber.assertValues("Hello", "World", "I love", "RxJava");
}
```

### 3.2。`MergeDelayError`

`mergeDelayError`方法与`merge`的相同之处在于它将多个`Observables`合并为一个，但是**如果在合并过程中出现错误，它允许无错误的项目在传播错误**之前继续:

```java
@Test
public void givenMutipleObservablesOneThrows_whenMerged_thenCombineBeforePropagatingError() {
    TestSubscriber<String> testSubscriber = new TestSubscriber<>();

    Observable.mergeDelayError(
      Observable.from(new String[] { "hello", "world" }),
      Observable.error(new RuntimeException("Some exception")),
      Observable.from(new String[] { "rxjava" })
    ).subscribe(testSubscriber);

    testSubscriber.assertValues("hello", "world", "rxjava");
    testSubscriber.assertError(RuntimeException.class);
}
```

上面的例子**发出所有无错值**:

```java
hello
world
rxjava
```

**注意，如果我们使用`merge`而不是`mergeDelayError`，将不会发出`String` " *rxjava"* ，因为当错误发生时`merge`会立即停止来自`Observables`的数据流。**

### 3.3。`Zip`

`zip`扩展方法**将两个值序列组合成对**:

```java
@Test
public void givenTwoObservables_whenZipped_thenReturnCombinedResults() {
    List<String> zippedStrings = new ArrayList<>();

    Observable.zip(
      Observable.from(new String[] { "Simple", "Moderate", "Complex" }), 
      Observable.from(new String[] { "Solutions", "Success", "Hierarchy"}),
      (str1, str2) -> str1 + " " + str2).subscribe(zippedStrings::add);

    assertThat(zippedStrings).isNotEmpty();
    assertThat(zippedStrings.size()).isEqualTo(3);
    assertThat(zippedStrings).contains("Simple Solutions", "Moderate Success", "Complex Hierarchy");
}
```

### 3.4。带间隔的 zip

在这个例子中，我们将使用`[interval](https://web.archive.org/web/20221205171539/http://reactivex.io/documentation/operators/interval.html)` 压缩一个流，这实际上将延迟第一个流的元素的发出:

```java
@Test
public void givenAStream_whenZippedWithInterval_shouldDelayStreamEmmission() {
    TestSubscriber<String> testSubscriber = new TestSubscriber<>();

    Observable<String> data = Observable.just("one", "two", "three", "four", "five");
    Observable<Long> interval = Observable.interval(1L, TimeUnit.SECONDS);

    Observable
      .zip(data, interval, (strData, tick) -> String.format("[%d]=%s", tick, strData))
      .toBlocking().subscribe(testSubscriber);

    testSubscriber.assertCompleted();
    testSubscriber.assertValueCount(5);
    testSubscriber.assertValues("[0]=one", "[1]=two", "[2]=three", "[3]=four", "[4]=five");
}
```

## 4。总结

在本文中，我们已经看到了一些将`Observables`与 RxJava 相结合的方法。你可以在[官方 RxJava 文档](https://web.archive.org/web/20221205171539/https://github.com/ReactiveX/RxJava/wiki/Combining-Observables)中了解到其他类似`[combineLatest](https://web.archive.org/web/20221205171539/http://reactivex.io/documentation/operators/combinelatest.html)`、`[join](https://web.archive.org/web/20221205171539/http://reactivex.io/documentation/operators/join.html)`、`[groupJoin](https://web.archive.org/web/20221205171539/http://reactivex.io/documentation/operators/join.html)`、`[switchOnNext](https://web.archive.org/web/20221205171539/http://reactivex.io/documentation/operators/switch.html)`的方法。

一如既往，本文的源代码可以在我们的 [GitHub repo](https://web.archive.org/web/20221205171539/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-observables) 中获得。