# 处理 Project Reactor 中的异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reactor-exceptions>

## 1.概观

在本教程中，我们将看看在[项目反应器](/web/20220625074631/https://www.baeldung.com/reactor-core)中处理异常的几种方法。代码示例中引入的运算符在 [`Mono`](https://web.archive.org/web/20220625074631/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) 和 [`Flux`](https://web.archive.org/web/20220625074631/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) 类中都有定义。然而，**我们将只关注`Flux`类**中的方法。

## 2.Maven 依赖性

让我们从添加[反应堆堆芯依赖性](https://web.archive.org/web/20220625074631/https://search.maven.org/search?q=g:%22io.projectreactor%22%20a:%22reactor-core%22)开始:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId
    <version>3.4.9</version>
</dependency>
```

## 3.直接在管道运算符中引发异常

处理一个`Exception`的最简单的方法是抛出它。如果在处理一个流元素的过程中发生了异常，**我们可以抛出一个带有`throw`关键字的`Exception`，就好像它是一个正常的方法执行**。

假设我们需要解析一个从`String`到`Integer`的流。如果一个元素不是数字`String`，我们将需要抛出一个`Exception`。

使用`map`操作符进行这种转换是一种常见的做法:

```java
Function<String, Integer> mapper = input -> {
    if (input.matches("\\D")) {
        throw new NumberFormatException();
    } else {
        return Integer.parseInt(input);
    }
};

Flux<String> inFlux = Flux.just("1", "1.5", "2");
Flux<Integer> outFlux = inFlux.map(mapper);
```

正如我们看到的，如果输入元素无效，操作符抛出一个`Exception`。当我们以这种方式抛出`Exception`时，**反应堆捕捉到它，并向下游**发出错误信号:

```java
StepVerifier.create(outFlux)
    .expectNext(1)
    .expectError(NumberFormatException.class)
    .verify();
```

这种解决方案是可行的，但并不优雅。根据反应流规范[规则 2.13](https://web.archive.org/web/20220625074631/https://github.com/reactive-streams/reactive-streams-jvm#2-subscriber-code) 的规定，操作员必须正常返回。Reactor 通过将`Exception`转换成错误信号帮助了我们。然而，我们可以做得更好。

本质上，**反应流依靠`onError`方法来指示故障条件**。在大多数情况下，这个条件**必须通过调用 [`Publisher`](https://web.archive.org/web/20220625074631/https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Publisher.html)** 上的`error`方法来触发。在这个用例中使用一个`Exception`将我们带回传统编程。

## 4.在`handle`操作符中处理异常

类似于`map`操作符，我们可以使用 [`handle`](https://web.archive.org/web/20220625074631/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#handle-java.util.function.BiConsumer-) 操作符来逐个处理流中的项目。不同之处在于**反应器为`handle`操作符提供了一个输出接收器**，允许我们应用更复杂的转换。

让我们更新上一节的例子，使用`handle`操作符:

```java
BiConsumer<String, SynchronousSink<Integer>> handler = (input, sink) -> {
    if (input.matches("\\D")) {
        sink.error(new NumberFormatException());
    } else {
        sink.next(Integer.parseInt(input));
    }
};

Flux<String> inFlux = Flux.just("1", "1.5", "2");
Flux<Integer> outFlux = inFlux.handle(handler);
```

与`map`操作符不同，**`handle`操作符接收一个功能性的[消费者](https://web.archive.org/web/20220625074631/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiConsumer.html)，为每个元素**调用一次。这个消费者有两个参数:一个来自上游的元素和一个 [`SynchronousSink`](https://web.archive.org/web/20220625074631/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/SynchronousSink.html) 构建要发送到下游的输出。

如果输入元素是一个数字`String`，我们在接收器上调用`next`方法，为它提供从输入转换而来的`Integer`。如果它不是一个数字`String`，我们将通过用一个`Exception`对象调用`error`方法来指示这种情况。

注意，**调用`error`方法将取消对上游的订阅，并调用下游**的`onError`方法。`error`和`onError`的这种合作是在反应流中处理`Exception`的标准方式。

让我们验证输出流:

```java
StepVerifier.create(outFlux)
    .expectNext(1)
    .expectError(NumberFormatException.class)
    .verify();
```

## 5.在`flatMap`操作符中处理异常

**另一个支持错误处理的常用运算符是 [`flatMap`](https://web.archive.org/web/20220625074631/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMap-java.util.function.Function-)** 。该操作符将输入元素转换成`Publisher` s，然后将`Publisher` s 展平成一个新的流。我们可以利用这些`Publisher`来表示一个错误的状态。

让我们使用`flatMap`来尝试同样的例子:

```java
Function<String, Publisher<Integer>> mapper = input -> {
    if (input.matches("\\D")) {
        return Mono.error(new NumberFormatException());
    } else {
        return Mono.just(Integer.parseInt(input));
    }
};

Flux<String> inFlux = Flux.just("1", "1.5", "2");
Flux<Integer> outFlux = inFlux.flatMap(mapper);

StepVerifier.create(outFlux)
    .expectNext(1)
    .expectError(NumberFormatException.class)
    .verify();
```

不出所料，结果和之前一样。

注意**`handle`和`flatMap`在错误处理方面的唯一区别是`handle`操作符在接收器上调用`error`方法，而`flatMap`在`Publisher`** 上调用它。

**如果我们正在处理一个由`Flux`对象表示的流，我们也可以使用 [`concatMap`](https://web.archive.org/web/20220625074631/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatMap-java.util.function.Function-) 来处理错误**。这个方法的行为与`flatMap`非常相似，但是它不支持异步处理。

## 6.避免`NullPointerException`

本节涵盖了对`null`引用的处理，这通常会导致`NullPointerException` s，这是 Java 中经常遇到的`Exception`。为了避免这种异常，我们通常将一个变量与`null`进行比较，如果这个变量实际上是`null`的话，我们就以不同的方式执行。在反应流中做同样的事情很有诱惑力:

```java
Function<String, Integer> mapper = input -> {
    if (input == null) {
        return 0;
    } else {
        return Integer.parseInt(input);
    }
};
```

我们可能认为`NullPointerException`不会发生，因为我们已经处理了输入值为`null`的情况。然而，现实告诉我们一个不同的故事:

```java
Flux<String> inFlux = Flux.just("1", null, "2");
Flux<Integer> outFlux = inFlux.map(mapper);

StepVerifier.create(outFlux)
    .expectNext(1)
    .expectError(NullPointerException.class)
    .verify();
```

显然， **a `NullPointerException`触发了下游的错误，意味着我们的`null`检查没有起作用**。

为了理解为什么会这样，我们需要回到反应流规范。[规范的规则 2.13](https://web.archive.org/web/20220625074631/https://github.com/reactive-streams/reactive-streams-jvm#2-subscriber-code) 说“调用`onSubscribe`、`onNext`、`onError`或`onComplete`必须正常返回，除非当任何一个提供的参数是`null`时，在这种情况下它必须抛出一个`java.lang.NullPointerException`给调用者”。

**根据规范要求，当`null`值达到`map`函数**时，反应堆抛出`NullPointerException`。

因此，当一个`null`值到达某个流时，我们对此无能为力。在将它传递到下游之前，我们不能处理它或将其转换为非`null`值。因此，**避免`NullPointerException`的唯一方法是确保`null`值不会到达流水线**。

## 7.结论

在本文中，我们已经走过了 Project Reactor 中的`Exception`处理。我们讨论了几个例子并阐明了过程。我们还讨论了在处理一个反应流时可能发生的一个特殊的异常情况— `NullPointerException`。

像往常一样，我们的应用程序的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220625074631/https://github.com/eugenp/tutorials/tree/master/reactor-core)