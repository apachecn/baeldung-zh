# 带有 Ratpack 的反应流 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-reactive-streams-api>

## 1.介绍

Ratpack 是构建在 [Netty](/web/20220525133756/https://www.baeldung.com/netty) 引擎之上的框架，它允许我们快速构建 HTTP 应用程序。我们已经在[之前的文章](/web/20220525133756/https://www.baeldung.com/?s=ratpack)中介绍了它的基本用法。**这一次，我们将展示如何使用其流式 API 来实现反应式应用**。

## 2.快速回顾反应流

在进入实际实现之前，让我们先快速回顾一下什么是反应式应用程序。根据[原作者](https://web.archive.org/web/20220525133756/https://github.com/reactivemanifesto/reactivemanifesto)的说法，此类应用必须具备以下属性:

*   应答的
*   弹回的
*   弹性的
*   消息驱动

那么，反应流如何帮助我们实现这些属性呢？嗯，在这个上下文中，`message-driven`并不一定意味着使用消息中间件。相反，解决这一点真正需要的是**异步请求处理和对非阻塞背压的支持**。

Ratpack 反应式支持使用 JVM 的反应式流 API 标准作为其实现的基础。因此，它允许与 Project Reactor 和 RxJava 等其他兼容框架的互操作性。

## 3.使用 Ratpacks '类

**Ratpack 的`[Streams](https://web.archive.org/web/20220525133756/https://ratpack.io/manual/current/api/ratpack/stream/Streams.html)`类提供了几个实用方法来创建`Publisher`实例，然后我们可以用它们来创建数据处理管道。**

一个很好的起点是`publish()`方法，我们可以用它从任何`Iterable`创建一个`Publisher`:

```java
Publisher<String> pub = Streams.publish(Arrays.asList("hello", "hello again"));
LoggingSubscriber<String> sub = new LoggingSubscriber<String>();
pub.subscribe(sub);
sub.block(); 
```

这里，`LoggingSubscriber`是`Subscriber`接口的一个测试实现，它只记录发布者发出的每个对象。它还包括一个助手方法`block()`，顾名思义，它阻塞调用者，直到发布者发出它的所有对象或者产生一个错误。

运行测试用例，我们将看到预期的事件序列:

```java
onSubscribe: sub=7311908
onNext: sub=7311908, value=hello
onNext: sub=7311908, value=hello again
onComplete: sub=7311908 
```

另一个有用的方法是`yield()`。它有一个单一的`Function`参数，该参数接收一个`YieldRequest`对象并返回下一个要发出的对象:

```java
@Test
public void whenYield_thenSuccess() {

    Publisher<String> pub = Streams.yield((t) -> {
        return t.getRequestNum() < 5 ? "hello" : null;
    });

    LoggingSubscriber<String> sub = new LoggingSubscriber<String>();
    pub.subscribe(sub);
    sub.block();
    assertEquals(5, sub.getReceived());
} 
```

`YieldRequest`参数允许我们使用它的`getRequestNum()`方法，根据目前发出的对象数量实现逻辑。在我们的例子中，我们使用这个信息来定义结束条件，我们通过返回一个`null`值来发出信号。

现在，让我们看看如何为周期性事件创建一个`Publisher`:

```java
@Test
public void whenPeriodic_thenSuccess() {
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
    Publisher<String> pub = Streams.periodically(executor, Duration.ofSeconds(1), (t) -> {
        return t < 5 ? String.format("hello %d",t): null; 
    });

    LoggingSubscriber<String> sub = new LoggingSubscriber<String>();
    pub.subscribe(sub);
    sub.block();
    assertEquals(5, sub.getReceived());
} 
```

返回的发布者使用`ScheduledExecutorService`周期性地调用生产者函数，直到它返回一个`null`值。producer 函数接收一个与已经发出的对象数量相对应的整数值，我们用它来终止流。

## 4.使用`TransformablePublisher`

仔细看看`Streams’` 方法，我们可以看到它们通常会返回一个 [`TransformablePublisher`](https://web.archive.org/web/20220525133756/https://ratpack.io/manual/current/api/ratpack/stream/TransformablePublisher.html) 。这个接口用几个实用方法扩展了`Publisher`，很像我们在 Project Reactor 的`Flux` 和`Mono`、**中发现的，使得从单个步骤**创建复杂的处理管道变得更加容易。

作为一个例子，让我们使用`map`方法将一个整数序列转换成字符串:

```java
@Test
public void whenMap_thenSuccess() throws Exception {
    TransformablePublisher<String> pub = Streams.yield( t -> {
        return t.getRequestNum() < 5 ? t.getRequestNum() : null;
      })
      .map(v -> String.format("item %d", v));

    ExecResult<List<String>> result = ExecHarness.yieldSingle((c) -> pub.toList());
    assertTrue("should succeed", result.isSuccess());
    assertEquals("should have 5 items",5,result.getValue().size());
} 
```

这里，实际的执行发生在由测试实用程序类 [`ExecHarness`](https://web.archive.org/web/20220525133756/https://ratpack.io/manual/current/api/ratpack/test/exec/ExecHarness.html) 管理的线程池中。由于`yieldSingle()`期待一个`[Promise](/web/20220525133756/https://www.baeldung.com/ratpack-http-client#3-ratpack-promises)`，我们使用`toList()`来修改我们的发布者。这个方法收集订阅者产生的所有结果，并将它们存储在一个`List`中。

正如文档中所述，我们在使用这种方法时必须小心。把它应用到一个无边界的发布者身上会很快让 JVM 耗尽内存！为了避免这种情况，我们应该将它的使用限制在单元测试中。

除了`map()`，`TransformablePublisher`还有几个有用的操作符:

*   `filter()`:根据 a `Predicate`过滤上游对象
*   `take()`:仅发射来自上游`Publisher`的第一个`n`物体
*   `wiretap()`:增加一个观察点，当数据和事件流经管道时，我们可以在此观察它们
*   `reduce()`:将上游对象缩减为单个值
*   `transform()`:在流中注入一个常规的`Publisher`

## 5.使用不符合`Publishers`的`buffer()`

**在某些场景中，我们必须处理一个`Publisher`，它发送给订阅者的条目比请求的多**。为了解决这些情况，Ratpack 的流提供了一个`buffer()`方法，将这些额外的项目保存在内存中，直到用户使用它们。

为了说明这是如何工作的，让我们创建一个简单的不兼容的`Publisher`，它忽略了所请求的项目的数量。相反，它将总是比请求的多生产至少 5 个项目:

```java
private class NonCompliantPublisher implements Publisher<Integer> {

    @Override
    public void subscribe(Subscriber<? super Integer> subscriber) {
        log.info("subscribe");
        subscriber.onSubscribe(new NonCompliantSubscription(subscriber));
    }

    private class NonCompliantSubscription implements Subscription {
        private Subscriber<? super Integer> subscriber;
        private int recurseLevel = 0;

        public NonCompliantSubscription(Subscriber<? super Integer> subscriber) {
            this.subscriber = subscriber;
        }

        @Override
        public void request(long n) {
            log.info("request: n={}", n);
            if ( recurseLevel > 0 ) {
               return;
            }
            recurseLevel++;
            for (int i = 0 ; i < (n + 5) ; i ++ ) {
                subscriber.onNext(i);
            }
            subscriber.onComplete();
        }

        @Override
        public void cancel() {
        }
    }
} 
```

首先，让我们使用我们的`LoggingSubscriber. `测试这个发布者，我们将使用`take()`操作符，这样它将只接收第一个项目

```java
@Test
public void whenNonCompliantPublisherWithoutBuffer_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new NonCompliantPublisher())
      .wiretap(new LoggingAction(""))
      .take(1);

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
} 
```

**运行这个测试，我们看到尽管收到了一个`cancel()`请求，我们不符合要求的出版商继续生产新的项目:**

```java
RatpackStreamsUnitTest - : event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=583189145, value=0
RatpackStreamsUnitTest - : event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
NonCompliantPublisher - request: n=1
RatpackStreamsUnitTest - : event=StreamEvent[CancelEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=583189145
RatpackStreamsUnitTest - : event=StreamEvent[DataEvent{subscriptionId=0, data=1}]
... more expurious data event
RatpackStreamsUnitTest - : event=StreamEvent[CompletionEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=583189145
```

现在，让我们在这个流程中添加一个`buffer()`步骤。我们将在它之前添加两个`wiretap`步骤来记录事件，因此它的效果变得更加明显:

```java
@Test
public void whenNonCompliantPublisherWithBuffer_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new NonCompliantPublisher())
      .wiretap(new LoggingAction("before buffer"))
      .buffer()
      .wiretap(new LoggingAction("after buffer"))
      .take(1);

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
} 
```

这一次，运行这段代码会产生一个不同的日志序列:

```java
LoggingSubscriber - onSubscribe: sub=675852144
RatpackStreamsUnitTest - after buffer: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
NonCompliantPublisher - subscribe
RatpackStreamsUnitTest - before buffer: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
NonCompliantPublisher - request: n=1
RatpackStreamsUnitTest - before buffer: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
... more data events
RatpackStreamsUnitTest - before buffer: event=StreamEvent[CompletionEvent{subscriptionId=0}]
RatpackStreamsUnitTest - after buffer: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=675852144, value=0
RatpackStreamsUnitTest - after buffer: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
RatpackStreamsUnitTest - after buffer: event=StreamEvent[CancelEvent{subscriptionId=0}]
RatpackStreamsUnitTest - before buffer: event=StreamEvent[CancelEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=67585214
```

“before buffer”消息显示，我们不兼容的发布者能够在第一次调用`request`后发送所有值。**尽管如此，下游值仍然被一个接一个地发送**，考虑到`LoggingSubscriber`请求的数量。

## 6.对慢速订阅者使用`batch()`

另一个可能降低应用程序吞吐量的场景是当下游订户请求少量数据时。我们的`LoggingSubscriber`就是一个很好的例子:它一次只请求一个项目。

**在现实世界的应用中，这会导致大量的上下文切换，这会损害整体性能。**更好的方法是一次请求更多的项目。`batch()`方法允许上游发布者使用更有效的请求大小，同时允许下游订阅者使用更小的请求大小。

让我们看看这在实践中是如何工作的。和以前一样，我们将从一个没有`batch`的流开始:

```java
@Test
public void whenCompliantPublisherWithoutBatch_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new CompliantPublisher(10))
      .wiretap(new LoggingAction(""));

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
} 
```

这里，`CompliantPublisher`只是一个测试`Publisher`,它产生整数，但不包括传递给构造函数的值。让我们运行它来看看非批处理行为:

```java
CompliantPublisher - subscribe
LoggingSubscriber - onSubscribe: sub=-779393331
RatpackStreamsUnitTest - : event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
CompliantPublisher - request: requested=1, available=10
RatpackStreamsUnitTest - : event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=-779393331, value=0
... more data events omitted
CompliantPublisher - request: requested=1, available=1
RatpackStreamsUnitTest - : event=StreamEvent[CompletionEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=-779393331 
```

**输出显示生产者逐个发出数值**。现在，让我们将步骤`batch()`添加到我们的管道中，这样上游发布者一次最多可以生成五个项目:

```java
@Test
public void whenCompliantPublisherWithBatch_thenSuccess() throws Exception {

    TransformablePublisher<Integer> pub = Streams.transformable(new CompliantPublisher(10))
      .wiretap(new LoggingAction("before batch"))
      .batch(5, Action.noop())
      .wiretap(new LoggingAction("after batch"));

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
} 
```

`batch()` 方法有两个参数:每个`request()`调用请求的项目数和一个用于处理丢弃项目(即请求但未消费的项目)的`Action`。如果出现错误或下游用户调用`cancel()`，就会出现这种情况。让我们看看生成的执行日志:

```java
LoggingSubscriber - onSubscribe: sub=-1936924690
RatpackStreamsUnitTest - after batch: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
CompliantPublisher - subscribe
RatpackStreamsUnitTest - before batch: event=StreamEvent[RequestEvent{requestAmount=5, subscriptionId=0}]
CompliantPublisher - request: requested=5, available=10
RatpackStreamsUnitTest - before batch: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
... first batch data events omitted
RatpackStreamsUnitTest - before batch: event=StreamEvent[RequestEvent{requestAmount=5, subscriptionId=0}]
CompliantPublisher - request: requested=5, available=6
RatpackStreamsUnitTest - before batch: event=StreamEvent[DataEvent{subscriptionId=0, data=5}]
... second batch data events omitted
RatpackStreamsUnitTest - before batch: event=StreamEvent[RequestEvent{requestAmount=5, subscriptionId=0}]
CompliantPublisher - request: requested=5, available=1
RatpackStreamsUnitTest - before batch: event=StreamEvent[CompletionEvent{subscriptionId=0}]
RatpackStreamsUnitTest - after batch: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=-1936924690, value=0
RatpackStreamsUnitTest - after batch: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
RatpackStreamsUnitTest - after batch: event=StreamEvent[DataEvent{subscriptionId=0, data=1}]
... downstream data events omitted
LoggingSubscriber - onComplete: sub=-1936924690 
```

**我们可以看到，现在发布者每次都会收到五个项目的请求**。注意，在这个测试场景中，我们甚至在登录订阅者获得第一项之前就看到了对生产者的`two`请求。原因是，在这个测试场景中，我们有一个单线程执行，所以`batch`()继续缓冲项目，直到它得到`onComplete()`信号。

## 7.在 Web 应用程序中使用流

Ratpack 支持将反应流与其异步 web 框架结合使用。

### 7.1.接收数据流

对于传入的数据，通过处理程序的`Context`可用的`Request` 对象提供了`getBodyStream()`方法，该方法返回一个`ByteBuf`对象的`TransformablePublisher`。

从这个发布者，我们可以建立我们的处理管道:

```java
@Bean
public Action<Chain> uploadFile() {

    return chain -> chain.post("upload", ctx -> {
        TransformablePublisher<? extends ByteBuf> pub = ctx.getRequest().getBodyStream();
        pub.subscribe(new Subscriber<ByteBuf>() {
            private Subscription sub;
            @Override
            public void onSubscribe(Subscription sub) {
                this.sub = sub;
                sub.request(1);
            }

            @Override
            public void onNext(ByteBuf t) {
                try {
                    ... do something useful with received data
                    sub.request(1);
                }
                finally {
                    // DO NOT FORGET to RELEASE !
                    t.release();
                }
            }

            @Override
            public void onError(Throwable t) {
                ctx.getResponse().status(500);
            }

            @Override
            public void onComplete() {
                ctx.getResponse().status(202);
            }
        }); 
    });
} 
```

在实现订阅者时，有几个细节需要考虑。首先，我们必须确保在某个时候调用了`ByteBuf`的`release()`方法。**否则会导致内存泄漏**。其次，任何异步处理必须只使用 Ratpack 的原语。这些包括`Promise`、`Blocking,`和类似的构造。

### 7.2.发送数据流

发送数据流最直接的方法是使用`Response.sendStream()`。该方法接受一个`ByteBuf`发布者参数，并将数据发送到客户端，根据需要应用反压力以避免溢出:

```java
@Bean
public Action<Chain> download() {
    return chain -> chain.get("download", ctx -> {
        ctx.getResponse().sendStream(new RandomBytesPublisher(1024,512));
    });
} 
```

虽然很简单，但使用这种方法也有不好的一面:**它不会自己设置任何头，包括`Content-Length`，这对客户来说可能是个问题:**

```java
$ curl -v --output data.bin http://localhost:5050/download
... request messages omitted
< HTTP/1.1 200 OK
< transfer-encoding: chunked
... download progress messages omitted 
```

**或者，更好的方法是使用句柄的`Context` `render()`方法，传递一个`ResponseChunks`对象**。在这种情况下，响应将使用“ [chunked](https://web.archive.org/web/20220525133756/https://en.wikipedia.org/wiki/Chunked_transfer_encoding) ”传输编码方法。创建 [`ResponseChunks`](https://web.archive.org/web/20220525133756/https://ratpack.io/manual/current/api/ratpack/http/ResponseChunks.html) 实例最直接的方法是通过该类中的一个静态方法:

```java
@Bean
public Action<Chain> downloadChunks() {
    return chain -> chain.get("downloadChunks", ctx -> {
        ctx.render(ResponseChunks.bufferChunks("application/octetstream",
          new RandomBytesPublisher(1024,512)));
    });
} 
```

经过这一更改，响应现在包括了`content-type`头:

```java
$ curl -v --output data.bin http://localhost:5050/downloadChunks
... request messages omitted
< HTTP/1.1 200 OK
< transfer-encoding: chunked
< content-type: application/octetstream
<
... progress messages omitted
```

### 7.3.使用服务器端事件

对服务器端事件(SSE)的支持也使用了`render()`方法。然而，在这种情况下，我们使用 [`ServerSentEvents`](https://web.archive.org/web/20220525133756/https://ratpack.io/manual/current/api/ratpack/sse/ServerSentEvents.html) 将来自`Producer`的项目改编为`Event` 对象，这些对象包含一些元数据以及事件有效载荷:

```java
@Bean
public Action<Chain> quotes() {
    ServerSentEvents sse = ServerSentEvents.serverSentEvents(quotesService.newTicker(), (evt) -> {
        evt
          .id(Long.toString(idSeq.incrementAndGet()))
          .event("quote")
          .data( q -> q.toString());
    });

    return chain -> chain.get("quotes", ctx -> ctx.render(sse));
} 
```

在这里，`QuotesService`只是一个示例服务，它创建了一个`Publisher`,以固定的时间间隔生成随机报价。第二个参数是一个准备发送事件的函数。这包括添加一个`id`、一个事件类型和有效负载本身。

我们可以使用`curl`来测试这个方法，产生一个显示随机引用序列的输出，以及事件元数据:

```java
$ curl -v http://localhost:5050/quotes
... request messages omitted
< HTTP/1.1 200 OK
< content-type: text/event-stream;charset=UTF-8
< transfer-encoding: chunked
... other response headers omitted
id: 10
event: quote
data: Quote [ts=2021-10-11T01:20:52.081Z, symbol=ORCL, value=53.0]

... more quotes
```

### 7.4.广播 Websocket 数据

**我们可以使用 [`Websockets.websocketBroadcast()`](https://web.archive.org/web/20220525133756/https://ratpack.io/manual/current/api/ratpack/websocket/WebSockets.html#websocketBroadcast(ratpack.handling.Context,org.reactivestreams.Publisher)) :** 将数据从任何`Publisher`传输到 WebSocket 连接

```java
@Bean
public Action<Chain> quotesWS() {
    Publisher<String> pub = Streams.transformable(quotesService.newTicker())
      .map(Quote::toString);
    return chain -> chain.get("quotes-ws", ctx -> WebSockets.websocketBroadcast(ctx, pub));
} 
```

这里，我们使用我们之前见过的相同的`QuotesService`作为向客户广播报价的事件源。让我们再次使用`curl`来模拟一个 WebSocket 客户端:

```java
$ curl --include -v \
     --no-buffer \
     --header "Connection: Upgrade" \
     --header "Upgrade: websocket" \
     --header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" \
     --header "Sec-WebSocket-Version: 13" \
     http://localhost:5050/quotes-ws
... request messages omitted
< HTTP/1.1 101 Switching Protocols
HTTP/1.1 101 Switching Protocols
< upgrade: websocket
upgrade: websocket
< connection: upgrade
connection: upgrade
< sec-websocket-accept: qGEgH3En71di5rrssAZTmtRTyFk=
sec-websocket-accept: qGEgH3En71di5rrssAZTmtRTyFk=

<
<Quote [ts=2021-10-11T01:39:42.915Z, symbol=ORCL, value=63.0]
... more quotes omitted 
```

## 8.结论

在本文中，我们探讨了 Ratpack 对反应流的支持，以及如何在不同的场景中应用它。

像往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525133756/https://github.com/eugenp/tutorials/tree/master/ratpack)