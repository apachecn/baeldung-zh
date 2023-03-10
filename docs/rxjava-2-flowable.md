# rx Java 2–可流动

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-2-flowable>

## 1。简介

RxJava 是一个反应式扩展 Java 实现，它允许我们编写事件驱动的异步应用程序。关于如何使用 RxJava 的更多信息可以在我们的[介绍文章中找到。](/web/20220628145529/https://www.baeldung.com/rx-java)

RxJava 2 从头重写，带来了多个新特性；其中一些是针对以前版本的框架中存在的问题而创建的。

这样的特征之一就是 **`io.reactivex.Flowable`** 。

## 2。`Observable` vs`. Flowable`

在 RxJava 的早期版本中，只有一个基类用于处理背压感知和非背压感知源—`Observable.`

RxJava 2 引入了这两种源之间的明显区别——背压感知源现在用一个专用的类来表示—`Flowable.`

**`Observable`源不支持背压。正因为如此，我们应该把它用在我们仅仅消费而不能影响的资源上。**

此外，如果我们处理大量元素，根据`Observable`的类型，可能会出现两种与[背压](/web/20220628145529/https://www.baeldung.com/rxjava-backpressure)相关的情况。

在使用所谓的**的情况下，冷`Observable**“,**` 事件被惰性地发射，所以我们不会溢出一个观察者。**

 **当使用一个**【热`Observable`**时，这将继续发出事件，即使消费者跟不上。****

 ****## 3。创造一个`Flowable`

创建一个`Flowable`有不同的方法。为了方便起见，这些方法看起来类似于 RxJava 第一版中的`Observable`中的方法。

### 3.1。简单的`Flowable`

我们可以使用`just()`方法创建一个`Flowable`，就像我们使用`Observable :` 一样

```java
Flowable<Integer> integerFlowable = Flowable.just(1, 2, 3, 4);
```

尽管使用`just()`非常简单，但从静态数据创建`Flowable`并不常见，它用于测试目的。

### 3.2。`Flowable` 从`Observable`到

**当我们有一个`Observable`时，我们可以使用`toFlowable()`方法**轻松地将其转换为`Flowable` :

```java
Observable<Integer> integerObservable = Observable.just(1, 2, 3);
Flowable<Integer> integerFlowable = integerObservable
  .toFlowable(BackpressureStrategy.BUFFER);
```

注意，为了能够执行转换，我们需要用一个`BackpressureStrategy.` 来丰富`Observable`，我们将在下一节描述可用的策略。

### 3.3。`Flowable`从`FlowableOnSubscribe`到

**RxJava 2 引入了一个函数接口`FlowableOnSubscribe`，它代表一个`Flowable` ，在消费者订阅它之后，它开始发出事件。**

因此，所有客户端都将接收到相同的事件集，这使得`FlowableOnSubscribe` 背压是安全的。

当我们有了`FlowableOnSubscribe`之后，我们可以用它来创建`Flowable`:

```java
FlowableOnSubscribe<Integer> flowableOnSubscribe
 = flowable -> flowable.onNext(1);
Flowable<Integer> integerFlowable = Flowable
  .create(flowableOnSubscribe, BackpressureStrategy.BUFFER);
```

文档描述了更多创建`Flowable.`的方法

## 4。`Flowable` `BackpressureStrategy`

像`toFlowable()`或`create()` 这样的方法采用一个`BackpressureStrategy`作为参数。

**`BackpressureStrategy`是一个枚举，它定义了我们将应用于** `**Flowable.**` 的背压行为

它可以缓存或丢弃事件，或者根本不实现任何行为，在最后一种情况下，我们将负责使用背压运算符来定义它。

`BackpressureStrategy`类似于 RxJava 早期版本中的`BackpressureMode`。

RxJava 2 中有五种不同的策略。

### 4.1。缓冲器

**如果我们使用`BackpressureStrategy.BUFFER` `,` ，源将缓冲所有事件，直到订户可以消费它们**:

```java
public void thenAllValuesAreBufferedAndReceived() {
    List testList = IntStream.range(0, 100000)
      .boxed()
      .collect(Collectors.toList());

    Observable observable = Observable.fromIterable(testList);
    TestSubscriber<Integer> testSubscriber = observable
      .toFlowable(BackpressureStrategy.BUFFER)
      .observeOn(Schedulers.computation()).test();

    testSubscriber.awaitTerminalEvent();

    List<Integer> receivedInts = testSubscriber.getEvents()
      .get(0)
      .stream()
      .mapToInt(object -> (int) object)
      .boxed()
      .collect(Collectors.toList());

    assertEquals(testList, receivedInts);
}
```

这类似于在`Flowable,` 上调用`onBackpressureBuffer()` 方法，但是它不允许显式定义缓冲区大小或 onOverflow 动作。

### 4.2。掉落

**我们可以使用`BackpressureStrategy.DROP`来丢弃不能消费的事件，而不是缓冲它们。**

这同样类似于在`Flowable`上使用`onBackpressureDrop` `()`:

```java
public void whenDropStrategyUsed_thenOnBackpressureDropped() {

    Observable observable = Observable.fromIterable(testList);
    TestSubscriber<Integer> testSubscriber = observable
      .toFlowable(BackpressureStrategy.DROP)
      .observeOn(Schedulers.computation())
      .test();
    testSubscriber.awaitTerminalEvent();
    List<Integer> receivedInts = testSubscriber.getEvents()
      .get(0)
      .stream()
      .mapToInt(object -> (int) object)
      .boxed()
      .collect(Collectors.toList());

    assertThat(receivedInts.size() < testList.size());
    assertThat(!receivedInts.contains(100000));
 }
```

### 4.3。最新

**使用`BackpressureStrategy.LATEST`将强制源仅保留最新的事件，因此如果消费者跟不上，将覆盖任何先前的值:**

```java
public void whenLatestStrategyUsed_thenTheLastElementReceived() {

    Observable observable = Observable.fromIterable(testList);
    TestSubscriber<Integer> testSubscriber = observable
      .toFlowable(BackpressureStrategy.LATEST)
      .observeOn(Schedulers.computation())
      .test();

    testSubscriber.awaitTerminalEvent();
    List<Integer> receivedInts = testSubscriber.getEvents()
      .get(0)
      .stream()
      .mapToInt(object -> (int) object)
      .boxed()
      .collect(Collectors.toList());

    assertThat(receivedInts.size() < testList.size());
    assertThat(receivedInts.contains(100000));
 }
```

当我们看代码时，看起来非常相似。

**但是，`BackpressureStrategy.LATEST`会覆盖我们的订阅者无法处理的元素，只保留最新的，因此得名。**

`BackpressureStrategy.DROP,` 另一方面，会丢弃无法处理的元素。这意味着最新的元素不一定会被释放出来。

### 4.4。错误

当我们使用`BackpressureStrategy.ERROR,`时，我们只是说**我们不希望背压出现**。因此，如果消费者跟不上来源，应该抛出一个`MissingBackpressureException`:

```java
public void whenErrorStrategyUsed_thenExceptionIsThrown() {
    Observable observable = Observable.range(1, 100000);
    TestSubscriber subscriber = observable
      .toFlowable(BackpressureStrategy.ERROR)
      .observeOn(Schedulers.computation())
      .test();

    subscriber.awaitTerminalEvent();
    subscriber.assertError(MissingBackpressureException.class);
}
```

### 4.5。缺少

**如果我们使用`BackpressureStrategy.MISSING`，源将推送元素而不丢弃或缓冲。**

在这种情况下，下游必须处理溢出:

```java
public void whenMissingStrategyUsed_thenException() {
    Observable observable = Observable.range(1, 100000);
    TestSubscriber subscriber = observable
      .toFlowable(BackpressureStrategy.MISSING)
      .observeOn(Schedulers.computation())
      .test();
    subscriber.awaitTerminalEvent();
    subscriber.assertError(MissingBackpressureException.class);
}
```

在我们的测试中，对于`ERROR` 和`MISSING` 策略，我们都排除了`MissingbackpressureException`。因为当源的内部缓冲区溢出时，它们都会抛出这样的异常。

然而，值得注意的是，他们两人的目的不同。

当我们根本不希望有反压力，并且希望源在出现反压力时抛出异常时，我们应该使用前一种方法。

如果我们不想在创建`Flowable`时指定默认行为，可以使用后一种方法。我们稍后会用反压算子来定义它。

## 5。总结

在本教程中，我们介绍了 RxJava `2`中引入的新类`Flowable.`

要找到更多关于`Flowable`本身及其 API 的信息，我们可以参考[文档](https://web.archive.org/web/20220628145529/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html)。

像往常一样，所有的代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628145529/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-libraries)******