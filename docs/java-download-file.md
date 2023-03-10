# 用 Java 从 URL 下载文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-download-file>

## 1。概述

在本教程中，我们将看到几种下载文件的方法。

我们将讨论一些例子，从 Java IO 的基本用法到 NIO 包，以及一些常见的库，如 AsyncHttpClient 和 Apache Commons IO。

最后，我们将讨论如果我们的连接在整个文件被读取之前失败，我们如何恢复下载。

## 2。使用 Java IO

我们可以用来下载文件的最基本的 API 是 [Java IO](https://web.archive.org/web/20220717111859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/package-summary.html) 。我们可以使用`URL `类打开一个到我们想要下载的文件的连接。

为了有效地读取文件，我们将使用`openStream()`方法获得一个`InputStream`:

```java
BufferedInputStream in = new BufferedInputStream(new URL(FILE_URL).openStream())
```

**从`InputStream`开始读取时，建议将其包裹在`BufferedInputStream`中以提高性能。**

性能的提高来自缓冲。当使用`read()`方法一次读取一个字节时，每个方法调用都意味着对底层文件系统的系统调用。当 JVM 调用`read()`系统调用时，程序执行上下文从用户模式切换到内核模式，然后再切换回来。

从性能角度来看，这种上下文切换代价很高。当我们读取大量字节时，应用程序的性能会很差，这是由于涉及到大量的上下文切换。

为了将从 URL 读取的字节写入我们的本地文件，我们将使用来自`FileOutputStream `类的`write()`方法:

```java
try (BufferedInputStream in = new BufferedInputStream(new URL(FILE_URL).openStream());
  FileOutputStream fileOutputStream = new FileOutputStream(FILE_NAME)) {
    byte dataBuffer[] = new byte[1024];
    int bytesRead;
    while ((bytesRead = in.read(dataBuffer, 0, 1024)) != -1) {
        fileOutputStream.write(dataBuffer, 0, bytesRead);
    }
} catch (IOException e) {
    // handle exception
}
```

当使用`BufferedInputStream`时，`read()`方法将读取我们为缓冲区大小设置的字节数。在我们的例子中，我们已经通过一次读取 1024 字节的块来做到这一点，所以`BufferedInputStream`是不必要的。

上面的例子非常冗长，但幸运的是，从 Java 7 开始，我们有了包含用于处理 IO 操作的 helper 方法的`Files`类。

**我们可以使用`Files.copy()`** 方法从`InputStream`中读取所有的字节，并将它们复制到一个本地文件中:

```java
InputStream in = new URL(FILE_URL).openStream();
Files.copy(in, Paths.get(FILE_NAME), StandardCopyOption.REPLACE_EXISTING);
```

我们的代码运行良好，但可以改进。它的主要缺点是字节被缓冲到内存中。

幸运的是，Java 为我们提供了 NIO 包，它提供了在两个`Channels`之间直接传输字节而无需缓冲的方法。

我们将在下一节详细介绍。

## 3。使用 NIO

Java NIO 包提供了在两个`Channels`之间传输字节的可能性，而无需将它们缓冲到应用程序内存中。

为了从我们的 URL 中读取文件，我们将从`URL `流中创建一个新的`ReadableByteChannel`:

```java
ReadableByteChannel readableByteChannel = Channels.newChannel(url.openStream());
```

从`ReadableByteChannel`读取的字节将被传输到与将要下载的文件对应的`FileChannel`:

```java
FileOutputStream fileOutputStream = new FileOutputStream(FILE_NAME);
FileChannel fileChannel = fileOutputStream.getChannel();
```

我们将使用来自`ReadableByteChannel`类的`transferFrom()`方法从给定的 URL 下载字节到我们的`FileChannel`:

```java
fileOutputStream.getChannel()
  .transferFrom(readableByteChannel, 0, Long.MAX_VALUE);
```

`transferTo()`和`transferFrom()`方法比简单地使用缓冲区从流中读取更有效。根据底层的操作系统，**数据可以直接从文件系统缓存传输到我们的文件，而不需要将任何字节复制到应用程序内存中。**

**在 Linux 和 UNIX 系统上，这些方法使用了`zero-copy`技术，减少了内核模式和用户模式之间的上下文切换次数。**

## 4。使用库

在上面的例子中，我们已经看到了如何通过使用 Java 核心功能从 URL 下载内容。

当不需要性能调整时，我们还可以利用现有库的功能来简化我们的工作。

例如，在真实的场景中，我们需要我们的下载代码是异步的。

我们可以将所有的逻辑打包到一个`Callable`中，或者我们可以使用一个现有的库来完成这个任务。

### 4.1。异步客户端

[AsyncHttpClient](/web/20220717111859/https://www.baeldung.com/async-http-client) 是一个使用 Netty 框架执行异步 HTTP 请求的流行库。我们可以使用它来执行对文件 URL 的 GET 请求，并获取文件内容。

首先，我们需要创建一个 HTTP 客户端:

```java
AsyncHttpClient client = Dsl.asyncHttpClient();
```

下载的内容将被放入`FileOutputStream`:

```java
FileOutputStream stream = new FileOutputStream(FILE_NAME);
```

接下来，我们创建一个 HTTP GET 请求并注册一个`AsyncCompletionHandler`处理程序来处理下载的内容:

```java
client.prepareGet(FILE_URL).execute(new AsyncCompletionHandler<FileOutputStream>() {

    @Override
    public State onBodyPartReceived(HttpResponseBodyPart bodyPart) 
      throws Exception {
        stream.getChannel().write(bodyPart.getBodyByteBuffer());
        return State.CONTINUE;
    }

    @Override
    public FileOutputStream onCompleted(Response response) 
      throws Exception {
        return stream;
    }
})
```

注意，我们已经覆盖了`onBodyPartReceived()`方法。**默认实现将接收到的 HTTP 块累积到一个`ArrayList`中。**这可能导致高内存消耗，或者在试图下载大文件时出现`OutOfMemory`异常。

不是将每个`HttpResponseBodyPart`累积到内存中，**我们使用一个`FileChannel`将字节直接写入我们的本地文件。**我们将使用`getBodyByteBuffer()`方法通过`ByteBuffer`访问身体部分内容。

**`ByteBuffer` s 的优点是内存分配在 JVM 堆之外，所以不会影响我们的应用内存。**

### 4.2 .Apache common me〔t1〕

另一个经常使用的 IO 操作库是 [Apache Commons IO](https://web.archive.org/web/20220717111859/https://commons.apache.org/proper/commons-io/) 。我们可以从 Javadoc 中看到，有一个名为 [`FileUtils`](https://web.archive.org/web/20220717111859/https://commons.apache.org/proper/commons-io/apidocs/index.html?org/apache/commons/io/FileUtils.html) 的实用程序类，我们用它来完成一般的文件操作任务。

要从 URL 下载文件，我们可以使用这个命令行程序:

```java
FileUtils.copyURLToFile(
  new URL(FILE_URL), 
  new File(FILE_NAME), 
  CONNECT_TIMEOUT, 
  READ_TIMEOUT);
```

从性能的角度来看，这段代码与第 2 节中的代码相同。

底层代码使用相同的概念，在一个循环中从一个`InputStream`读取一些字节，并将它们写入一个`OutputStream`。

一个区别是这里的`URLConnection`类用于控制连接超时，这样下载就不会长时间阻塞:

```java
URLConnection connection = source.openConnection();
connection.setConnectTimeout(connectionTimeout);
connection.setReadTimeout(readTimeout);
```

## 5。可恢复下载

考虑到互联网连接有时会失败，能够恢复下载是有用的，而不是从零字节重新下载文件。

让我们重写前面的第一个例子来添加这个功能。

首先要知道的是**我们可以通过使用 HTTP HEAD 方法**从给定的 URL 读取文件的大小，而不需要实际下载它:

```java
URL url = new URL(FILE_URL);
HttpURLConnection httpConnection = (HttpURLConnection) url.openConnection();
httpConnection.setRequestMethod("HEAD");
long removeFileSize = httpConnection.getContentLengthLong();
```

现在我们有了文件的总内容大小，我们可以检查我们的文件是否被部分下载。

如果是这样，我们将从磁盘上记录的最后一个字节开始继续下载:

```java
long existingFileSize = outputFile.length();
if (existingFileSize < fileLength) {
    httpFileConnection.setRequestProperty(
      "Range", 
      "bytes=" + existingFileSize + "-" + fileLength
    );
}
```

这里**我们已经配置了`URLConnection`来请求特定范围内的文件字节。**该范围将从最后下载的字节开始，到对应于远程文件大小的字节结束。

使用`Range`头的另一种常见方式是通过设置不同的字节范围来下载文件。例如，要下载 2 KB 的文件，我们可以使用范围 0–1024 和 1024–2048。

与第 2 节中的代码的另一个细微区别是， **`FileOutputStream`是在`append`参数设置为 true** 的情况下打开的:

```java
OutputStream os = new FileOutputStream(FILE_NAME, true);
```

在我们做了这个更改之后，剩下的代码与第 2 节中的代码完全相同。

## 6。结论

在本文中，我们已经看到了用 Java 从 URL 下载文件的几种方法。

最常见的实现是在执行读/写操作时缓冲字节。即使对于大文件，这种实现也是安全的，因为我们不会将整个文件加载到内存中。

我们还看到了如何使用 Java NIO `Channels`实现零拷贝下载。这很有用，因为它最大限度地减少了读取和写入字节时的上下文切换次数，并且通过使用直接缓冲区，字节不会加载到应用程序内存中。

此外，因为下载文件通常是通过 HTTP 完成的，所以我们展示了如何使用 AsyncHttpClient 库来实现这一点。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220717111859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)