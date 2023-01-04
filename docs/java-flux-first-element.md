# 如何访问通量的第一个元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-flux-first-element>

## 1.概观

在本教程中，我们将探索用 [Spring 5 WebFlux](/web/20221208143856/https://www.baeldung.com/spring-webflux) 访问`Flux`的第一个元素的各种方法。

首先，我们将使用 API 的非阻塞方法，比如`next()`和`take()`。之后，我们将看到如何借助`elementAt()`方法实现同样的事情，这里我们需要指定索引。

最后，我们将学习 API 的阻塞方法，我们将使用`blockFirst()`来访问`flux`的第一个元素。

## 2.测试设置

对于本文中的代码示例，我们将使用`Payment`类，它只有一个字段，付款`amount`:

```java
public class Payment {
    private final int amount;
    // constructor and getter
}
```

在测试中，我们将使用名为`fluxOfThreePayments`的测试助手方法构建一个`flux`支付:

```java
private Flux<Payment> fluxOfThreePayments() {
    return Flux.just(paymentOf100, new Payment(200), new Payment(300));
}
```

之后，我们将使用 Spring Reactor 的`[StepVerifier](/web/20221208143856/https://www.baeldung.com/reactive-streams-step-verifier-test-publisher)`来测试结果。

## 3.`next()`

首先，我们来试试`next()`的方法。这个方法将返回通量的第一个元素，包装成反应式`[Mono](/web/20221208143856/https://www.baeldung.com/java-string-from-mono)`类型:

```java
@Test
void givenAPaymentFlux_whenUsingNext_thenGetTheFirstPaymentAsMono() {
    Mono<Payment> firstPayment = fluxOfThreePayments().next();

    StepVerifier.create(firstPayment)
      .expectNext(paymentOf100)
      .verifyComplete();
}
```

另一方面，如果我们在空的`flux`上调用`next()`，结果将是一个`empty Mono.` ，因此，阻塞它将返回`null`:

```java
@Test
void givenAEmptyFlux_whenUsingNext_thenGetAnEmptyMono() {
    Flux<Payment> emptyFlux = Flux.empty();

    Mono<Payment> firstPayment = emptyFlux.next();

    StepVerifier.create(firstPayment)
      .verifyComplete();
} 
```

## 4.`take()`

**无功通量的`take()`方法相当于 Java 8 流的`limit()`。**换句话说，我们可以使用`take(1)`将`flux`限制为一个元素，然后以阻塞或非阻塞的方式使用它:

```java
@Test
void givenAPaymentFlux_whenUsingTake_thenGetTheFirstPaymentAsFlux() {
    Flux<Payment> firstPaymentFlux = fluxOfThreePayments().take(1);

    StepVerifier.create(firstPaymentFlux)
      .expectNext(paymentOf100)
      .verifyComplete();
}
```

同样，对于一个空通量，`take(1)`将返回一个空通量:

```java
@Test
void givenAEmptyFlux_whenUsingNext_thenGetAnEmptyFlux() {
    Flux<Payment> emptyFlux = Flux.empty();

    Flux<Payment> firstPaymentFlux = emptyFlux.take(1);

    StepVerifier.create(firstPaymentFlux)
      .verifyComplete();
}
```

## 5.`elementAt()`

`Flux` API 也提供了`elementAt()`方法。我们可以用`elementAt(0)`以非阻塞的方式得到一个通量的第一个元素:

```java
@Test
void givenAPaymentFlux_whenUsingElementAt_thenGetTheFirstPaymentAsMono() {
    Mono<Payment> firstPayment = fluxOfThreePayments().elementAt(0);

    StepVerifier.create(firstPayment)
      .expectNext(paymentOf100)
      .verifyComplete();
}
```

但是，如果作为参数传递的索引大于 flux 发出的元素数，将会发出一个错误:

```java
@Test
void givenAEmptyFlux_whenUsingElementAt_thenGetAnEmptyMono() {
    Flux<Payment> emptyFlux = Flux.empty();

    Mono<Payment> firstPayment = emptyFlux.elementAt(0);

    StepVerifier.create(firstPayment)
      .expectError(IndexOutOfBoundsException.class);
}
```

## 6.`blockFirst()`

或者，我们也可以使用`blockFirst().`，尽管顾名思义，这是一种阻塞方法。因此，如果我们使用`blockFirst(),`，我们将离开被动世界，我们将失去它的所有好处:

```java
@Test
void givenAPaymentFlux_whenUsingBlockFirst_thenGetTheFirstPayment() {
    Payment firstPayment = fluxOfThreePayments().blockFirst();

    assertThat(firstPayment).isEqualTo(paymentOf100);
}
```

## 7.`toStream()`

最后，我们可以将流量转换成 Java 8 流，然后访问第一个元素:

```java
@Test
void givenAPaymentFlux_whenUsingToStream_thenGetTheFirstPaymentAsOptional() {
    Optional<Payment> firstPayment = fluxOfThreePayments().toStream()
      .findFirst();

    assertThat(firstPayment).contains(paymentOf100);
}
```

但是，如果我们这样做，我们将无法继续使用反应管道。

## 8.结论

在本文中，我们讨论了 Java 的[反应流](https://web.archive.org/web/20221208143856/http://www.reactive-streams.org/)的 API。我们已经看到了访问一个`Flux,`的第一个元素的各种方法，我们知道如果我们想使用反应式管道，我们应该坚持非阻塞解决方案。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/spring-5-webflux-2)