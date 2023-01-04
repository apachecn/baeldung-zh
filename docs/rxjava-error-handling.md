# RxJava 和错误处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-error-handling>

## 1。简介

在本文中，我们将了解如何使用 RxJava 处理异常和错误。

首先，记住`Observable`通常不会抛出异常。相反，默认情况下，`Observable`调用它的`Observer's onError()`方法，通知观察者刚刚发生了一个不可恢复的错误，然后退出，不再调用它的`Observer's`方法。

**我们将要引入的错误处理操作符通过恢复或重试`Observable`序列来改变默认行为。**

## 2。Maven 依赖关系

首先，让我们在`pom.xml`中添加 RxJava:

```
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.1.3</version>
</dependency>
```

最新版本的神器可以在[这里](https://web.archive.org/web/20220627084917/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.reactivex.rxjava2%22%20AND%20a%3A%22rxjava%22)找到。

## 3。错误处理

当错误发生时，我们通常需要以某种方式处理它。例如，改变相关的外部状态，用默认结果恢复序列，或者简单地让它保持原样，以便错误可以传播。

### 3.1。对错误的操作

使用 [`doOnError`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#doOnError-io.reactivex.functions.Consumer-) ，我们可以在出现错误时调用任何需要的操作:

```
@Test
public void whenChangeStateOnError_thenErrorThrown() {
    TestObserver testObserver = new TestObserver();
    AtomicBoolean state = new AtomicBoolean(false);
    Observable
      .error(UNKNOWN_ERROR)
      .doOnError(throwable -> state.set(true))
      .subscribe(testObserver);

    testObserver.assertError(UNKNOWN_ERROR);
    testObserver.assertNotComplete();
    testObserver.assertNoValues();

    assertTrue("state should be changed", state.get());
}
```

如果在执行动作时抛出异常，RxJava 会将异常封装在一个 [`CompositeException`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/exceptions/CompositeException.html) 中:

```
@Test
public void whenExceptionOccurOnError_thenCompositeExceptionThrown() {
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_ERROR)
      .doOnError(throwable -> {
          throw new RuntimeException("unexcepted");
      })
      .subscribe(testObserver);

    testObserver.assertError(CompositeException.class);
    testObserver.assertNotComplete();
    testObserver.assertNoValues();
}
```

### 3.2。恢复默认项目

虽然我们可以用`doOnError`调用动作，但是错误仍然打破了标准的顺序流程。有时我们想用默认选项恢复序列，这就是 [`onErrorReturnItem`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#onErrorReturnItem-T-) 所做的:

```
@Test
public void whenHandleOnErrorResumeItem_thenResumed(){
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_ERROR)
      .onErrorReturnItem("singleValue")
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertValueCount(1);
    testObserver.assertValue("singleValue");
}
```

如果动态默认物料供应商是首选，我们可以使用 [`onErrorReturn`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#onErrorReturn-io.reactivex.functions.Function-) :

```
@Test
public void whenHandleOnErrorReturn_thenResumed() {
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_ERROR)
      .onErrorReturn(Throwable::getMessage)
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertValueCount(1);
    testObserver.assertValue("unknown error");
}
```

### 3.3。用另一个序列继续

当遇到错误时，我们可以使用 [`onErrorResumeNext`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#onErrorResumeNext-io.reactivex.ObservableSource-) 提供回退数据序列，而不是回退到单个项目。这将有助于防止错误传播:

```
@Test
public void whenHandleOnErrorResume_thenResumed() {
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_ERROR)
      .onErrorResumeNext(Observable.just("one", "two"))
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertValueCount(2);
    testObserver.assertValues("one", "two");
}
```

如果回退序列根据具体的异常类型而不同，或者该序列需要由函数生成，我们可以将函数传递给`onErrorResumeNext:`

```
@Test
public void whenHandleOnErrorResumeFunc_thenResumed() {
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_ERROR)
      .onErrorResumeNext(throwable -> Observable
        .just(throwable.getMessage(), "nextValue"))
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertValueCount(2);
    testObserver.assertValues("unknown error", "nextValue");
}
```

### 3.4。仅处理异常

 **RxJava 还提供了一个回退方法，当出现异常(但没有错误)时，允许使用提供的`Observable`继续序列:

```
@Test
public void whenHandleOnException_thenResumed() {
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_EXCEPTION)
      .onExceptionResumeNext(Observable.just("exceptionResumed"))
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertValueCount(1);
    testObserver.assertValue("exceptionResumed");
}

@Test
public void whenHandleOnException_thenNotResumed() {
    TestObserver testObserver = new TestObserver();
    Observable
      .error(UNKNOWN_ERROR)
      .onExceptionResumeNext(Observable.just("exceptionResumed"))
      .subscribe(testObserver);

    testObserver.assertError(UNKNOWN_ERROR);
    testObserver.assertNotComplete();
}
```

如上面的代码所示，当一个错误发生时，`onExceptionResumeNext`不会开始恢复序列。

## 4。出错时重试

正常顺序可能会被暂时的系统故障或后端错误打破。在这些情况下，我们希望重试并等待，直到序列被修复。

幸运的是，RxJava 为我们提供了实现这一点的选项。

### 4.1。重试

通过使用[`retry`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#retry--)`,``Observable`将被无限次重新订阅，直到没有错误为止。但是大多数情况下，我们更喜欢固定的重试次数:

```
@Test
public void whenRetryOnError_thenRetryConfirmed() {
    TestObserver testObserver = new TestObserver();
    AtomicInteger atomicCounter = new AtomicInteger(0);
    Observable
      .error(() -> {
          atomicCounter.incrementAndGet();
          return UNKNOWN_ERROR;
      })
      .retry(1)
      .subscribe(testObserver);

    testObserver.assertError(UNKNOWN_ERROR);
    testObserver.assertNotComplete();
    testObserver.assertNoValues();
    assertTrue("should try twice", atomicCounter.get() == 2);
}
```

### 4.2。在条件下重试

条件重试在 RxJava 中也是可行的，使用带有谓词的[重试或使用](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#retry-io.reactivex.functions.BiPredicate-)`[retryUntil](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#retryUntil-io.reactivex.functions.BooleanSupplier-)`:

```
@Test
public void whenRetryConditionallyOnError_thenRetryConfirmed() {
    TestObserver testObserver = new TestObserver();
    AtomicInteger atomicCounter = new AtomicInteger(0);
    Observable
      .error(() -> {
          atomicCounter.incrementAndGet();
          return UNKNOWN_ERROR;
      })
      .retry((integer, throwable) -> integer < 4)
      .subscribe(testObserver);

    testObserver.assertError(UNKNOWN_ERROR);
    testObserver.assertNotComplete();
    testObserver.assertNoValues();
    assertTrue("should call 4 times", atomicCounter.get() == 4);
}

@Test
public void whenRetryUntilOnError_thenRetryConfirmed() {
    TestObserver testObserver = new TestObserver();
    AtomicInteger atomicCounter = new AtomicInteger(0);
    Observable
      .error(UNKNOWN_ERROR)
      .retryUntil(() -> atomicCounter.incrementAndGet() > 3)
      .subscribe(testObserver);
    testObserver.assertError(UNKNOWN_ERROR);
    testObserver.assertNotComplete();
    testObserver.assertNoValues();
    assertTrue("should call 4 times", atomicCounter.get() == 4);
}
```

### 4.3。`RetryWhen`

除了这些基本选项，还有一个有趣的重试方法:`retryWhen`。

这将返回一个发出与源[`ObservableSource`](https://web.archive.org/web/20220627084917/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/ObservableSource.html)`“OldO”,`相同值的`Observable,``“NewO”,`，但是如果返回的`Observable`“NewO”调用`onComplete`或`onError`，订户的`onComplete`或`onError`将被调用。

并且如果`“NewO”`发出任何项目，将触发对源 `ObservableSource` `“OldO”`的重新订阅。

下面的测试显示了这是如何工作的:

```
@Test
public void whenRetryWhenOnError_thenRetryConfirmed() {
    TestObserver testObserver = new TestObserver();
    Exception noretryException = new Exception("don't retry");
    Observable
      .error(UNKNOWN_ERROR)
      .retryWhen(throwableObservable -> Observable.error(noretryException))
      .subscribe(testObserver);

    testObserver.assertError(noretryException);
    testObserver.assertNotComplete();
    testObserver.assertNoValues();
}

@Test
public void whenRetryWhenOnError_thenCompleted() {
    TestObserver testObserver = new TestObserver();
    AtomicInteger atomicCounter = new AtomicInteger(0);
    Observable
      .error(() -> {
        atomicCounter.incrementAndGet();
        return UNKNOWN_ERROR;
      })
      .retryWhen(throwableObservable -> Observable.empty())
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertNoValues();
    assertTrue("should not retry", atomicCounter.get()==0);
}

@Test
public void whenRetryWhenOnError_thenResubscribed() {
    TestObserver testObserver = new TestObserver();
    AtomicInteger atomicCounter = new AtomicInteger(0);
    Observable
      .error(() -> {
        atomicCounter.incrementAndGet();
        return UNKNOWN_ERROR;
      })
      .retryWhen(throwableObservable -> Observable.just("anything"))
      .subscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertNoValues();
    assertTrue("should retry once", atomicCounter.get()==1);
}
```

`retryWhen`的典型用法是可变延迟的有限重试:

```
@Test
public void whenRetryWhenForMultipleTimesOnError_thenResumed() {
    TestObserver testObserver = new TestObserver();
    long before = System.currentTimeMillis();
    Observable
      .error(UNKNOWN_ERROR)
      .retryWhen(throwableObservable -> throwableObservable
        .zipWith(Observable.range(1, 3), (throwable, integer) -> integer)
        .flatMap(integer -> Observable.timer(integer, TimeUnit.SECONDS)))
      .blockingSubscribe(testObserver);

    testObserver.assertNoErrors();
    testObserver.assertComplete();
    testObserver.assertNoValues();
    long secondsElapsed = (System.currentTimeMillis() - before)/1000;
    assertTrue("6 seconds should elapse",secondsElapsed == 6 );
}
```

请注意这个逻辑是如何重试三次并逐渐延迟每次重试的。

## 5。总结

在本文中，我们介绍了 RxJava 中处理错误和异常的多种方法。

还有几个与错误处理相关的 RxJava 特有的异常——查看官方 wiki 了解更多细节。

和往常一样，完整的实现可以在 Github 上找到[。](https://web.archive.org/web/20220627084917/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)**