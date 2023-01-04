# Java 输入流到字节数组和字节缓冲区

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-input-stream-to-array-of-bytes>

## 1。概述

在这个快速教程中，我们将看看如何将一个`InputStream`转换成一个`byte[]` 和`ByteBuffer`——首先使用普通 Java，然后使用 Guava 和 Commons IO。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 延伸阅读:

## [春天的流水简介](/web/20221013193919/https://www.baeldung.com/spring-stream-utils)

Discover Spring's StreamUtils class.[Read more](/web/20221013193919/https://www.baeldung.com/spring-stream-utils) →

## [Java 序列化简介](/web/20221013193919/https://www.baeldung.com/java-serialization)

We learn how to serialize and deserialize objects in Java.[Read more](/web/20221013193919/https://www.baeldung.com/java-serialization) →

## [Java 字符串转换](/web/20221013193919/https://www.baeldung.com/java-string-conversions)

Quick and practical examples focused on converting String objects to different data types in Java.[Read more](/web/20221013193919/https://www.baeldung.com/java-string-conversions) →

## 2。转换成字节数组

让我们看看从简单的输入流中获得一个字节数组`.` 字节数组的重要方面是**，它支持对存储在内存中的每个 8 位(一个字节)值进行索引(快速)访问**。因此，您可以操纵这些字节来控制每一位。我们将看看如何将一个简单的输入流转换成一个`byte[]`——首先使用普通的 Java，然后使用[番石榴](https://web.archive.org/web/20221013193919/https://github.com/google/guava)和 [Apache Commons IO](https://web.archive.org/web/20221013193919/https://commons.apache.org/proper/commons-io/) 。

### 2.1。使用普通 Java 进行转换

让我们从一个专注于处理固定大小流的 Java 解决方案开始:

```java
@Test
public void givenUsingPlainJavaOnFixedSizeStream_whenConvertingAnInputStreamToAByteArray_thenCorrect() 
  throws IOException {
    InputStream is = new ByteArrayInputStream(new byte[] { 0, 1, 2 });
    byte[] targetArray = new byte[is.available()];

    is.read(targetArray);
}
```

在缓冲流的情况下——我们处理的是缓冲流，并且不知道底层数据的确切大小，我们需要使实现更加灵活:

```java
@Test
public void givenUsingPlainJavaOnUnknownSizeStream_whenConvertingAnInputStreamToAByteArray_thenCorrect() 
  throws IOException {
    InputStream is = new ByteArrayInputStream(new byte[] { 0, 1, 2, 3, 4, 5, 6 }); // not really known
    ByteArrayOutputStream buffer = new ByteArrayOutputStream();

    int nRead;
    byte[] data = new byte[4];

    while ((nRead = is.read(data, 0, data.length)) != -1) {
        buffer.write(data, 0, nRead);
    }

    buffer.flush();
    byte[] targetArray = buffer.toByteArray();
}
```

从 Java 9 开始，我们可以用一个专用的`readNbytes`方法实现同样的功能:

```java
@Test
public void givenUsingPlainJava9OnUnknownSizeStream_whenConvertingAnInputStreamToAByteArray_thenCorrect() 
  throws IOException {
    InputStream is = new ByteArrayInputStream(new byte[] { 0, 1, 2, 3, 4, 5, 6 });
    ByteArrayOutputStream buffer = new ByteArrayOutputStream();

    int nRead;
    byte[] data = new byte[4];

    while ((nRead = is.readNBytes(data, 0, data.length)) != 0) {
        System.out.println("here " + nRead);
        buffer.write(data, 0, nRead);
    }

    buffer.flush();
    byte[] targetArray = buffer.toByteArray();
}
```

这两种方法之间的差别非常微妙。

第一个 **`read​(byte[] b, int off, int len)`，从输入流**中读取最多`len` 个字节的数据，而第二个 **`readNBytes​(byte[] b, int off, int len)`，读取所请求的字节数**。

此外，如果输入流中没有更多的数据，那么`read`将返回`-1`。然而，`readNbytes`总是返回读入缓冲区的实际字节数。

我们也可以一次读取所有字节:

```java
@Test
public void
  givenUsingPlainJava9_whenConvertingAnInputStreamToAByteArray_thenCorrect()
  throws IOException {
    InputStream is = new ByteArrayInputStream(new byte[] { 0, 1, 2 });

    byte[] data = is.readAllBytes();
}
```

### 2.2。使用番石榴进行转换

现在让我们看看简单的基于番石榴的解决方案——使用方便的 ByteStreams 实用程序类:

```java
@Test
public void givenUsingGuava_whenConvertingAnInputStreamToAByteArray_thenCorrect() 
  throws IOException {
    InputStream initialStream = ByteSource.wrap(new byte[] { 0, 1, 2 }).openStream();

    byte[] targetArray = ByteStreams.toByteArray(initialStream);
}
```

### 2.3。使用通用 IO 转换

最后，使用 Apache Commons IO 的简单解决方案:

```java
@Test
public void givenUsingCommonsIO_whenConvertingAnInputStreamToAByteArray_thenCorrect() 
  throws IOException {
    ByteArrayInputStream initialStream = new ByteArrayInputStream(new byte[] { 0, 1, 2 });

    byte[] targetArray = IOUtils.toByteArray(initialStream);
}
```

方法`IOUtils.toByteArray()`在内部缓冲输入，所以当需要缓冲时**不需要使用`BufferedInputStream`实例**。

## 3。转换为`ByteBuffer`

现在，让我们看看如何从一个`InputStream.` 中获得一个 [`ByteBuffer`](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/ByteBuffer.html) ，每当我们需要在内存中进行快速直接的低级 I/O 操作时，这个**就很有用。**

使用与上一节相同的方法，我们将看看如何将`InputStream`转换为`ByteBuffer`——首先使用普通 Java，然后使用 Guava 和 Commons IO。

### 3.1。使用普通 Java 进行转换

在字节流的情况下，我们知道底层数据的确切大小。让我们使用`ByteArrayInputStream#available` 方法将字节流读入一个`ByteBuffer`:

```java
@Test
public void givenUsingCoreClasses_whenByteArrayInputStreamToAByteBuffer_thenLengthMustMatch() 
  throws IOException {
    byte[] input = new byte[] { 0, 1, 2 };
    InputStream initialStream = new ByteArrayInputStream(input);
    ByteBuffer byteBuffer = ByteBuffer.allocate(3);
    while (initialStream.available() > 0) {
        byteBuffer.put((byte) initialStream.read());
    }

    assertEquals(byteBuffer.position(), input.length);
}
```

### 3.2。使用番石榴进行转换

现在让我们看看一个简单的基于番石榴的解决方案——使用方便的`ByteStreams`实用程序类:

```java
@Test
public void givenUsingGuava__whenByteArrayInputStreamToAByteBuffer_thenLengthMustMatch() 
  throws IOException {
    InputStream initialStream = ByteSource
      .wrap(new byte[] { 0, 1, 2 })
      .openStream();
    byte[] targetArray = ByteStreams.toByteArray(initialStream);
    ByteBuffer bufferByte = ByteBuffer.wrap(targetArray);
    while (bufferByte.hasRemaining()) {
        bufferByte.get();
    }

    assertEquals(bufferByte.position(), targetArray.length);
}
```

这里我们使用一个带有方法`hasRemaining` 的 while 循环来展示将所有字节读入`ByteBuffer.`的不同方式，否则断言将失败，因为`ByteBuffer`索引位置将为零。

### 3.3。使用通用 IO 转换

最后，使用 Apache Commons IO 和`IOUtils`类:

```java
@Test
public void givenUsingCommonsIo_whenByteArrayInputStreamToAByteBuffer_thenLengthMustMatch() 
  throws IOException {
    byte[] input = new byte[] { 0, 1, 2 };
    InputStream initialStream = new ByteArrayInputStream(input);
    ByteBuffer byteBuffer = ByteBuffer.allocate(3);
    ReadableByteChannel channel = newChannel(initialStream);
    IOUtils.readFully(channel, byteBuffer);

    assertEquals(byteBuffer.position(), input.length);
}
```

## 4.结论

本文展示了使用普通 Java、Guava 和 Apache Commons IO 将原始输入流转换为字节数组和`ByteBuffer`的各种方法。

所有这些例子的实现都可以在我们的 [GitHub 项目](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-improvements)中找到。