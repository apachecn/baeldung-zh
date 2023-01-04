# 用 Java 将字节[]写入文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-write-byte-array-file>

## 1.概观

在这个快速教程中，我们将学习几种不同的方法来将 Java 字节数组写入文件。我们将从头开始，使用 Java IO 包。接下来，我们将看一个使用 Java NIO 的例子。之后，我们将使用 Google Guava 和 Apache Commons IO。

## 2.Java IO

Java 的 IO 包从 JDK 1.0 开始就存在了，它提供了一系列用于读写数据的类和接口。

让我们用一个`FileOutputStream`将图像写到一个文件中:

```
File outputFile = tempFolder.newFile("outputFile.jpg");
try (FileOutputStream outputStream = new FileOutputStream(outputFile)) {
    outputStream.write(dataForWriting);
}
```

我们打开一个到目标文件的输出流，然后我们可以简单地将我们的`byte[]` `dataForWriting`传递给`write`方法。注意，我们在这里使用了一个 [`try-with-resources`块](/web/20221013193919/https://www.baeldung.com/java-try-with-resources)来确保在抛出`IOException`时关闭`OutputStream`。

## 3.Java 九

Java NIO 包是在 Java 1.4 中引入的，NIO 的[文件系统 API 是在 Java 7 中作为扩展引入的。 **Java NIO 使用缓冲并且是非阻塞的，而 Java IO 使用阻塞流。**在`java.nio.file`包中，创建文件资源的语法更加简洁。](/web/20221013193919/https://www.baeldung.com/java-nio-2-file-api)

我们可以使用`Files`类在一行中编写我们的`byte[]`:

```
Files.write(outputFile.toPath(), dataForWriting);
```

我们的示例要么创建一个文件，要么截断一个现有文件并打开它进行写入。我们也可以使用`Paths.get(“path/to/file”)` 或`Paths.get(“path”, “to”, “file”)`来构建`Path`,它描述了我们的文件将被存储在哪里。`Path`是 Java NIO 表达路径的本地方式。

如果我们需要覆盖文件打开行为，我们也可以向`write`方法提供`[OpenOption](/web/20221013193919/https://www.baeldung.com/java-file-options)` 。

## 4.谷歌番石榴

[Guava](/web/20221013193919/https://www.baeldung.com/guava-write-to-file-read-from-file) 是 Google 的一个库，它提供了多种类型来执行 Java 中的常见操作，包括 IO。

让我们将[番石榴](https://web.archive.org/web/20221013193919/https://search.maven.org/search?q=g:com.google.guava%20a:guava)导入到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency> 
```

### 4.1.番石榴文件

与 Java NIO 包一样，我们可以在一行中编写我们的`byte[]`:

```
Files.write(dataForWriting, outputFile);
```

Guava 的`Files.write`方法也有一个可选的`OptionOptions`，并使用与`java.nio.Files.write`相同的默认值。

不过这里有一个陷阱:番石榴 **`Files.write`方法标有`@Beta`注释**。[根据文档](https://web.archive.org/web/20221013193919/https://github.com/google/guava#important-warnings)，这意味着它可以随时改变，因此不建议在图书馆使用。

因此，如果我们正在编写一个库项目，我们应该考虑使用`ByteSink`。

### 4.2.`ByteSink`

我们也可以创建一个`ByteSink`来写我们的`byte[]`:

```
ByteSink byteSink = Files.asByteSink(outputFile);
byteSink.write(dataForWriting);
```

**`ByteSink`是我们可以写入字节的目的地。**向目的地提供一个`OutputStream`。

如果我们需要使用一个`java.nio.files.Path`或者提供一个特殊的`OpenOption`，我们可以使用`MoreFiles`类来获取我们的`ByteSink`:

```
ByteSink byteSink = MoreFiles.asByteSink(outputFile.toPath(), 
    StandardOpenOption.CREATE, 
    StandardOpenOption.WRITE);
byteSink.write(dataForWriting);
```

## 5.Apache commons me(Apache 公用程式)

Apache [Commons IO](/web/20221013193919/https://www.baeldung.com/apache-commons-io) 提供了一些常见的文件任务。

让我们导入最新版本的 [commons-io](https://web.archive.org/web/20221013193919/https://search.maven.org/search?q=g:commons-io%20a:commons-io) :

```
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency> 
```

现在，让我们使用`FileUtils`类编写我们的`byte[]`:

```
FileUtils.writeByteArrayToFile(outputFile, dataForWriting);
```

`FileUtils.writeByteArrayToFile`方法类似于我们使用过的其他方法，我们给它一个`File`,代表我们想要的目的地和我们正在写入的二进制数据。**如果我们的目标文件或任何父目录不存在，它们将被创建。**

## 6.结论

在这个简短的教程中，我们学习了如何使用普通 Java 和两个流行的 Java 实用程序库:Google Guava 和 Apache Commons IO 将二进制数据从一个`byte[]`写入一个文件。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)