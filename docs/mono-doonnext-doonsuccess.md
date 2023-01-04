# Mono 的 doOnNext()和 doOnSuccess()的比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mono-doonnext-doonsuccess>

## 1.概观

在这个简短的教程中，我们将探索来自 [Spring 5 WebFlux](/web/20221208143856/https://www.baeldung.com/spring-webflux) 的 Mono 对象的各种监听器。我们将比较`doOnNext()`和`doOnSuccess()`方法，发现尽管它们很相似，但对于空的`Mono`来说，它们的行为是不同的

## 2.`doOnNext`

**`Mono`的`doOnNext()`允许我们附加一个监听器，当数据发出时会被触发。对于本文中的代码示例，我们将使用`PaymentService`类。在这种情况下，我们将只在`paymentMono`发出数据时调用`processPayment`方法，使用`doOnNext()`:**

```java
@Test
void givenAPaymentMono_whenCallingServiceOnNext_thenCallServiceWithPayment() {
    Payment paymentOf100 = new Payment(100);
    Mono<Payment> paymentMono = Mono.just(paymentOf100);

    paymentMono.doOnNext(paymentService::processPayment)
        .block();

    verify(paymentService).processPayment(paymentOf100);
} 
```

**但是，空的`Mono`不会发出任何数据，`doOnNext`也不会被触发。**因此，如果我们使用`Mono.empty()`重复测试，则`processPayment`方法将不再被调用:

```java
@Test
void givenAnEmptyMono_whenCallingServiceOnNext_thenDoNotCallService() {
    Mono<Payment> emptyMono = Mono.empty();

    emptyMono.doOnNext(paymentService::processPayment)
        .block();

    verify(paymentService, never()).processPayment(any());
}
```

## 3.`doOnSuccess`

**我们可以使用`doOnSuccess`来附加一个监听器，当`Mono`成功完成时会触发这个监听器。**让我们重复测试，但这次使用`doOnSuccess`:

```java
@Test
void givenAPaymentMono_whenCallingServiceOnSuccess_thenCallServiceWithPayment() {
    Payment paymentOf100 = new Payment(100);
    Mono<Payment> paymentMono = Mono.just(paymentOf100);

    paymentMono.doOnSuccess(paymentService::processPayment)
        .block();

    verify(paymentService).processPayment(paymentOf100);
}
```

**但是，我们应该注意，即使没有数据发出，也认为`Mono`成功完成。**因此，对于一个空的`Mono`，上面的代码将调用带有`null` `Payment`的`processPayment`方法:

```java
Test
void givenAnEmptyMono_whenCallingServiceOnSuccess_thenCallServiceWithNull() {
    Mono<Payment> emptyMono = Mono.empty();

    emptyMono.doOnSuccess(paymentService::processPayment)
        .block();

    verify(paymentService).processPayment(null);
}
```

## 4.结论

在这篇短文中，我们了解了`Mono`的`doOnNext`和`doOnSuccess`的听众之间的区别。我们看到，如果我们想要对收到的数据做出反应，可以使用`doOnNext`。另一方面，如果我们希望方法调用在`Mono`成功完成时发生，不管它是否发出数据，我们都应该使用`doOnSuccess`。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/spring-5-webflux-2)