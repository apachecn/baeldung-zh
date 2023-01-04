# Mono.defer()是做什么的？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mono-defer>

## 1.概观

在[反应式编程](/web/20220526052247/https://www.baeldung.com/reactor-core)中，我们有很多方法可以创建 [`Mono`](https://web.archive.org/web/20220526052247/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) 或 [`Flux`](https://web.archive.org/web/20220526052247/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) 类型的发布者。这里，我们将看看如何使用`defer`方法来延迟`Mono`发布者的执行。

## 2.什么是 Mono.defer 方法？

我们可以使用`Mono`的 [`defer`](https://web.archive.org/web/20220526052247/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#defer-java.util.function.Supplier-) 方法创建一个最多产生一个值的冷发布器。让我们看看方法签名:

```
public static <T> Mono<T> defer(Supplier<? extends Mono<? extends T>> supplier)
```

这里，`defer`接收`Mono`发行商的 [`Supplier`](https://web.archive.org/web/20220526052247/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Supplier.html) ，并在下游订阅时延迟返回该`Mono`。

然而，问题是，什么是冷酷的发布者，什么是懒惰的发布者？让我们调查一下。

**只有当消费者订阅冷发布者时，执行线程才评估冷发布者。** **当热门出版商在任何订阅前都热切地评估时。**我们有方法`Mono.just()`给出类型`Mono`的热发布者。

## 3.它是如何工作的？

让我们来探索一个具有类型`Mono`的 `Supplier`的样例用例:

```
private Mono<String> sampleMsg(String str) {
    log.debug("Call to Retrieve Sample Message!! --> {} at: {}", str, System.currentTimeMillis());
    return Mono.just(str);
}
```

这里，这个方法返回一个热门的`Mono`发布者。让我们急切地订阅这个:

```
public void whenUsingMonoJust_thenEagerEvaluation() throws InterruptedException {

    Mono<String> msg = sampleMsg("Eager Publisher");

    log.debug("Intermediate Test Message....");

    StepVerifier.create(msg)
      .expectNext("Eager Publisher")
      .verifyComplete();

    Thread.sleep(5000);

    StepVerifier.create(msg)
      .expectNext("Eager Publisher")
      .verifyComplete();
}
```

在执行时，我们可以在日志中看到以下内容:

```
20:44:30.250 [main] DEBUG com.baeldung.mono.MonoUnitTest - Call to Retrieve Sample Message!! --> Eager Publisher at: 1622819670247
20:44:30.365 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
20:44:30.365 [main] DEBUG com.baeldung.mono.MonoUnitTest - Intermediate Test Message....
```

在这里，我们可以注意到:

*   根据指令序列，`main`线程急切地执行方法`sampleMsg`。
*   在使用`StepVerifier`的两个订阅中，`main`线程使用`sampleMsg`的相同输出。因此，没有新的评价。

让我们看看`Mono.defer()`如何将其转化为一个冷酷(懒惰)的发行商:

```
public void whenUsingMonoDefer_thenLazyEvaluation() throws InterruptedException {

    Mono<String> deferMsg = Mono.defer(() -> sampleMsg("Lazy Publisher"));

    log.debug("Intermediate Test Message....");

    StepVerifier.create(deferMsg)
      .expectNext("Lazy Publisher")
      .verifyComplete();

    Thread.sleep(5000);

    StepVerifier.create(deferMsg)
      .expectNext("Lazy Publisher")
      .verifyComplete();

}
```

在执行此方法时，我们可以在控制台中看到以下日志:

```
20:01:05.149 [main] DEBUG com.baeldung.mono.MonoUnitTest - Intermediate Test Message....
20:01:05.187 [main] DEBUG com.baeldung.mono.MonoUnitTest - Call to Retrieve Sample Message!! --> Lazy Publisher at: 1622817065187
20:01:10.197 [main] DEBUG com.baeldung.mono.MonoUnitTest - Call to Retrieve Sample Message!! --> Lazy Publisher at: 1622817070197
```

这里，我们可以注意到日志序列中的几个点:

*   `StepVerifier`在每个订阅上执行方法`sampleMsg`，而不是在我们定义它的时候。
*   延迟 5 秒钟后，订阅方法`sampleMsg`的第二个消费者再次执行它。

`defer`方法就是这样把热的变成冷的发布者。

## 4.`Mono.defer`的用例？

让我们看看可以使用`Mono.defer()`方法的可能用例:

*   当我们必须有条件地订阅某个发布者时
*   当每个预订的执行可能产生不同的结果时
*   **[`deferContextual`](https://web.archive.org/web/20220526052247/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#deferContextual-java.util.function.Function-) 可用于当前发布者**的上下文评估

### 4.1.示例用法

让我们看一个使用条件`Mono.defer()`方法的例子:

```
public void whenEmptyList_thenMonoDeferExecuted() {

    Mono<List<String>> emptyList = Mono.defer(() -> monoOfEmptyList());

    //Empty list, hence Mono publisher in switchIfEmpty executed after condition evaluation
    Flux<String> emptyListElements = emptyList.flatMapIterable(l -> l)
      .switchIfEmpty(Mono.defer(() -> sampleMsg("EmptyList")))
      .log();

    StepVerifier.create(emptyListElements)
      .expectNext("EmptyList")
      .verifyComplete();
}
```

这里将`Mono`发布者`sampleMsg`的`supplier`放在`switchIfEmpty`方法中进行条件执行。因此，`sampleMsg`只有当它被懒洋洋地认购时才被执行。

现在，让我们看看非空列表的相同代码:

```
public void whenNonEmptyList_thenMonoDeferNotExecuted() {

    Mono<List<String>> nonEmptyist = Mono.defer(() -> monoOfList());

    //Non empty list, hence Mono publisher in switchIfEmpty won't evaluated.
    Flux<String> listElements = nonEmptyist.flatMapIterable(l -> l)
      .switchIfEmpty(Mono.defer(() -> sampleMsg("NonEmptyList")))
      .log();

    StepVerifier.create(listElements)
      .expectNext("one", "two", "three", "four")
      .verifyComplete();
}
```

这里，`sampleMsg `没有被执行，因为它没有被订阅。

## 5.结论

在本文中，我们讨论了`Mono.defer()`方法和热/冷发布者。另外，我们如何把一个热的发布者转化成一个冷的发布者。最后，我们还讨论了它与样例用例的工作。

与往常一样，GitHub 上的[提供了代码示例。](https://web.archive.org/web/20220526052247/https://github.com/eugenp/tutorials/tree/master/reactor-core)