# 如何将单声道`<list>`转换成通量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mono-list-to-flux>

## 1.概观

有时在[反应式编程](/web/20221128045436/https://www.baeldung.com/reactor-core)中，我们可能有一个大型项目集合的发布者。在某些情况下，此发布者的使用者可能无法一次性处理所有项目。因此，我们可能需要异步发布每个项目，以匹配消费者的处理速度。

在本教程中，我们将研究一些方法，通过这些方法我们可以将一个`Collection`的`Mono`转换为`Collection's`项目的`Flux`。

## 2.问题描述

当处理反应流时，我们使用一个`[Publisher](https://web.archive.org/web/20221128045436/https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Publisher.html?is-external=true)`及其两个实现`[Flux](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)`和`[Mono](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)`。虽然`Mono<T>`是`Publisher<T>`的一种，可以发射 0 或 1 个`T`类型的物品，但是`Flux<T>`可以发射 0 到`T`类型的`N`物品。

假设我们有一个持有`Mono<List<T>>`——`T`类型项目的可迭代集合的`Mono`发布者。我们的要求是使用`Flux<T>` : [![stream of list](img/dc79e7746d80eb9cff7f7622a6a60a98.png)](/web/20221128045436/https://www.baeldung.com/wp-content/uploads/2021/05/stream-of-list.png) 异步产生集合项目

在这里，我们可以看到我们需要一个操作符来执行这个转换。首先，我们将从流发布者`Mono`中提取集合项，然后像`Flux`一样一个接一个地异步产生项。

**`Mono`发布器包含一个可以同步转换`Mono`的`[map](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#map-java.util.function.Function-)`操作符和一个可以异步转换`Mono`的`[flatMap](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#flatMap-java.util.function.Function-)`操作符**。此外，这两个操作符都生成一个项目作为输出。

**但是，** **对于我们的用例来说，在展平`Mono<List<T>>`之后产生很多条目，我们可以使用`[flatMapMany](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#flatMapMany-java.util.function.Function-)`或者`[flatMapIterable](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#flatMapIterable-java.util.function.Function-)`** 。

让我们探索一下如何使用这些操作符。

## 3.`flatMapMany`

让我们从`String`的示例`List`开始，创建我们的`Mono`发布者:

```java
private Mono<List<String>> monoOfList() {
    List<String> list = new ArrayList<>();
    list.add("one");
    list.add("two");
    list.add("three");
    list.add("four");

    return Mono.just(list);
}
```

`flatMapMany` 是`Mono`上的一个通用操作符，它返回一个`Publisher.` ，让我们将`flatMapMany`应用到我们的解决方案中:

```java
private <T> Flux<T> monoTofluxUsingFlatMapMany(Mono<List<T>> monoList) {
    return monoList
      .flatMapMany(Flux::fromIterable)
      .log();
}
```

在这种情况下，`flatMapMany `获取`Mono`的`List`，展平它，并使用`Flux`操作符`[fromIterable](https://web.archive.org/web/20221128045436/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#fromIterable-java.lang.Iterable-).` 创建一个`Flux`发布器，我们在这里还使用了`log()` 来记录生成的每个元素。所以，**这个会像`one`、`two`、`three`、`four`一样一个一个的输出元素，然后终止**。

## 4.`flatMapIterable`

对于`String,` 的同一个样本`List`，我们现在将探索`flatMapIterable`——一个定制的操作器。

这里，我们不需要从`List`显式创建`Flux`；我们只需要提供`List`。这个操作符从它的元素中隐式地创建了一个`Flux`。让我们使用`flatMapIterable`作为我们的解决方案:

```java
private <T> Flux<T> monoTofluxUsingFlatMapIterable(Mono<List<T>> monoList) {
    return monoList
      .flatMapIterable(list -> list)
      .log();
}
```

在这里， `flatMapIterable`获取`Mono`的`List`，并在内部将其转换为其元素的`Flux`。因此，与`flatMapMany`操作符相比，它更加优化。**而这个会输出同样的“`one`”、“`two`”、“`three`”、“`four`”，然后终止**。

## 5.结论

在本文中，我们讨论了使用操作符`flatMapMany`和`flatMapIterable`将`Mono<List<T>>` 转换成 `Flux<T>`的不同方法。两者都是易于使用的操作符。鉴于`flatMapMany `对于更一般的出版商来说是有用的，`flatMapIterable `更适合这种目的。

与往常一样，代码示例是 GitHub 上可用的 [。](https://web.archive.org/web/20221128045436/https://github.com/eugenp/tutorials/tree/master/reactor-core)