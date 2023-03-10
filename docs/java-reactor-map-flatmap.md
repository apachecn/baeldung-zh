# 项目反应器:map()与 flatMap()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reactor-map-flatmap>

## 1.概观

本教程介绍了[项目反应器](https://web.archive.org/web/20220526060607/https://projectreactor.io/)中的`map`和`flatMap`操作符。它们是在 [`Mono`](https://web.archive.org/web/20220526060607/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) 和 [`Flux`](https://web.archive.org/web/20220526060607/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) 类中定义的，用于在处理流时转换项目。

在接下来的章节中，**我们将重点关注`Flux`类**中的`map`和`flatMap`方法。那些在`Mono`类中同名的只是以同样的方式工作。

## 2.Maven 依赖性

为了编写一些代码示例，我们需要[反应堆核心依赖关系](https://web.archive.org/web/20220526060607/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.projectreactor%22%20AND%20a%3A%22reactor-core%22):

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.3.9.RELEASE</version>
</dependency>
```

## 3.`map`操作员

现在，让我们看看如何使用`map`操作符。

`Flux#map`方法需要一个单独的`Function`参数，可以很简单:

```java
Function<String, String> mapper = String::toUpperCase;
```

这个映射器将字符串转换成大写形式。我们可以将其应用于`Flux`流:

```java
Flux<String> inFlux = Flux.just("baeldung", ".", "com");
Flux<String> outFlux = inFlux.map(mapper);
```

给定的映射器**将输入流中的每个项目转换为输出中的一个新项目，并保持顺序**。

让我们证明:

```java
StepVerifier.create(outFlux)
  .expectNext("BAELDUNG", ".", "COM")
  .expectComplete()
  .verify();
```

请注意，当调用`map`方法时，不会执行 mapper 函数。相反，**在我们订阅流**的时候运行。

## 4.`flatMap`操作员

是时候转到`flatMap`操作符了。

### 4.1.代码示例

类似于`map`,`flatMap`操作符有一个类型为`Function`的单一参数。然而，与使用`map`的函数不同，`flatMap`映射函数**将输入项转换成`Publisher`** 而不是普通对象。

这里有一个例子:

```java
Function<String, Publisher<String>> mapper = s -> Flux.just(s.toUpperCase().split(""));
```

在这种情况下，mapper 函数将字符串转换为大写形式，然后将其拆分为单独的字符。最后，该函数从这些字符构建一个新的流。

我们现在可以将给定的映射器传递给一个`flatMap`方法:

```java
Flux<String> inFlux = Flux.just("baeldung", ".", "com");
Flux<String> outFlux = inFlux.flatMap(mapper);
```

我们看到的平面映射操作用三个字符串项从上游创建了三个新的流。之后，来自这三个流的元素被分割并交织在一起，形成另一个新的流。这个最终流包含所有三个输入字符串中的字符。

然后，我们可以订阅这个新形成的流来触发管道并验证输出:

```java
List<String> output = new ArrayList<>();
outFlux.subscribe(output::add);
assertThat(output).containsExactlyInAnyOrder("B", "A", "E", "L", "D", "U", "N", "G", ".", "C", "O", "M");
```

请注意，由于来自不同来源的项目交错，**它们在输出中的顺序可能与我们在输入**中看到的不同。

### 4.2.管道操作说明

我们刚刚定义了一个映射器，将它传递给一个`flatMap`操作符，并在流上调用这个操作符。是时候深入细节，看看为什么输出中的项目可能是无序的。

首先，让我们弄清楚**在流被订阅之前**不会发生任何操作。当这种情况发生时，管道执行并调用传递给`flatMap`方法的映射函数。

此时，映射器对输入流中的元素执行必要的转换。**这些元素中的每一个都可以被转换成多个条目，然后这些条目被用来创建一个新的流**。在我们的代码示例中，表达式`Flux.just(s.toUpperCase().split(""))`的值表示这样一个流。

一旦一个新的流——由一个`Publisher`实例表示——准备好了，`flatMap`就会急切地订阅。**操作者不等待发布者完成就继续下一个流**，这意味着订阅是非阻塞的。

因为管道同时处理所有派生的流，所以它们的项目可能随时进来。结果，原来的订单丢失了。如果项目的顺序很重要，可以考虑使用`[flatMapSequential](https://web.archive.org/web/20220526060607/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMapSequential-java.util.function.Function-)`操作符。

## 5.`map`和`flatMap`的区别

到目前为止，我们已经讨论了`map`和`flatMap`操作符。让我们总结一下它们之间的主要区别。

### 5.1.一对一与一对多

**`map`操作符对流元素应用一对一的转换，而`flatMap`执行一对多的**。当查看方法签名时，这种区别很明显:

*   `<V> Flux<V> map(Function<? super T, ? extends V> mapper)`–映射器将类型`T`的单个值转换为类型`V`的单个值
*   `Flux<R> flatMap(Function<? super T, ? extends Publisher<? extends R>> mapper)`–映射器将类型为`T`的单个值转换为类型为`R`的元素的`Publisher`

我们可以看到，在功能上，Project Reactor 中的`map`和`flatMap`的区别类似于 Java Stream API 中的`map`和`flatMap`的区别。

### 5.2.同步与异步

以下是反应堆堆芯库 API 规范的两个摘录:

*   [`map`](https://web.archive.org/web/20220526060607/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#map-java.util.function.Function-) :通过对每个项目应用同步函数来转换此`Flux`发出的项目
*   [`flatMap`](https://web.archive.org/web/20220526060607/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMap-java.util.function.Function-) :将这个`Flux`发出的元素异步转换成`Publishers`

很容易看出 **`map`是一个同步操作符**——它只是一个将一个值转换成另一个值的方法。此方法与调用方在同一个线程中执行。

另一种说法——**`flatMap`是异步的**——并不清楚。事实上，元素到`Publishers`的转换可以是同步的，也可以是异步的。

在我们的示例代码中，操作是同步的，因为我们用`Flux#just`方法发出元素。然而，**当处理引入高延迟的源时，比如远程服务器，异步处理是更好的选择**。

重要的一点是，管道不关心元素来自哪个线程——它只关注发布者本身。

## 6.结论

在本文中，我们已经遍历了 Project Reactor 中的`map`和`flatMap`操作符。我们讨论了几个例子并阐明了过程。

像往常一样，我们的应用程序的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220526060607/https://github.com/eugenp/tutorials/tree/master/reactor-core)