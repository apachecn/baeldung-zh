# RxJava 的 RxRelay 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rx-relay>

## 1。简介

RxJava 的流行导致了多个扩展其功能的第三方库的产生。

其中许多库是开发人员在使用 RxJava 时遇到的典型问题的答案。 **[RxRelay](https://web.archive.org/web/20220628234859/https://github.com/JakeWharton/RxRelay)** 就是这些方案之一。

## 2。`Subject`对付一个

简单地说，一个*主题*充当`Observable`和`Observer.` 之间的桥梁，因为它是一个`Observer`，它可以订阅一个或多个`Observables` 并从它们那里接收事件。

此外，假设它同时是一个`Observable`，它可以重新发送事件或者向它的订阅者发送新的事件。关于`Subject`的更多信息可以在这篇[文章](/web/20220628234859/https://www.baeldung.com/rx-java)中找到。

`Subject` 的一个问题是，在它接收到`onComplete()` 或`onError()`后，它不再能够移动数据。有时是想要的行为，但有时不是。

在不希望出现这种行为的情况下，我们应该考虑使用`RxRelay`。

## 3。`Relay`

**一个*继电器*基本上是一个`Subject`，但是没有调用`onComplete()` 和`onError(),` 的能力，因此它能够不断地发射数据。**

这允许我们在不同类型的 API 之间建立桥梁，而不用担心意外触发终端状态。

要使用`RxRelay` ,我们需要向我们的项目添加以下依赖项:

```java
<dependency>
  <groupId>com.jakewharton.rxrelay2</groupId>
  <artifactId>rxrelay</artifactId>
  <version>1.2.0</version>
</dependency>
```

## 4。`Relay`种类

图书馆里有三种不同类型的书。我们将在这里快速探究这三个问题。

### 4.1.`PublishRelay`

一旦`Observer`订阅了事件，这种类型的`Relay`将重新提交所有事件。

事件将发送给所有订阅者:

```java
public void whenObserverSubscribedToPublishRelay_itReceivesEmittedEvents() {
    PublishRelay<Integer> publishRelay = PublishRelay.create();
    TestObserver<Integer> firstObserver = TestObserver.create();
    TestObserver<Integer> secondObserver = TestObserver.create();

    publishRelay.subscribe(firstObserver);
    firstObserver.assertSubscribed();
    publishRelay.accept(5);
    publishRelay.accept(10);
    publishRelay.subscribe(secondObserver);
    secondObserver.assertSubscribed();
    publishRelay.accept(15);
    firstObserver.assertValues(5, 10, 15);

    // second receives only the last event
    secondObserver.assertValue(15);
}
```

在这种情况下没有事件缓冲，所以这种行为类似于感冒`Observable.`

### 4.2.`BehaviorRelay`

一旦 O `bserver`订阅，这种类型的`Relay`将重新提交最近观察到的事件和所有后续事件:

```java
public void whenObserverSubscribedToBehaviorRelay_itReceivesEmittedEvents() {
    BehaviorRelay<Integer> behaviorRelay = BehaviorRelay.create();
    TestObserver<Integer> firstObserver = TestObserver.create();
    TestObserver<Integer> secondObserver = TestObserver.create();
    behaviorRelay.accept(5);     
    behaviorRelay.subscribe(firstObserver);
    behaviorRelay.accept(10);
    behaviorRelay.subscribe(secondObserver);
    behaviorRelay.accept(15);
    firstObserver.assertValues(5, 10, 15);
    secondObserver.assertValues(10, 15);
}
```

当我们创建`BehaviorRelay`时，我们可以指定默认值，如果没有其他事件要发出，就会发出这个值。

要指定缺省值我们可以使用`createDefault()`方法:

```java
public void whenObserverSubscribedToBehaviorRelay_itReceivesDefaultValue() {
    BehaviorRelay<Integer> behaviorRelay = BehaviorRelay.createDefault(1);
    TestObserver<Integer> firstObserver = new TestObserver<>();
    behaviorRelay.subscribe(firstObserver);
    firstObserver.assertValue(1);
}
```

如果我们不想指定默认值，我们可以使用`create()` 方法:

```java
public void whenObserverSubscribedToBehaviorRelayWithoutDefaultValue_itIsEmpty() {
    BehaviorRelay<Integer> behaviorRelay = BehaviorRelay.create();
    TestObserver<Integer> firstObserver = new TestObserver<>();
    behaviorRelay.subscribe(firstObserver);
    firstObserver.assertEmpty();
}
```

### 4.3.`ReplayRelay`

这种类型的`Relay`缓冲它接收到的所有事件，然后将其重新发送给订阅它的所有订阅者:

```java
 public void whenObserverSubscribedToReplayRelay_itReceivesEmittedEvents() {
    ReplayRelay<Integer> replayRelay = ReplayRelay.create();
    TestObserver<Integer> firstObserver = TestObserver.create();
    TestObserver<Integer> secondObserver = TestObserver.create();
    replayRelay.subscribe(firstObserver);
    replayRelay.accept(5);
    replayRelay.accept(10);
    replayRelay.accept(15);
    replayRelay.subscribe(secondObserver);
    firstObserver.assertValues(5, 10, 15);
    secondObserver.assertValues(5, 10, 15);
}
```

所有的元素都被缓冲，所有的订阅者都会收到相同的事件，所以**这个行为类似于冷`Observable`。**

当我们创建`ReplayRelay`时，我们可以为事件提供最大的缓冲区大小和生存时间。

要创建缓冲区大小有限的`Relay`，我们可以使用`createWithSize()`方法。当要缓冲的事件多于设置的缓冲区时，之前的元素将被丢弃:

```java
public void whenObserverSubscribedToReplayRelayWithLimitedSize_itReceivesEmittedEvents() {
    ReplayRelay<Integer> replayRelay = ReplayRelay.createWithSize(2);
    TestObserver<Integer> firstObserver = TestObserver.create();
    replayRelay.accept(5);
    replayRelay.accept(10);
    replayRelay.accept(15);
    replayRelay.accept(20);
    replayRelay.subscribe(firstObserver);
    firstObserver.assertValues(15, 20);
}
```

我们还可以使用`createWithTime()`方法创建`ReplayRelay`，为缓冲事件留出最大时间:

```java
public void whenObserverSubscribedToReplayRelayWithMaxAge_thenItReceivesEmittedEvents() {
    SingleScheduler scheduler = new SingleScheduler();
    ReplayRelay<Integer> replayRelay =
      ReplayRelay.createWithTime(2000, TimeUnit.MILLISECONDS, scheduler);
    long current =  scheduler.now(TimeUnit.MILLISECONDS);
    TestObserver<Integer> firstObserver = TestObserver.create();
    replayRelay.accept(5);
    replayRelay.accept(10);
    replayRelay.accept(15);
    replayRelay.accept(20);
    Thread.sleep(3000);
    replayRelay.subscribe(firstObserver);
    firstObserver.assertEmpty();
}
```

## 5。`Relay`风俗

上面描述的所有类型都扩展了通用抽象类`Relay,`,这给了我们编写自己定制的`Relay` 类的能力。

为了创建一个自定义的`Relay`，我们需要实现三个方法:`accept(), hasObservers()` 和`subscribeActual().`

让我们编写一个简单的中继，将事件重新发送给随机选择的一个订阅者:

```java
public class RandomRelay extends Relay<Integer> {
    Random random = new Random();

    List<Observer<? super Integer>> observers = new ArrayList<>();

    @Override
    public void accept(Integer integer) {
        int observerIndex = random.nextInt() % observers.size();
        observers.get(observerIndex).onNext(integer);
    }

    @Override
    public boolean hasObservers() {
        return observers.isEmpty();
    }

    @Override
    protected void subscribeActual(Observer<? super Integer> observer) {
        observers.add(observer);
        observer.onSubscribe(Disposables.fromRunnable(
          () -> System.out.println("Disposed")));
    }
}
```

我们现在可以测试只有一个订户会收到该事件:

```java
public void whenTwoObserversSubscribedToRandomRelay_thenOnlyOneReceivesEvent() {
    RandomRelay randomRelay = new RandomRelay();
    TestObserver<Integer> firstObserver = TestObserver.create();
    TestObserver<Integer> secondObserver = TestObserver.create();
    randomRelay.subscribe(firstObserver);
    randomRelay.subscribe(secondObserver);
    randomRelay.accept(5);
    if(firstObserver.values().isEmpty()) {
        secondObserver.assertValue(5);
    } else {
        firstObserver.assertValue(5);
        secondObserver.assertEmpty();
    }
}
```

## 6。结论

在本教程中，我们看了一下 RxRelay，一种类似于`Subject`的类型，但是没有触发终端状态的能力。

更多信息可在[文档](https://web.archive.org/web/20220628234859/https://jakewharton.github.io/RxRelay/)中找到。和往常一样，所有的代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628234859/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-libraries)