# 组合 RxJava Completables

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-completable>

## 1。概述

在本教程中，我们将使用 [RxJava 的](/web/20220627171839/https://www.baeldung.com/rx-java) `Completable`类型，它表示一个没有实际值的计算结果。

## 2。RxJava 依赖关系

让我们将 RxJava 2 依赖项包含到我们的 Maven 项目中:

```java
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.2</version>
</dependency>
```

我们通常可以在 [Maven Central](https://web.archive.org/web/20220627171839/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.reactivex.rxjava2%22%20AND%20a%3A%22rxjava%22) 上找到最新版本。

## 3。可完成类型

**[`Completable`](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html)** **与 [`Observable`](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html) 相似，唯一的区别是前者发出完成或错误信号，但没有项目。**`Completable `类包含几个方便的方法，用于从不同的反应源创建或获取它。

**我们可以使用 [`Completable.complete()`](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html#complete--) 生成一个立即完成的实例。**

那么，我们可以通过使用 [`DisposableCompletableObserver`](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/observers/DisposableCompletableObserver.html) 来观察它的状态:

```java
Completable
  .complete()
  .subscribe(new DisposableCompletableObserver() {
    @Override
    public void onComplete() {
        System.out.println("Completed!");
    }

    @Override
    public void onError(Throwable e) {
        e.printStackTrace();
    }
});
```

**另外，我们可以从`Callable, [Action](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/functions/Action.html), and Runnable`** **:** 构造一个`Completable`实例

```java
Completable.fromRunnable(() -> {});
```

同样，我们可以使用`Completable.from()`或调用[上的`ignoreElement()`从其他类型中获取`Completable`实例，可能是](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Maybe.html#ignoreElement--)、[单个](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Single.html#ignoreElement--)、[可流动](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html#ignoreElements--)和[可观察](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#ignoreElements--)源本身:

```java
Flowable<String> flowable = Flowable
  .just("request received", "user logged in");
Completable flowableCompletable = Completable
  .fromPublisher(flowable);
Completable singleCompletable = Single.just(1)
  .ignoreElement();
```

## 4。链接`Completables`

当我们只关心操作的成功时，我们可以在许多真实世界的用例中使用`Completables`的链接:

*   全有或全无的操作，比如执行一个更新远程对象的 PUT 请求，然后在成功后更新本地数据库
*   事后记录和日志
*   多个操作的协调，例如，在摄取操作完成后运行分析作业

我们将保持例子简单和问题不可知。考虑我们有几个`Completable`实例:

```java
Completable first = Completable
  .fromSingle(Single.just(1));
Completable second = Completable
  .fromRunnable(() -> {});
Throwable throwable = new RuntimeException();
Completable error = Single.error(throwable)
  .ignoreElement();
```

**要将两个`Completables`合并成一个，我们可以使用`[andThen](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html#andThen-io.reactivex.CompletableSource-)()`操作符**:

```java
first
  .andThen(second)
  .test()
  .assertComplete();
```

我们可以根据需要把许多`Completables`链接起来。同时，**如果至少有一个源未能完成，导致`Completable`不会触发 *onComplete()* 以及**:

```java
first
  .andThen(second)
  .andThen(error)
  .test()
  .assertError(throwable);
```

此外，**如果其中一个源是无限的或者由于某种原因没有到达`onComplete`，那么产生的`Completable`将不会触发`onComplete() `也不会触发`onError() as well.`和**

好在我们仍然可以测试这个场景:

```java
...
  .andThen(Completable.never())
  .test()
  .assertNotComplete();
```

## 5。组成可完成的系列

假设我们有一堆`Completables.` 作为实际用例，假设我们需要在几个独立的子系统中注册一个用户。

**要把所有的`Completables`连接成一个，我们可以使用 [`merge`](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html#merge-java.lang.Iterable-) ()族的方法。**`merge()`操作者允许订阅所有资源。

一旦所有的源完成，结果实例**也就完成了。此外，当任何来源发出错误时，它以`onError`终止:**

```java
Completable.mergeArray(first, second)
  .test()
  .assertComplete();

Completable.mergeArray(first, second, error)
  .test()
  .assertError(throwable);
```

让我们转到一个稍微不同的用例。假设我们需要对从`Flowable.`中获得的每个元素执行一个动作

然后，我们需要一个单独的`Completable`来完成上游和所有元素级的动作。在这种情况下， [`flatMapCompletable`](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html#flatMapCompletable-io.reactivex.functions.Function-) ()操作员会提供帮助:

```java
Completable allElementsCompletable = Flowable
  .just("request received", "user logged in")
  .flatMapCompletable(message -> Completable
      .fromRunnable(() -> System.out.println(message))
  );
allElementsCompletable
  .test()
  .assertComplete();
```

类似地，上述方法可用于其余的基本反应类，如`Observable`、`Maybe`或`Single.`

作为`flatMapCompletable()`的实际背景，我们可以考虑用一些副作用来装饰每一件物品。我们可以为每个完成的元素写一个日志条目，或者在每个成功的操作后制作一个存储快照。

最后，**我们可能需要从几个其他来源构建一个`Completable`，并在其中任何一个完成时终止它。**这正是`amb`运营商可以提供帮助的地方。

前缀`amb`是“ambiguous”的简写，意味着不确定哪个`Completable`确切完成。例如，`[ambArray()](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html#ambArray-io.reactivex.CompletableSource...-):`

```java
Completable.ambArray(first, Completable.never(), second)
  .test()
  .assertComplete();
```

注意，上面的`Completable `也可以用`onError()`而不是`onComplete()`终止，这取决于哪个源 completable 首先终止:

```java
Completable.ambArray(error, first, second)
  .test()
  .assertError(throwable);
```

此外，一旦第一个源终止，剩余的源肯定会被处理掉。

这意味着所有剩余运行的`Completables`将通过 [Disposable.dispose()](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/disposables/Disposable.html#dispose--) 停止，相应的[completable observer](https://web.archive.org/web/20220627171839/http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/CompletableObserver.html)将被取消订阅。

关于一个实际的例子，当我们将一个备份文件流式传输到几个同等的远程存储器时，我们可以使用`amb()`。一旦最佳备份完成，我们就完成这一过程，或者在出现错误时重复这一过程。

## 6。结论

在本文中，我们简要回顾了 RxJava 的`Completable `类型。

我们从获得`Completable` 实例的不同选项开始，然后通过使用`andThen(), merge(), flatMapCompletable(), `和`amb…()`操作符链接和组合`Completables`。

我们可以在 GitHub 上找到所有代码样本[的源代码。](https://web.archive.org/web/20220627171839/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)