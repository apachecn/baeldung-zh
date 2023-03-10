# Java–文件到阅读器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-file-to-reader>

在这个快速教程中，我们将演示**如何使用普通 Java、Guava 或 Apache Commons IO 将`File`转换为`Reader`** 。让我们开始吧。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用普通 Java

让我们首先看看简单的 Java 解决方案:

```java
@Test
public void givenUsingPlainJava_whenConvertingFileIntoReader_thenCorrect() 
  throws IOException {
    File initialFile = new File("src/test/resources/initialFile.txt");
    initialFile.createNewFile();
    Reader targetReader = new FileReader(initialFile);
    targetReader.close();
}
```

## 2。有番石榴

现在，让我们看看相同的转换，这次使用的是番石榴库:

```java
@Test
public void givenUsingGuava_whenConvertingFileIntoReader_thenCorrect() throws 
  IOException {
    File initialFile = new File("src/test/resources/initialFile.txt");
    com.google.common.io.Files.touch(initialFile);
    Reader targetReader = Files.newReader(initialFile, Charset.defaultCharset());
    targetReader.close();
}
```

## 3。带公共 IO

最后，让我们以 Commons IO 代码示例结束，通过中间字节数组进行转换:

```java
@Test
public void givenUsingCommonsIO_whenConvertingFileIntoReader_thenCorrect() 
  throws IOException {
    File initialFile = new File("src/test/resources/initialFile.txt");
    FileUtils.touch(initialFile);
    FileUtils.write(initialFile, "With Commons IO");
    byte[] buffer = FileUtils.readFileToByteArray(initialFile);
    Reader targetReader = new CharSequenceReader(new String(buffer));
    targetReader.close();
}
```

现在我们有了将`File`转换成`Reader` 的三种方法——首先用普通 Java，然后用 Guava，最后用 Apache Commons IO 库。请务必在 GitHub 上查看样本[。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)