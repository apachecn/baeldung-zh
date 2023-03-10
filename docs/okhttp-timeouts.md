# OkHttp 中超时的快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/okhttp-timeouts>

## 1.概观

在这个快速教程中，我们将关注我们可以为 [OkHttp](https://web.archive.org/web/20221227153527/https://square.github.io/okhttp/) 客户端设置的不同类型的超时。

关于 OkHttp 库的更多概述，请查看我们的介绍性 OkHttp 指南。

## 2.连接超时

连接超时定义了一个**时间段，在这个时间段内，我们的客户端应该与目标主机**建立连接。

**默认情况下，对于`OkHttpClient`，此超时设置为 10 秒**。

但是，我们可以使用 [`OkHttpClient.Builder#connectTimeout`](https://web.archive.org/web/20221227153527/https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#connectTimeout-long-java.util.concurrent.TimeUnit-) 方法轻松地更改它的值。零值意味着根本没有超时。

现在让我们看看如何构建和使用带有自定义连接超时的`OkHttpClient`:

```java
@Test
public void whenConnectTimeoutExceeded_thenSocketTimeoutException() {
    OkHttpClient client = new OkHttpClient.Builder()
      .connectTimeout(10, TimeUnit.MILLISECONDS)
      .build();

    Request request = new Request.Builder()
      .url("http://203.0.113.1") // non routable address
      .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(SocketTimeoutException.class);
}
```

上面的例子显示，当连接尝试超过配置的超时时间时，客户端抛出一个`SocketTimeoutException`。

## 3.读取超时

从客户端和目标主机之间的连接成功建立的那一刻起，便会应用读取超时。

它定义了在等待服务器响应时，两个数据包之间不活动的最大时间**。**

使用 [`OkHttpClient.Builder#readTimeout`](https://web.archive.org/web/20221227153527/https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#readTimeout-long-java.util.concurrent.TimeUnit-) 可以更改**默认的 10 秒**超时。与连接超时类似，零值表示没有超时。

现在让我们看看如何在实践中配置自定义读取超时:

```java
@Test
public void whenReadTimeoutExceeded_thenSocketTimeoutException() {
    OkHttpClient client = new OkHttpClient.Builder()
      .readTimeout(10, TimeUnit.MILLISECONDS)
      .build();

    Request request = new Request.Builder()
      .url("https://httpbin.org/delay/2") // 2-second response time
      .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(SocketTimeoutException.class);
}
```

正如我们所看到的，服务器没有在定义的 500 毫秒超时内返回响应。结果，`OkHttpClient`抛出了一个`SocketTimeoutException.`

## 4.写入超时

写超时定义了向服务器发送请求时两个数据包之间不活动的最大时间。

同样，对于连接和读取超时，**我们可以使用 [`OkHttpClient.Builder#writeTimeout`](https://web.archive.org/web/20221227153527/https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#writeTimeout-long-java.util.concurrent.TimeUnit-)** 来覆盖默认值 10 秒。按照惯例，零值意味着根本没有超时。

在下面的示例中，我们设置了一个非常短的写入超时(10 ms ),并向服务器发送一个 1 MB 的内容:

```java
@Test
public void whenWriteTimeoutExceeded_thenSocketTimeoutException() {
    OkHttpClient client = new OkHttpClient.Builder()
      .writeTimeout(10, TimeUnit.MILLISECONDS)
      .build();

    Request request = new Request.Builder()
      .url("https://httpbin.org/delay/2")
      .post(RequestBody.create(MediaType.parse("text/plain"), create1MBString()))
      .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(SocketTimeoutException.class);
}
```

正如我们所看到的，由于负载过大，我们的客户机无法在定义的超时时间内向服务器发送请求体。因此，`OkHttpClient`抛出一个`SocketTimeoutException`。

## 5.呼叫超时

调用超时与我们已经讨论过的连接、读取和写入超时稍有不同。

**它定义了一个完整 HTTP 调用的时间限制**。这包括解析 DNS、连接、编写请求正文、服务器处理以及读取响应正文。

与其他超时不同，**的默认值设置为零，这意味着没有超时**。但是当然，我们可以使用 [`OkHttpClient.Builder#callTimeout`](https://web.archive.org/web/20221227153527/https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#callTimeout-long-java.util.concurrent.TimeUnit-) 方法配置一个自定义值。

让我们来看一个实际的使用例子:

```java
@Test
public void whenCallTimeoutExceeded_thenInterruptedIOException() {
    OkHttpClient client = new OkHttpClient.Builder()
      .callTimeout(1, TimeUnit.SECONDS)
      .build();

    Request request = new Request.Builder()
      .url("https://httpbin.org/delay/2")
      .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(InterruptedIOException.class);
}
```

正如我们所看到的，调用超时并且`OkHttpClient`抛出一个`InterruptedIOException.`

## 6.每个请求超时

**建议创建一个单独的`OkHttpClient`实例，并在我们的应用程序中为所有的 HTTP 调用**重用它。

然而，有时我们知道某个请求比所有其他请求花费更多的时间。在这种情况下，我们需要**只为那个特定的调用**延长给定的超时。

在这种情况下，我们可以使用一种 [`OkHttpClient#newBuilder`](https://web.archive.org/web/20221227153527/https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.html#newBuilder--) 的方法。这将构建一个共享相同设置的新客户端。然后，我们可以根据需要使用构建器方法来调整超时设置。

现在让我们看看如何在实践中做到这一点:

```java
@Test
public void whenPerRequestTimeoutExtended_thenResponseSuccess() throws IOException {
    OkHttpClient defaultClient = new OkHttpClient.Builder()
      .readTimeout(1, TimeUnit.SECONDS)
      .build();

    Request request = new Request.Builder()
      .url("https://httpbin.org/delay/2")
      .build();

    Throwable thrown = catchThrowable(() -> defaultClient.newCall(request).execute());

    assertThat(thrown).isInstanceOf(InterruptedIOException.class);

    OkHttpClient extendedTimeoutClient = defaultClient.newBuilder()
      .readTimeout(5, TimeUnit.SECONDS)
      .build();

    Response response = extendedTimeoutClient.newCall(request).execute();
    assertThat(response.code()).isEqualTo(200);
}
```

正如我们看到的，`defaultClient`由于超过读取超时而未能完成 HTTP 调用。

这就是为什么我们创建了`extendedTimeoutClient,`调整了超时值，并成功执行了请求。

## 7.摘要

在本文中，我们**探索了可以为`OkHttpClient`** 配置的不同超时。

我们还简要描述了在 HTTP 调用期间何时应用连接、读取和写入超时。

此外，**我们展示了只为单个请求**更改某个超时值是多么容易。

像往常一样，GitHub 上的所有代码示例[都可用。](https://web.archive.org/web/20221227153527/https://github.com/eugenp/tutorials/tree/master/libraries-http)