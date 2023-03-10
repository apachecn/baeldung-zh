# Java–将文件转换为输入流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-file-to-input-stream>

## 1。概述

在这个快速教程中，我们将展示如何**将`File`转换为`InputStream`** ——首先使用普通 Java，然后使用 Guava 和 Apache Commons IO 库。

这篇文章是 Baeldung 网站上[Java 基础知识系列](/web/20221013193922/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [Java 扫描仪](/web/20221013193922/https://www.baeldung.com/java-scanner)

A quick and practical set of examples for using the core Scanner Class in Java - to work with Strings, Files and user input.[Read more](/web/20221013193922/https://www.baeldung.com/java-scanner) →

## [番石榴–写入文件，从文件中读取](/web/20221013193922/https://www.baeldung.com/guava-write-to-file-read-from-file)

How to Write to File and Read from a File using the Guava IO support and utilities.[Read more](/web/20221013193922/https://www.baeldung.com/guava-write-to-file-read-from-file) →

## [Java 字节数组到 InputStream](/web/20221013193922/https://www.baeldung.com/convert-byte-array-to-input-stream)

How to convert a byte[] to an InputStream using plain Java or Guava.[Read more](/web/20221013193922/https://www.baeldung.com/convert-byte-array-to-input-stream) →

## 2。使用 Java 转换

**我们可以使用 Java 的 [IO 包](https://web.archive.org/web/20221013193922/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/package-summary.html)将一个`File`转换成不同的`InputStream` s.**

### 2.1。`FileInputStream`

让我们从第一个也是最简单的**开始——使用`FileInputStream`T2:**

```java
@Test
public void givenUsingPlainJava_whenConvertingFileToInputStream_thenCorrect() 
  throws IOException {
    File initialFile = new File("src/main/resources/sample.txt");
    InputStream targetStream = new FileInputStream(initialFile);
}
```

### 2.2。`DataInputStream`

让我们看看另一种方法，我们可以使用`DataInputStream`从文件中读取二进制或原始数据:

```java
@Test
public final void givenUsingPlainJava_whenConvertingFileToDataInputStream_thenCorrect() 
  throws IOException {
      final File initialFile = new File("src/test/resources/sample.txt");
      final InputStream targetStream = 
        new DataInputStream(new FileInputStream(initialFile));
}
```

### 2.3.`SequenceInputStream`

最后，让我们看看**如何使用`SequenceInputStream` 将两个文件的输入流连接成一个`InputStream`T3:**

```java
@Test
public final void givenUsingPlainJava_whenConvertingFileToSequenceInputStream_thenCorrect() 
  throws IOException {
      final File initialFile = new File("src/test/resources/sample.txt");
      final File anotherFile = new File("src/test/resources/anothersample.txt");
      final InputStream targetStream = new FileInputStream(initialFile);
      final InputStream anotherTargetStream = new FileInputStream(anotherFile);

      InputStream sequenceTargetStream = 
        new SequenceInputStream(targetStream, anotherTargetStream);
}
```

注意，为了清晰起见，我们没有关闭这些例子中的结果流。

## 3。使用番石榴进行转换

接下来，让我们看看**番石榴溶液**，使用一个中介`ByteSource`:

```java
@Test
public void givenUsingGuava_whenConvertingFileToInputStream_thenCorrect() 
  throws IOException {
    File initialFile = new File("src/main/resources/sample.txt");
    InputStream targetStream = Files.asByteSource(initialFile).openStream();
}
```

## 4。使用通用 IO 转换

最后，让我们看一个使用 Apache Commons IO 的解决方案:

```java
@Test
public void givenUsingCommonsIO_whenConvertingFileToInputStream_thenCorrect() 
  throws IOException {
    File initialFile = new File("src/main/resources/sample.txt");
    InputStream targetStream = FileUtils.openInputStream(initialFile);
}
```

现在我们有了。从 Java 文件中打开流的三个简单干净的解决方案。

## 5。结论

在本文中，我们探索了通过使用不同的库将`File`转换为`InputStream`的各种方法。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221013193922/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)