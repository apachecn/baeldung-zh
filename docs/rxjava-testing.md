# 如何测试 RxJava？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-testing>

## 1。概述

在这篇文章中，我们将看看测试使用 [RxJava](https://web.archive.org/web/20220815040207/https://github.com/ReactiveX/RxJava) 编写的代码的方法。

我们用 RxJava 创建的典型流由一个`Observable` 和一个`Observer.` 组成。可观察对象是一个数据源，它是一个元素序列。一个或多个观察者订阅它来接收发出的事件。

通常，观察者和可观察对象以异步方式在单独的线程中执行——这使得代码难以用传统方式测试。

幸运的是， **RxJava 提供了一个`[TestSubscriber](https://web.archive.org/web/20220815040207/http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html)` 类，让我们能够测试异步的、事件驱动的流。**

## 2。测试 rx Java——传统方式

**让我们从一个例子**开始——我们有一个字母序列，我们想用一个从 1 开始的整数序列压缩它。

我们的测试应该断言，侦听 zipped observable 发出的事件的订阅者接收到的是用整数压缩的信件。

以传统方式编写这样的测试意味着我们需要保存一个结果列表，并从观察者那里更新该列表。向整数列表添加元素意味着我们的可观察对象和观察者需要在同一个线程中工作——它们不能异步工作。

因此我们将错过 RxJava 最大的优势之一——在单独的线程中处理事件。

下面是该测试的有限版本:

```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
List<String> results = new ArrayList<>();
Observable<String> observable = Observable
  .from(letters)
  .zipWith(
     Observable.range(1, Integer.MAX_VALUE), 
     (string, index) -> index + "-" + string);

observable.subscribe(results::add);

assertThat(results, notNullValue());
assertThat(results, hasSize(5));
assertThat(results, hasItems("1-A", "2-B", "3-C", "4-D", "5-E"));
```

我们通过向一个`results`列表添加元素来聚集来自观察者的结果。观察者和被观察对象在同一个线程中工作，所以我们的断言会正确地阻塞，并等待一个`subscribe()` 方法完成。

## 3。使用`TestSubscriber` 测试 RxJava

RxJava 附带了一个`TestSubsriber`类，允许我们编写使用异步事件处理的测试。这是一个普通的观察者，认同可观察的事物。

在测试中，我们可以检查一个`TestSubscriber`的状态，并对该状态做出断言:

```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
TestSubscriber<String> subscriber = new TestSubscriber<>();

Observable<String> observable = Observable
  .from(letters)
  .zipWith(
    Observable.range(1, Integer.MAX_VALUE), 
    ((string, index) -> index + "-" + string));

observable.subscribe(subscriber);

subscriber.assertCompleted();
subscriber.assertNoErrors();
subscriber.assertValueCount(5);
assertThat(
  subscriber.getOnNextEvents(),
  hasItems("1-A", "2-B", "3-C", "4-D", "5-E"));
```

我们将一个`TestSubscriber`实例传递给可观察对象上的一个`subscribe()` 方法。然后我们可以检查这个订阅者的状态。

**`TestSubscriber` 有一些非常有用的断言方法**，我们将用它们来验证我们的期望。订阅者应该收到一个观察者发出的 5 个元素，我们通过调用`assertValueCount()`方法断言。

我们可以通过调用`getOnNextEvents()`方法来检查订户收到的所有事件。

调用`assertCompleted()` 方法检查观察者订阅的流是否完成。`assertNoErrors()`方法断言在订阅流时没有错误。

## 4。测试预期异常

有时在我们的处理过程中，当一个可观察对象正在发射事件或者一个观察者正在处理事件时，就会发生错误。`TestSubscriber`有一个检查错误状态的特殊方法——`assertError()` 方法将异常的类型作为参数:

```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
TestSubscriber<String> subscriber = new TestSubscriber<>();

Observable<String> observable = Observable
  .from(letters)
  .zipWith(Observable.range(1, Integer.MAX_VALUE), ((string, index) -> index + "-" + string))
  .concatWith(Observable.error(new RuntimeException("error in Observable")));

observable.subscribe(subscriber);

subscriber.assertError(RuntimeException.class);
subscriber.assertNotCompleted();
```

我们正在使用`concatWith()`方法创建与另一个可观察对象相结合的可观察对象。第二个可观察对象在发射下一个事件时抛出一个`RuntimeException`。我们可以通过调用`assertError()` 方法来检查`TestSubsciber`上的异常类型。

收到错误的观察器停止处理，并以未完成状态结束。该状态可以通过`assertNotCompleted()`方法来检查。

## 5。测试基于时间的`Observable`

假设我们有一个每秒发出一个事件的`Observable`，我们想用一个`TestSubsciber`来测试这个行为。

我们可以使用`Observable.interval()` 方法定义一个基于时间的`Observable`，并传递一个`TimeUnit` 作为参数:

```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
TestScheduler scheduler = new TestScheduler();
TestSubscriber<String> subscriber = new TestSubscriber<>();
Observable<Long> tick = Observable.interval(1, TimeUnit.SECONDS, scheduler);

Observable<String> observable = Observable.from(letters)
  .zipWith(tick, (string, index) -> index + "-" + string);

observable.subscribeOn(scheduler)
  .subscribe(subscriber);
```

可观察到的`tick`每一秒钟都会发出一个新值。

在测试开始时，我们处于时间零点，因此我们的`TestSubscriber`将不会完成:

```java
subscriber.assertNoValues();
subscriber.assertNotCompleted();
```

为了在测试中模拟时间流逝，我们需要使用一个`[TestScheduler](https://web.archive.org/web/20220815040207/http://reactivex.io/RxJava/javadoc/rx/schedulers/TestScheduler.html)` 类。我们可以通过在一个`TestScheduler`上调用`advanceTimeBy()` 方法来模拟这一秒钟的过程:

```java
scheduler.advanceTimeBy(1, TimeUnit.SECONDS);
```

`advanceTimeBy()` 方法将使一个可观察产生一个事件。我们可以断言一个事件是通过调用一个`assertValueCount()` 方法产生的:

```java
subscriber.assertNoErrors();
subscriber.assertValueCount(1);
subscriber.assertValues("0-A");
```

我们的列表`letters` 中有 5 个元素，所以当我们想让一个可观察对象发出所有事件时，需要经过 6 秒钟的处理。为了模拟这 6 秒，我们使用了`advanceTimeTo()`方法:

```java
scheduler.advanceTimeTo(6, TimeUnit.SECONDS);

subscriber.assertCompleted();
subscriber.assertNoErrors();
subscriber.assertValueCount(5);
assertThat(subscriber.getOnNextEvents(), hasItems("0-A", "1-B", "2-C", "3-D", "4-E"));
```

在模拟过去的时间之后，我们可以在一个`TestSubscriber`上执行断言。我们可以断言所有事件都是通过调用`assertValueCount()`方法产生的。

## 6。结论

在本文中，我们研究了在 RxJava 中测试观察者和可观察对象的方法。我们研究了一种测试发射事件、错误和基于时间的可观察性的方法。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220815040207/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。