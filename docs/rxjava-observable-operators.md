# RxJava 中的可观察效用运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-observable-operators>

## 1。概述

在本文中，我们将发现一些在 RxJava 中使用`Observables` 的实用操作符，以及如何实现自定义操作符。

**操作符是一个函数，它接受并改变上游`Observable<T>`的行为，并返回下游`Observable<R> or Subscriber`** ，其中类型 T 和 R 可能相同，也可能不同。

运营商包装现有的`Observables`并通常通过拦截订阅来增强它们。这听起来可能很复杂，但实际上非常灵活，并不难掌握。

## 2。`Do`

有多种操作可以改变`Observable` 生命周期事件。

**d`oOnNext`操作符修改了`Observable` 源，以便在`onNext` 被调用`.`** 时调用一个动作

**`doOnCompleted`操作符注册一个动作，如果结果`Observable`正常终止**则调用这个动作，调用`Observer`的`onCompleted` 方法`:`

```java
Observable.range(1, 10)
  .doOnNext(r -> receivedTotal += r)
  .doOnCompleted(() -> result = "Completed")
  .subscribe();

assertTrue(receivedTotal == 55);
assertTrue(result.equals("Completed"));
```

`doOnEach` 操作符修改了`Observable`源，以便它为每个项目通知一个`Observer`,并建立一个回调，每次发出一个项目时都会调用这个回调。

`doOnSubscribe`操作符注册了一个动作，每当一个`Observer`订阅结果`Observable`时就会调用这个动作。

还有与`doOnSubscribe:`相反的`doOnUnsubscribe operator`

```java
Observable.range(1, 10)
  .doOnEach(new Observer<Integer>() {
      @Override
      public void onCompleted() {
          System.out.println("Complete");
      }
      @Override
      public void onError(Throwable e) {
          e.printStackTrace();
      }
      @Override
      public void onNext(Integer value) {
          receivedTotal += value;
      }
  })
  .doOnSubscribe(() -> result = "Subscribed")
  .subscribe();
assertTrue(receivedTotal == 55);
assertTrue(result.equals("Subscribed"));
```

当一个`Observable`错误完成时，我们可以使用`doOnError` 操作符来执行一个动作。

注册一个动作，当一个可观察对象成功或出错完成时，该动作将被调用:

```java
thrown.expect(OnErrorNotImplementedException.class);
Observable.empty()
  .single()
  .doOnError(throwable -> { throw new RuntimeException("error");})
  .doOnTerminate(() -> result += "doOnTerminate")
  .doAfterTerminate(() -> result += "_doAfterTerminate")
  .subscribe();
assertTrue(result.equals("doOnTerminate_doAfterTerminate"));
```

还有一个 **`FinallyDo`操作符——这个操作符被弃用，取而代之的是`doAfterTerminate.` ,它在`Observable`完成时注册一个动作。**

## 3。`ObserveOn`vs`SubscribeOn`

**默认情况下，`Observable`和操作符链将在调用其`Subscribe` 方法的同一个线程上操作。**

`ObserveOn` 操作符指定了一个不同的`Scheduler`，`Observable`将使用它向`Observers:`发送通知

```java
Observable.range(1, 5)
  .map(i -> i * 100)
  .doOnNext(i -> {
      emittedTotal += i;
      System.out.println("Emitting " + i
        + " on thread " + Thread.currentThread().getName());
  })
  .observeOn(Schedulers.computation())
  .map(i -> i * 10)
  .subscribe(i -> {
      receivedTotal += i;
      System.out.println("Received " + i + " on thread "
        + Thread.currentThread().getName());
  });

Thread.sleep(2000);
assertTrue(emittedTotal == 1500);
assertTrue(receivedTotal == 15000);
```

我们看到元素是在`main thread`中产生的，并且一直被推到第一个`map`调用。

但是在那之后，`observeOn`将处理重定向到一个`computation thread`，它在处理`map`和最后的`Subscriber.`时使用

