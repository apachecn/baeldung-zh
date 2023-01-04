# Java–阅读器的输入流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-inputstream-to-reader>

在这个快速教程中，我们将看看**如何使用 Java，然后是 Guava，最后是 Apache Commons IO 将`InputStream`转换为`Reader`** 。

这篇文章是 Baeldung 网站上的“**Java——回到基础的**”系列文章的一部分。

## 1。用 Java

首先，让我们看看简单的 Java 解决方案——使用现成的`InputStreamReader`:

```java
@Test
public void givenUsingPlainJava_whenConvertingInputStreamIntoReader_thenCorrect() 
  throws IOException {
    InputStream initialStream = new ByteArrayInputStream("With Java".getBytes());

    Reader targetReader = new InputStreamReader(initialStream);

    targetReader.close();
}
```

## 2。有番石榴

接下来——让我们看看**番石榴解**—使用中间字节数组和字符串:

```java
@Test
public void givenUsingGuava_whenConvertingInputStreamIntoReader_thenCorrect() 
  throws IOException {
    InputStream initialStream = ByteSource.wrap("With Guava".getBytes()).openStream();

    byte[] buffer = ByteStreams.toByteArray(initialStream);
    Reader targetReader = CharSource.wrap(new String(buffer)).openStream();

    targetReader.close();
}
```

注意，Java 解决方案比这种方法简单。

## 3。带公共 IO

最后，使用 Apache Commons IO 的解决方案也使用了中间字符串:

```java
@Test
public void givenUsingCommonsIO_whenConvertingInputStreamIntoReader_thenCorrect() 
  throws IOException {
    InputStream initialStream = IOUtils.toInputStream("With Commons IO");

    byte[] buffer = IOUtils.toByteArray(initialStream);
    Reader targetReader = new CharSequenceReader(new String(buffer));

    targetReader.close();
}
```

现在你有了 3 种将输入流转换成 Java `Reader`的快速方法。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20220526040216/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)