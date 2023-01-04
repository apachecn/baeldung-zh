# Java–写入文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-write-to-file>

## 1。概述

在本教程中，我们将探索用 Java 写文件的不同方法。我们将使用`BufferedWriter`、`PrintWriter`、`FileOutputStream`、`DataOutputStream`、`RandomAccessFile`、`FileChannel,`和 Java 7 `Files`实用程序类。

我们还将了解在写入时锁定文件，并讨论写入文件的一些最终要点。

本教程是 Baeldung 上的 Java“回归基础”系列的一部分。

## 延伸阅读:

## [Java–将数据追加到文件中](/web/20220927030111/https://www.baeldung.com/java-append-to-file)

A quick and practical guide to appending data to files.[Read more](/web/20220927030111/https://www.baeldung.com/java-append-to-file) →

## Java 中的 filenotfounindexception

A quick and practical guide to FileNotFoundException in Java.[Read more](/web/20220927030111/https://www.baeldung.com/java-filenotfound-exception) →

## [如何用 Java 复制文件](/web/20220927030111/https://www.baeldung.com/java-copy-file)

Take a look at some common ways of copying files in Java.[Read more](/web/20220927030111/https://www.baeldung.com/java-copy-file) →

## 2。`BufferedWriter`写着

让我们从简单的**开始，使用`BufferedWriter`将一个`String`写入一个新文件**:

```java
public void whenWriteStringUsingBufferedWritter_thenCorrect() 
  throws IOException {
    String str = "Hello";
    BufferedWriter writer = new BufferedWriter(new FileWriter(fileName));
    writer.write(str);

    writer.close();
}
```

文件中的输出将是:

```java
Hello
```

然后我们可以**将一个 `String`添加到现有的文件**中:

```java
@Test
public void whenAppendStringUsingBufferedWritter_thenOldContentShouldExistToo() 
  throws IOException {
    String str = "World";
    BufferedWriter writer = new BufferedWriter(new FileWriter(fileName, true));
    writer.append(' ');
    writer.append(str);

    writer.close();
}
```

该文件将成为:

```java
Hello World
```

## 3。`PrintWriter`写着

接下来，让我们看看如何使用`PrintWriter`将格式化文本写入文件:

```java
@Test
public void givenWritingStringToFile_whenUsingPrintWriter_thenCorrect() 
  throws IOException {
    FileWriter fileWriter = new FileWriter(fileName);
    PrintWriter printWriter = new PrintWriter(fileWriter);
    printWriter.print("Some String");
    printWriter.printf("Product name is %s and its price is %d $", "iPhone", 1000);
    printWriter.close();
}
```

生成的文件将包含:

```java
Some String
Product name is iPhone and its price is 1000$
```

请注意，我们不仅将原始的`String`写入文件，还用`printf`方法写入了一些格式化的文本。

我们可以使用`FileWriter`、`BufferedWriter`，甚至`System.out`来创建作者。

## 4。`FileOutputStream`写着

现在让我们看看如何使用`FileOutputStream`到**将二进制数据写入文件。**

下面的代码将一个`String`转换成字节，并使用`FileOutputStream`将字节写入一个文件:

```java
@Test
public void givenWritingStringToFile_whenUsingFileOutputStream_thenCorrect() 
  throws IOException {
    String str = "Hello";
    FileOutputStream outputStream = new FileOutputStream(fileName);
    byte[] strToBytes = str.getBytes();
    outputStream.write(strToBytes);

    outputStream.close();
}
```

文件中的输出当然是:

```java
Hello
```

## 5。`DataOutputStream`写着

接下来，让我们看看如何使用`DataOutputStream`将`String`写入文件:

```java
@Test
public void givenWritingToFile_whenUsingDataOutputStream_thenCorrect() 
  throws IOException {
    String value = "Hello";
    FileOutputStream fos = new FileOutputStream(fileName);
    DataOutputStream outStream = new DataOutputStream(new BufferedOutputStream(fos));
    outStream.writeUTF(value);
    outStream.close();

    // verify the results
    String result;
    FileInputStream fis = new FileInputStream(fileName);
    DataInputStream reader = new DataInputStream(fis);
    result = reader.readUTF();
    reader.close();

    assertEquals(value, result);
}
```

## 6。`RandomAccessFile`写着

现在让我们来说明如何在现有文件中写入和编辑**，而不是仅仅写入一个全新的文件或追加到一个现有文件中。简单地说:我们需要随机存取。**

`RandomAccessFile`使我们能够在文件中给定偏移量的特定位置写入数据——从文件的开头开始——以字节为单位。

**这段代码从文件的开头开始写一个带有给定偏移量的整数值:**

```java
private void writeToPosition(String filename, int data, long position) 
  throws IOException {
    RandomAccessFile writer = new RandomAccessFile(filename, "rw");
    writer.seek(position);
    writer.writeInt(data);
    writer.close();
}
```

如果我们想**读取存储在特定位置**的`int`，我们可以使用这个方法:

```java
private int readFromPosition(String filename, long position) 
  throws IOException {
    int result = 0;
    RandomAccessFile reader = new RandomAccessFile(filename, "r");
    reader.seek(position);
    result = reader.readInt();
    reader.close();
    return result;
}
```

为了测试我们的函数，让我们写一个整数，编辑它，最后读回来:

```java
@Test
public void whenWritingToSpecificPositionInFile_thenCorrect() 
  throws IOException {
    int data1 = 2014;
    int data2 = 1500;

    writeToPosition(fileName, data1, 4);
    assertEquals(data1, readFromPosition(fileName, 4));

    writeToPosition(fileName2, data2, 4);
    assertEquals(data2, readFromPosition(fileName, 4));
}
```

## 7。`FileChannel`写着

**如果我们处理的是大文件，`FileChannel`可以比标准 IO 更快。**下面的代码使用`FileChannel`将`String`写入一个文件:

```java
@Test
public void givenWritingToFile_whenUsingFileChannel_thenCorrect() 
  throws IOException {
    RandomAccessFile stream = new RandomAccessFile(fileName, "rw");
    FileChannel channel = stream.getChannel();
    String value = "Hello";
    byte[] strBytes = value.getBytes();
    ByteBuffer buffer = ByteBuffer.allocate(strBytes.length);
    buffer.put(strBytes);
    buffer.flip();
    channel.write(buffer);
    stream.close();
    channel.close();

    // verify
    RandomAccessFile reader = new RandomAccessFile(fileName, "r");
    assertEquals(value, reader.readLine());
    reader.close();
}
```

## 8。用`Files`类写作

Java 7 引入了一种处理文件系统的新方法，以及一个新的实用程序类:`Files`。

使用`Files`类，我们可以创建、移动、复制和删除文件和目录。它还可以用于读取和写入文件:

```java
@Test
public void givenUsingJava7_whenWritingToFile_thenCorrect() 
  throws IOException {
    String str = "Hello";

    Path path = Paths.get(fileName);
    byte[] strToBytes = str.getBytes();

    Files.write(path, strToBytes);

    String read = Files.readAllLines(path).get(0);
    assertEquals(str, read);
}
```

## 9。写入临时文件

现在让我们试着写入一个临时文件。以下代码创建一个临时文件，并向其中写入一个`String`:

```java
@Test
public void whenWriteToTmpFile_thenCorrect() throws IOException {
    String toWrite = "Hello";
    File tmpFile = File.createTempFile("test", ".tmp");
    FileWriter writer = new FileWriter(tmpFile);
    writer.write(toWrite);
    writer.close();

    BufferedReader reader = new BufferedReader(new FileReader(tmpFile));
    assertEquals(toWrite, reader.readLine());
    reader.close();
}
```

正如我们所见，有趣和不同的只是临时文件的创建。之后，写入文件是相同的。

## 10。写入前锁定文件

最后，当写入文件时，我们有时需要额外确保没有其他人同时写入该文件。基本上，我们需要能够在写入时锁定该文件。

让我们利用`FileChannel` 在写入文件之前尝试锁定文件:

```java
@Test
public void whenTryToLockFile_thenItShouldBeLocked() 
  throws IOException {
    RandomAccessFile stream = new RandomAccessFile(fileName, "rw");
    FileChannel channel = stream.getChannel();

    FileLock lock = null;
    try {
        lock = channel.tryLock();
    } catch (final OverlappingFileLockException e) {
        stream.close();
        channel.close();
    }
    stream.writeChars("test lock");
    lock.release();

    stream.close();
    channel.close();
}
```

请注意，如果我们试图获取锁时文件已经被锁定，将会抛出一个`OverlappingFileLockException`。

## 11。注释

在探索了这么多写入文件的方法之后，让我们讨论一些重要的注意事项:

*   如果我们试图读取一个不存在的文件，就会抛出一个`FileNotFoundException`。
*   如果我们试图写入一个不存在的文件，该文件将首先被创建，不会抛出异常。
*   在使用流之后关闭它是非常重要的，因为它不是隐式关闭的，以释放与其相关联的任何资源。
*   在输出流中，`close()`方法在释放资源之前调用`flush()`,这会强制将任何缓冲的字节写入流中。

**看看常见的使用惯例，我们可以看到，比如`PrintWriter`用来写格式化文本，`FileOutputStream`用来写二进制数据，`DataOutputStream`用来写原始数据类型，`RandomAccessFile`用来写特定位置，`FileChannel`用来写更大的文件更快。这些类的一些 API 确实允许更多，但是这是一个好的开始。**

## 12。结论

本文展示了使用 Java 将数据写入文件的许多选项。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220927030111/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2 "Example of processing lines in a large file efficiently")