# Jetty 流 HTTP 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jetty-reactivestreams-http-client>

## 1。概述

在本教程中，我们将学习如何使用来自 Jetty 的[反应式 HTTP 客户端。我们将通过创建小的测试用例来展示它在不同反应库上的用法。](https://web.archive.org/web/20221205170624/https://github.com/jetty-project/jetty-reactive-httpclient)

## 2。什么是无功`HttpClient`？

Jetty 的 [`HttpClient`](https://web.archive.org/web/20221205170624/https://www.eclipse.org/jetty/documentation/current/http-client.html) 允许我们阻塞 HTTP 请求。然而，当我们处理反应式 API 时，我们不能使用标准的 HTTP 客户端。为了填补这个空白，Jetty 创建了一个包装器来包装`HttpClient`API，这样它也支持`ReactiveStreams` API。

**被动`HttpClient`用于通过 HTTP 调用消费或产生数据流。**

我们在这里要演示的例子将有一个反应式 HTTP 客户端，它将使用不同的反应式库与 Jetty 服务器通信。我们还将讨论由 Reactive `HttpClient`提供的请求和响应事件。

我们建议阅读我们关于 [Project Reactor](/web/20221205170624/https://www.baeldung.com/reactor-core) 、 [RxJava](/web/20221205170624/https://www.baeldung.com/rx-java) 和 [Spring WebFlux](/web/20221205170624/https://www.baeldung.com/spring-webflux) 的文章，以便更好地理解反应式编程概念及其术语。

## 3。Maven 依赖关系

让我们从将[反应流](https://web.archive.org/web/20221205170624/https://search.maven.org/search?q=g:org.reactivestreams%20AND%20a:reactive-streams)、[项目反应器](https://web.archive.org/web/20221205170624/https://search.maven.org/search?q=g:org.springframework%20AND%20a:spring-webflux)、 [RxJava](https://web.archive.org/web/20221205170624/https://search.maven.org/search?q=g:io.reactivex.rxjava2%20AND%20a:rxjava) 、 [Spring WebFlux](https://web.archive.org/web/20221205170624/https://search.maven.org/search?q=g:io.projectreactor%20AND%20a:reactor-core) 和 [Jetty 的反应流`HTTPClient`、](https://web.archive.org/web/20221205170624/https://search.maven.org/search?q=g:org.eclipse.jetty%20AND%20a:jetty-reactive-httpclient)的依赖项添加到我们的`pom.xml. `开始这个示例。除此之外，我们还将添加 [Jetty 服务器](https://web.archive.org/web/20221205170624/https://search.maven.org/search?q=g:org.eclipse.jetty%20AND%20a:jetty-server)的依赖项，以便创建服务器:

```java
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-reactive-httpclient</artifactId>
    <version>1.0.3</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>9.4.19.v20190610</version>
</dependency>
<dependency>
    <groupId>org.reactivestreams</groupId>
    <artifactId>reactive-streams</artifactId>
    <version>1.0.3</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.2.12.RELEASE</version>
</dependency>
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.11</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>
```

## 4。创建服务器和客户端

现在，让我们创建一个服务器并添加一个请求处理程序，该处理程序只是将请求正文写入响应:

```java
public class RequestHandler extends AbstractHandler {
    @Override
    public void handle(String target, Request jettyRequest, HttpServletRequest request,
      HttpServletResponse response) throws IOException, ServletException {
        jettyRequest.setHandled(true);
        response.setContentType(request.getContentType());
        IO.copy(request.getInputStream(), response.getOutputStream());
    }
}

...

Server server = new Server(8080);
server.setHandler(new RequestHandler());
server.start();
```

然后我们可以写`HttpClient`:

```java
HttpClient httpClient = new HttpClient();
httpClient.start();
```

现在我们已经创建了客户机和服务器，让我们看看如何将这个阻塞的 HTTP 客户机转换成一个非阻塞的客户机，并创建请求:

```java
Request request = httpClient.newRequest("http://localhost:8080/"); 
ReactiveRequest reactiveRequest = ReactiveRequest.newBuilder(request).build();
Publisher<ReactiveResponse> publisher = reactiveRequest.response();
```

因此，在这里，Jetty 提供的 *ReactiveRequest* 包装器使我们的阻塞 HTTP 客户端具有反应性。让我们来看看它在不同反应库上的用法。

## 5。`ReactiveStreams`用法

Jetty 的`HttpClient` 本身支持[反应流](https://web.archive.org/web/20221205170624/http://www.reactive-streams.org/announce-1.0.3)，所以让我们从那里开始吧。

现在， **Reactive Streams 只是一组接口**，因此，对于我们的测试，让我们实现一个简单的阻塞订阅者:

```java
public class BlockingSubscriber implements Subscriber<ReactiveResponse> {
    BlockingQueue<ReactiveResponse> sink = new LinkedBlockingQueue<>(1);

    @Override
    public void onSubscribe(Subscription subscription) { 
        subscription.request(1); 
    }

    @Override 
    public void onNext(ReactiveResponse response) { 
        sink.offer(response);
    } 

    @Override 
    public void onError(Throwable failure) { } 

    @Override 
    public void onComplete() { }

    public ReactiveResponse block() throws InterruptedException {
        return sink.poll(5, TimeUnit.SECONDS);
    }   
} 
```

注意，根据 JavaDoc，我们需要**调用`Subscription#request`** ，JavaDoc 声明`“No events will be sent by a [Publisher](https://web.archive.org/web/20221205170624/http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Publisher.html "interface in org.reactivestreams") until demand is signaled via this method.” `

此外，请注意，我们添加了一个安全机制，以便我们的测试可以在 5 秒内没有看到值的情况下退出。

现在，我们可以快速测试我们的 HTTP 请求:

```java
BlockingSubscriber subscriber = new BlockingSubscriber();
publisher.subscribe(subscriber);
ReactiveResponse response = subscriber.block();
Assert.assertNotNull(response);
Assert.assertEquals(response.getStatus(), HttpStatus.OK_200);
```

## 6。项目反应器使用量

现在让我们看看如何将反应式`HttpClient`用于项目反应堆。发布者的创建与上一节中的非常相似。

在发布者创建之后，让我们使用 Project Reactor 中的`Mono`类来获得反应性响应:

```java
ReactiveResponse response = Mono.from(publisher).block();
```

然后，我们可以测试结果反应:

```java
Assert.assertNotNull(response);
Assert.assertEquals(response.getStatus(), HttpStatus.OK_200);
```

### 6.1。Spring WebFlux 用法

当与 Spring WebFlux 一起使用时，将阻塞的 HTTP 客户机转换成反应式客户机是很容易的。**Spring web flux 附带了一个反应式客户端`WebClient`，它可以与各种 HTTP 客户端库**一起使用。我们可以用它作为使用直接项目反应器代码的替代方法。

首先，让我们用`JettyClientHttpConnector`将 Jetty 的 HTTP 客户端与`WebClient:`绑定在一起

```java
ClientHttpConnector clientConnector = new JettyClientHttpConnector(httpClient);
```

然后将这个连接器传递给`WebClient` 来执行非阻塞 HTTP 请求:

```java
WebClient client = WebClient.builder().clientConnector(clientConnector).build();
```

接下来，让我们使用刚刚创建的反应式 HTTP 客户端进行实际的 HTTP 调用，并测试结果:

```java
String responseContent = client.post()
  .uri("http://localhost:8080/").contentType(MediaType.TEXT_PLAIN)
  .body(BodyInserters.fromPublisher(Mono.just("Hello World!"), String.class))
  .retrieve()
  .bodyToMono(String.class)
  .block();
Assert.assertNotNull(responseContent);
Assert.assertEquals("Hello World!", responseContent);
```

## 7。`RxJava2`用法

现在让我们继续，看看反应式 HTTP 客户端如何与 RxJava2 `. `一起使用

现在，让我们稍微改变一下我们的例子，在请求中包含一个主体:

```java
ReactiveRequest reactiveRequest = ReactiveRequest.newBuilder(request)
  .content(ReactiveRequest.Content
    .fromString("Hello World!", "text/plain", StandardCharsets.UTF_8))
  .build();
Publisher<String> publisher = reactiveRequest
  .response(ReactiveResponse.Content.asString()); 
```

代码`ReactiveResponse.Content.asString()`将响应体转换成一个字符串。如果我们只对请求的状态感兴趣，也可以使用`ReactiveResponse.Content.discard()`方法丢弃响应。

现在，我们可以看到，使用 RxJava2 获得响应实际上与 Project Reactor 非常相似。基本上，我们只是用`Single`代替`Mono`:

```java
String responseContent = Single.fromPublisher(publisher)
  .blockingGet();

Assert.assertEquals("Hello World!", responseContent);
```

## 8。请求和响应事件

被动 HTTP 客户端在执行过程中会发出许多事件。它们被分为请求事件和响应事件。这些事件有助于窥视被动 HTTP 客户端的生命周期。

这一次，让我们通过使用 HTTP 客户端而不是请求，使我们的反应式请求略有不同:

```java
ReactiveRequest request = ReactiveRequest.newBuilder(httpClient, "http://localhost:8080/")
  .content(ReactiveRequest.Content.fromString("Hello World!", "text/plain", UTF_8))
  .build(); 
```

现在让我们来看看 HTTP 请求事件的`Publisher`:

```java
Publisher<ReactiveRequest.Event> requestEvents = request.requestEvents(); 
```

现在，让我们再次使用 RxJava。这一次，我们将创建一个包含事件类型的列表，并通过在事件发生时订阅请求事件来填充它:

```java
List<Type> requestEventTypes = new ArrayList<>();

Flowable.fromPublisher(requestEvents)
  .map(ReactiveRequest.Event::getType).subscribe(requestEventTypes::add);
Single<ReactiveResponse> response = Single.fromPublisher(request.response()); 
```

然后，由于我们在测试中，我们可以阻止我们的响应并验证:

```java
int actualStatus = response.blockingGet().getStatus();

Assert.assertEquals(6, requestEventTypes.size());
Assert.assertEquals(HttpStatus.OK_200, actualStatus);
```

类似地，我们也可以订阅响应事件。因为它们类似于请求事件订阅，所以我们在这里只添加了后者。请求和响应事件的完整实现可以在 GitHub 资源库中找到，本文末尾有链接。

## 9。结论

在本教程中，我们学习了 Jetty 提供的`ReactiveStreams HttpClient`,它与各种反应库的用法，以及与反应请求相关的生命周期事件。

文章中提到的所有代码片段都可以在我们的 [GitHub 资源库](https://web.archive.org/web/20221205170624/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)中找到。