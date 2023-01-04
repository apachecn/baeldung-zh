# 用 Java 删除文件的内容

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-delete-file-contents>

## 1.介绍

在本教程中，我们将看到如何使用 Java 删除文件内容，而不删除文件本身。既然有很多简单的方法，那就让我们一个一个地探索一下吧。

## 2.使用`PrintWriter`

Java 的 [`PrintWriter`](/web/20221102194145/https://www.baeldung.com/java-write-to-file) 类扩展了`Writer `类。它将对象的格式化表示打印到文本输出流中。

我们将进行一个简单的测试。让我们创建一个指向现有文件的`PrintWriter`实例，通过关闭它来删除文件的现有内容，然后确保文件长度为空:

```java
new PrintWriter(FILE_PATH).close();
assertEquals(0, StreamUtils.getStringFromInputStream(new FileInputStream(FILE_PATH)).length());
```

另外，请注意，如果我们不需要进一步处理`PrintWriter`对象，这是最好的选择。然而，如果我们需要`PrintWriter`对象来进行进一步的文件操作，我们可以用不同的方式来做:

```java
PrintWriter writer = new PrintWriter(FILE_PATH);
writer.print("");
// other operations
writer.close();
```

## 3.使用`FileWriter`

Java 的`FileWriter` 是一个标准的 Java IO API 类，它提供了将面向字符的数据写入文件的方法。

现在让我们看看如何使用`FileWriter:`来做同样的操作

```java
new FileWriter(FILE_PATH, false).close();
```

类似地，如果我们需要进一步处理` FileWriter`对象，我们可以将它赋给一个变量并用一个空字符串更新。

## 4.使用`FileOutputStream`

Java 的 [`FileOutputStream`](/web/20221102194145/https://www.baeldung.com/convert-input-stream-to-a-file) 是用于将字节数据写入文件的输出流。

现在，让我们使用`FileOutputStream:`删除文件的内容

```java
new FileOutputStream(FILE_PATH).close(); 
```

## 5.使用 Apache Commons IO `FileUtils`

Apache Commons IO 是一个包含实用程序类的库，可以帮助解决常见的 IO 问题。我们可以使用它的一个实用程序类来删除文件的内容—`FileUtils.`

为了了解这是如何工作的，让我们将 [Apache Commons IO](https://web.archive.org/web/20221102194145/https://search.maven.org/search?q=g:commons-io%20a:commons-io) 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency> 
```

之后，让我们举一个简单的例子来演示文件内容的删除:

```java
FileUtils.write(new File(FILE_PATH), "", Charset.defaultCharset());
```

## 6.使用 Java NIO `Files`

[Java NIO File](/web/20221102194145/https://www.baeldung.com/java-nio-2-file-api) was introduced in JDK 7\. It defines interfaces and classes to access files, file attributes, and file systems.

我们也可以使用`java.nio.file.Files`删除文件内容:

```java
BufferedWriter writer = Files.newBufferedWriter(Paths.get(FILE_PATH));
writer.write("");
writer.flush();
```

## 7.使用 Java NIO `FileChannel`

Java NIO FileChannel 是 NIO 连接文件的实现。它还补充了标准的 Java IO 包。

我们也可以使用`java.nio.channels.FileChannel`删除文件内容:

```java
FileChannel.open(Paths.get(FILE_PATH), StandardOpenOption.WRITE).truncate(0).close();
```

## 8.用番石榴

Guava 是一个开源的基于 Java 的库，它提供了进行 I/O 操作的实用方法。让我们看看如何使用 Guava API 删除文件内容。

首先，我们需要在我们的`pom.xml`中添加[番石榴](https://web.archive.org/web/20221102194145/https://search.maven.org/search?q=g:com.google.guava%20a:guava)依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency> 
```

之后，让我们看一个使用 Guava 删除文件内容的快速示例:

```java
File file = new File(FILE_PATH);
byte[] empty = new byte[0];
com.google.common.io.Files.write(empty, file);
```

## 9.结论

总而言之，我们已经看到了多种删除文件内容而不删除文件本身的方法。

本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221102194145/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2)