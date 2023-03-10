# 将字符串列表写入文本文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-to-text-file>

## 1.概观

在这个快速教程中，我们将以不同的方式将一个`Strings`的`List`写入一个 Java 文本文件。首先，我们将讨论`FileWriter`，然后`BufferedWriter`，最后是`Files.writeString`。

## 2.使用`FileWriter`

`java.io`包包含一个`FileWriter`类，我们可以用它将字符数据写入文件。如果我们看看层次结构，我们会看到`FileWriter`类扩展了`OutputStreamWriter`类，后者又扩展了`Writer`类。

让我们看看初始化一个`FileWriter`可用的构造函数:

```java
FileWriter f = new FileWriter(File file);
FileWriter f = new FileWriter(File file, boolean append);
FileWriter f = new FileWriter(FileDescriptor fd);
FileWriter f = new FileWriter(File file, Charset charset);
FileWriter f = new FileWriter(File file, Charset charset, boolean append);
FileWriter f = new FileWriter(String fileName);
FileWriter f = new FileWriter(String fileName, Boolean append);
FileWriter f = new FileWriter(String fileName, Charset charset);
FileWriter f = new FileWriter(String fileName, Charset charset, boolean append); 
```

注意，`FileWriter`类的所有构造函数都假设默认的字节缓冲区大小和默认的字符编码是可以接受的。

现在，让我们学习如何使用`FileWriter`将`Strings`的`List`写入文本文件:

```java
FileWriter fileWriter = new FileWriter(TEXT_FILENAME);
for (String str : stringList) {
    fileWriter.write(str + System.lineSeparator());
}
fileWriter.close();
return TEXT_FILENAME;
```

示例中需要注意的一点是，如果不存在的话， **`FileWriter`会创建`sampleTextFile.txt`。如果文件存在，那么我们可以根据构造函数的选择来覆盖它或追加它。**

## 3.使用`BufferedWriter`

`java.io`包包含一个`BufferedWriter` 类，可以和其他`Writers`一起使用，以更好的性能写入角色数据。但是如何更有效率呢？

当我们使用`BufferedWriter,`时，字符被写入缓冲区而不是磁盘。**当缓冲区填满时，所有数据会一次性写入磁盘，从而减少进出磁盘的流量**，因此有助于提高性能。

如果说层次的话，`BufferedWriter` 类扩展了`Writer`类。

让我们看看初始化一个`BufferedWriter`可用的构造函数:

```java
BufferedWriter b = new BufferedWriter(Writer w);
BufferedWriter b = new BufferedWriter(Writer w, int size);
```

请注意，如果我们不指定缓冲区大小，它将采用默认值。

现在，让我们探索如何使用`BufferedWriter`将`Strings`的`List`写入文本文件:

```java
BufferedWriter br = new BufferedWriter(new FileWriter(TEXT_FILENAME));
for (String str : stringList) {
    br.write(str + System.lineSeparator());
}
br.close();
return TEXT_FILENAME;
```

在上面的代码中，我们用`BufferedWriter,`包装了`FileWriter`，这减少了读取调用，从而提高了性能。

## 3.使用`Files.writeString`

`java.nio`包包含一个来自`Files`类的方法`writeString()`,用于将字符写入文件。这个方法是在 Java 11 中引入的。

让我们看看`java.nio.file.Files`中可用的两个重载方法:

```java
public static Path writeString​(Path path, CharSequence csq, OpenOption… options) throws IOException
public static Path writeString​(Path path, CharSequence csq, Charset cs, OpenOption… options) throws IOException
```

注意，在第一种方法中，我们不必指定`Charset`。默认情况下，它采用`UTF-8 Charset`，而在第二种方法中，我们可以指定`Charset`。最后，让我们学习如何使用`Files.writeString`将`Strings`的`List`写入文本文件:

```java
Path filePath = Paths.get(TEXT_FILENAME);
Files.deleteIfExists(filePath);
Files.createFile(filePath);
for (String str : stringList) {
    Files.writeString(filePath, str + System.lineSeparator(),
    StandardOpenOption.APPEND);
}
return filePath.toString();
```

在上面的例子中，我们删除了已经存在的文件，然后[使用`Files.createFile`方法](/web/20221007203217/https://www.baeldung.com/java-how-to-create-a-file)创建了一个文件。注意，我们已经将`OpenOption`字段设置为`StandardOperation.APPEND,`，这意味着文件将以追加模式打开。如果我们不指定`OpenOption,`，那么我们可以预期会发生两件事:

*   如果文件已经存在，它将被覆盖
*   如果文件不存在，将创建一个新文件并在其上写入

## 4.测试

对于 JUnit 测试，让我们定义一个`Strings`的`List`:

```java
private static final List<String> stringList = Arrays.asList("Hello", "World"); 
```

这里，我们将使用`NIO Files.lines`和`count()`找出输出文件的行数。计数应该等于`List`中`Strings`的数量。

现在，让我们开始测试我们的`FileWriter`实现:

```java
@Test
public void givenUsingFileWriter_whenStringList_thenGetTextFile() throws IOException {
    String fileName = FileWriterExample.generateFileFromStringList(stringList);
    long count = Files.lines(Paths.get(fileName)).count();
    assertTrue("No. of lines in file should be equal to no. of Strings in List", ((int) count) == stringList.size());
}
```

接下来，我们将测试`BufferedWriter`的实现:

```java
@Test
public void givenUsingBufferedWriter_whenStringList_thenGetTextFile() throws IOException {
    String fileName = BufferedWriterExample.generateFileFromStringList(stringList);
    long count = Files.lines(Paths.get(fileName)).count();
    assertTrue("No. of lines in file should be equal to no. of Strings in List", ((int) count) == stringList.size());
}
```

最后，让我们测试一下我们的`Files.writeString`实现:

```java
@Test
public void givenUsingFileWriteString_whenStringList_thenGetTextFile() throws IOException {
    String fileName = FileWriteStringExample.generateFileFromStringList(stringList);
    long count = Files.lines(Paths.get(fileName)).count();
    assertTrue("No. of lines in file should be equal to no. of Strings in List", ((int) count) == stringList.size());
}
```

## 5.结论

在本文中，我们探索了三种将字符串`List` 写入文本文件的常见方法。此外，我们通过编写 JUnit 测试来测试我们的实现。

和往常一样，教程的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221007203217/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-3)