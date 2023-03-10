# Java 9 反应流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-reactive-streams>

## 1.概观

在本文中，我们将关注 Java 9 反应流。简单地说，我们将能够使用`Flow` 类，它包含了构建反应式流处理逻辑的主要构件。

`Reactive Streams`是无阻塞背压异步流处理的标准。该规范在`[Reactive Manifesto](https://web.archive.org/web/20221129021658/http://www.reactive-streams.org/),` 中定义，有多种实现方式，例如`RxJava`或`Akka-Streams.`

## 2。反应式 API 概述

为了构建一个`Flow`，我们可以使用三个主要的抽象，并将它们组合成异步处理逻辑。

**每个`Flow` 都需要处理发布者实例**发布给它的事件；`Publisher` 有一个方法——`subscribe().`

如果任何订阅者想要接收它发布的事件，他们需要订阅给定的`Publisher.`

**消息的接收者需要实现`Subscriber`接口。**通常这是每个`Flow` 处理的结束，因为它的实例不再发送消息。

我们可以把`Subscriber` 看作一个`Sink.`，它有四个方法需要被覆盖——`onSubscribe(), onNext(), onError(),` 和`onComplete().` ,我们将在下一节中讨论这些方法。

**如果我们想要转换传入的消息，并将其进一步传递给下一个`Subscriber,` ，我们需要实现`Processor` 接口。**它既作为`Subscriber` ，因为它接收消息，又作为`Publisher` ，因为它处理这些消息并发送它们以供进一步处理。

## 3。发布和消费消息

假设我们想要创建一个简单的`Flow,` ，其中有一个`Publisher` 发布消息，一个简单的`Subscriber`在消息到达时消费消息——一次一个。

让我们创建一个`EndSubscriber` 类。我们需要实现`Subscriber` 接口。接下来，我们将覆盖所需的方法。

在处理开始之前调用`onSubscribe()` 方法。`Subscription` 的实例作为参数传递。这个类用于控制`Subscriber`和`Publisher:`之间的消息流

```java
public class EndSubscriber<T> implements Subscriber<T> {
    private Subscription subscription;
    public List<T> consumedElements = new LinkedList<>();

    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }
}
```

我们还初始化了一个空的`consumedElements` 的`List`，它将在测试中使用。

现在，我们需要从`Subscriber` 接口实现剩余的方法。这里的主要方法是 on next()——每当`Publisher`发布新消息时都会调用这个方法:

```java
@Override
public void onNext(T item) {
    System.out.println("Got : " + item);
    consumedElements.add(item);
    subscription.request(1);
}
```

注意，当我们在`onSubscribe()` 方法中开始订阅时，以及当我们处理一条消息时，我们需要在`Subscription` 上调用`request()` 方法，以表示当前的`Subscriber` 已经准备好消费更多的消息。

最后，我们需要实现`onError()`——每当处理过程中出现异常时都会调用它，当`Publisher`关闭时也会调用`onComplete() –` :

```java
@Override
public void onError(Throwable t) {
    t.printStackTrace();
}

@Override
public void onComplete() {
    System.out.println("Done");
}
```

让我们为处理`Flow.` 编写一个测试，我们将使用`SubmissionPublisher` 类——来自`java.util.concurrent`的一个构造——它实现了`Publisher` 接口。

我们将向`Publisher`提交`N`个元素——我们的`EndSubscriber` 将会收到:

```java
@Test
public void whenSubscribeToIt_thenShouldConsumeAll() 
  throws InterruptedException {

    // given
    SubmissionPublisher<String> publisher = new SubmissionPublisher<>();
    EndSubscriber<String> subscriber = new EndSubscriber<>();
    publisher.subscribe(subscriber);
    List<String> items = List.of("1", "x", "2", "x", "3", "x");

    // when
    assertThat(publisher.getNumberOfSubscribers()).isEqualTo(1);
    items.forEach(publisher::submit);
    publisher.close();

    // then
     await().atMost(1000, TimeUnit.MILLISECONDS)
       .until(
         () -> assertThat(subscriber.consumedElements)
         .containsExactlyElementsOf(items)
     );
}
```

注意，我们在`EndSubscriber.`的实例上调用`close()` 方法，它将在给定的`Publisher.` 的每个`Subscriber`上调用下面的`onComplete()` 回调

运行该程序将产生以下输出:

```java
Got : 1
Got : x
Got : 2
Got : x
Got : 3
Got : x
Done
```

## 4。消息的转换

假设我们想要在一个`Publisher` 和一个`Subscriber`之间建立类似的逻辑，但是也要应用一些转换。

