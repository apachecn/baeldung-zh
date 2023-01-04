# OkHttp 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guide-to-okhttp>

## 1。简介

在本教程中，我们将探索发送不同类型的 HTTP 请求，以及接收和解释 HTTP 响应的基础知识。然后我们将学习如何用 OkHttp 配置客户端[。](https://web.archive.org/web/20220925052718/https://square.github.io/okhttp/)

最后，我们将讨论用定制头、超时、响应缓存等配置客户机的更高级的用例。

## 2。OkHttp 概述

OkHttp 是一个适用于 Android 和 Java 应用程序的高效 HTTP & HTTP/2 客户端。

它附带了一些高级功能，比如连接池(如果 HTTP/2 不可用)、透明 GZIP 压缩和响应缓存，以完全避免网络重复请求。

它还能够从常见的连接问题中恢复；在连接失败时，如果一个服务有多个 IP 地址，它可以重试对备用地址的请求。

在高层次上，客户端是为阻塞同步调用和非阻塞异步调用而设计的。

OkHttp 支持 Android 2.3 及以上版本。对于 Java，最低要求是 1.7。

现在我们已经给出了一个简要的概述，让我们来看一些使用示例。

## 3。Maven 依赖关系

首先，我们将把这个库作为一个依赖项添加到`pom.xml`中:

```
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.1</version>
</dependency>
```

要查看这个库的最新依赖项，请查看 [Maven Central](https://web.archive.org/web/20220925052718/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22okhttp%22) 上的页面。

## 4。用 OkHttp 同步 GET

为了发送同步 GET 请求，我们需要基于一个`URL`构建一个`Request`对象，并制作一个`Call`。在它执行之后，我们将得到一个`Response` 的实例:

```
@Test
public void whenGetRequest_thenCorrect() throws IOException {
    Request request = new Request.Builder()
      .url(BASE_URL + "/date")
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 5。与 OkHttp 的异步 GET

为了进行异步 GET，我们需要让一个`Call`入队。一个`Callback`允许我们读取可读的响应。这发生在响应头准备好之后。

读取响应正文可能仍会阻塞。OkHttp 目前不提供任何异步 API 来接收部分响应体:

```
@Test
public void whenAsynchronousGetRequest_thenCorrect() {
    Request request = new Request.Builder()
      .url(BASE_URL + "/date")
      .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        public void onResponse(Call call, Response response) 
          throws IOException {
            // ...
        }

        public void onFailure(Call call, IOException e) {
            fail();
        }
    });
}
```

## 6。用查询参数获取

最后，为了向 GET 请求添加查询参数，我们可以利用`HttpUrl.Builder`。

在我们构建了 URL 之后，我们可以将它传递给我们的`Request`对象:

```
@Test
public void whenGetRequestWithQueryParameter_thenCorrect() 
  throws IOException {

    HttpUrl.Builder urlBuilder 
      = HttpUrl.parse(BASE_URL + "/ex/bars").newBuilder();
    urlBuilder.addQueryParameter("id", "1");

    String url = urlBuilder.build().toString();

    Request request = new Request.Builder()
      .url(url)
      .build();
    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 7。发布请求

现在让我们看一个简单的 POST 请求，我们构建一个`RequestBody`来发送参数`“username”`和`“password”`:

```
@Test
public void whenSendPostRequest_thenCorrect() 
  throws IOException {
    RequestBody formBody = new FormBody.Builder()
      .add("username", "test")
      .add("password", "test")
      .build();

    Request request = new Request.Builder()
      .url(BASE_URL + "/users")
      .post(formBody)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

我们的文章“用 OkHttp 发布请求的快速指南”有更多用 OkHttp 发布请求的例子。

## 8。文件上传

### 8.1。上传文件

在这个例子中，我们将演示如何上传一个`File`。我们将使用`MultipartBody.Builder`上传“`test.ext”` 文件:

```
@Test
public void whenUploadFile_thenCorrect() throws IOException {
    RequestBody requestBody = new MultipartBody.Builder()
      .setType(MultipartBody.FORM)
      .addFormDataPart("file", "file.txt",
        RequestBody.create(MediaType.parse("application/octet-stream"), 
          new File("src/test/resources/test.txt")))
      .build();

    Request request = new Request.Builder()
      .url(BASE_URL + "/users/upload")
      .post(requestBody)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

### 8.2。获取文件上传进度

然后我们将学习如何获得一个`File`上传的进度。我们将扩展`RequestBody` 来获得上传过程的可见性。

上传方法如下:

```
@Test
public void whenGetUploadFileProgress_thenCorrect() 
  throws IOException {
    RequestBody requestBody = new MultipartBody.Builder()
      .setType(MultipartBody.FORM)
      .addFormDataPart("file", "file.txt",
        RequestBody.create(MediaType.parse("application/octet-stream"), 
          new File("src/test/resources/test.txt")))
      .build();

    ProgressRequestWrapper.ProgressListener listener 
      = (bytesWritten, contentLength) -> {
        float percentage = 100f * bytesWritten / contentLength;
        assertFalse(Float.compare(percentage, 100) > 0);
    };

    ProgressRequestWrapper countingBody
      = new ProgressRequestWrapper(requestBody, listener);

    Request request = new Request.Builder()
      .url(BASE_URL + "/users/upload")
      .post(countingBody)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
} 
```

现在这里是界面`ProgressListener,`，它使我们能够观察上传进度:

```
public interface ProgressListener {
    void onRequestProgress(long bytesWritten, long contentLength);
}
```

接下来是`ProgressRequestWrapper,`，它是`RequestBody`的扩展版本:

```
public class ProgressRequestWrapper extends RequestBody {

    @Override
    public void writeTo(BufferedSink sink) throws IOException {
        BufferedSink bufferedSink;

        countingSink = new CountingSink(sink);
        bufferedSink = Okio.buffer(countingSink);

        delegate.writeTo(bufferedSink);

        bufferedSink.flush();
    }
}
```

最后，这里是`CountingSink,`，它是`ForwardingSink`的扩展版本:

```
protected class CountingSink extends ForwardingSink {

    private long bytesWritten = 0;

    public CountingSink(Sink delegate) {
        super(delegate);
    }

    @Override
    public void write(Buffer source, long byteCount)
      throws IOException {
        super.write(source, byteCount);

        bytesWritten += byteCount;
        listener.onRequestProgress(bytesWritten, contentLength());
    }
}
```

请注意:

*   当将`ForwardingSink` 扩展到`“CountingSink,”`时，我们覆盖了 `write()`方法来计算写入(传输)的字节数
*   当将`RequestBody` 扩展到`ProgressRequestWrapper,`时，我们重写了`writeTo()`方法来使用我们的`“ForwardingSink”`

## 9。设置自定义标题

### 9.1。在请求上设置标题

要在`Request,`上设置任何自定义标题，我们可以使用一个简单的`addHeader`调用:

```
@Test
public void whenSetHeader_thenCorrect() throws IOException {
    Request request = new Request.Builder()
      .url(SAMPLE_URL)
      .addHeader("Content-Type", "application/json")
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();
    response.close();
}
```

### 9.2。设置默认标题

在这个例子中，我们将看到如何在客户机上配置一个默认的头，而不是在每个请求上设置它。

例如，如果我们想为每个请求设置一个内容类型`“application/json”`，我们需要为我们的客户端设置一个拦截器:

```
@Test
public void whenSetDefaultHeader_thenCorrect() 
  throws IOException {

    OkHttpClient client = new OkHttpClient.Builder()
      .addInterceptor(
        new DefaultContentTypeInterceptor("application/json"))
      .build();

    Request request = new Request.Builder()
      .url(SAMPLE_URL)
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();
    response.close();
}
```

这里的`DefaultContentTypeInterceptor,`是`Interceptor`的扩展版本:

```
public class DefaultContentTypeInterceptor implements Interceptor {

    public Response intercept(Interceptor.Chain chain) 
      throws IOException {

        Request originalRequest = chain.request();
        Request requestWithUserAgent = originalRequest
          .newBuilder()
          .header("Content-Type", contentType)
          .build();

        return chain.proceed(requestWithUserAgent);
    }
}
```

请注意，拦截器将头添加到原始请求中。

## 10。 **不跟随重定向**

在这个例子中，我们将看到如何配置`OkHttpClient`来停止跟随重定向。

默认情况下，如果一个 GET 请求用一个`HTTP 301 Moved Permanently,`来回答，那么重定向就会自动执行。在某些用例中，这完全没问题，但在其他用例中，这并不理想。

为了实现这种行为，当我们构建客户端时，我们需要将`followRedirects`设置为`false`。

请注意，响应将返回一个`HTTP 301`状态代码:

```
@Test
public void whenSetFollowRedirects_thenNotRedirected() 
  throws IOException {

    OkHttpClient client = new OkHttpClient().newBuilder()
      .followRedirects(false)
      .build();

    Request request = new Request.Builder()
      .url("http://t.co/I5YYd9tddw")
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(301));
} 
```

如果我们用一个`true`参数打开重定向(或者完全删除它)，客户端将遵循重定向，测试将失败，因为返回代码将是 HTTP 200。

## 11。超时

当对方不可达时，我们可以使用超时来使呼叫失败。网络故障可能是由客户端连接问题、服务器可用性问题或两者之间的任何问题引起的。OkHttp 支持连接、读取和写入超时。

在这个例子中，我们用 1 秒的`readTimeout`构建了我们的客户端，而 URL 服务有 2 秒的延迟:

```
@Test
public void whenSetRequestTimeout_thenFail() 
  throws IOException {
    OkHttpClient client = new OkHttpClient.Builder()
      .readTimeout(1, TimeUnit.SECONDS)
      .build();

    Request request = new Request.Builder()
      .url(BASE_URL + "/delay/2")
      .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

请注意，测试将会失败，因为客户端超时低于资源响应时间。

## 12。取消通话

我们可以使用`Call.cancel()`立即停止正在进行的通话。如果一个线程正在写一个请求或者读一个响应，一个`IOException`将被抛出。

当不再需要通话时，我们使用这种方法来保存网络，例如当用户离开某个应用程序时:

```
@Test(expected = IOException.class)
public void whenCancelRequest_thenCorrect() 
  throws IOException {
    ScheduledExecutorService executor
      = Executors.newScheduledThreadPool(1);

    Request request = new Request.Builder()
      .url(BASE_URL + "/delay/2")  
      .build();

    int seconds = 1;
    long startNanos = System.nanoTime();

    Call call = client.newCall(request);

    executor.schedule(() -> {
        logger.debug("Canceling call: "  
            + (System.nanoTime() - startNanos) / 1e9f);

        call.cancel();

        logger.debug("Canceled call: " 
            + (System.nanoTime() - startNanos) / 1e9f);

    }, seconds, TimeUnit.SECONDS);

    logger.debug("Executing call: " 
      + (System.nanoTime() - startNanos) / 1e9f);

    Response response = call.execute();

    logger.debug(Call was expected to fail, but completed: " 
      + (System.nanoTime() - startNanos) / 1e9f, response);
}
```

## 13。响应缓存

为了创建一个`Cache`，我们需要一个可以读写的缓存目录，以及对缓存大小的限制。

客户端将使用它来缓存响应:

```
@Test
public void  whenSetResponseCache_thenCorrect() 
  throws IOException {
    int cacheSize = 10 * 1024 * 1024;

    File cacheDirectory = new File("src/test/resources/cache");
    Cache cache = new Cache(cacheDirectory, cacheSize);

    OkHttpClient client = new OkHttpClient.Builder()
      .cache(cache)
      .build();

    Request request = new Request.Builder()
      .url("http://publicobject.com/helloworld.txt")
      .build();

    Response response1 = client.newCall(request).execute();
    logResponse(response1);

    Response response2 = client.newCall(request).execute();
    logResponse(response2);
}
```

启动测试后，第一次调用的响应不会被缓存。对方法`cacheResponse`的调用将返回`null`，而对方法`networkResponse`的调用将从网络返回响应。

缓存文件夹也将充满缓存文件。

第二次调用执行将产生相反的效果，因为响应已经被缓存了。这意味着对`networkResponse`的调用将返回`null,`，而对`cacheResponse`的调用将从缓存中返回响应。

为了防止响应使用缓存，我们可以使用`CacheControl.FORCE_NETWORK`。为了防止它使用网络，我们可以使用`CacheControl.FORCE_CACHE`。

需要注意的是，如果我们使用`FORCE_CACHE,`并且响应需要网络，`OkHttp`将返回一个 504 不可满足请求响应。

## 14。结论

在本文中，我们探索了几个如何使用 OkHttp 作为 HTTP & HTTP/2 客户机的例子。

与往常一样，示例代码可以在 [GitHub 项目](https://web.archive.org/web/20220925052718/https://github.com/eugenp/tutorials/tree/master/libraries-http)中找到。