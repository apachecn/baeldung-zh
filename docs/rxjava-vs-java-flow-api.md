# RxJava API 和 Java 9 Flow API 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-vs-java-flow-api>

## 1.介绍

Java 流 API 是在 Java 9 中作为反应式流规范的实现而引入的。

在本教程中，我们将首先研究反应流。然后，我们将了解它与 RxJava 和 Flow API 的关系。

## 2.什么是反应流？

[反应式宣言](https://web.archive.org/web/20220625232110/https://www.reactive-streams.org/)引入了反应式流，为具有非阻塞背压的异步流处理指定了标准。

反应流规范的范围是定义**一组最小的接口**来实现这些目标:

*   `[org.reactivestreams.Publisher](https://web.archive.org/web/20220625232110/http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Publisher.html)`是一个数据提供者，它根据用户的需求向用户发布数据
*   [`org.reactivestreams.Subscriber`](https://web.archive.org/web/20220625232110/http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Subscriber.html) 是数据的消费者——它可以在订阅发布者后接收数据
*   `[org.reactivestreams.Subscription](https://web.archive.org/web/20220625232110/http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Subscription.html)`是在发布者接受订阅者时创建的
*   `[org.reactivestreams.Processor](https://web.archive.org/web/20220625232110/http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Processor.html)`既是订阅者又是发布者——它订阅发布者，处理数据，然后将处理后的数据传递给订阅者

流程 API 源于规范。RxJava 领先于它，但是从 2.0 开始，RxJava 也支持这个规范。

我们将深入探讨这两者，但首先，让我们看一个实际的用例。

## 3.用例

在本教程中，我们将使用一个直播视频服务作为用例。

与点播视频流相反，直播视频流不依赖于消费者。因此，服务器以自己的速度发布流，消费者有责任适应。

在最简单的形式中，我们的模型由视频流发布者和作为订阅者的视频播放器组成。

让我们实现 `VideoFrame`作为我们的数据项:

```java
public class VideoFrame {
    private long number;
    // additional data fields

    // constructor, getters, setters
}
```

然后让我们一个接一个地检查我们的 Flow API 和 RxJava 实现。

## 4.使用流程 API 实施

JDK 9 中的流 API 对应于反应流规范。使用 Flow API，如果应用程序最初请求 N 个项目，那么发布者最多向订阅者推送 N 个项目。

流程 API 接口都在`[java.util.concurrent.Flow](https://web.archive.org/web/20220625232110/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Flow.html)`接口中。它们在语义上等同于它们各自对应的反应流。

让我们将`VideoStreamServer`实现为`VideoFrame`的发布者。

```java
public class VideoStreamServer extends SubmissionPublisher<VideoFrame> {

    public VideoStreamServer() {
        super(Executors.newSingleThreadExecutor(), 5);
    }
}
```

**我们从`SubmissionPublisher`扩展了我们的`VideoStreamServer`，而不是直接实现`Flow::Publisher.`** `SubmissionPublisher` 是`Flow::Publisher` 的 JDK 实现，用于与订户的异步通信，所以它让我们的`VideoStreamServer`按照自己的节奏发射。

此外，它还有助于背压和缓冲处理，因为当调用`SubmissionPublisher::subscribe`时，它会创建一个`BufferedSubscription`的实例，然后将新的订阅添加到它的订阅链中。`BufferedSubscription`最多可以缓冲`SubmissionPublisher#maxBufferCapacity`的已发放物品。

现在让我们定义`VideoPlayer,` ，它消耗了`VideoFrame.` 的流，因此它必须实现`Flow::Subscriber`。

```java
public class VideoPlayer implements Flow.Subscriber<VideoFrame> {

    Flow.Subscription subscription = null;

    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }

    @Override
    public void onNext(VideoFrame item) {
        log.info("play #{}" , item.getNumber());
        subscription.request(1);
    }

    @Override
    public void onError(Throwable throwable) {
        log.error("There is an error in video streaming:{}" , throwable.getMessage());

    }

    @Override
    public void onComplete() {
        log.error("Video has ended");
    }
}
```

`VideoPlayer`订阅`VideoStreamServer,` ，订阅成功后`VideoPlayer` :: `onSubscribe`方法被调用，请求一帧。`VideoPlayer`::下一次接收帧并请求新的帧。请求帧的数量取决于用例及`Subscriber`实施。

最后，让我们把事情放在一起:

```java
VideoStreamServer streamServer = new VideoStreamServer();
streamServer.subscribe(new VideoPlayer());

// submit video frames

ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
AtomicLong frameNumber = new AtomicLong();
executor.scheduleWithFixedDelay(() -> {
    streamServer.offer(new VideoFrame(frameNumber.getAndIncrement()), (subscriber, videoFrame) -> {
        subscriber.onError(new RuntimeException("Frame#" + videoFrame.getNumber()
        + " droped because of backpressure"));
        return true;
    });
}, 0, 1, TimeUnit.MILLISECONDS);

sleep(1000);
```

## 5.用 RxJava 实现

[RxJava](/web/20220625232110/https://www.baeldung.com/rx-java) 是[react vex](https://web.archive.org/web/20220625232110/http://reactivex.io/)的 Java 实现。ReactiveX(或 Reactive Extensions)项目旨在提供一个反应式编程概念。它是 [观察者模式](/web/20220625232110/https://www.baeldung.com/java-observer-pattern)、迭代器模式和函数式编程的组合。T12


RxJava 的最新主要版本是 3.x. RxJava 从 2.x 版本开始就支持带有`Flowable`基类的反应流，但是它比带有几个基类的反应流更重要，比如`Flowable`、`Observable`、`Single`、`Completable.`

[`Flowable`](/web/20220625232110/https://www.baeldung.com/rxjava-2-flowable) 作为反应性流的柔顺元件是一个有背压处理的 0 到 N 项的流。`Flowable`从反应流延伸出`Publisher`。因此，许多 RxJava 操作符直接接受`Publisher`，并允许与其他反应流实现直接互操作。

现在，让我们制作一个视频流生成器，它是一个无限的惰性流:

```java
Stream<VideoFrame> videoStream = Stream.iterate(new VideoFrame(0), videoFrame -> {
    // sleep for 1ms;
    return new VideoFrame(videoFrame.getNumber() + 1);
});
```

然后我们定义一个`Flowable`实例在一个单独的线程上生成帧:

```java
Flowable
  .fromStream(videoStream)
  .subscribeOn(Schedulers.from(Executors.newSingleThreadExecutor()))
```

需要注意的是，无限流对我们来说已经足够了，但是如果我们需要一种更灵活的方式来生成我们的流，那么`Flowable.create`是一个不错的选择。

```java
Flowable
  .create(new FlowableOnSubscribe<VideoFrame>() {
      AtomicLong frame = new AtomicLong();
      @Override
      public void subscribe(@NonNull FlowableEmitter<VideoFrame> emitter) {
          while (true) {
              emitter.onNext(new VideoFrame(frame.incrementAndGet()));
              //sleep for 1 ms to simualte delay
          }
      }
  }, /* Set Backpressure Strategy Here */)
```

然后，在下一步，VideoPlayer 订阅这个流，并在一个单独的线程上观察项目。

```java
videoFlowable
  .observeOn(Schedulers.from(Executors.newSingleThreadExecutor()))
  .subscribe(item -> {
      log.info("play #" + item.getNumber());
      // sleep for 30 ms to simualate frame display
  });
```

最后，我们将配置[背压](/web/20220625232110/https://www.baeldung.com/rxjava-backpressure)的策略。如果我们想在帧丢失的情况下停止视频，因此我们必须在缓冲区已满时使用`BackpressureOverflowStrategy::ERROR`。

```java
Flowable
  .fromStream(videoStream)
  .subscribeOn(Schedulers.from(Executors.newSingleThreadExecutor()))
  .onBackpressureBuffer(5, null, BackpressureOverflowStrategy.ERROR)
  .observeOn(Schedulers.from(Executors.newSingleThreadExecutor()))
  .subscribe(item -> {
       log.info("play #" + item.getNumber()); 
       // sleep for 30 ms to simualate frame display 
  });
```

## 6.RxJava 和 Flow API 的比较

即使在这两个简单的实现中，我们也可以看到 RxJava 的 API 是多么的丰富，尤其是缓冲区管理、错误处理和背压策略。它用流畅的 API 给了我们更多的选择和更少的代码行。现在让我们考虑更复杂的情况。

假设我们的播放器没有编解码器就不能显示视频帧。因此，对于 Flow API，我们需要实现一个`Processor`来模拟编解码器，并位于服务器和播放器之间。有了 RxJava，我们可以用 [`Flowable::flatMap` 或者 `Flowable::map`](/web/20220625232110/https://www.baeldung.com/java-difference-map-and-flatmap) 来做。

或者让我们想象一下，我们的播放器也要广播现场翻译音频，所以我们必须组合来自不同发行商的视频和音频流。使用 RxJava，我们可以使用`[Flowable::combineLatest](/web/20220625232110/https://www.baeldung.com/rxjava-combine-observables),` 但是使用 Flow API，这并不是一件容易的事情。

尽管可以编写一个定制的`Processor`来订阅这两个流，并将合并后的数据发送给我们的`VideoPlayer.`,但是实现起来令人头疼。

## 7.为什么是 Flow API？

此时，我们可能会有一个疑问，Flow API 背后的哲学是什么？

如果我们在 JDK 中搜索流 API 用法，我们可以在`java.net.http`和`jdk.internal.net.http.` 中找到一些东西

此外，我们可以在[反应堆项目](/web/20220625232110/https://www.baeldung.com/reactor-core)或反应流包中找到适配器。例如，`org.reactivestreams.FlowAdapters` 拥有将流 API 接口转换成反应流接口的方法，反之亦然。因此，它有助于流 API 和具有反应式流支持的库之间的互操作性 T4。

所有这些事实都有助于我们理解 Flow API 的用途:它是在 JDK 创建的一组反应式规范接口，不依赖于第三方。此外，Java 期望 Flow API 被接受为反应式规范的标准接口，并被用于 JDK 或其他基于 Java 的库，这些库实现了中间件和实用程序的反应式规范。

## 8.结论

在本教程中，我们介绍了反应式流规范、流 API 和 RxJava。

此外，我们已经看到了一个用于实时视频流的 Flow API 和 RxJava 实现的实际例子。

但是这里并没有涵盖 Flow API 和 RxJava 的所有方面，如`Flow::Processor`、`Flowable::map`和`Flowable::flatMap`或背压策略。

和往常一样，你可以在 GitHub 上找到教程的完整代码[。](https://web.archive.org/web/20220625232110/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)