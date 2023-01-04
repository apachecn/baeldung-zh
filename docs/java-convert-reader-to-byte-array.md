# Java–读取字节数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-reader-to-byte-array>

如果您在 Java 生态系统中有几年的经验，并且您有兴趣与社区分享这些经验(当然，您的工作也可以获得报酬)，请查看[“为我们而写”页面](/web/20210613230852/https://www.baeldung.com/contribution-guidelines)。干杯，尤金

这个快速教程将展示如何使用普通 Java、Guava 和 Apache Commons IO 库将阅读器转换成 byte[]。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用 Java

让我们从简单的 Java 解决方案开始——通过一个中间字符串:

```java
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoByteArray_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("With Java");

    char[] charArray = new char[8 * 1024];
    StringBuilder builder = new StringBuilder();
    int numCharsRead;
    while ((numCharsRead = initialReader.read(charArray, 0, charArray.length)) != -1) {
        builder.append(charArray, 0, numCharsRead);
    }
    byte[] targetArray = builder.toString().getBytes();

    initialReader.close();
}
```

注意，阅读是成块进行的，而不是一次一个字符。

## 2。有番石榴

接下来，让我们来看看番石榴溶液，同样使用中间字符串:

```java
@Test
public void givenUsingGuava_whenConvertingReaderIntoByteArray_thenCorrect() 
  throws IOException {
    Reader initialReader = CharSource.wrap("With Google Guava").openStream();

    byte[] targetArray = CharStreams.toString(initialReader).getBytes();

    initialReader.close();
}
```

请注意，我们使用内置的实用程序 API，不必对普通 Java 示例进行任何低级转换。

## 3。带公共 IO

最后，这是一个直接的解决方案，它受 Commons IO 的支持:

```java
@Test
public void givenUsingCommonsIO_whenConvertingReaderIntoByteArray_thenCorrect() 
  throws IOException {
    StringReader initialReader = new StringReader("With Commons IO");

    byte[] targetArray = IOUtils.toByteArray(initialReader);

    initialReader.close();
}
```

现在你有了 3 种将 java `Reader`转换成字节数组的快速方法。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20210613230852/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)