# Java 字符串阅读器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-reader-to-string>

在这个快速教程中，我们将使用普通 Java、Guava 和 Apache Commons IO 库将一个`Reader`转换成一个字符串。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用 Java

让我们从一个简单的 Java 解决方案开始，**从`Reader`** 开始顺序读取字符:

```
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoStringV1_thenCorrect() 
  throws IOException {
    StringReader reader = new StringReader("text");
    int intValueOfChar;
    String targetString = "";
    while ((intValueOfChar = reader.read()) != -1) {
        targetString += (char) intValueOfChar;
    }
    reader.close();
}
```

如果要读取大量内容，批量读取解决方案会更好:

```
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoStringV2_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("text");
    char[] arr = new char[8 * 1024];
    StringBuilder buffer = new StringBuilder();
    int numCharsRead;
    while ((numCharsRead = initialReader.read(arr, 0, arr.length)) != -1) {
        buffer.append(arr, 0, numCharsRead);
    }
    initialReader.close();
    String targetString = buffer.toString();
}
```

## 2。有番石榴

Guava 提供了一个实用程序，可以直接进行转换:

```
@Test
public void givenUsingGuava_whenConvertingReaderIntoString_thenCorrect() 
  throws IOException {
    Reader initialReader = CharSource.wrap("With Google Guava").openStream();
    String targetString = CharStreams.toString(initialReader);
    initialReader.close();
}
```

## 3。带公共 IO

Apache Commons IO 也是如此–有一个 IO 实用程序能够执行**直接转换**:

```
@Test
public void givenUsingCommonsIO_whenConvertingReaderIntoString_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("With Apache Commons");
    String targetString = IOUtils.toString(initialReader);
    initialReader.close();
}
```

现在你有了 4 种将`Reader`转换成普通字符串的方法。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20220627082323/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)