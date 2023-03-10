# OkHTTP 中的事件指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-okhttp-events>

## 1。概述

通常，当在我们的 web 应用程序中处理 HTTP 调用时，我们需要一种方法来捕获关于请求和响应的某种度量。通常，这是为了监控我们的应用程序发出的 HTTP 调用的大小和频率。

OkHttp 是一款适用于 Android 和 Java 应用的高效 HTTP & HTTP/2 客户端。在之前的教程中，我们学习了如何使用 OkHttp 的[基础知识](/web/20220630004056/https://www.baeldung.com/guide-to-okhttp)。

在本教程中，我们将学习如何使用事件捕获这些类型的指标。

## 2。事件

顾名思义，事件为我们记录与整个 HTTP 调用生命周期相关的应用程序指标提供了一个强大的机制。

**为了订阅我们所有感兴趣的事件，我们需要做的是定义一个`EventListener`并覆盖我们想要捕捉的事件的方法。**

例如，如果我们只想监视失败和成功的调用，这就特别有用。在这种情况下，我们只需在事件侦听器类中覆盖对应于这些事件的特定方法。稍后我们将更详细地了解这一点。

在我们的应用程序中使用事件至少有几个好处:

*   我们可以使用事件来监控应用程序发出的 HTTP 调用的大小和频率
*   这可以帮助我们快速确定我们的应用程序中哪里有瓶颈

最后，我们还可以使用事件来确定我们的网络是否存在潜在问题。

## 3.属国

当然，我们需要将标准的 [`okhttp`依赖关系](https://web.archive.org/web/20220630004056/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22okhttp%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.1</version>
</dependency>
```

我们还需要另一个专门用于测试的依赖项。再来补充一下 OkHttp [`mockwebserver`神器](https://web.archive.org/web/20220630004056/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22mockwebserver%22):

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>4.9.1</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必需的依赖项，我们可以继续编写我们的第一个事件侦听器了。

## 4.事件方法和顺序

但是在我们开始定义我们自己的事件监听器之前，**我们将后退一步，简要地看一下我们可以使用哪些事件方法，以及我们期望事件到达**的顺序。这将有助于我们以后深入研究一些真实的例子。

假设我们正在处理一个没有重定向或重试的成功的 HTTP 调用。然后我们可以期待这个典型的方法调用流。

### 4.1.`callStart()`

这个方法是我们的入口点，一旦我们将一个调用排队或者我们的客户机执行它，我们就调用它。

### 4.2.`proxySelectStart()` 和 `proxySelectEnd()`

第一种方法在代理选择之前调用，同样在代理选择之后调用，包括代理列表，按照它们将被尝试的顺序。当然，如果没有配置代理，这个列表可以是空的。

### 4.3.`dnsStart()`和`dnsEnd()`

这些方法在 DNS 查找之前和 DNS 解析之后立即被调用。

### 4.4.`connectStart()`和`connectEnd()`

这些方法在建立和关闭套接字连接之前被调用。

### 4.5.`secureConnectStart()`和`secureConnectEnd()`

如果我们的调用使用 HTTPS，那么穿插在`connectStart`和`connectEnd`之间，我们将有这些安全连接变体。

### 4.6.`connectionAcquired()`和`connectionReleased()`

在获取或释放连接后调用。

### 4.7.`requestHeadersStart()`和`requestHeadersEnd()`

这些方法将在发送请求头之前和之后立即被调用。

### 4.8.`requestBodyStart()`和`requestBodyEnd()`

顾名思义，在发送请求体之前调用。当然，这只适用于包含主体的请求。

### 4.9.`responseHeadersStart()`和`responseHeadersEnd()`

当响应头第一次从服务器返回时以及在收到响应头后立即调用这些方法。

### 4.10.`responseBodyStart()`和`responseBodyEnd()`

同样，当响应正文第一次从服务器返回时，在收到正文后立即调用。

**除了这些方法，我们还有另外三种方法可以用来捕获故障:**

### 4.11.`callFailed()`、`responseFailed()`和`requestFailed()`

如果我们的调用永久失败，则请求有写失败，或者响应有读失败。

## 5.定义简单的事件侦听器

让我们从定义我们自己的 even 监听器开始。为了简单起见，**我们的事件监听器将记录调用开始和结束的时间，以及一些请求和响应头信息**:

```java
public class SimpleLogEventsListener extends EventListener {

    private static final Logger LOGGER = LoggerFactory.getLogger(SimpleLogEventsListener.class);

    @Override
    public void callStart(Call call) {
        LOGGER.info("callStart at {}", LocalDateTime.now());
    }

    @Override
    public void requestHeadersEnd(Call call, Request request) {
        LOGGER.info("requestHeadersEnd at {} with headers {}", LocalDateTime.now(), request.headers());
    }

    @Override
    public void responseHeadersEnd(Call call, Response response) {
        LOGGER.info("responseHeadersEnd at {} with headers {}", LocalDateTime.now(), response.headers());
    }

    @Override
    public void callEnd(Call call) {
        LOGGER.info("callEnd at {}", LocalDateTime.now());
    }
}
```

正如我们所见，**要创建我们的监听器，我们需要做的就是从`EventListener`类扩展。**然后我们可以继续为我们关心的事件重写方法。

在我们的简单侦听器中，我们记录调用开始和结束的时间，以及它们到达时的请求和响应头。

### 5.1.将它们连接在一起

要真正利用这个监听器，我们需要做的就是在构建我们的`OkHttpClient`实例时调用`eventListener`方法，它应该可以正常工作:

```java
OkHttpClient client = new OkHttpClient.Builder() 
  .eventListener(new SimpleLogEventsListener())
  .build();
```

在下一节中，我们将看看如何测试我们的新监听器。

### 5.2.测试事件监听器

现在，我们已经定义了第一个事件监听器；让我们继续编写我们的第一个集成测试:

```java
@Rule
public MockWebServer server = new MockWebServer();

@Test
public void givenSimpleEventLogger_whenRequestSent_thenCallsLogged() throws IOException {
    server.enqueue(new MockResponse().setBody("Hello Baeldung Readers!"));

    OkHttpClient client = new OkHttpClient.Builder()
      .eventListener(new SimpleLogEventsListener())
      .build();

    Request request = new Request.Builder()
      .url(server.url("/"))
      .build();

    try (Response response = client.newCall(request).execute()) {
        assertEquals("Response code should be: ", 200, response.code());
        assertEquals("Body should be: ", "Hello Baeldung Readers!", response.body().string());
    }
 }
```

首先，我们使用 OkHttp `MockWebServer` [JUnit 规则](/web/20220630004056/https://www.baeldung.com/junit-4-rules)。

这是一个轻量级的、可脚本化的 web 服务器，用于测试 HTTP 客户端，我们将用它来测试我们的事件监听器。通过使用这个规则，我们将为每个集成测试创建一个干净的服务器实例。

记住这一点，现在让我们来看看测试的关键部分:

*   首先，我们设置一个模拟响应，其主体中包含一条简单的消息
*   然后，我们构建我们的`OkHttpClient`并配置我们的`SimpleLogEventsListener`
*   **最后，我们发送请求，并使用断言检查收到的响应代码和主体**

### 5.3.运行测试

当我们运行测试时，我们将看到我们的事件被记录:

```java
callStart at 2021-05-04T17:51:33.024
...
requestHeadersEnd at 2021-05-04T17:51:33.046 with headers User-Agent: A Baeldung Reader
Host: localhost:51748
Connection: Keep-Alive
Accept-Encoding: gzip
...
responseHeadersEnd at 2021-05-04T17:51:33.053 with headers Content-Length: 23
callEnd at 2021-05-04T17:51:33.055
```

## 6.把所有的放在一起

现在，让我们假设我们想要构建简单的日志示例，并记录调用链中每个步骤的运行时间:

```java
public class EventTimer extends EventListener {

    private long start;

    private void logTimedEvent(String name) {
        long now = System.nanoTime();
        if (name.equals("callStart")) {
            start = now;
        }
        long elapsedNanos = now - start;
        System.out.printf("%.3f %s%n", elapsedNanos / 1000000000d, name);
    }

    @Override
    public void callStart(Call call) {
        logTimedEvent("callStart");
    }

    // More event listener methods
}
```

这与我们的第一个例子非常相似，但是这一次我们捕获了从每个事件的调用开始所经过的时间。**通常，检测网络延迟**会非常有趣。

让我们来看看我们是否在一个真实的网站上运行它，比如我们自己的[https://www.baeldung.com/](/web/20220630004056/https://www.baeldung.com/):

```java
0.000 callStart
0.012 proxySelectStart
0.012 proxySelectEnd
0.012 dnsStart
0.175 dnsEnd
0.183 connectStart
0.248 secureConnectStart
0.608 secureConnectEnd
0.608 connectEnd
0.609 connectionAcquired
0.612 requestHeadersStart
0.613 requestHeadersEnd
0.706 responseHeadersStart
0.707 responseHeadersEnd
0.765 responseBodyStart
0.765 responseBodyEnd
0.765 connectionReleased
0.765 callEnd 
```

由于这个呼叫经过 HTTPS，我们还将看到`secureConnectStart`和`secureConnectStart`事件。

## 7.监控失败的呼叫

到目前为止，我们关注的是成功的 HTTP 请求，但是我们也可以捕获失败的事件:

```java
@Test (expected = SocketTimeoutException.class)
public void givenConnectionError_whenRequestSent_thenFailedCallsLogged() throws IOException {
    OkHttpClient client = new OkHttpClient.Builder()
      .eventListener(new EventTimer())
      .build();

    Request request = new Request.Builder()
      .url(server.url("/"))
      .build();

    client.newCall(request).execute();
}
```

在这个例子中，我们故意避免设置我们的模拟 web 服务器，这当然意味着，我们将看到一个以`SocketTimeoutException`形式出现的灾难性故障。

现在让我们来看看运行测试时的输出:

```java
0.000 callStart
...
10.008 responseFailed
10.009 connectionReleased
10.009 callFailed
```

**正如所料，我们将看到我们的呼叫开始，然后在 10 秒钟后，连接超时发生，因此，我们看到记录的`responseFailed`和`callFailed`事件。**

## 8.简单说说并发性

到目前为止，我们假设没有多个调用同时执行[和](/web/20220630004056/https://www.baeldung.com/cs/aba-concurrency)。**如果我们想要适应这个场景，那么我们需要在配置`OkHttpClient`** 时使用`eventListenerFactory` 方法。

我们可以使用工厂为每个 HTTP 调用创建一个新的`EventListener`实例。当我们使用这种方法时，可以在我们的侦听器中保持特定于调用的状态。

## 9.结论

在本文中，我们已经了解了如何使用 OkHttp 捕获事件。首先，我们开始解释什么是事件，理解什么类型的事件对我们可用，以及它们在处理 HTTP 调用时到达的顺序。

然后我们看了看如何定义一个简单的事件记录器来捕获我们的 HTTP 调用的一部分，以及如何编写一个集成测试。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220630004056/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)