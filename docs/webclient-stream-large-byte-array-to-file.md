# 使用 WebClient 将大字节[]流式传输到文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/webclient-stream-large-byte-array-to-file>

## 1.介绍

在这个快速教程中，我们将使用 [`WebClient`](/web/20221218213041/https://www.baeldung.com/spring-5-webclient) 从服务器传输一个大文件。为了说明，我们将创建一个简单的[控制器](/web/20221218213041/https://www.baeldung.com/spring-controllers)和两个客户端。**最终，我们将学习如何以及何时使用 [Spring](/web/20221218213041/https://www.baeldung.com/spring-tutorial) 的 [`DataBuffer`](/web/20221218213041/https://www.baeldung.com/spring-reactive-read-flux-into-inputstream) 和 [`DataBufferUtils`](/web/20221218213041/https://www.baeldung.com/spring-reactive-read-flux-into-inputstream#bodyextractors-databufferutils) 。**

## 2.我们使用简单服务器的场景

**我们将从一个简单的[控制器开始，用于下载](/web/20221218213041/https://www.baeldung.com/spring-controller-return-image-file)任意文件。**首先，我们将构造一个 [`FileSystemResource`](https://web.archive.org/web/20221218213041/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/FileSystemResource.html) ，传递一个文件 [`Path`](/web/20221218213041/https://www.baeldung.com/java-nio-2-path) ，然后将它作为一个体包装到我们的 [`ResponseEntity`](/web/20221218213041/https://www.baeldung.com/spring-response-entity) :

```java
@RestController
@RequestMapping("/large-file")
public class LargeFileController {

    @GetMapping
    ResponseEntity<Resource> get() {
        return ResponseEntity.ok()
          .body(new FileSystemResource(Paths.get("/tmp/large.dat")));
    }
}
```

其次，我们需要生成我们引用的文件。**由于内容对于理解教程并不重要，我们将使用`fallocate`在磁盘上保留一个指定的大小，而不写任何东西**。所以，让我们[通过运行这个命令来创建我们的大文件](/web/20221218213041/https://www.baeldung.com/linux/create-large-file):

```java
fallocate -l 128M /tmp/large.dat
```

最后，我们有一个客户可以下载的文件。所以，我们准备开始给客户写信了。

## 3.`WebClient`与`ExchangeStrategies`用于大文件

我们将从一个简单但有限的 [`WebClient`](/web/20221218213041/https://www.baeldung.com/spring-5-webclient) 开始下载我们的文件。**我们将使用 [`ExchangeStrategies`](https://web.archive.org/web/20221218213041/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/ExchangeStrategies.html) 来提高可用于`exchange()`操作的内存限制。**通过这种方式，我们可以操作更多的字节，但是我们仍然受限于 JVM 可用的最大内存。让我们使用`bodyToMono()`从服务器获得一个`Mono<byte[]>`:

```java
public class LimitedFileDownloadWebClient {

    public static long fetch(WebClient client, String destination) {
        Mono<byte[]> mono = client.get()
          .retrieve()
          .bodyToMono(byte[].class);

        byte[] bytes = mono.block();

        Path path = Paths.get(destination);
        Files.write(path, bytes);
        return bytes.length;
    }

    // ...
}
```

换句话说，**我们正在将整个响应内容检索到一个`byte[]`** 中。之后，我们[将这些字节写入](/web/20221218213041/https://www.baeldung.com/java-write-byte-array-file#java-nio)到我们的`path`中，并返回下载的字节数。让我们创建一个 [`main()`](/web/20221218213041/https://www.baeldung.com/java-main-method) 方法来测试它:

```java
public static void main(String... args) {
    String baseUrl = args[0];
    String destination = args[1];

    WebClient client = WebClient.builder()
      .baseUrl(baseUrl)
      .exchangeStrategies(useMaxMemory())
      .build();

    long bytes = fetch(client, destination);
    System.out.printf("downloaded %d bytes", bytes);
}
```

此外，我们需要两个[参数](/web/20221218213041/https://www.baeldung.com/java-command-line-arguments):下载 URL 和一个`destination`将其保存在本地。**为了避免在我们的`client`中出现 [`DataBufferLimitException`](/web/20221218213041/https://www.baeldung.com/spring-webflux-databufferlimitexception) ，让我们配置一个交换策略来限制可加载到内存中的字节数。**我们将使用 [`Runtime`](/web/20221218213041/https://www.baeldung.com/java-heap-memory-api#3-maximum-memory) 获取为我们的应用程序配置的总内存，而不是定义一个固定的大小。**注意，不建议这样做，这只是为了演示目的**:

```java
private static ExchangeStrategies useMaxMemory() {
    long totalMemory = Runtime.getRuntime().maxMemory();

    return ExchangeStrategies.builder()
      .codecs(configurer -> configurer.defaultCodecs()
        .maxInMemorySize((int) totalMemory)
      )
      .build();
}
```

澄清一下，交换策略定制了我们的`client`处理请求的方式。在这种情况下，我们使用构建器中的`codecs()`方法，所以我们不替换任何默认设置。

### 3.1.用内存调整运行我们的客户端

随后，我们将把我们的项目打包成`/tmp/app.jar`中的 [jar](/web/20221218213041/https://www.baeldung.com/java-create-jar) ，并在 [`localhost:8081`](/web/20221218213041/https://www.baeldung.com/spring-boot-change-port) 上运行我们的服务器。然后，让我们定义一些变量，并从命令行运行我们的客户端:

```java
limitedClient='com.baeldung.streamlargefile.client.LimitedFileDownloadWebClient' 
endpoint='http://localhost:8081/large-file' 
java -Xmx256m -cp /tmp/app.jar $limitedClient $endpoint /tmp/download.dat 
```

注意，我们允许应用程序使用两倍于 128M 文件大小的内存。事实上，我们将下载我们的文件并得到以下输出:

```java
downloaded 134217728 bytes
```

另一方面，**如果我们不分配足够的内存，我们会得到一个 [`OutOfMemoryError`](/web/20221218213041/https://www.baeldung.com/java-permgen-space-error)** :

```java
$ java -Xmx64m -cp /tmp/app.jar $limitedClient $endpoint /tmp/download.dat
reactor.netty.ReactorNetty$InternalNettyException: java.lang.OutOfMemoryError: Direct buffer memory 
```

这种方法不依赖于 Spring 核心工具。但是，它是有限的，因为**我们不能下载任何大小接近我们的应用程序**的最大内存的文件。

## 4.`WebClient`对于任何文件大小用`DataBuffer`

**一个更安全的方法是使用`DataBuffer`和 [`DataBufferUtils`](/web/20221218213041/https://www.baeldung.com/spring-reactive-read-flux-into-inputstream#bodyextractors-databufferutils) 来流传输我们的下载，这样整个文件就不会被加载到内存中。**然后，这一次，我们将使用`bodyToFlux()`来检索一个`Flux<DataBuffer>`，将其写入我们的`path`，并返回其字节大小:

```java
public class LargeFileDownloadWebClient {

    public static long fetch(WebClient client, String destination) {
        Flux<DataBuffer> flux = client.get()
          .retrieve()
          .bodyToFlux(DataBuffer.class);

        Path path = Paths.get(destination);
        DataBufferUtils.write(flux, path)
          .block();

        return Files.size(path);
    }

    // ...
}
```

**最后，让我们编写 main 方法来接收我们的参数，创建一个`WebClient`，并获取我们的文件**:

```java
public static void main(String... args) {
    String baseUrl = args[0];
    String destination = args[1];

    WebClient client = WebClient.create(baseUrl);

    long bytes = fetch(client, destination);
    System.out.printf("downloaded %d bytes", bytes);
}
```

仅此而已。**这种方法更加通用，因为我们不依赖于文件或内存大小。**让我们将最大内存设置为文件大小的四分之一，并使用之前相同的`endpoint`运行它:

```java
client='com.baeldung.streamlargefile.client.LargeFileDownloadWebClient'
java -Xmx32m -cp /tmp/app.jar $client $endpoint /tmp/download.dat
```

最终，我们将获得一个成功的输出，即使我们的应用程序的总内存小于我们的文件大小:

```java
downloaded 134217728 bytes
```

## 5.结论

在本文中，我们学习了使用`WebClient`下载任意大文件的不同方法。首先，我们学习了如何为我们的`WebClient`操作定义可用的内存量。然后，我们看到了这种方法的弊端。**最重要的是，我们学会了如何让我们的客户高效地使用内存。**

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221218213041/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-client-2)