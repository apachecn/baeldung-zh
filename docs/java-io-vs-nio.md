# Java IO 与 NIO

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-io-vs-nio>

## 1.概观

处理输入和输出是 Java 程序员的常见任务。在本教程中，我们将看看**最初的 [`java.io`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/package-summary.html) ( [IO](/web/20220628100032/https://www.baeldung.com/java-io) )库和更新的 [`java.nio`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/package-summary.html) ( [NIO](/web/20220628100032/https://www.baeldung.com/tag/java-nio/) )库**，以及它们在跨网络通信时有何不同。

## 2.关键特征

让我们先来看看这两个包的关键特性。

### 2.1.IO-`java.io`

Java 1.0 中引入了 **`java.io`包**，Java 1.1 中引入了`Reader`。它提供:

*   `InputStream`和[`OutputStream`](/web/20220628100032/https://www.baeldung.com/java-outputstream)——每次提供一个字节的数据
*   `Reader`和`Writer`–流的便利包装器
*   阻塞模式–等待完整的消息

### 二点二。九—`java.nio`

在 Java 1.4 中引入了 **`java.nio`包，并在 Java 1.7 (NIO.2)** 中用[增强的文件操作](/web/20220628100032/https://www.baeldung.com/java-nio-2-file-api)和一个`[ASynchronousSocketChannel](/web/20220628100032/https://www.baeldung.com/java-nio2-async-socket-channel)`更新了**。它提供:**

*   `Buffer` `–` 一次读取大块数据
*   `CharsetDecoder`–用于将原始字节映射到可读字符或从可读字符映射到原始字节
*   `Channel`–用于与外界沟通
*   `[Selector](/web/20220628100032/https://www.baeldung.com/java-nio-selector)`–启用`SelectableChannel`上的多路复用，并提供对任何准备好进行 I/O 的`Channel`的访问
*   非阻塞模式——读取任何准备好的内容

现在，让我们看看当我们向服务器发送数据或读取其响应时，如何使用这些包。

## 3.配置我们的测试服务器

这里我们将使用 [WireMock](/web/20220628100032/https://www.baeldung.com/introduction-to-wiremock) 来模拟另一个服务器，这样我们就可以独立运行我们的测试。

我们将配置它来监听我们的请求，并向我们发送响应，就像真正的 web 服务器一样。我们还将使用一个动态端口，这样我们就不会与本地机器上的任何服务发生冲突。

让我们为 WireMock 添加 Maven 依赖项，作用域为`test`:

```java
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.26.3</version>
    <scope>test</scope>
</dependency>
```

在一个测试类中，让我们定义一个 JUnit `@Rule`来在一个空闲端口上启动 WireMock up。然后，当我们请求预定义的资源时，我们将对其进行配置，以返回 HTTP 200 响应，消息体为 JSON 格式的一些文本:

```java
@Rule public WireMockRule wireMockRule = new WireMockRule(wireMockConfig().dynamicPort());

private String REQUESTED_RESOURCE = "/test.json";

@Before
public void setup() {
    stubFor(get(urlEqualTo(REQUESTED_RESOURCE))
      .willReturn(aResponse()
      .withStatus(200)
      .withBody("{ \"response\" : \"It worked!\" }")));
}
```

现在我们已经设置了模拟服务器，我们准备运行一些测试。

## 4.阻塞 IO-`java.io`

我们通过从一个网站上读取一些数据来看看最初的阻塞 IO 模型是如何工作的。我们将使用一个 [`java.net.Socket`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/Socket.html) 来访问操作系统的一个端口。

### 4.1.发送请求

在这个例子中，我们将创建一个 GET 请求来检索我们的资源。首先，让我们**创建一个`Socket`来访问我们的 WireMock 服务器正在监听的端口**:

```java
Socket socket = new Socket("localhost", wireMockRule.port())
```

对于普通的 HTTP 或 HTTPS 通信，端口应该是 80 或 443。然而，在这种情况下，我们使用`wireMockRule.port()` 来访问我们之前设置的动态端口。

现在让我们用**在**的插座上打开一个`OutputStream`，用一个`OutputStreamWriter`包裹，然后把它传给一个`PrintWriter`来写我们的消息。让我们确保刷新缓冲区，以便发送我们的请求:

```java
OutputStream clientOutput = socket.getOutputStream();
PrintWriter writer = new PrintWriter(new OutputStreamWriter(clientOutput));
writer.print("GET " + TEST_JSON + " HTTP/1.0\r\n\r\n");
writer.flush();
```

### 4.2.等待回应

让我们**在 socket** 上打开一个`InputStream` **来访问响应，用一个`[BufferedReader](/web/20220628100032/https://www.baeldung.com/java-buffered-reader)`读取流，并存储在一个`StringBuilder`中:**

```java
InputStream serverInput = socket.getInputStream();
BufferedReader reader = new BufferedReader(new InputStreamReader(serverInput));
StringBuilder ourStore = new StringBuilder();
```

让我们使用`reader.readLine()`来阻塞，等待一个完整的行，然后将该行追加到我们的存储中。我们将继续读取，直到得到一个表示流结束的`null,` :

```java
for (String line; (line = reader.readLine()) != null;) {
   ourStore.append(line);
   ourStore.append(System.lineSeparator());
}
```

## 5.非阻塞 IO–`java.nio`

现在，让我们看一下 **`nio` 包的非阻塞 IO 模型**如何与同一个例子一起工作。

这一次，我们将**创造一个 [`java.nio.channel`。`SocketChannel`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/SocketChannel.html) 访问我们服务器上的端口**而不是`java.net.Socket`，并传递给它一个`InetSocketAddress`。

### 5.1.发送请求

首先，让我们打开我们的`SocketChannel`:

```java
InetSocketAddress address = new InetSocketAddress("localhost", wireMockRule.port());
SocketChannel socketChannel = SocketChannel.open(address);
```

现在，让我们用一个标准的 UTF-8 `Charset`来编码和书写我们的信息:

```java
Charset charset = StandardCharsets.UTF_8;
socket.write(charset.encode(CharBuffer.wrap("GET " + REQUESTED_RESOURCE + " HTTP/1.0\r\n\r\n")));
```

### 5.2.阅读回复

发送请求后，我们可以使用原始缓冲区以非阻塞模式读取响应。

既然我们要处理文本，我们需要一个 [`ByteBuffer`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/ByteBuffer.html) 用于原始字节，一个 [`CharBuffer`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/CharBuffer.html) 用于转换后的字符(由 [`CharsetDecoder`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/CharsetDecoder.html) 辅助):

```java
ByteBuffer byteBuffer = ByteBuffer.allocate(8192);
CharsetDecoder charsetDecoder = charset.newDecoder();
CharBuffer charBuffer = CharBuffer.allocate(8192);
```

如果数据以多字节字符集发送，我们的`CharBuffer`将会有剩余空间。

注意，如果我们需要特别快的性能，我们可以使用`ByteBuffer.allocateDirect()`在本机内存中创建一个 [`MappedByteBuffer`](https://web.archive.org/web/20220628100032/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/MappedByteBuffer.html) 。然而，在我们的例子中，使用标准堆中的`allocate()` 已经足够快了。

**在处理缓冲区时，我们需要知道缓冲区有多大**(容量)**(当前位置)**以及我们能走多远**(极限)。**

 **因此，让我们从我们的`SocketChannel` 中读取**，将它传递给我们的`ByteBuffer`来存储我们的数据。我们来自`SocketChannel`的`read`将以我们`ByteBuffer`的**当前位置设置为下一个要写入**(就在写入的最后一个字节之后)**的字节结束，但其限制不变**:**

```java
socketChannel.read(byteBuffer)
```

**我们的 *SocketChannel.read()* 返回读取的字节数**可以写入我们的缓冲区。如果套接字断开连接，则该值为-1。

当我们的缓冲区没有任何剩余空间时，因为我们还没有处理完它的所有数据，那么`SocketChannel.read()` 将返回零字节读取，但是我们的`buffer.position()`仍然大于零。

为了确保我们从缓冲区中的正确位置开始读取，我们将使用 **`Buffer.flip`()将`ByteBuffer`的当前位置设置为零，并将其限制为由`SocketChannel`** `.`写入的最后一个字节。然后，我们将使用我们的`storeBufferContents`方法保存缓冲区内容，我们将在后面讨论。最后，我们将使用`buffer.compact()`压缩缓冲区，并设置当前位置，为下一次从`SocketChannel.` 读取做好准备

由于我们的数据可能是部分到达的，让我们用终止条件将我们的缓冲区读取代码包装在一个循环中，以检查我们的套接字是否仍然连接，或者我们是否已经断开连接，但仍然有数据留在我们的缓冲区中:

```java
while (socketChannel.read(byteBuffer) != -1 || byteBuffer.position() > 0) {
    byteBuffer.flip();
    storeBufferContents(byteBuffer, charBuffer, charsetDecoder, ourStore);
    byteBuffer.compact();
}
```

让我们不要忘记`close()`我们的套接字(除非我们在 try-with-resources 块中打开它):

```java
socketChannel.close();
```

### 5.3.从我们的缓冲区存储数据

来自服务器的响应将包含头，这可能会使数据量超过我们的缓冲区的大小。因此，当消息到达时，我们将使用一个`StringBuilder`来构建完整的消息。

为了存储我们的消息，我们首先在我们的`CharBuffer` 中将原始字节解码成字符。然后我们将翻转指针，这样我们就可以读取我们的字符数据，并将它附加到我们的可扩展的`StringBuilder.`中。最后，我们将清除`CharBuffer`，为下一个写/读周期做好准备。

现在，让我们实现我们完整的`storeBufferContents()`方法，在我们的缓冲区中传递`CharsetDecoder`和`StringBuilder`:

```java
void storeBufferContents(ByteBuffer byteBuffer, CharBuffer charBuffer, 
  CharsetDecoder charsetDecoder, StringBuilder ourStore) {
    charsetDecoder.decode(byteBuffer, charBuffer, true);
    charBuffer.flip();
    ourStore.append(charBuffer);
    charBuffer.clear();
}
```

## 6.结论

在本文中，我们已经看到了**的原始 `java.io`模型如何阻塞**，等待请求，并使用`Stream` s 来操作它接收到的数据。

相比之下，**`java.nio`库允许使用`Buffer`和`Channel`进行无阻塞通信**，并且可以提供直接内存访问以获得更快的性能。然而，随着这种速度而来的是处理缓冲区的额外复杂性。

和往常一样，这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628100032/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2)**