我们将创建实现`Processor` 并扩展`SubmissionPublisher –` 的`TransformProcessor` 类，因为这将是`P` `ublisher`和 S `ubscriber.`

我们将传入一个将输入转换成输出的`Function`:

```java
public class TransformProcessor<T, R> 
  extends SubmissionPublisher<R> 
  implements Flow.Processor<T, R> {

    private Function<T, R> function;
    private Flow.Subscription subscription;

    public TransformProcessor(Function<T, R> function) {
        super();
        this.function = function;
    }

    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }

    @Override
    public void onNext(T item) {
        submit(function.apply(item));
        subscription.request(1);
    }

    @Override
    public void onError(Throwable t) {
        t.printStackTrace();
    }

    @Override
    public void onComplete() {
        close();
    }
}
```

现在让我们**用一个处理流程编写一个快速测试**，其中`Publisher` 正在发布`String` 元素。

我们的`TransformProcessor` 将把`String`解析为`Integer`——这意味着这里需要进行转换:

```java
@Test
public void whenSubscribeAndTransformElements_thenShouldConsumeAll()
  throws InterruptedException {

    // given
    SubmissionPublisher<String> publisher = new SubmissionPublisher<>();
    TransformProcessor<String, Integer> transformProcessor 
      = new TransformProcessor<>(Integer::parseInt);
    EndSubscriber<Integer> subscriber = new EndSubscriber<>();
    List<String> items = List.of("1", "2", "3");
    List<Integer> expectedResult = List.of(1, 2, 3);

    // when
    publisher.subscribe(transformProcessor);
    transformProcessor.subscribe(subscriber);
    items.forEach(publisher::submit);
    publisher.close();

    // then
     await().atMost(1000, TimeUnit.MILLISECONDS)
       .until(() -> 
         assertThat(subscriber.consumedElements)
         .containsExactlyElementsOf(expectedResult)
     );
}
```

注意，调用基类`Publisher` 上的`close()` 方法将导致调用`TransformProcessor`上的`onComplete()` 方法。

请记住，处理链中的所有发布者都需要以这种方式关闭。

## 5。使用`Subscription` 控制消息需求

假设我们只想使用订阅中的第一个元素，应用一些逻辑并完成处理。我们可以使用`request()` 方法来实现这一点。

让我们修改我们的`EndSubscriber` ,只消耗 N 条消息。我们将传递这个数字作为`howMuchMessagesConsume` 构造函数的参数:

```java
public class EndSubscriber<T> implements Subscriber<T> {

    private AtomicInteger howMuchMessagesConsume;
    private Subscription subscription;
    public List<T> consumedElements = new LinkedList<>();

    public EndSubscriber(Integer howMuchMessagesConsume) {
        this.howMuchMessagesConsume 
          = new AtomicInteger(howMuchMessagesConsume);
    }

    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }

    @Override
    public void onNext(T item) {
        howMuchMessagesConsume.decrementAndGet();
        System.out.println("Got : " + item);
        consumedElements.add(item);
        if (howMuchMessagesConsume.get() > 0) {
            subscription.request(1);
        }
    }
    //...

}
```

只要我们愿意，我们可以请求元素。

让我们编写一个测试，其中我们只想使用给定`Subscription:`中的一个元素

```java
@Test
public void whenRequestForOnlyOneElement_thenShouldConsumeOne()
  throws InterruptedException {

    // given
    SubmissionPublisher<String> publisher = new SubmissionPublisher<>();
    EndSubscriber<String> subscriber = new EndSubscriber<>(1);
    publisher.subscribe(subscriber);
    List<String> items = List.of("1", "x", "2", "x", "3", "x");
    List<String> expected = List.of("1");

    // when
    assertThat(publisher.getNumberOfSubscribers()).isEqualTo(1);
    items.forEach(publisher::submit);
    publisher.close();

    // then
    await().atMost(1000, TimeUnit.MILLISECONDS)
      .until(() -> 
        assertThat(subscriber.consumedElements)
       .containsExactlyElementsOf(expected)
    );
}
```

虽然`publisher` 发布了六个元素，但是我们的`EndSubscriber` 将只消耗一个元素，因为它发出了只处理这一个元素的信号。

通过在`Subscription,` 上使用`request()` 方法，我们可以实现一个更复杂的背压机制来控制消息消费的速度。

## 6。结论

在本文中，我们看了一下 Java 9 反应流。

我们看到了如何创建由一个`Publisher` 和一个`Subscriber.` 组成的处理`Flow` ，我们使用`Processors`创建了一个更复杂的元素转换处理流程。

最后，我们使用`Subscription` 来控制`Subscriber.` 对元素的需求

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221129021658/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。