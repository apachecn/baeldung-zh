# 使用 Java MappedByteBuffer

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mapped-byte-buffer>

## 1。概述

在这篇简短的文章中，我们将关注`java.nio`包中的`[MappedByteBuffer](https://web.archive.org/web/20220625162422/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/MappedByteBuffer.html)` 。该实用程序对于高效的文件读取非常有用。

## 2。如何`MappedByteBuffer W`works

当我们加载文件的一个区域时，我们可以将它加载到特定的内存区域，以便以后访问。

当我们知道我们需要多次读取一个文件的内容时，优化这个昂贵的过程是一个好主意，例如，通过将内容保存在内存中。由于这一点，文件的这一部分的后续查找将只到主存储器，而不需要从光盘加载数据，大大减少了延迟。

使用`MappedByteBuffer` 时，我们需要小心的一件事是，当我们从光盘上处理非常大的文件时——**我们需要确保文件适合内存**。

否则，我们可能会填满整个内存，结果遇到常见的`OutOfMemoryException.` ,我们可以通过只加载文件的一部分来解决这个问题——例如基于使用模式。

## 3。使用`MappedByteBuffer`阅读文件

假设我们有一个名为`fileToRead.txt`的文件，其内容如下:

```java
This is a content of the file
```

该文件位于`/resource`目录中，因此我们可以使用以下函数加载它:

```java
Path getFileURIFromResources(String fileName) throws Exception {
    ClassLoader classLoader = getClass().getClassLoader();
    return Paths.get(classLoader.getResource(fileName).getPath());
}
```

为了从一个文件创建`MappedByteBuffer` ，首先我们需要从它创建一个`FileChannel`。一旦我们创建了通道，我们就可以调用它的`map()` 方法，传入我们想要读取的`MapMode,` a `position` ，以及指定我们想要多少字节的`size` 参数:

```java
CharBuffer charBuffer = null;
Path pathToRead = getFileURIFromResources("fileToRead.txt");

try (FileChannel fileChannel (FileChannel) Files.newByteChannel(
  pathToRead, EnumSet.of(StandardOpenOption.READ))) {

    MappedByteBuffer mappedByteBuffer = fileChannel
      .map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());

    if (mappedByteBuffer != null) {
        charBuffer = Charset.forName("UTF-8").decode(mappedByteBuffer);
    }
}
```

一旦我们将文件映射到内存映射缓冲区，我们就可以将数据从那里读入到`CharBuffer.` 中。需要注意的是，虽然我们在调用`decode()` 方法传递`MappedByteBuffer,` 时读取文件的内容，但是我们是从内存中读取的，而不是从磁盘中读取的。因此读取将会非常快。

我们可以断言，从我们的文件中读取的内容是`fileToRead.txt`文件的实际内容:

```java
assertNotNull(charBuffer);
assertEquals(
  charBuffer.toString(), "This is a content of the file");
```

从`mappedByteBuffer` 开始的每个后续读取都将非常快，因为文件的内容在内存中映射，并且读取不需要从光盘中查找数据。

## 4。使用`MappedByteBuffer`向文件写入

假设我们想使用`MappedByteBuffer` API 将一些内容写入文件`fileToWriteTo.txt` 。为了实现这一点，我们需要打开`FileChannel` 并调用它的`map()` 方法，传入`FileChannel.MapMode.READ_WRITE.`

接下来，我们可以使用来自`MappedByteBuffer:`的`put()` 方法将`CharBuffer`的内容保存到文件中

```java
CharBuffer charBuffer = CharBuffer
  .wrap("This will be written to the file");
Path pathToWrite = getFileURIFromResources("fileToWriteTo.txt");

try (FileChannel fileChannel = (FileChannel) Files
  .newByteChannel(pathToWrite, EnumSet.of(
    StandardOpenOption.READ, 
    StandardOpenOption.WRITE, 
    StandardOpenOption.TRUNCATE_EXISTING))) {

    MappedByteBuffer mappedByteBuffer = fileChannel
      .map(FileChannel.MapMode.READ_WRITE, 0, charBuffer.length());

    if (mappedByteBuffer != null) {
        mappedByteBuffer.put(
          Charset.forName("utf-8").encode(charBuffer));
    }
}
```

我们可以断言`charBuffer` 的实际内容是通过读取它的内容写入文件的:

```java
List<String> fileContent = Files.readAllLines(pathToWrite);
assertEquals(fileContent.get(0), "This will be written to the file");
```

## 5。结论

在这个快速教程中，我们看到了来自`java.nio`包的`MappedByteBuffer` 构造。

这是多次读取文件内容的一种非常有效的方式，因为文件被映射到内存中，后续读取不需要每次都到磁盘。

所有这些例子和代码片段都可以在 GitHub 上找到[——这是一个 Maven 项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220625162422/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio-2)