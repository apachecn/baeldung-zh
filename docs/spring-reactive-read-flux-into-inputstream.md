# 使用 Spring Reactive WebClient 将流量读入单个 InputStream

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-reactive-read-flux-into-inputstream>

## 1.概观

在本教程中，我们将深入 [Java 反应式编程](/web/20220910090243/https://www.baeldung.com/spring-reactive-guide)到**解决一个有趣的问题，如何将`[Flux](/web/20220910090243/https://www.baeldung.com/reactor-core)<DataBuffer>`读入单个`InputStream`** 。

## 2.请求设置

作为解决将`Flux<DataBuffer>`读入单个`InputStream`的问题的第一步，我们将使用 [Spring reactive WebClient](/web/20220910090243/https://www.baeldung.com/spring-5-webclient) 发出一个`GET`请求。此外，我们可以**使用由[gorest.co.in](https://web.archive.org/web/20220910090243/https://gorest.co.in/)**托管的公共 API 端点之一来进行这样的测试场景:

```java
String REQUEST_ENDPOINT = "https://gorest.co.in/public/v2/users"; 
```

接下来，让我们定义获取`WebClient` 类的新实例的`getWebClient()`方法:

```java
static WebClient getWebClient() {
    WebClient.Builder webClientBuilder = WebClient.builder();
    return webClientBuilder.build();
}
```

此时，我们已经准备好向`/public/v2/users` 端点发出`GET`请求。但是，我们必须将响应体作为一个`Flux<DataBuffer>`对象。因此，让我们转到下一节关于`BodyExtractors`的内容来做这件事。

## 3.`BodyExtractors`和`DataBufferUtils`

我们可以**使用`[spring-webflux](/web/20220910090243/https://www.baeldung.com/spring-webflux)`中可用的`BodyExtractors`类的`toDataBuffers()`方法将响应体提取到`Flux<DataBuffer>`T5。**

让我们继续创建`body`作为`Flux<DataBuffer>`类型的实例:

```java
Flux<DataBuffer> body = client
  .get(
  .uri(REQUEST_ENDPOINT)
    .exchangeToFlux( clientResponse -> {
        return clientResponse.body(BodyExtractors.toDataBuffers());
    });
```

接下来，当我们需要将这些`DataBuffer`流收集到单个`InputStream`流中时，一个好的策略是使用 [`PipedInputStream`](https://web.archive.org/web/20220910090243/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/io/class-use/PipedInputStream.html) 和`[PipedOutputStream](https://web.archive.org/web/20220910090243/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/io/class-use/PipedOutputStream.html)`来实现。

此外，我们打算写入`PipedOutputStream`并最终从`PipedInputStream`读取。那么，让我们看看如何创建这两个相连的流:

```java
PipedOutputStream outputStream = new PipedOutputStream();
PipedInputStream inputStream = new PipedInputStream(1024*10);
inputStream.connect(outputStream);
```

我们必须注意，默认大小是`1024`字节。然而，我们预计从`Flux<DataBuffer>`收集的结果可能会超过默认值。因此，我们需要为大小明确指定一个更大的值，在本例中是`1024*10`。

最后，我们使用`DataBufferUtils`类中可用的`write()`实用程序方法，将`body`作为发布者编写到`outputStream`:

```java
DataBufferUtils.write(body, outputStream).subscribe();
```

我们必须注意，在声明的时候，我们**将`inputStream`连接到了`outputStream`。**所以，我们好读读`inputStream`。让我们转到下一部分来看看这是如何操作的。

## 4.从`PipedInputStream`开始阅读

首先，让我们定义一个助手方法`readContent()`，将一个`InputStream`读取为一个`String`对象:

```java
String readContent(InputStream stream) throws IOException {
    StringBuffer contentStringBuffer = new StringBuffer();
    byte[] tmp = new byte[stream.available()];
    int byteCount = stream.read(tmp, 0, tmp.length);
    contentStringBuffer.append(new String(tmp));
    return String.valueOf(contentStringBuffer);
}
```

接下来，因为在不同的线程 中读取`PipedInputStream`是一个典型的 [**练习，所以让我们创建一个`readContentFromPipedInputStream()`方法，该方法通过调用`readContent()`方法在内部产生一个新的线程来将内容从`PipedInputStream`读取到一个`String`对象中:**](https://web.archive.org/web/20220910090243/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/PipedInputStream.html)

```java
String readContentFromPipedInputStream(PipedInputStream stream) throws IOException {
    StringBuffer contentStringBuffer = new StringBuffer();
    try {
        Thread pipeReader = new Thread(() -> {
            try {
                contentStringBuffer.append(readContent(stream));
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
        pipeReader.start();
        pipeReader.join();
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    } finally {
        stream.close();
    }

    return String.valueOf(contentStringBuffer);
}
```

在这个阶段，我们的代码已经准备好用于模拟。让我们来看看它的实际应用:

```java
WebClient webClient = getWebClient();
InputStream inputStream = getResponseAsInputStream(webClient, REQUEST_ENDPOINT);
Thread.sleep(3000);
String content = readContentFromPipedInputStream((PipedInputStream) inputStream);
logger.info("response content: \n{}", content.replace("}","}\n"));
```

因为我们处理的是异步系统，所以在从流中读取之前，我们将读取延迟了任意 3 秒，以便我们能够看到完整的响应。此外，在记录日志时，我们会插入一个换行符来将长输出分成多行。

最后，让我们验证代码执行生成的输出:

```java
20:45:04.120 [main] INFO com.baeldung.databuffer.DataBufferToInputStream - response content: 
[{"id":2642,"name":"Bhupen Trivedi","email":"[[email protected]](/web/20220910090243/https://www.baeldung.com/cdn-cgi/l/email-protection)","gender":"male","status":"active"}
,{"id":2637,"name":"Preity Patel","email":"[[email protected]](/web/20220910090243/https://www.baeldung.com/cdn-cgi/l/email-protection)","gender":"female","status":"inactive"}
,{"id":2633,"name":"Brijesh Shah","email":"[[email protected]](/web/20220910090243/https://www.baeldung.com/cdn-cgi/l/email-protection)","gender":"male","status":"inactive"}
...
,{"id":2623,"name":"Mohini Mishra","email":"[[email protected]](/web/20220910090243/https://www.baeldung.com/cdn-cgi/l/email-protection)","gender":"female","status":"inactive"}
] 
```

就是这样！看起来我们已经准备好了。

## 5.结论

在本文中，我们**使用管道流的概念和`BodyExtractors`和`DataBufferUtils`类**中可用的实用方法将`Flux<DataBuffer>`读入单个`InputStream`。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220910090243/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-3)