**可能出现的一个问题是，底部气流产生排放的速度可能比顶部气流处理排放的速度快。**这可能会导致我们不得不考虑的[背压](/web/20221128045627/https://www.baeldung.com/rxjava-backpressure)的问题。

为了指定`Observable`应该在哪个`Scheduler`上操作，我们可以使用`subscribeOn`操作符:

```java
Observable.range(1, 5)
  .map(i -> i * 100)
  .doOnNext(i -> {
      emittedTotal += i;
      System.out.println("Emitting " + i
        + " on thread " + Thread.currentThread().getName());
  })
  .subscribeOn(Schedulers.computation())
  .map(i -> i * 10)
  .subscribe(i -> {
      receivedTotal += i;
      System.out.println("Received " + i + " on thread "
        + Thread.currentThread().getName());
  });

Thread.sleep(2000);
assertTrue(emittedTotal == 1500);
assertTrue(receivedTotal == 15000);
```

`SubscribeOn` 指示源`Observable` 使用哪个线程来发送项目——只有这个线程会将项目推送到`Subscriber`。它可以放在流中的任何位置，因为它只影响订阅。

实际上，我们只能使用一个`subscribeOn`，但是我们可以有任意数量的`observeOn`操作符。我们可以使用`observeOn.`轻松地将发射从一个线程切换到另一个线程

## 4。`Single`和`SingleOrDefault`

**操作符`Single` 返回一个`Observable`，它发出源`Observable:`** 发出的单项

```java
Observable.range(1, 1)
  .single()
  .subscribe(i -> receivedTotal += i);
assertTrue(receivedTotal == 1);
```

如果源`Observable`产生零个或多个元素，将会抛出一个异常:

```java
Observable.empty()
  .single()
  .onErrorReturn(e -> receivedTotal += 10)
  .subscribe();
assertTrue(receivedTotal == 10);
```

另一方面，操作符`SingleOrDefault`与`Single,` 非常相似，这意味着它也返回一个从源发出单个项目的`Observable`，但是另外，我们可以指定一个默认值:

```java
Observable.empty()
  .singleOrDefault("Default")
  .subscribe(i -> result +=i);
assertTrue(result.equals("Default"));
```

但是如果`Observable`源发出多个项目，它仍然抛出一个`IllegalArgumentExeption:`

```java
Observable.range(1, 3)
  .singleOrDefault(5)
  .onErrorReturn(e -> receivedTotal += 10)
  .subscribe();
assertTrue(receivedTotal == 10);
```

简单的结论是:

*   如果预计源`Observable` 可能没有元素或只有一个元素，则应该使用`SingleOrDefault`
*   如果我们正在处理在我们的`<samp class="ph codeph">Observable</samp>` 中发出的可能不止一个项目，并且我们只想发出第一个或最后一个值，我们可以使用其他操作符，比如`first`或`last`

## 5。`Timestamp`

**`Timestamp` 操作符将时间戳附加到源`Observable`** 发出的每一项上，然后再按顺序重新发出该项。时间戳表示项目发出的时间:

```java
Observable.range(1, 10)
  .timestamp()
  .map(o -> result = o.getClass().toString() )
  .last()
  .subscribe();

assertTrue(result.equals("class rx.schedulers.Timestamped"));
```

## 6。`Delay`

**该操作符通过在发出每个源`Observable’s`项目之前暂停特定的时间增量来修改其源`Observable`。**

它使用提供的值偏移整个序列:

```java
Observable source = Observable.interval(1, TimeUnit.SECONDS)
  .take(5)
  .timestamp();

Observable delayedObservable
  = source.delay(2, TimeUnit.SECONDS);

source.subscribe(
  value -> System.out.println("source :" + value),
  t -> System.out.println("source error"),
  () -> System.out.println("source completed"));

delayedObservable.subscribe(
  value -> System.out.println("delay : " + value),
  t -> System.out.println("delay error"),
  () -> System.out.println("delay completed"));
Thread.sleep(8000);
```

有一个替代的操作符，我们可以用它来延迟订阅源可观测值`delaySubscription`。

默认情况下， `Delay`操作符在`computation` `[Scheduler](https://web.archive.org/web/20221128045627/http://reactivex.io/documentation/scheduler.html)` 上运行，但是我们可以通过将它作为可选的第三个参数传递给`delaySubscription`来选择不同的`Scheduler`。

## 7。`Repeat`

**`Repeat`简单地截取来自上游的完成通知，而不是将其传递给下游，它重新订阅。**

因此，不能保证`repeat`将通过相同的事件序列保持循环，但当上游是固定流时，情况恰好如此:

```java
Observable.range(1, 3)
  .repeat(3)
  .subscribe(i -> receivedTotal += i);

assertTrue(receivedTotal == 18);
```

## 8。`Cache`

`cache` 操作员站在`subscribe`和我们的客户`Observable`之间。

**当第一个订阅者出现时，`cache`将订阅委托给底层`Observable` ，并向下游转发所有通知(事件、完成或错误)。**

但是，与此同时，它会在内部保留所有通知的副本。当后续订户想要接收推送的通知时，`cache`不再委托给底层`Observable`，而是提供缓存的值:

```java
Observable<Integer> source =
  Observable.<Integer>create(subscriber -> {
      System.out.println("Create");
      subscriber.onNext(receivedTotal += 5);
      subscriber.onCompleted();
  }).cache();
source.subscribe(i -> {
  System.out.println("element 1");
  receivedTotal += 1;
});
source.subscribe(i -> {
  System.out.println("element 2");
  receivedTotal += 2;
});

assertTrue(receivedTotal == 8);
```

## 9。`Using`

当一个`observer`订阅从`using()`返回的`Observable`时，它将使用`Observable`工厂函数来创建`observer`将…观察的`Observable`，同时使用资源工厂函数来创建我们设计的任何资源。

当`observer`取消订阅`Observable`时，或者当`Observable`终止时，`using`将调用第三个函数来处理创建的资源:

```java
Observable<Character> values = Observable.using(
  () -> "resource",
  r -> {
      return Observable.create(o -> {
          for (Character c : r.toCharArray()) {
              o.onNext(c);
          }
          o.onCompleted();
      });
  },
  r -> System.out.println("Disposed: " + r)
);
values.subscribe(
  v -> result += v,
  e -> result += e
);
assertTrue(result.equals("resource"));
```

## 10。结论

在本文中，我们讨论了如何使用 RxJava 实用操作符，以及如何探索它们最重要的特性。

RxJava 的真正威力在于它的操作符。数据流的声明式转换是安全的，同时又具有表达性和灵活性。

有了函数式编程的坚实基础，操作符在 RxJava 的采用中起着决定性的作用。掌握内置操作符是本库成功的关键。

该项目的完整源代码，包括这里使用的所有代码样本，可以在 GitHub 上找到[。](https://web.archive.org/web/20221128045627/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-operators)