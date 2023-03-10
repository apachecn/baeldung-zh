# Java–阅读器的字节数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-byte-array-to-reader>

在这个快速教程中，我们将使用普通 Java、Guava 和 Apache Commons IO 库将一个简单的**字节数组转换成一个`Reader`** 。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用普通 Java

让我们从简单的 Java 示例开始，通过中间字符串进行转换:

```java
@Test
public void givenUsingPlainJava_whenConvertingByteArrayIntoReader_thenCorrect() 
  throws IOException {
    byte[] initialArray = "With Java".getBytes();
    Reader targetReader = new StringReader(new String(initialArray));
    targetReader.close();
}
```

另一种方法是使用一个`InputStreamReader`和一个`ByteArrayInputStream`:

```java
@Test
public void givenUsingPlainJava2_whenConvertingByteArrayIntoReader_thenCorrect() 
  throws IOException {
    byte[] initialArray = "Hello world!".getBytes();
    Reader targetReader = new InputStreamReader(new ByteArrayInputStream(initialArray));
    targetReader.close();
}
```

## 2。有番石榴

接下来，让我们来看看番石榴溶液，同样使用中间字符串:

```java
@Test
public void givenUsingGuava_whenConvertingByteArrayIntoReader_thenCorrect() 
  throws IOException {
    byte[] initialArray = "With Guava".getBytes();
    String bufferString = new String(initialArray);
    Reader targetReader = CharSource.wrap(bufferString).openStream();
    targetReader.close();
}
```

不幸的是，Guava `ByteSource`实用程序不允许直接转换，所以我们仍然需要使用中间字符串表示。

## 3。使用 Apache Commons IO

类似地——Commons IO 也使用中间字符串表示将`byte[]`转换为读取器:

```java
@Test
public void givenUsingCommonsIO_whenConvertingByteArrayIntoReader_thenCorrect() 
  throws IOException {
    byte[] initialArray = "With Commons IO".getBytes();
    Reader targetReader = new CharSequenceReader(new String(initialArray));
    targetReader.close();
}
```

现在我们有了 3 种简单的方法来将字节数组转换成 Java 阅读器。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20220929022938/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)