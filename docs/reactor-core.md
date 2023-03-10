# 反应堆堆芯简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reactor-core>

## 1。简介

[Reactor Core](https://web.archive.org/web/20220710141727/https://github.com/reactor/reactor-core) 是一个 Java 8 库，实现了反应式编程模型。它建立在[反应流](https://web.archive.org/web/20220710141727/http://www.reactive-streams.org/)规范之上，这是构建反应式应用程序的标准。

从非反应式 Java 开发的背景来看，走向反应式可能是一个相当陡峭的学习曲线。与 Java 8 `Stream` API 相比，这变得更具挑战性，因为它们可能被误认为是相同的高级抽象。

在本文中，我们将试图揭开这一范式的神秘面纱。我们将一步一步地学习 Reactor，直到我们对如何编写反应式代码有一个大致的了解，为后续系列中更高级的文章打下基础。

## 2。反应流规格

在我们看反应器之前，我们应该看看反应流规格。这就是 Reactor 实现的，它为库奠定了基础。

本质上，反应流是异步流处理的规范。

换句话说，这是一个异步产生和消费大量事件的系统。想象一下，每秒钟有成千上万的股票更新进入一个财务应用程序，它必须及时响应这些更新。

这样做的主要目的之一是解决背压问题。如果我们有一个生产者，它向消费者发送事件的速度比它处理事件的速度快，那么最终消费者将被事件淹没，耗尽系统资源。

背压意味着我们的消费者应该能够告诉生产者发送多少数据，以防止这种情况，这就是规范中所规定的。

## 3。Maven 依赖关系

在我们开始之前，让我们添加我们的 [Maven](https://web.archive.org/web/20220710141727/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22io.projectreactor%22%20AND%20a%3A%22reactor-core%22)%20OR%20(g%3A%22ch.qos.logback%22%20AND%20a%3A%22logback-classic%22)) 依赖项:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.4.16</version>
</dependency>

<dependency> 
    <groupId>ch.qos.logback</groupId> 
    <artifactId>logback-classic</artifactId> 
    <version>1.2.6</version> 
</dependency>
```

我们还添加了 [Logback](https://web.archive.org/web/20220710141727/https://logback.qos.ch/) 作为依赖项。这是因为我们将记录反应堆的输出，以便更好地理解数据流。

## 4。产生数据流

为了让应用程序具有反应能力，它必须能够做的第一件事就是生成数据流。

这可能类似于我们之前给出的股票更新示例。没有这些数据，我们就没有任何反应，这就是为什么这是合乎逻辑的第一步。

反应式核心为我们提供了两种数据类型，使我们能够做到这一点。

### 4.1。通量

第一种方法是用`[Flux](https://web.archive.org/web/20220710141727/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html).` 它是一个可以发出`0..n` 元素的流。让我们试着创建一个简单的:

```java
Flux<Integer> just = Flux.just(1, 2, 3, 4);
```

在这种情况下，我们有一个包含四个元素的静态流。

### 4.2。单声道

第二种方法是使用一个由`0..1` 元素组成的流`[Mono](https://web.archive.org/web/20220710141727/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html),` 。让我们试着实例化一个:

```java
Mono<Integer> just = Mono.just(1);
```

这看起来和行为几乎与`Flux`完全相同，只是这次我们被限制在不超过一个元素。

### 4.3。为什么不仅仅是通量？

在进一步实验之前，有必要强调一下为什么我们有这两种数据类型。

首先，应该注意到`Flux`和`Mono`都是反应流`[Publisher](https://web.archive.org/web/20220710141727/http://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Publisher.html)` 接口的实现。这两个类都符合规范，我们可以用这个接口来代替它们:

```java
Publisher<String> just = Mono.just("foo");
```

但实际上，知道这个基数是有用的。这是因为一些操作只对两种类型中的一种有意义，并且因为它更有表现力(想象一下`findOne()` 在一个存储库中)。

## 5。订阅流

现在我们对如何产生数据流有了一个高层次的概述，我们需要订阅它以便它发出元素。

### 5.1。收集元素

让我们使用 `subscribe()` 方法收集流中的所有元素:

```java
List<Integer> elements = new ArrayList<>();

Flux.just(1, 2, 3, 4)
  .log()
  .subscribe(elements::add);

assertThat(elements).containsExactly(1, 2, 3, 4);
```

在我们订阅之前，数据不会开始流动。请注意，我们还添加了一些日志记录，这将有助于我们了解幕后发生的事情。

### 5.2。元素的流动

有了日志记录，我们可以使用它来可视化数据是如何流经我们的流的:

```java
20:25:19.550 [main] INFO  reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
20:25:19.553 [main] INFO  reactor.Flux.Array.1 - | request(unbounded)
20:25:19.553 [main] INFO  reactor.Flux.Array.1 - | onNext(1)
20:25:19.553 [main] INFO  reactor.Flux.Array.1 - | onNext(2)
20:25:19.553 [main] INFO  reactor.Flux.Array.1 - | onNext(3)
20:25:19.553 [main] INFO  reactor.Flux.Array.1 - | onNext(4)
20:25:19.553 [main] INFO  reactor.Flux.Array.1 - | onComplete()
```

首先，一切都在主线程上运行。我们不要详细讨论这个问题，因为我们将在本文后面进一步研究并发性。不过，这确实让事情变得简单了，因为我们可以按顺序处理所有事情。

现在，让我们逐一查看我们记录的序列:

1.  当我们订阅我们的流时，这被调用
2.  `request(unbounded) –` 当我们调用`subscribe`时，我们在幕后创建了一个`[Subscription](https://web.archive.org/web/20220710141727/http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Subscription.html).` ，这个订阅从流中请求元素。在这种情况下，它默认为`unbounded,` ，这意味着它请求所有可用的元素
3.  这在每个元素上都被调用
4.  在接收到最后一个元素后，这被称为 last。实际上也有一个`onError()`,如果有异常就会被调用，但是在这个例子中，没有异常

这是作为反应流规范的一部分在`[Subscriber](https://web.archive.org/web/20220710141727/http://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Subscriber.html)` 接口中布置的流，实际上，这是我们调用`onSubscribe().` 时在幕后实例化的内容。这是一个有用的方法，但是为了更好地理解发生了什么，让我们直接提供一个`Subscriber`接口:

```java
Flux.just(1, 2, 3, 4)
  .log()
  .subscribe(new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
      s.request(Long.MAX_VALUE);
    }

    @Override
    public void onNext(Integer integer) {
      elements.add(integer);
    }

    @Override
    public void onError(Throwable t) {}

    @Override
    public void onComplete() {}
});
```

我们可以看到，上面流程中的每个可能的阶段都映射到了`Subscriber` 实现中的一个方法。恰好`Flux` 为我们提供了一个 helper 方法来减少这种冗长。

### 5.3。与 Java 8 `Streams`的比较

看起来我们仍然有一些 Java 8 `Stream`做收集的同义词:

```java
List<Integer> collected = Stream.of(1, 2, 3, 4)
  .collect(toList());
```

只是我们没有。

核心区别在于，Reactive 是推模型，而 Java 8 `Streams`是拉模型。**在反应式方法中，事件在订户进来时就对他们来说是`pushed` 。**

接下来要注意的是一个`Streams` 终端操作符，终端，提取所有数据并返回一个结果。使用 Reactive，我们可以从外部资源获得一个无限的流，在特定的基础上附加和删除多个订户。我们还可以做一些事情，比如合并流、节流流和应用背压，这将在接下来讨论。

## 6。背压

接下来我们要考虑的是背压。在我们的例子中，订阅者告诉生产者一次推送每个元素。这最终可能会使订户不堪重负，耗尽其所有资源。

**背压是指下游可以告诉上游发送更少的数据，以防止其被淹没**。

我们可以修改我们的`Subscriber` 实现来应用背压。让我们使用`request()`告诉上游一次只发送两个元素:

```java
Flux.just(1, 2, 3, 4)
  .log()
  .subscribe(new Subscriber<Integer>() {
    private Subscription s;
    int onNextAmount;

    @Override
    public void onSubscribe(Subscription s) {
        this.s = s;
        s.request(2);
    }

    @Override
    public void onNext(Integer integer) {
        elements.add(integer);
        onNextAmount++;
        if (onNextAmount % 2 == 0) {
            s.request(2);
        }
    }

    @Override
    public void onError(Throwable t) {}

    @Override
    public void onComplete() {}
});
```

现在，如果我们再次运行我们的代码，我们会看到`request(2)` 被调用，接着是两个`onNext()` 调用，然后是`request(2)` 。

```java
23:31:15.395 [main] INFO  reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
23:31:15.397 [main] INFO  reactor.Flux.Array.1 - | request(2)
23:31:15.397 [main] INFO  reactor.Flux.Array.1 - | onNext(1)
23:31:15.398 [main] INFO  reactor.Flux.Array.1 - | onNext(2)
23:31:15.398 [main] INFO  reactor.Flux.Array.1 - | request(2)
23:31:15.398 [main] INFO  reactor.Flux.Array.1 - | onNext(3)
23:31:15.398 [main] INFO  reactor.Flux.Array.1 - | onNext(4)
23:31:15.398 [main] INFO  reactor.Flux.Array.1 - | request(2)
23:31:15.398 [main] INFO  reactor.Flux.Array.1 - | onComplete()
```

本质上，这是反作用拉背压。我们要求上游只推送一定数量的元素，而且只有在我们准备好的时候。

如果我们想象我们正在从 Twitter 上接收推文，那么将由上游来决定做什么。如果有推文传入，但是下游没有请求，那么上游可以丢弃项目，将它们存储在缓冲区中，或者采用其他策略。

## 7。在流上操作

我们还可以对流中的数据执行操作，以我们认为合适的方式响应事件。

### 7.1。在流中映射数据

我们可以执行的一个简单操作是应用转换。在这种情况下，让我们将流中的所有数字加倍:

```java
Flux.just(1, 2, 3, 4)
  .log()
  .map(i -> i * 2)
  .subscribe(elements::add);
```

调用`onNext()` 时将应用`map()` 。

### 7.2。合并两个流

然后，我们可以通过将另一个流与这个流相结合来使事情变得更有趣。让我们通过使用`zip()` 函数`:`来尝试一下

```java
Flux.just(1, 2, 3, 4)
  .log()
  .map(i -> i * 2)
  .zipWith(Flux.range(0, Integer.MAX_VALUE), 
    (one, two) -> String.format("First Flux: %d, Second Flux: %d", one, two))
  .subscribe(elements::add);

assertThat(elements).containsExactly(
  "First Flux: 2, Second Flux: 0",
  "First Flux: 4, Second Flux: 1",
  "First Flux: 6, Second Flux: 2",
  "First Flux: 8, Second Flux: 3");
```

在这里，我们创建了另一个`Flux`,它一直递增 1，并与原来的那个一起流式传输。通过检查日志，我们可以看到这些是如何协同工作的:

```java
20:04:38.064 [main] INFO  reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
20:04:38.065 [main] INFO  reactor.Flux.Array.1 - | onNext(1)
20:04:38.066 [main] INFO  reactor.Flux.Range.2 - | onSubscribe([Synchronous Fuseable] FluxRange.RangeSubscription)
20:04:38.066 [main] INFO  reactor.Flux.Range.2 - | onNext(0)
20:04:38.067 [main] INFO  reactor.Flux.Array.1 - | onNext(2)
20:04:38.067 [main] INFO  reactor.Flux.Range.2 - | onNext(1)
20:04:38.067 [main] INFO  reactor.Flux.Array.1 - | onNext(3)
20:04:38.067 [main] INFO  reactor.Flux.Range.2 - | onNext(2)
20:04:38.067 [main] INFO  reactor.Flux.Array.1 - | onNext(4)
20:04:38.067 [main] INFO  reactor.Flux.Range.2 - | onNext(3)
20:04:38.067 [main] INFO  reactor.Flux.Array.1 - | onComplete()
20:04:38.067 [main] INFO  reactor.Flux.Array.1 - | cancel()
20:04:38.067 [main] INFO  reactor.Flux.Range.2 - | cancel()
```

请注意，我们现在每个`Flux`都有一个订阅。`onNext()` 调用也是交替的，所以当我们应用`zip()` 函数时，流中每个元素的索引都将匹配。

## 8。热流

目前，我们主要关注冷流。这些是静态的、固定长度的流，很容易处理。reactive 的一个更现实的用例可能是无限发生的事情。

例如，我们可以有一个不断需要响应的鼠标移动流或一个 Twitter feed。这些类型的流称为热流，因为它们总是在运行，并且可以在任何时间点订阅，错过了数据的开始。

### 8.1。创造一个`ConnectableFlux`

产生热流的一种方法是将冷流转换成热流。让我们创建一个持久的`Flux`，将结果输出到控制台，这将模拟来自外部资源的无限数据流:

```java
ConnectableFlux<Object> publish = Flux.create(fluxSink -> {
    while(true) {
        fluxSink.next(System.currentTimeMillis());
    }
})
  .publish();
```

通过调用`publish()` ,我们得到一个`[ConnectableFlux](https://web.archive.org/web/20220710141727/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/ConnectableFlux.html).` ,这意味着调用`subscribe()`不会导致它开始发射，允许我们添加多个订阅:

```java
publish.subscribe(System.out::println);        
publish.subscribe(System.out::println);
```

如果我们尝试运行这段代码，什么也不会发生。直到我们调用`connect(),` 时，`Flux`才会开始发射:

```java
publish.connect();
```

### 8.2。节流

如果我们运行我们的代码，我们的控制台将被日志淹没。这是在模拟一种情况，向我们的消费者传递了太多的数据。让我们试着用节流来解决这个问题:

```java
ConnectableFlux<Object> publish = Flux.create(fluxSink -> {
    while(true) {
        fluxSink.next(System.currentTimeMillis());
    }
})
  .sample(ofSeconds(2))
  .publish();
```

这里，我们介绍了一个间隔两秒钟的`sample()`方法。现在，每两秒钟就会向我们的订阅者推送一次值，这意味着控制台将不再那么繁忙。

当然，有多种策略可以减少向下游发送的数据量，比如窗口和缓冲，但是它们不在本文的讨论范围之内。

## 9。并发性

我们上面的所有例子目前都运行在主线程上。然而，如果我们愿意，我们可以控制我们的代码在哪个线程上运行。 [`Scheduler`](https://web.archive.org/web/20220710141727/https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Scheduler.html) 接口提供了异步代码的抽象，为我们提供了许多实现。让我们尝试订阅不同的线程来维护:

```java
Flux.just(1, 2, 3, 4)
  .log()
  .map(i -> i * 2)
  .subscribeOn(Schedulers.parallel())
  .subscribe(elements::add);
```

调度程序将导致我们的订阅在不同的线程上运行，我们可以通过查看日志来证明这一点。我们看到第一个条目来自于`main`线程，而 Flux 在另一个叫做`parallel-1`的线程中运行。

```java
20:03:27.505 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
20:03:27.529 [parallel-1] INFO  reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
20:03:27.531 [parallel-1] INFO  reactor.Flux.Array.1 - | request(unbounded)
20:03:27.531 [parallel-1] INFO  reactor.Flux.Array.1 - | onNext(1)
20:03:27.531 [parallel-1] INFO  reactor.Flux.Array.1 - | onNext(2)
20:03:27.531 [parallel-1] INFO  reactor.Flux.Array.1 - | onNext(3)
20:03:27.531 [parallel-1] INFO  reactor.Flux.Array.1 - | onNext(4)
20:03:27.531 [parallel-1] INFO  reactor.Flux.Array.1 - | onComplete()
```

并发 get 比这更有趣，值得我们在另一篇文章中探讨。

## 10。结论

在本文中，我们对反应式核心进行了高层次的端到端概述。我们已经解释了如何发布和订阅流、应用背压、操作流以及异步处理数据。这应该有望为我们编写反应式应用程序奠定基础。

本系列的后续文章将涵盖更高级的并发性和其他反应性概念。还有另一篇文章涉及带弹簧的反应器。

GitHub 上的[提供了我们应用程序的源代码；这是一个 Maven 项目，应该能够按原样运行。](https://web.archive.org/web/20220710141727/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-reactive)