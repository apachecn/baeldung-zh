# 通量和单声道的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reactor-flux-vs-mono>

## 1.概观

在本教程中，我们将学习[反应堆堆芯](/web/20221006123232/https://www.baeldung.com/reactor-core)库的`[Flux](https://web.archive.org/web/20221006123232/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)`和`[Mono](https://web.archive.org/web/20221006123232/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)`之间的区别。

## 2.什么是`Mono`？

`Mono`是`[Publisher](https://web.archive.org/web/20221006123232/https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Publisher.html)`的一种特殊类型。**一个`Mono`对象代表一个单值或空值**。这意味着它最多只能为`onNext()`请求发出一个值，然后以`onComplete()`信号终止。如果失败，它只发出一个`onError()`信号。

让我们看一个带有完成信号的`Mono`的例子:

```java
@Test
public void givenMonoPublisher_whenSubscribeThenReturnSingleValue() {
    Mono<String> helloMono = Mono.just("Hello");
    StepVerifier.create(helloMono)
      .expectNext("Hello")
      .expectComplete()
      .verify();
}
```

我们在这里可以看到，当`helloMono`被订阅时，它只发出一个值，然后发出完成的信号。

## 3.什么是`Flux`？

`Flux`是代表 0 到 N 个异步序列值的标准`Publisher`。这意味着**它可以为`onNext()`请求发出 0 到多个值，可能是无限个值，然后以完成或错误信号终止。**

让我们看一个带有完成信号的`Flux`的例子:

```java
@Test
public void givenFluxPublisher_whenSubscribedThenReturnMultipleValues() {
    Flux<String> stringFlux = Flux.just("Hello", "Baeldung");
    StepVerifier.create(stringFlux)
      .expectNext("Hello")
      .expectNext("Baeldung")
      .expectComplete()
      .verify();
}
```

现在，让我们看一个带有错误信号的`Flux`的例子:

```java
@Test
public void givenFluxPublisher_whenSubscribeThenReturnMultipleValuesWithError() {
    Flux<String> stringFlux = Flux.just("Hello", "Baeldung", "Error")
      .map(str -> {
          if (str.equals("Error"))
              throw new RuntimeException("Throwing Error");
          return str;
      });
    StepVerifier.create(stringFlux)
      .expectNext("Hello")
      .expectNext("Baeldung")
      .expectError()
      .verify();
}
```

我们可以看到，从`Flux,`中获得两个值后，我们得到一个错误。

## 4.`Mono`对`Flux`

`Mono`和`Flux`都是`Publisher`接口的实现。简单来说，我们可以说，当我们正在做一些事情，比如计算或者向数据库或外部服务发出请求，并且期望最多一个结果时，那么我们应该使用`Mono`。

当我们期望计算、数据库或外部服务调用产生多个结果时，我们应该使用`Flux`。

`Mono`与 Java 中的 [`Optional`](/web/20221006123232/https://www.baeldung.com/java-optional) 类更相关，因为它包含 0 或 1 值，`Flux`与 [`List`](/web/20221006123232/https://www.baeldung.com/java-arraylist) 更相关，因为它可以有 N 个值。

## 5.结论

在本文中，我们已经了解了`Mono`和`Flux`的区别。

和往常一样，GitHub 上的[提供了完整的示例源代码。](https://web.archive.org/web/20221006123232/https://github.com/eugenp/tutorials/tree/master/reactor-core)