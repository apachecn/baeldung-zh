# Java——将输入流写入文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-input-stream-to-a-file>

## 1。概述

在这个快速教程中，我们将演示如何**将`InputStream`写入文件。**首先我们将使用普通 Java，然后是 Guava，最后是 Apache Commons IO 库。

这篇文章是 Baeldung 上的["**Java-回到基础**教程](/web/20220525002142/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [Java–阅读器的输入流](/web/20220525002142/https://www.baeldung.com/java-convert-inputstream-to-reader)

How to convert an InputStream to a Reader using Java, Guava and the Apache Commons IO library.[Read more](/web/20220525002142/https://www.baeldung.com/java-convert-inputstream-to-reader) →

## [Java–将文件转换为输入流](/web/20220525002142/https://www.baeldung.com/convert-file-to-input-stream)

How to open an InputStream from a Java File - using plain Java, Guava and the Apache Commons IO library.[Read more](/web/20220525002142/https://www.baeldung.com/convert-file-to-input-stream) →

## [Java InputStream 到字节数组和字节缓冲区](/web/20220525002142/https://www.baeldung.com/convert-input-stream-to-array-of-bytes)

How to convert an InputStream to a byte[] using plain Java, Guava or Commons IO.[Read more](/web/20220525002142/https://www.baeldung.com/convert-input-stream-to-array-of-bytes) →

## 2。使用普通 Java 进行转换

让我们从 Java 解决方案开始:

```java
@Test
public void whenConvertingToFile_thenCorrect() throws IOException {
    Path path = Paths.get("src/test/resources/sample.txt");
    byte[] buffer = java.nio.file.Files.readAllBytes(path);

    File targetFile = new File("src/test/resources/targetFile.tmp");
    OutputStream outStream = new FileOutputStream(targetFile);
    outStream.write(buffer);

    IOUtils.closeQuietly(outStream);
}
```

注意，在这个例子中，输入流有已知的和预先确定的数据，比如磁盘上的文件或内存流。因此，**我们不需要做任何边界检查**，如果内存允许，我们可以简单地一次性读取和写入。

如果输入流链接到一个正在进行的数据流，就像来自一个正在进行的连接的 HTTP 响应，那么一次读取整个流是不可行的。在这种情况下，我们需要确保我们**继续阅读，直到我们到达流的结尾**:

```java
@Test
public void whenConvertingInProgressToFile_thenCorrect() 
  throws IOException {

    InputStream initialStream = new FileInputStream(
      new File("src/main/resources/sample.txt"));
    File targetFile = new File("src/main/resources/targetFile.tmp");
    OutputStream outStream = new FileOutputStream(targetFile);

    byte[] buffer = new byte[8 * 1024];
    int bytesRead;
    while ((bytesRead = initialStream.read(buffer)) != -1) {
        outStream.write(buffer, 0, bytesRead);
    }
    IOUtils.closeQuietly(initialStream);
    IOUtils.closeQuietly(outStream);
}
```

最后，这里有另一个简单的方法，我们可以**使用 Java 8 来做同样的操作:**

```java
@Test
public void whenConvertingAnInProgressInputStreamToFile_thenCorrect2() 
  throws IOException {

    InputStream initialStream = new FileInputStream(
      new File("src/main/resources/sample.txt"));
    File targetFile = new File("src/main/resources/targetFile.tmp");

    java.nio.file.Files.copy(
      initialStream, 
      targetFile.toPath(), 
      StandardCopyOption.REPLACE_EXISTING);

    IOUtils.closeQuietly(initialStream);
}
```

## 3。使用番石榴进行转换

接下来，让我们看看一个更简单的基于番石榴的解决方案:

```java
@Test
public void whenConvertingInputStreamToFile_thenCorrect3() 
  throws IOException {

    InputStream initialStream = new FileInputStream(
      new File("src/main/resources/sample.txt"));
    byte[] buffer = new byte[initialStream.available()];
    initialStream.read(buffer);

    File targetFile = new File("src/main/resources/targetFile.tmp");
    Files.write(buffer, targetFile);
}
```

## 4。使用通用 IO 转换

最后，这里有一个更快的 Apache Commons IO 解决方案:

```java
@Test
public void whenConvertingInputStreamToFile_thenCorrect4() 
  throws IOException {
    InputStream initialStream = FileUtils.openInputStream
      (new File("src/main/resources/sample.txt"));

    File targetFile = new File("src/main/resources/targetFile.tmp");

    FileUtils.copyInputStreamToFile(initialStream, targetFile);
}
```

现在我们有了将`InputStream`写入文件的 3 种快速方法。

所有这些例子的实现都可以在我们的 [GitHub 项目](https://web.archive.org/web/20220525002142/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2)中找到。