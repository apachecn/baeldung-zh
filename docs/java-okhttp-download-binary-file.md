# 使用 OkHttp 下载一个二进制文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-okhttp-download-binary-file>

## 1.概观

本教程将给出一个如何使用 [OkHttp 库](/web/20220524121329/https://www.baeldung.com/guide-to-okhttp)T3**下载二进制文件的实例。**

## 2.Maven 依赖性

我们将从添加基础库 [okhttp](https://web.archive.org/web/20220524121329/https://search.maven.org/artifact/com.squareup.okhttp3/okhttp) 依赖关系开始:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.1</version>
</dependency>
```

然后，**如果我们想为用 OkHttp 库实现的模块编写一个集成测试，我们可以使用 [mockwebserver](https://web.archive.org/web/20220524121329/https://search.maven.org/artifact/com.squareup.okhttp3/mockwebserver) 库**。这个库有模仿服务器及其响应的工具:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>4.9.1</version>
    <scope>test</scope>
</dependency>
```

## 3.请求二进制文件

我们将首先实现一个类，**接收一个 URL** 作为参数，从那里下载文件，**为这个 URL 创建并执行一个 HTTP 请求**。

为了使类可测试，我们将**在构造函数中注入`OkHttpClient`和作者**:

```java
public class BinaryFileDownloader implements AutoCloseable {

    private final OkHttpClient client;
    private final BinaryFileWriter writer;

    public BinaryFileDownloader(OkHttpClient client, BinaryFileWriter writer) {
        this.client = client;
        this.writer = writer;
    }
}
```

接下来，我们将实现**从 URL** 下载文件的方法:

```java
public long download(String url) throws IOException {
    Request request = new Request.Builder().url(url).build();
    Response response = client.newCall(request).execute();
    ResponseBody responseBody = response.body();
    if (responseBody == null) {
        throw new IllegalStateException("Response doesn't contain a file");
    }
    double length = Double.parseDouble(Objects.requireNonNull(response.header(CONTENT_LENGTH, "1")));
    return writer.write(responseBody.byteStream(), length);
}
```

下载文件的过程有四个步骤。使用 URL 创建请求。执行请求并接收响应。获取响应的主体，如果是`null.` 则失败，将响应主体的`bytes`写入文件。

## 4.将响应写入本地文件

为了将从响应中接收到的`bytes`写入本地文件，我们将实现一个`BinaryFileWriter`类，其中**将一个`InputStream`和一个`OutputStream`作为输入，并将内容从 [`InputStream`](/web/20220524121329/https://www.baeldung.com/convert-input-stream-to-a-file) 复制到`OutputStream`** 。

`OutputStream`将被注入到构造函数中，以便该类是可测试的:

```java
public class BinaryFileWriter implements AutoCloseable {

    private final OutputStream outputStream;

    public BinaryFileWriter(OutputStream outputStream) {
        this.outputStream = outputStream;
    }
}
```

我们现在将实现将内容从`InputStream`复制到`OutputStream`的方法。该方法首先用一个`BufferedInputStream`包装`InputStream`，这样我们可以一次读取更多的`bytes`。然后，我们准备一个数据缓冲区，临时存储来自`InputStream`的`bytes`。

最后，我们将把缓冲的数据`write`到`OutputStream`。只要`InputStream`有数据要读取，我们就这样做:

```java
public long write(InputStream inputStream) throws IOException {
    try (BufferedInputStream input = new BufferedInputStream(inputStream)) {
        byte[] dataBuffer = new byte[CHUNK_SIZE];
        int readBytes;
        long totalBytes = 0;
        while ((readBytes = input.read(dataBuffer)) != -1) {
            totalBytes += readBytes;
            outputStream.write(dataBuffer, 0, readBytes);
        }
        return totalBytes;
    }
}
```

## 5.获取文件下载进度

在某些情况下，我们可能想告诉用户文件下载的进度。

我们首先需要创建一个[功能接口](/web/20220524121329/https://www.baeldung.com/java-8-functional-interfaces):

```java
public interface ProgressCallback {
    void onProgress(double progress);
}
```

然后，我们将在`BinaryFileWriter`类中使用它。这将在每一步给出下载者到目前为止所写的总数`bytes`。

首先，我们将**将`ProgressCallback`作为一个字段添加到 writer 类**中。然后，我们将把`write`方法更新为**，接收响应**的`length`作为参数。这将帮助我们计算进度。

然后，我们将**调用`onProgress`方法，并根据目前编写的`totalBytes`和`length:`计算进度**

```java
public class BinaryFileWriter implements AutoCloseable {
    private final ProgressCallback progressCallback;
    public long write(InputStream inputStream, double length) {
        //...
        progressCallback.onProgress(totalBytes / length * 100.0);
    }
}
```

最后，我们将用总响应长度将`BinaryFileDownloader`类更新为**调用`write`方法。我们将从`Content-Length`头中获取响应长度，然后将其传递给`write`方法:**

```java
public class BinaryFileDownloader {
    public long download(String url) {
        double length = getResponseLength(response);
        return write(responseBody, length);
    }
    private double getResponseLength(Response response) {
        return Double.parseDouble(Objects.requireNonNull(response.header(CONTENT_LENGTH, "1")));
    }
}
```

## 6.结论

在本文中，我们实现了一个简单而实用的例子**，使用 OkHttp 库**从 URL 下载二进制文件。

关于文件下载应用程序的完整实现，以及单元测试，请查看 GitHub 上的项目[。](https://web.archive.org/web/20220524121329/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)