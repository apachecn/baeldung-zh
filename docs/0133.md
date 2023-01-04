# rx Java single . just()vs single . from callable()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-single-just-single-fromcallable>

## 1.概观

在这个简短的教程中，我们将比较在 RxJava 中创建一个`Single`对象的两种流行方法，并且我们将使用一个 [`TestSubscriber`](/web/20221209231817/https://www.baeldung.com/rxjava-testing) 来测试实现。首先，我们将看看`Single.just()`工厂方法，并急切地使用它来创建对象的实例。之后，我们将学习`Single.fromCallable()`，看看如何使用它来提高性能。

## 2.`Single.just()`

`Single.just()`是创建一个`Observable.`实例的简单方法，它将一个对象作为参数，并将其包装在 RxJava 的`Single`中:

```
Single<String> employee = Single.just("John Doe");
```

不过，大多数时候，我们会以某种方式检索或计算这些数据。出于演示的目的，让我们假设我们正在从`EmployeeRepository`类中检索雇员的姓名。为了测试，我们将使用 [Mockito](/web/20221209231817/https://www.baeldung.com/mockito-series) 来检查与这个存储库的交互，使用 [`TestSubscriber`](/web/20221209231817/https://www.baeldung.com/rxjava-testing) 来测试结果可观察对象发布的值:

```
@Test
void givenASubscriber_whenUsingJust_thenReturnTheCorrectValue() {
    TestSubscriber<String> testSubscriber = new TestSubscriber<>();
    Mockito.when(repository.findById(123L)).thenReturn("John Doe");

    Single<String> employee = Single.just(repository.findById(123L));
    employee.subscribe(testSubscriber);

    testSubscriber.assertValue("John Doe");
    testSubscriber.assertCompleted();
}
```

正如所料，`testSubscriber` 发布的值与存储库返回的值相同。另一方面，**即使没有订户，数据仍然会被提取**。让我们使用`Mockito`来验证与`EmployeeRepostory`的交互次数:

```
@Test
void givenNoSubscriber_whenUsingJust_thenDataIsFetched() {
    Mockito.when(repository.findById(123L)).thenReturn("John Doe");

    Single<String> employee = Single.just(repository.findById(123L));

    Mockito.verify(repository, times(1)).findById(123L);
}
```

## 3.`Single.fromCallable()`

作为替代，我们可以使用`Single.fromCallable()`。在这种情况下，我们需要提供一个*可调用*接口或 lambda 表达式的实现。换句话说，我们可以传递一个获取数据的函数，这个函数只有在有人订阅时才会被调用。因此，**如果没有订阅者，就不会与`repository`** 进行交互:

```
@Test
void givenNoSubscriber_whenUsingFromCallable_thenNoDataIsFetched() {
    Single<String> employee = Single.fromCallable(() -> repository.findById(123L));

    Mockito.verify(repository, never()).findById(123L);
}
```

但是，一旦有人订阅了`employee`，数据就会被获取并发布。让我们添加另一个测试，检查产生的`Single` 对象是否发布了正确的值:

```
@Test
void givenASubscriber_whenUsingFromCallable_thenReturnCorrectValue() {
    TestSubscriber<String> testSubscriber = new TestSubscriber<>();
    Mockito.when(repository.findById(123L)).thenReturn("John Doe");

    Single<String> employee = Single.fromCallable(() -> repository.findById(123L));
    employee.subscribe(testSubscriber);

    Mockito.verify(repository, times(1)).findById(123L);
    testSubscriber.assertCompleted();
    testSubscriber.assertValue("John Doe");
}
```

## 4.结论

在本文中，我们比较了`Single`的`just()`和`fromCallable()`的工厂方法。我们了解到，如果我们想在获取数据时利用惰性评估，我们应该使用`fromCallable()`选项。

该项目的完整源代码，包括这里使用的所有代码样本，可以在 GitHub 上找到[。](https://web.archive.org/web/20221209231817/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)