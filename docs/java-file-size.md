# Java 中的文件大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-size>

## 1。概述

在这个快速教程中，我们将学习如何在 Java 中获得文件的**大小——使用 Java 7、新的 Java 8 和 Apache Common IO。**

最后，我们还将获得文件大小的人类可读表示。

## 2。标准 Java IO

让我们从一个简单的计算文件大小的例子开始——使用 `[File.length()](https://web.archive.org/web/20220617075808/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#length() "API usage")`方法:

```java
private long getFileSize(File file) {
    long length = file.length();
    return length;
}
```

我们可以相对简单地测试我们的实现:

```java
@Test
public void whenGetFileSize_thenCorrect() {
    long expectedSize = 12607;

    File imageFile = new File("src/test/resources/image.jpg");
    long size = getFileSize(imageFile);

    assertEquals(expectedSize, size);
}
```

请注意，默认情况下，文件大小是以字节计算的。

## 3。用 Java NIO

接下来，让我们看看如何使用 NIO 库来获取文件的大小。

在下面的例子中，我们将使用`[FileChannel.size()](https://web.archive.org/web/20220617075808/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#size() "API usage")` API 来获取文件的字节大小:

```java
@Test
public void whenGetFileSizeUsingNioApi_thenCorrect() throws IOException {
    long expectedSize = 12607;

    Path imageFilePath = Paths.get("src/test/resources/image.jpg");
    FileChannel imageFileChannel = FileChannel.open(imageFilePath);

    long imageFileSize = imageFileChannel.size();
    assertEquals(expectedSize, imageFileSize);
} 
```

## 4。使用 Apache Commons IO

接下来，让我们看看如何使用 **Apache Commons IO** 获得文件大小。在下面的例子中，我们简单地使用`[FileUtils.sizeOf()](https://web.archive.org/web/20220617075808/https://commons.apache.org/proper/commons-io/javadocs/api-2.5/org/apache/commons/io/FileUtils.html#sizeOf(java.io.File) "API usage")`来获得文件大小:

```java
@Test
public void whenGetFileSizeUsingApacheCommonsIO_thenCorrect() {
    long expectedSize = 12607;

    File imageFile = new File("src/test/resources/image.jpg");
    long size = FileUtils.sizeOf(imageFile);

    assertEquals(expectedSize, size);
}
```

请注意，对于安全受限文件，`FileUtils.sizeOf()`会将大小报告为零。

## 5。人类可读尺寸

最后，让我们看看如何使用 **Apache Commons IO** 获得用户更容易理解的文件大小表示，而不仅仅是以字节为单位的大小:

```java
@Test
public void whenGetReadableFileSize_thenCorrect() {
    File imageFile = new File("src/test/resources/image.jpg");
    long size = getFileSize(imageFile);

    assertEquals("12 KB", FileUtils.byteCountToDisplaySize(size));
} 
```

## 6。结论

在本教程中，我们举例说明了如何使用 Java 和 Apache Commons IO 来计算文件系统中文件的大小。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行。