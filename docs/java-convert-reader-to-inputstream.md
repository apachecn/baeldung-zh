# Java–输入流的阅读器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-reader-to-inputstream>

在这个快速教程中，我们将看看从`Reader`到`InputStream` 的**转换——首先是普通 Java，然后是 Guava，最后是 Apache Commons IO 库。**

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用 Java

让我们从 Java 解决方案开始:

```
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoInputStream_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("With Java");

    char[] charBuffer = new char[8 * 1024];
    StringBuilder builder = new StringBuilder();
    int numCharsRead;
    while ((numCharsRead = initialReader.read(charBuffer, 0, charBuffer.length)) != -1) {
        builder.append(charBuffer, 0, numCharsRead);
    }
    InputStream targetStream = new ByteArrayInputStream(
      builder.toString().getBytes(StandardCharsets.UTF_8));

    initialReader.close();
    targetStream.close();
}
```

请注意，我们一次读取(和写入)大量数据。

## 2。有番石榴

接下来，让我们看看**更简单的番石榴溶液**:

```
@Test
public void givenUsingGuava_whenConvertingReaderIntoInputStream_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("With Guava");

    InputStream targetStream = 
      new ByteArrayInputStream(CharStreams.toString(initialReader)
      .getBytes(Charsets.UTF_8));

    initialReader.close();
    targetStream.close();
}
```

请注意，我们使用了一个开箱即用的输入流，它将整个转换变成了一个线性的。

## 3。带公共 IO

最后，让我们看看几个**通用 IO 解决方案**——也是简单的一行程序。

首先，使用`[ReaderInputStream](https://web.archive.org/web/20221013193919/https://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/input/ReaderInputStream.html):`

```
@Test
public void givenUsingCommonsIOReaderInputStream_whenConvertingReaderIntoInputStream() 
  throws IOException {
    Reader initialReader = new StringReader("With Commons IO");

    InputStream targetStream = new ReaderInputStream(initialReader, Charsets.UTF_8);

    initialReader.close();
    targetStream.close();
}
```

最后，使用`[IOUtils](https://web.archive.org/web/20221013193919/https://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/IOUtils.html#toString-java.io.Reader-)`进行相同的转换:

```
@Test
public void givenUsingCommonsIOUtils_whenConvertingReaderIntoInputStream() 
  throws IOException {
    Reader initialReader = new StringReader("With Commons IO");

    InputStream targetStream = 
      IOUtils.toInputStream(IOUtils.toString(initialReader), Charsets.UTF_8);

    initialReader.close();
    targetStream.close();
}
```

注意，我们这里处理的是任何类型的读取器——但是如果您专门处理文本数据，那么显式地指定字符集而不是使用 JVM 缺省值总是一个好主意。

## 4.结论

现在你有了——将`Reader`转换成`InputStream`的 3 种简单方法。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)