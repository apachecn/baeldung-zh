# 使用 StepVerifier 和 TestPublisher 测试反应流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reactive-streams-step-verifier-test-publisher>

## 1.概观

在本教程中，我们将仔细研究用`StepVerifier`和`TestPublisher`测试[反应流](https://web.archive.org/web/20221126234152/http://www.reactive-streams.org/)。

我们将基于一个包含一系列反应堆操作的 [`Spring Reactor`](/web/20221126234152/https://www.baeldung.com/spring-reactor) 应用程序进行研究。

## 2.Maven 依赖性

Spring Reactor 附带了几个用于测试反应流的类。

我们可以通过添加[的`reactor-test`依赖](https://web.archive.org/web/20221126234152/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.projectreactor%22%20AND%20a%3A%22reactor-test%22)来得到这些:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
    <version>3.2.3.RELEASE</version>
</dependency>
```

## 3.`StepVerifier`

一般来说，`reactor-test`有两个主要用途:

*   用`StepVerifier`创建分步测试
*   使用`TestPublisher `生成预定义数据，以测试下游操作员

测试反应流最常见的情况是当我们在代码中定义了一个发布者(一个`Flux `或`Mono`)时。**我们想知道当有人订阅时它会有什么反应。**

使用`StepVerifier` API，我们可以根据**我们期望什么元素以及当我们的流完成**时会发生什么来定义我们对发布元素的期望。

首先，我们用一些操作符创建一个发布者。

我们将使用一个`Flux.just(T elements).`这个方法将创建一个`Flux `来发出给定的元素，然后完成。

由于高级操作符超出了本文的范围，我们将只创建一个简单的 publisher，它只输出映射到大写字母的四个字母的名称:

```java
Flux<String> source = Flux.just("John", "Monica", "Mark", "Cloe", "Frank", "Casper", "Olivia", "Emily", "Cate")
  .filter(name -> name.length() == 4)
  .map(String::toUpperCase);
```

### 3.1.逐步情景

现在，让我们用`StepVerifier` **来测试我们的`source `，以便测试当有人订阅**时会发生什么:

```java
StepVerifier
  .create(source)
  .expectNext("JOHN")
  .expectNextMatches(name -> name.startsWith("MA"))
  .expectNext("CLOE", "CATE")
  .expectComplete()
  .verify();
```

首先，我们用`create `方法创建一个`StepVerifier `构建器。

接下来，我们包装我们正在测试的`Flux `源。第一个信号用`expectNext(T element), `验证，但是实际上，**我们可以传递任意数量的元素给`expectNext`。**

我们也可以使用`expectNextMatches `并提供一个`Predicate<T> `来进行更加定制的匹配。

对于我们最后的期望，我们期望我们的流完成。

最后，**我们使用`verify()`来触发我们的测试**。

### 3.2.`StepVerifier`中的异常

现在，让我们把我们的`Flux`出版商和`Mono` `.`连接起来

**当订阅**时，我们将让此`Mono `因错误而立即终止:

```java
Flux<String> error = source.concatWith(
  Mono.error(new IllegalArgumentException("Our message"))
);
```

现在，在四个元素之后，**我们期望我们的流以一个异常**终止:

```java
StepVerifier
  .create(error)
  .expectNextCount(4)
  .expectErrorMatches(throwable -> throwable instanceof IllegalArgumentException &&
    throwable.getMessage().equals("Our message")
  ).verify();
```

**我们只能使用一种方法来验证异常。**`OnError` 信号通知订阅者**发布者因错误状态而关闭。因此，我们不能在**之后增加更多的期待。

如果没有必要立刻检查异常的类型和消息，那么我们可以使用一个专用的方法:

*   期待任何种类的错误
*   预期特定类型的错误
*   `expectErrorMessage(String errorMessage) – `预期有特定消息的错误
*   `expectErrorMatches(Predicate<Throwable> predicate) `–预期出现与给定谓词匹配的错误
*   `expectErrorSatisfies(Consumer<Throwable> assertionConsumer) `–消耗一个`Throwable `以执行自定义断言

### 3.3.测试基于时间的发布者

有时我们的出版商是基于时间的。

例如，假设在我们现实生活的应用程序中，**在事件**之间有一天的延迟。现在，很明显，我们不希望我们的测试运行一整天来验证这样延迟的预期行为。

**`StepVerifier.withVirtualTime`构建器旨在避免长时间运行的测试。**

**我们通过调用`withVirtualTime`来创建一个构建器。** **注意，这个方法不把`Flux` 作为输入。相反，它需要一个`Supplier`，在设置好调度程序后，它会创建一个被测试的`Flux `的实例。**

为了演示我们如何测试事件之间的预期延迟，让我们创建一个运行两秒钟的间隔为一秒钟的`Flux `。**如果计时器运行正确，我们应该只得到两个元素:**

```java
StepVerifier
  .withVirtualTime(() -> Flux.interval(Duration.ofSeconds(1)).take(2))
  .expectSubscription()
  .expectNoEvent(Duration.ofSeconds(1))
  .expectNext(0L)
  .thenAwait(Duration.ofSeconds(1))
  .expectNext(1L)
  .verifyComplete();
```

注意，我们应该避免在代码的早期实例化`Flux `，然后让`Supplier `返回这个变量。相反，**我们应该总是在 lambda 中实例化`Flux `。**

有两种主要的处理时间的期望方法:

*   `thenAwait(Duration duration) –`暂停步骤的评估；在此期间可能会发生新的事件
*   `duration`期间出现任何事件时`expectNoEvent(Duration duration) – `失效；该序列将通过给定的`duration`

请注意，第一个信号是订阅事件，所以每个`expectNoEvent(Duration duration)`的**前面都应该有** `**expectSubscription()**.`

### 3.4.带有`StepVerifier`的执行后断言

因此，正如我们所看到的，逐步描述我们的期望是很简单的。

然而，有时我们需要在我们的整个场景成功结束后验证额外的状态。

让我们创建一个自定义发布者。**它将发射一些元素，然后完成，暂停，再发射一个元素，我们将删除**:

```java
Flux<Integer> source = Flux.<Integer>create(emitter -> {
    emitter.next(1);
    emitter.next(2);
    emitter.next(3);
    emitter.complete();
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    emitter.next(4);
}).filter(number -> number % 2 == 0);
```

**我们预计它将发出一个 2，但丢弃一个 4，因为我们首先调用了`emitter.complete`。**

所以，让我们通过使用`verifyThenAssertThat. `来验证这个行为。这个方法返回`StepVerifier.Assertions `，我们可以在上面添加我们的断言:

```java
@Test
public void droppedElements() {
    StepVerifier.create(source)
      .expectNext(2)
      .expectComplete()
      .verifyThenAssertThat()
      .hasDropped(4)
      .tookLessThan(Duration.ofMillis(1050));
}
```

## 4.用`TestPublisher`产生数据

有时，我们可能需要一些特殊数据来触发选定的信号。

例如，我们可能有一个非常特殊的情况需要测试。

或者，我们可以选择实现自己的操作符，并测试它的行为。

对于这两种情况，我们都可以使用`TestPublisher<T>`，其中**允许我们以编程方式触发各种信号:**

*   `next(T value)` 或`next(T value, T rest) – `向用户发送一个或多个信号
*   `emit(T value) – `与`next(T) `相同，但之后调用`complete()`
*   `complete()`–用`complete`信号终止信号源
*   `error(Throwable tr) – `终止出错的信号源
*   `flux() – `将一个`TestPublisher `包装成`Flux`的便捷方法
*   `mono() `–相同的美国`flux() `，但包装为`Mono`

### 4.1.创建一个`TestPublisher`

让我们创建一个简单的`TestPublisher `，它发出几个信号，然后以一个异常终止:

```java
TestPublisher
  .<String>create()
  .next("First", "Second", "Third")
  .error(new RuntimeException("Message"));
```

### 4.2.`TestPublisher`在行动

正如我们之前提到的，我们有时可能想要触发**一个精心选择的信号，与特定的情况紧密匹配。**

现在，在这种情况下，我们完全掌握数据的来源尤为重要。为了实现这一点，我们可以再次依靠`TestPublisher`。

首先，让我们创建一个使用`Flux<String> `作为构造函数参数来执行操作`getUpperCase()`的类:

```java
class UppercaseConverter {
    private final Flux<String> source;

    UppercaseConverter(Flux<String> source) {
        this.source = source;
    }

    Flux<String> getUpperCase() {
        return source
          .map(String::toUpperCase);
    }   
}
```

假设`UppercaseConverter `是我们具有复杂逻辑和操作符的类，我们需要从`source `发布者那里提供非常特殊的数据。

我们可以用`TestPublisher:`轻松实现这一点

```java
final TestPublisher<String> testPublisher = TestPublisher.create();

UppercaseConverter uppercaseConverter = new UppercaseConverter(testPublisher.flux());

StepVerifier.create(uppercaseConverter.getUpperCase())
  .then(() -> testPublisher.emit("aA", "bb", "ccc"))
  .expectNext("AA", "BB", "CCC")
  .verifyComplete();
```

在这个例子中，我们在`UppercaseConverter `构造函数参数中创建一个测试*流量*发布者。然后，我们的`TestPublisher`发出三个元素，完成。

### 4.3.行为不端`TestPublisher`

另一方面，**我们可以用`the createNonCompliant` 工厂方法创建一个行为不端的`TestPublisher`。**我们需要在构造函数中传递一个来自`TestPublisher.Violation.`的枚举值，这些值指定了我们的发布者可能会忽略规范的哪些部分。

让我们来看看一个不会为`null`元素抛出`NullPointerException `的`TestPublisher `:

```java
TestPublisher
  .createNoncompliant(TestPublisher.Violation.ALLOW_NULL)
  .emit("1", "2", null, "3"); 
```

除了`ALLOW_NULL,`之外，我们还可以使用`TestPublisher.Violation `来:

*   `REQUEST_OVERFLOW`–当请求数量不足时，允许调用`next() `而不抛出`IllegalStateException`
*   `CLEANUP_ON_TERMINATE – `允许连续多次发送任何终止信号
*   `DEFER_CANCELLATION –`允许我们忽略抵消信号，继续发射元件

## 5.结论

在本文中，**我们讨论了测试来自`Spring Reactor`项目的反应流的各种方法。**

首先，我们看到了如何使用`StepVerifier`来测试出版商。然后，我们看到了如何使用`TestPublisher.`同样，我们看到了如何操作一个行为不端的`TestPublisher`。

像往常一样，我们所有示例的实现都可以在 [Github 项目](https://web.archive.org/web/20221126234152/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-2)中找到。