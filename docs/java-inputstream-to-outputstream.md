# 将 Java 输入流写入输出流的简单方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-inputstream-to-outputstream>

## 1.概观

在这个快速教程中，**我们将学习如何编写一个 Java `InputStream`到一个 Java `OutputStream`** 。我们将首先使用 Java 8 和 Java 9 的核心功能。然后，我们将看看几个外部库— [Guava](/web/20221208041644/https://www.baeldung.com/category/guava/) 和 [Apache Commons IO 库](/web/20221208041644/https://www.baeldung.com/apache-commons-io)。

Java 9、Guava 和 Apache Commons IO 提供的实用程序方法不会刷新或关闭流。所以，我们需要使用 [`try-with-resources`](/web/20221208041644/https://www.baeldung.com/java-try-with-resources) 或 [`finally`](/web/20221208041644/https://www.baeldung.com/java-finally-keyword) 块来管理这些资源。

## 2.使用 Java 8

首先，我们将开始创建一个简单的方法，使用普通的 Java 将内容从`InputStream`复制到`OutputStream`:

```java
void copy(InputStream source, OutputStream target) throws IOException {
    byte[] buf = new byte[8192];
    int length;
    while ((length = source.read(buf)) != -1) {
        target.write(buf, 0, length);
    }
}
```

这段代码非常简单——我们只是读入一些字节，然后将它们写出来。

## 3.使用 Java 9

**Java 9 为这个任务**提供了一个实用方法`InputStream.transferTo()`。

让我们看看如何使用`transferTo()`方法:

```java
@Test
public void givenUsingJavaNine_whenCopyingInputStreamToOutputStream_thenCorrect() throws IOException {
    String initialString = "Hello World!";

    try (InputStream inputStream = new ByteArrayInputStream(initialString.getBytes());
         ByteArrayOutputStream targetStream = new ByteArrayOutputStream()) {
        inputStream.transferTo(targetStream);

        assertEquals(initialString, new String(targetStream.toByteArray()));
    }
}
```

注意，当处理文件流时，**使用`[Files.copy()](/web/20221208041644/https://www.baeldung.com/convert-input-stream-to-a-file)`比使用`transferTo()`** 方法更有效。

## 4.用番石榴

接下来，让我们看看我们将如何**使用番石榴的效用方法`ByteStreams.copy()`** 。

我们需要在我们的`pom.xml`中包含[番石榴属地](https://web.archive.org/web/20221208041644/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22):

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

让我们创建一个简单的测试用例来展示我们如何使用`ByteStreams`来复制数据:

```java
@Test
public void givenUsingGuava_whenCopyingInputStreamToOutputStream_thenCorrect() throws IOException {
    String initialString = "Hello World!";

    try (InputStream inputStream = new ByteArrayInputStream(initialString.getBytes());
         ByteArrayOutputStream targetStream = new ByteArrayOutputStream()) {
        ByteStreams.copy(inputStream, targetStream);

        assertEquals(initialString, new String(targetStream.toByteArray()));
    }
}
```

## 5.使用 Commons IO

最后，让我们看看如何使用 Commons IO `IOUtils.copy()` 方法来完成这个任务。

当然，我们需要将 [commons-io](https://web.archive.org/web/20221208041644/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22) 依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

让我们使用`IOUtils`创建一个简单的测试用例，将数据从输入流复制到输出流:

```java
@Test
public void givenUsingCommonsIO_whenCopyingInputStreamToOutputStream_thenCorrect() throws IOException {
    String initialString = "Hello World!";

    try (InputStream inputStream = new ByteArrayInputStream(initialString.getBytes());
         ByteArrayOutputStream targetStream = new ByteArrayOutputStream()) {
        IOUtils.copy(inputStream, targetStream);

        assertEquals(initialString, new String(targetStream.toByteArray()));
    }
}
```

注意:Commons IO 提供了处理`InputStream`和`OutputStream`的其他方法。每当需要复制 2 GB 或更多数据时，都应该使用 **`IOUtils.copyLarge()`。**

## 6.结论

在本文中，我们探索了将数据从一个`InputStream`复制到一个`OutputStream`**的简单方法。**

GitHub 上的[提供了这些例子的实现。](https://web.archive.org/web/20221208041644/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)