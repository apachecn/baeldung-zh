# 在 RxJava 中过滤可观察值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-filtering>

## 1。简介

在 RxJava 的[介绍之后，我们将看看过滤操作符。](/web/20220629002159/https://www.baeldung.com/rx-java)

特别是，我们将重点关注过滤、跳过、时间过滤和一些更高级的过滤操作。

## 2。过滤

当使用`Observable`时，有时只选择发射项目的子集是有用的。为此， **RxJava 提供了各种过滤功能**。

让我们开始看看`filter`方法。

### 2.1。`filter`操作员

**简单地说，`filter`操作符过滤一个`Observable`，确保发出的项目符合指定的条件**，它以`Predicate`的形式出现。

让我们看看如何从发出的值中只过滤出奇数值:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable
  .filter(i -> i % 2 != 0);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(1, 3, 5, 7, 9);
```

### 2.2。`take`操作员

**用`take`过滤时，逻辑导致前`n`项发射，忽略其余项。**

让我们看看如何过滤`source` `Observable`并仅发出前两项:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable.take(3);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(1, 2, 3);
```

### 2.3。`takeWhile`操作员

**使用`takeWhile,`时，过滤后的`Observable`会一直发射物品，直到遇到第一个不匹配`Predicate.`** 的元素

让我们看看如何使用过滤`Predicate:`的`takeWhile`

```java
Observable<Integer> sourceObservable = Observable.just(1, 2, 3, 4, 3, 2, 1);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable
  .takeWhile(i -> i < 4);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(1, 2, 3);
```

### 2.4。`takeFirst`操作员

**每当我们想只发出符合给定条件的第一个项目时，我们可以使用`takeFirst().`**

让我们快速看一下如何发出第一个大于 5 的项目:

```java
Observable<Integer> sourceObservable = Observable
  .just(1, 2, 3, 4, 5, 7, 6);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable
  .takeFirst(x -> x > 5);

filteredObservable.subscribe(subscriber);

subscriber.assertValue(7);
```

### 2.5。`first`和`firstOrDefault`运算符

使用`first` API 可以实现类似的行为:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable.first();

filteredObservable.subscribe(subscriber);

subscriber.assertValue(1);
```

**但是，如果我们想指定一个默认值，如果没有项目被发射，我们可以使用 *f* `irstOrDefault` :**

```java
Observable<Integer> sourceObservable = Observable.empty();

Observable<Integer> filteredObservable = sourceObservable.firstOrDefault(-1);

filteredObservable.subscribe(subscriber);

subscriber.assertValue(-1);
```

### 2.6。`takeLast`操作员

**接下来，如果我们想只发射由`Observable`发射的最后`n`个项目，我们可以使用`takeLast`。**

让我们看看如何只发射最后三个项目:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable.takeLast(3);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(8, 9, 10);
```

我们必须记住，这延迟了任何项目从源头`Observable`的发射，直到它完成。

### 2.7。`last`和`lastOrDefault`

**如果我们只想发射最后一个元素，除了使用`takeLast(1)`，我们可以使用`last`。**

这将过滤`Observable`，仅发出最后一个元素，该元素可选地验证过滤`Predicate`:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable
  .last(i -> i % 2 != 0);

filteredObservable.subscribe(subscriber);

subscriber.assertValue(9);
```

如果`Observable`为空，我们可以使用`lastOrDefault`，它过滤发出默认值的`Observable`。

如果使用了`lastOrDefault`操作符，并且没有任何验证过滤条件的项目，也会发出默认值:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = 
  sourceObservable.lastOrDefault(-1, i -> i > 10);

filteredObservable.subscribe(subscriber);

subscriber.assertValue(-1);
```

### 2.8。`elementAt`和`elementAtOrDefault`运算符

使用`elementAt`操作符，我们可以选择由源`Observable`发出的单个项目，并指定其索引:

```java
Observable<Integer> sourceObservable = Observable
  .just(1, 2, 3, 5, 7, 11);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable.elementAt(4);

filteredObservable.subscribe(subscriber);

subscriber.assertValue(7);
```

但是，如果指定的索引超过了发出的项目数，`elementAt`将抛出一个`IndexOutOfBoundException`。

为了避免这种情况，可以使用`elementAtOrDefault –`，如果索引超出范围，它将返回默认值:

```java
Observable<Integer> sourceObservable = Observable
  .just(1, 2, 3, 5, 7, 11);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable
 = sourceObservable.elementAtOrDefault(7, -1);

filteredObservable.subscribe(subscriber);

subscriber.assertValue(-1);
```

### 2.9。`ofType`操作员

**每当`Observable`发出`Object`物品时，可以根据它们的类型进行过滤。**

让我们看看如何只过滤发出的`String`类型项目:

```java
Observable sourceObservable = Observable.just(1, "two", 3, "five", 7, 11);
TestSubscriber subscriber = new TestSubscriber();

Observable filteredObservable = sourceObservable.ofType(String.class);

filteredObservable.subscribe(subscriber);

subscriber.assertValues("two", "five");
```

## 3。跳过

另一方面，当我们想要过滤掉或跳过由`Observable`、**发出的一些项目时，RxJava 提供了一些操作符作为过滤操作符**的对应操作符，我们之前已经讨论过了。

让我们开始看一下`skip`操作符，它是`take`的对应部分。

### 3.1。`skip`操作员

当一个`Observable`发出一系列项目时，可以使用`skip`过滤掉或跳过一些第一次发出的项目。

比如说。让我们看看如何跳过前四个元素:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable.skip(4);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(5, 6, 7, 8, 9, 10);
```

### 3.2。`skipWhile`操作员

**每当我们想要过滤掉由`Observable`发出的所有不符合过滤谓词的第一个值时，我们可以使用`skipWhile`操作符:**

```java
Observable<Integer> sourceObservable = Observable
  .just(1, 2, 3, 4, 5, 4, 3, 2, 1);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable
  .skipWhile(i -> i < 4);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(4, 5, 4, 3, 2, 1);
```

### 3.3。 `skipLast`符

**`skipLast`操作符允许我们跳过`Observable`发出的最后一个项目，只接受在它们之前发出的项目。**

例如，这样我们可以跳过最后五项:

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = sourceObservable.skipLast(5);

filteredObservable.subscribe(subscriber);

subscriber.assertValues(1, 2, 3, 4, 5);
```

### 3.4。`distinct`和`distinctUntilChanged`运算符

**`distinct`操作符返回一个`Observable`，它发出由`sourceObservable`发出的所有不同的项目:**

```java
Observable<Integer> sourceObservable = Observable
  .just(1, 1, 2, 2, 1, 3, 3, 1);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> distinctObservable = sourceObservable.distinct();

distinctObservable.subscribe(subscriber);

subscriber.assertValues(1, 2, 3);
```

然而，如果我们想获得一个发出所有由`sourceObservable`发出的项目的`Observable`，这些项目不同于它们的直接前任，我们可以使用`distinctUntilChanged`操作符:

```java
Observable<Integer> sourceObservable = Observable
  .just(1, 1, 2, 2, 1, 3, 3, 1);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> distinctObservable = sourceObservable.distinctUntilChanged();

distinctObservable.subscribe(subscriber);

subscriber.assertValues(1, 2, 1, 3, 1);
```

### 3.5。`ignoreElements`操作员

每当我们想忽略由`sourceObservable`发出的所有元素时，我们可以简单地使用`ignoreElements:`

```java
Observable<Integer> sourceObservable = Observable.range(1, 10);
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> ignoredObservable = sourceObservable.ignoreElements();

ignoredObservable.subscribe(subscriber);

subscriber.assertNoValues();
```

## 4。时间过滤运算符

当处理可观测序列时，时间轴是未知的，但有时从序列中获取及时的数据可能是有用的。

出于这个目的， **RxJava 提供了一些方法，允许我们使用时间轴**来处理`Observable`。

在进入第一个之前，让我们定义一个定时的`Observable`,它将每秒发出一个项目:

```java
TestScheduler testScheduler = new TestScheduler();

Observable<Integer> timedObservable = Observable
  .just(1, 2, 3, 4, 5, 6)
  .zipWith(Observable.interval(
    0, 1, TimeUnit.SECONDS, testScheduler), (item, time) -> item);
```

**`TestScheduler`是一个特殊的调度程序，允许以我们喜欢的任何速度手动提前时钟**。

### 4.1。`sample`和 `throttleLast`运算符

`sample`操作符过滤`timedObservable`，返回一个`Observable`,它在周期时间间隔内发出这个 API 发出的最近的项目。

让我们看看如何对`timedObservable`进行采样，每 2.5 秒过滤一次最后发出的项目:

```java
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> sampledObservable = timedObservable
  .sample(2500L, TimeUnit.MILLISECONDS, testScheduler);

sampledObservable.subscribe(subscriber);

testScheduler.advanceTimeBy(7, TimeUnit.SECONDS);

subscriber.assertValues(3, 5, 6);
```

这种行为也可以通过使用`throttleLast`操作符来实现。

### 4.2。`throttleFirst` 操作员

`throttleFirst`操作符与`throttleLast/sample`不同，因为它在每个采样周期内发出第一个由`timedObservable`发出的项目，而不是最近发出的项目。

让我们看看如何使用 4 秒的采样周期发射第一个项目:

```java
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = timedObservable
  .throttleFirst(4100L, TimeUnit.SECONDS, testScheduler);

filteredObservable.subscribe(subscriber);

testScheduler.advanceTimeBy(7, TimeUnit.SECONDS);

subscriber.assertValues(1, 6);
```

### 4.3。`debounce`和`throttleWithTimeout`运算符

使用`debounce`操作符，如果一个特定的时间间隔已经过去而没有发出另一个项目，那么可以只发出一个项目。

因此，如果我们选择的时间跨度大于`timedObservable`发射的项目之间的时间间隔，它将只发射最后一个`.` ，反之，如果它更小，它将发射所有由`timedObservable.`发射的项目

让我们看看第一个场景会发生什么:

```java
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = timedObservable
  .debounce(2000L, TimeUnit.MILLISECONDS, testScheduler);

filteredObservable.subscribe(subscriber);

testScheduler.advanceTimeBy(7, TimeUnit.SECONDS);

subscriber.assertValue(6);
```

这种行为也可以使用`throttleWithTimeout`来实现。

### 4.4。`timeout`操作员

`timeout`操作员镜像源`Observable`，但是如果源`Observable`在指定的时间间隔内没有发射任何物品，则发出通知错误，中止物品的发射。

让我们看看如果我们给我们的`timedObservable`指定 500 毫秒的超时会发生什么:

```java
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = timedObservable
  .timeout(500L, TimeUnit.MILLISECONDS, testScheduler);

filteredObservable.subscribe(subscriber);

testScheduler.advanceTimeBy(7, TimeUnit.SECONDS);

subscriber.assertError(TimeoutException.class); subscriber.assertValues(1);
```

## 5。多重可观测滤波

使用`Observable`时，完全可以根据第二个`Observable`来决定是过滤还是跳过项目。

在继续之前，让我们定义一个`delayedObservable`，它将在 3 秒后仅发射 1 个项目:

```java
Observable<Integer> delayedObservable = Observable.just(1)
  .delay(3, TimeUnit.SECONDS, testScheduler);
```

先说`takeUntil` 运算符。

### 5.1。`takeUntil`操作员

在第二个`Observable` ( `delayedObservable`)发出一个项目或终止后，`takeUntil`操作员丢弃由源`Observable` ( `timedObservable`)发出的任何项目；

```java
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = timedObservable
  .skipUntil(delayedObservable);

filteredObservable.subscribe(subscriber);

testScheduler.advanceTimeBy(7, TimeUnit.SECONDS);

subscriber.assertValues(4, 5, 6);
```

### 5.2。`skipUntil`操作员

另一方面，`skipUntil`丢弃由源`Observable` ( `timedObservable`)发出的任何物品，直到第二个`Observable` ( `delayedObservable`)发出一个物品:

```java
TestSubscriber<Integer> subscriber = new TestSubscriber();

Observable<Integer> filteredObservable = timedObservable
  .takeUntil(delayedObservable);

filteredObservable.subscribe(subscriber);

testScheduler.advanceTimeBy(7, TimeUnit.SECONDS);

subscriber.assertValues(1, 2, 3);
```

## 6。结论

在这篇内容丰富的教程中，我们探索了 RxJava 中可用的不同过滤操作符，并为每个操作符提供了一个简单的例子。

和往常一样，本文中的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220629002159/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-observables)