# RxJava 也许

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-maybe>

## 1。简介

在本教程中，我们将关注 RxJava 中的**`Maybe<T>`类型——它代表一个可以发出单个值的流，在空状态下完成或者报告一个错误**。

## 2。`Maybe`式

`Maybe`是一种特殊的 [`Observable`](/web/20220524120509/https://www.baeldung.com/rx-java) ，它只能发出零个或一个项，如果在某一点计算失败就报错。

在这方面，就像是`Single` 和`Completable`的结合。所有这些简化的类型，包括`Maybe,`提供了`Flowable` 操作符的子集。这意味着**我们可以像`Flowable` 一样使用`Maybe` ，只要操作对 0 或 1 项有意义。**

因为它只能发出一个值，所以它不像`Flowable`那样支持背压处理:

```java
Maybe.just(1)
  .map(x -> x + 7)
  .filter(x -> x > 0)
  .test()
  .assertResult(8);
```

从`Maybe` 信号源可以订阅`onSuccess, onError` 和`onComplete` 信号:

```java
Maybe.just(1)
    .subscribe(
        x -> System.out.print("Emitted item: " + x),
        ex -> System.out.println("Error: " + ex.getMessage()),
        () -> System.out.println("Completed. No items.")
     );
```

上面的代码将打印 `Emitted item: 1` ，因为这个源将发出一个成功值。

对于相同的订阅:

*   `Maybe.empty().subscribe(…)` 将打印`“Completed. No items.”`
*   `Maybe.error(new Exception(“error”)).subscribe(…)` 将打印`Error: error”`

**这些事件对于`Maybe`来说是互斥的。**即`onComplete` 在`onSuccess`之后不会被调用。这与`Flowable` 略有不同，因为`onComplete` 将在流完成时被调用，即使在可能的一些`onNext` 调用之后。

`Single` 没有像`Maybe` 那样的`onComplete` 信号，因为它意味着捕捉一个反应模式，该模式可以发射一个项目或失败。

另一方面，`Completable`缺少`onSuccess` ,因为它只处理完成/失败的情况。

`Maybe`类型的另一个用例是与`Flowable.` 结合使用。`firstElement()`方法可用于从`Flowable`创建`Maybe` :

```java
Flowable<String> visitors = ...
visitors
  .skip(1000)
  .firstElement()
  .subscribe(
    v -> System.out.println("1000th visitor: " + v + " won the prize"), 
    ex -> System.out.print("Error: " + ex.getMessage()), 
    () -> System.out.print("We need more marketing"));
```

## 4。结论

在这个简短的教程中，我们快速查看了 RxJava `Maybe<T>`的用法，以及它与其他反应类型如`Flowable, Single` 和`Completable.`的关系

像往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220524120509/https://github.com/eugenp/tutorials/tree/master/rxjava-core)