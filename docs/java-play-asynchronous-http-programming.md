# 用 Play 框架进行异步 HTTP 编程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-play-asynchronous-http-programming>

## 1.概观

通常我们的 web 服务需要使用其他 web 服务来完成它们的工作。很难在满足用户请求的同时保持较低的响应时间。缓慢的外部服务会增加我们的响应时间，并导致我们的系统堆积请求，使用更多的资源。这就是非阻塞方法非常有用的地方

在本教程中，我们将从一个 [Play Framework](/web/20221126234321/https://www.baeldung.com/java-intro-to-the-play-framework) 应用程序向一个服务发出多个异步请求。通过利用 Java 的非阻塞 HTTP 功能，我们将能够顺利地查询外部资源，而不会影响我们自己的主逻辑。

在我们的例子中，我们将探索 [Play WebService 库](https://web.archive.org/web/20221126234321/https://www.playframework.com/documentation/2.8.x/JavaWS)。

## 2.Play WebService (WS)库

WS 是一个强大的库，使用 Java 提供异步 HTTP 调用。

使用这个库，我们的代码发送这些请求并继续运行而不会阻塞。为了处理请求的结果，我们提供了一个消费函数，即`Consumer`接口的实现。

这种模式与 JavaScript 的回调实现、`Promises,`和`async/await`模式有一些相似之处。

让我们构建一个简单的`Consumer`来记录一些响应数据:

```java
ws.url(url)
  .thenAccept(r -> 
    log.debug("Thread#" + Thread.currentThread().getId() 
      + " Request complete: Response code = " + r.getStatus() 
      + " | Response: " + r.getBody() 
      + " | Current Time:" + System.currentTimeMillis()))
```

在这个例子中，我们的`Consumer`仅仅是登录。不过，消费者可以做我们需要对结果做的任何事情，比如将结果存储在数据库中。

如果我们更深入地研究库的实现，我们可以观察到 WS 包装并配置了 Java 的 [`AsyncHttpClient`](/web/20221126234321/https://www.baeldung.com/async-http-client) ，它是标准 JDK 的一部分，不依赖于 Play。

## 3.准备一个示例项目

为了试验这个框架，让我们创建一些单元测试来启动请求。我们将创建一个框架 web 应用程序来回答这些问题，并使用 WS 框架来发出 HTTP 请求。

### 3.1.框架 Web 应用程序

首先，我们使用`sbt new`命令创建初始项目:

```java
sbt new playframework/play-java-seed.g8
```

在新文件夹中，我们然后**编辑`build.sbt`文件并添加 WS 库依赖:**

```java
libraryDependencies += javaWs
```

现在我们可以用`sbt run`命令启动服务器:

```java
$ sbt run
...
--- (Running the application, auto-reloading is enabled) ---

[info] p.c.s.AkkaHttpServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000
```

一旦应用程序启动，我们可以通过浏览`[http://localhost:9000](https://web.archive.org/web/20221126234321/http://localhost:9000/)`来检查一切是否正常，这将打开 Play 的欢迎页面。

### 3.2.测试环境

为了测试我们的应用程序，我们将使用单元测试类`HomeControllerTest`。

首先，我们需要扩展`WithServer`，它将提供服务器生命周期:

```java
public class HomeControllerTest extends WithServer { 
```

多亏了它的父类**，这个类现在在运行测试之前，以测试模式在一个随机端口**上启动我们的框架 web 服务器。测试完成后,`WithServer`类也会停止应用程序。

接下来，我们需要提供一个应用程序来运行。

我们可以用 [`Guice`](/web/20221126234321/https://www.baeldung.com/guice) 的`GuiceApplicationBuilder`来创建它:

```java
@Override
protected Application provideApplication() {
    return new GuiceApplicationBuilder().build();
} 
```

最后，我们设置测试中使用的服务器 URL，使用测试服务器提供的端口号:

```java
@Override
@Before
public void setup() {
    OptionalInt optHttpsPort = testServer.getRunningHttpsPort();
    if (optHttpsPort.isPresent()) {
        port = optHttpsPort.getAsInt();
        url = "https://localhost:" + port;
    } else {
        port = testServer.getRunningHttpPort()
          .getAsInt();
        url = "http://localhost:" + port;
    }
}
```

现在我们准备好编写测试了。全面的测试框架让我们专注于编码我们的测试请求。

## 4.准备 WSRequest

让我们看看如何触发基本类型的请求，比如 GET 或 POST，以及文件上传的多部分请求。

### 4.1.初始化`WSRequest`对象

首先，我们需要获得一个`WSClient`实例来配置和初始化我们的请求。

在实际应用中，我们可以通过依赖注入获得一个使用默认设置自动配置的客户端:

```java
@Autowired
WSClient ws;
```

不过，在我们的测试类中，我们使用 [`WSTestClient`](https://web.archive.org/web/20221126234321/https://www.playframework.com/documentation/2.8.x/api/java/play/test/WSTestClient.html) ，可从[中获得测试框架](https://web.archive.org/web/20221126234321/https://www.playframework.com/documentation/2.8.x/JavaFunctionalTest):

```java
WSClient ws = play.test.WSTestClient.newClient(port);
```

一旦我们有了客户端，我们就可以通过调用`url`方法来初始化一个`WSRequest`对象:

```java
ws.url(url)
```

**`url`方法足以让我们发出请求。**但是，我们可以通过添加一些自定义设置来进一步自定义它:

```java
ws.url(url)
  .addHeader("key", "value")
  .addQueryParameter("num", "" + num);
```

正如我们所见，添加头和查询参数非常容易。

在我们完全配置好我们的请求之后，我们可以调用方法来初始化它。

### 4.2.通用获取请求

要触发 GET 请求，我们只需调用我们的`WSRequest`对象上的`get`方法:

```java
ws.url(url)
  ...
  .get();
```

因为这是一个非阻塞代码，所以它启动请求，然后在函数的下一行继续执行。

**由`get`返回的对象是一个`CompletionStage`实例**，它是 [`CompletableFuture` API](/web/20221126234321/https://www.baeldung.com/java-completablefuture) 的一部分。

一旦 HTTP 调用完成，这个阶段只执行几条指令。它将响应包装在一个`WSResponse`对象中。

通常，这个结果会被传递到执行链的下一个阶段。在这个例子中，我们没有提供任何消费函数，所以结果丢失了。

由于这个原因，这个请求是“发射并忘记”类型的。

### 4.3.提交表格

提交表单与`get`示例没有太大的不同。

要触发请求，我们只需调用`post`方法:

```java
ws.url(url)
  ...
  .setContentType("application/x-www-form-urlencoded")
  .post("key1=value1&key2;=value2");
```

**在这个场景中，我们需要传递一个 body 作为参数。**这可以是一个简单的字符串，如文件、json 或 xml 文档、`BodyWritable`或`Source`。

### 4.4.提交多部分/表单数据

多部分表单要求我们从附加文件或流中发送输入字段和数据。

**为了在框架中实现这一点，我们使用带有`Source`的`post`方法。**

在源内部，我们可以包装表单所需的所有不同数据类型:

```java
Source<ByteString, ?> file = FileIO.fromPath(Paths.get("hello.txt"));
FilePart<Source<ByteString, ?>> file = 
  new FilePart<>("fileParam", "myfile.txt", "text/plain", file);
DataPart data = new DataPart("key", "value");

ws.url(url)
...
  .post(Source.from(Arrays.asList(file, data)));
```

尽管这种方法增加了一些配置，但它仍然非常类似于其他类型的请求。

## 5.处理异步响应

到目前为止，我们只触发了一劳永逸的请求，其中我们的代码不对响应数据做任何事情。

现在让我们探索两种处理异步响应的技术。

**我们可以阻塞主线程，等待一个`CompletableFuture,` ，或者用一个`Consumer`异步消费。**

### 5.1.用`CompletableFuture`阻塞处理响应

即使在使用异步框架时，我们也可能选择阻塞代码的执行并等待响应。

使用`CompletableFuture` API，我们只需要对代码做一些修改就可以实现这个场景:

```java
WSResponse response = ws.url(url)
  .get()
  .toCompletableFuture()
  .get();
```

这可能是有用的，例如，提供强大的数据一致性，这是我们无法通过其他方式实现的。

### 5.2.异步处理响应

**为了无阻塞地处理异步响应，** **我们提供了一个`Consumer`或`**Function**`** ，当响应可用时由异步框架运行。

例如，让我们在前面的示例中添加一个`Consumer`来记录响应:

```java
ws.url(url)
  .addHeader("key", "value")
  .addQueryParameter("num", "" + 1)
  .get()
  .thenAccept(r -> 
    log.debug("Thread#" + Thread.currentThread().getId() 
      + " Request complete: Response code = " + r.getStatus() 
      + " | Response: " + r.getBody() 
      + " | Current Time:" + System.currentTimeMillis()));
```

然后，我们会在日志中看到响应:

```java
[debug] c.HomeControllerTest - Thread#30 Request complete: Response code = 200 | Response: {
  "Result" : "ok",
  "Params" : {
    "num" : [ "1" ]
  },
  "Headers" : {
    "accept" : [ "*/*" ],
    "host" : [ "localhost:19001" ],
    "key" : [ "value" ],
    "user-agent" : [ "AHC/2.1" ]
  }
} | Current Time:1579303109613
```

值得注意的是，我们使用了`thenAccept`，它需要一个`Consumer`函数，因为我们不需要在登录后返回任何东西。

当我们希望当前阶段返回某个东西，以便在下一阶段使用时，我们需要用`thenApply`来代替，这需要一个`Function`。

这些使用标准 [Java 功能接口](/web/20221126234321/https://www.baeldung.com/java-8-functional-interfaces)的约定。

### 5.3.大响应体

到目前为止，我们实现的代码对于小响应和大多数用例来说是一个很好的解决方案。然而，如果我们需要处理几百兆字节的数据，我们将需要一个更好的策略。

我们应该注意:**请求方法像`get`和`post`将整个响应加载到内存中。**

为了避免可能的`OutOfMemoryError`，我们可以使用 [Akka Streams](/web/20221126234321/https://www.baeldung.com/akka-streams) 来处理响应，而不让它填满我们的内存。

例如，我们可以将它的主体写在一个文件中:

```java
ws.url(url)
  .stream()
  .thenAccept(
    response -> {
        try {
            OutputStream outputStream = Files.newOutputStream(path);
            Sink<ByteString, CompletionStage<Done>> outputWriter =
              Sink.foreach(bytes -> outputStream.write(bytes.toArray()));
            response.getBodyAsSource().runWith(outputWriter, materializer);
        } catch (IOException e) {
            log.error("An error happened while opening the output stream", e);
        }
    });
```

`stream`方法返回一个`CompletionStage` ，其中`WSResponse`有一个`getBodyAsStream`方法提供一个`Source<ByteString, ?>`。

我们可以通过使用 Akka 的`Sink`来告诉代码如何处理这种类型的主体，在我们的示例中，它将简单地在`OutputStream`中写入任何通过的数据。

### 5.4.超时设定

在构建请求时，我们还可以设置一个特定的超时，这样如果我们没有及时收到完整的响应，请求就会被中断。

当我们看到我们正在查询的服务特别慢，并且可能导致打开的连接堆积在一起等待响应时，这是一个特别有用的特性。

我们可以使用调优参数为所有请求设置一个全局超时。对于特定于请求的超时，我们可以使用`setRequestTimeout`添加到请求中:

```java
ws.url(url)
  .setRequestTimeout(Duration.of(1, SECONDS));
```

尽管如此，仍然有一种情况需要处理:我们可能已经收到了所有的数据，但是我们的`Consumer`可能处理起来非常慢。如果有大量的数据处理、数据库调用等，这可能会发生。

在低吞吐量系统中，我们可以简单地让代码运行，直到它完成。然而，我们可能希望中止长时间运行的活动。

为了实现这一点，我们必须用一些`futures`处理来包装我们的代码。

让我们在代码中模拟一个非常长的过程:

```java
ws.url(url)
  .get()
  .thenApply(
    result -> { 
        try { 
            Thread.sleep(10000L); 
            return Results.ok(); 
        } catch (InterruptedException e) { 
            return Results.status(SERVICE_UNAVAILABLE); 
        } 
    });
```

这将在 10 秒后返回一个`OK`响应，但是我们不想等那么久。

相反，使用`timeout`包装器，我们指示我们的代码等待不超过 1 秒:

```java
CompletionStage<Result> f = futures.timeout(
  ws.url(url)
    .get()
    .thenApply(result -> {
        try {
            Thread.sleep(10000L);
            return Results.ok();
        } catch (InterruptedException e) {
            return Results.status(SERVICE_UNAVAILABLE);
        }
    }), 1L, TimeUnit.SECONDS); 
```

现在，我们的未来将以任何方式返回一个结果:如果`Consumer`及时完成，则返回计算结果，或者由于`futures`超时而导致的异常。

### 5.5.处理异常

在前面的例子中，我们创建了一个函数，它要么返回结果，要么因异常而失败。所以，现在我们需要处理这两种情况。

**我们可以用`handleAsync`方法处理成功和失败的场景。**

假设我们想要返回结果(如果我们已经得到了结果),或者记录错误并返回异常以便进一步处理:

```java
CompletionStage<Object> res = f.handleAsync((result, e) -> {
    if (e != null) {
        log.error("Exception thrown", e);
        return e.getCause();
    } else {
        return result;
    }
}); 
```

代码现在应该返回一个包含抛出的`TimeoutException`的`CompletionStage`。

我们可以通过简单地调用返回的异常对象的类上的`assertEquals`来验证它:

```java
Class<?> clazz = res.toCompletableFuture().get().getClass();
assertEquals(TimeoutException.class, clazz);
```

运行测试时，它还会记录我们收到的异常:

```java
[error] c.HomeControllerTest - Exception thrown
java.util.concurrent.TimeoutException: Timeout after 1 second
...
```

## 6.请求过滤器

有时，我们需要在请求被触发之前运行一些逻辑。

我们可以在初始化后操作`WSRequest`对象，但是更好的方法是设置一个`WSRequestFilter`。

**可以在初始化期间，调用触发方法之前设置过滤器，并将其附加到请求逻辑。**

我们可以通过实现`WSRequestFilter`接口来定义自己的过滤器，也可以添加一个现成的。

一个常见的场景是在执行请求之前记录它的样子。

在这种情况下，我们只需要设置`AhcCurlRequestLogger`:

```java
ws.url(url)
  ...
  .setRequestFilter(new AhcCurlRequestLogger())
  ...
  .get();
```

生成的日志具有类似于`curl`的格式:

```java
[info] p.l.w.a.AhcCurlRequestLogger - curl \
  --verbose \
  --request GET \
  --header 'key: value' \
  'http://localhost:19001'
```

我们可以通过更改我们的`logback.xml`配置来设置所需的日志级别。

## 7.缓存响应

**`WSClient`也支持响应的缓存。**

当同一个请求被多次触发，并且我们不需要每次都使用最新的数据时，这个特性特别有用。

当我们呼叫的服务暂时关闭时，它也有帮助。

### 7.1.添加缓存依赖项

要配置缓存，我们首先需要在我们的`build.sbt`中添加依赖关系:

```java
libraryDependencies += ehcache
```

这将 [Ehcache](/web/20221126234321/https://www.baeldung.com/ehcache) 配置为我们的缓存层。

如果我们不特别想要 Ehcache，我们可以使用任何其他的 [JSR-107](https://web.archive.org/web/20221126234321/https://github.com/jsr107/jsr107spec) 缓存实现。

### 7.2.强制缓存启发式

**默认情况下，如果服务器没有返回任何缓存配置，Play WS 不会缓存 HTTP 响应。**

为了避免这一点，我们可以通过向我们的`application.conf`添加一个设置来强制启发式缓存:

```java
play.ws.cache.heuristics.enabled=true
```

这将配置系统来决定何时缓存 HTTP 响应是有用的，而不管远程服务广告的缓存。

## 8.附加调谐

向外部服务发出请求可能需要一些客户端配置。我们可能需要处理重定向、慢速服务器或一些过滤，这取决于用户代理头。

**为了解决这个问题，我们可以使用`application.conf` :** 中的属性来调优我们的 WS 客户端

```java
play.ws.followRedirects=false
play.ws.useragent=MyPlayApplication
play.ws.compressionEnabled=true
# time to wait for the connection to be established
play.ws.timeout.connection=30
# time to wait for data after the connection is open
play.ws.timeout.idle=30
# max time available to complete the request
play.ws.timeout.request=300
```

**也可以直接配置底层`AsyncHttpClient`。**

可用属性的完整列表可以在 [`AhcConfig`](https://web.archive.org/web/20221126234321/https://github.com/playframework/play-ws/blob/master/play-ahc-ws-standalone/src/main/scala/play/api/libs/ws/ahc/AhcConfig.scala) 的源代码中查看。

## 9.结论

在本文中，我们探索了 Play WS 库及其主要特性。我们配置了我们的项目，学习了如何同步和异步地触发常见的请求并处理它们的响应。

我们处理大量数据下载，并了解如何缩短长时间运行的活动。

最后，我们研究了如何通过缓存来提高性能，以及如何调优客户端。

和往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221126234321/https://github.com/eugenp/tutorials/tree/master/web-modules/play-modules/async-http)