# Java 文件通道指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filechannel>

## 1。概述

在这个快速教程中，我们将看看 [`Java NIO`](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/package-summary.html) 库中提供的`[FileChannel](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html)`类。我们将讨论**如何使用`FileChannel`和`ByteBuffer`** 读写数据。

我们还将探索使用`FileChannel`和它的一些其他文件操作特性的优势。

## 2.`FileChannel`的优点

`FileChannel`的优点包括:

*   在文件的特定位置读写
*   将文件的一部分直接加载到内存中，这样效率会更高
*   我们可以以更快的速度将文件数据从一个通道传输到另一个通道
*   我们可以锁定文件的一部分来限制其他线程的访问
*   为了避免数据丢失，我们可以强制将文件更新立即写入存储

## 3.用`FileChannel`阅读

当我们读取大文件时，执行速度比标准 I/O 更快。

我们应该注意到，虽然部分`Java NIO`、**、`FileChannel`操作是阻塞**的，并没有非阻塞模式。

### 3.1.使用`FileChannel`读取文件

让我们了解如何在包含以下内容的文件上使用`FileChannel`来读取文件:

```java
Hello world
```

该测试读取文件并检查它是否正常读取:

```java
@Test
public void givenFile_whenReadWithFileChannelUsingRandomAccessFile_thenCorrect() 
  throws IOException {
    try (RandomAccessFile reader = new RandomAccessFile("src/test/resources/test_read.in", "r");
        FileChannel channel = reader.getChannel();
        ByteArrayOutputStream out = new ByteArrayOutputStream()) {

        int bufferSize = 1024;
        if (bufferSize > channel.size()) {
           bufferSize = (int) channel.size();
        }
        ByteBuffer buff = ByteBuffer.allocate(bufferSize);

        while (channel.read(buff) > 0) {
            out.write(buff.array(), 0, buff.position());
            buff.clear();
        }

     String fileContent = new String(out.toByteArray(), StandardCharsets.UTF_8);

     assertEquals("Hello world", fileContent);
    }
}
```

这里我们使用`FileChannel`、`RandomAccessFile`和`ByteBuffer.` 从文件中读取字节

我们还要注意的是，**多个并发线程可以安全地使用****`FileChannels`****。**然而，一次只允许一个线程执行更新通道位置或改变其文件大小的操作。这会阻止其他线程尝试类似的操作，直到前一个操作完成。

但是，提供显式通道位置的操作可以并发运行而不会被阻塞。

### 3.2.打开一个`FileChannel`

为了使用`FileChannel`读取文件，我们必须打开它。

让我们看看如何使用`RandomAccessFile`打开一个`FileChannel `:

```java
RandomAccessFile reader = new RandomAccessFile(file, "r");
FileChannel channel = reader.getChannel();
```

**模式“r”表示通道仅“开放读取”。**我们应该注意，关闭一个`RandomAccessFile `也会关闭相关的通道。

接下来，我们将看到使用`FileInputStream`打开一个`FileChannel`来读取一个文件:

```java
FileInputStream fin= new FileInputStream(file);
FileChannel channel = fin.getChannel();
```

同样，关闭一个`FileInputStream`也会关闭与之关联的通道。

### 3.3.从`FileChannel`读取数据

为了读取数据，我们可以使用`read` 方法之一。

让我们看看如何读取一个字节序列。我们将使用一个`ByteBuffer`来保存数据:

```java
ByteBuffer buff = ByteBuffer.allocate(1024);
int noOfBytesRead = channel.read(buff);
String fileContent = new String(buff.array(), StandardCharsets.UTF_8);

assertEquals("Hello world", fileContent);
```

接下来，我们将了解如何从文件位置开始读取一个字节序列:

```java
ByteBuffer buff = ByteBuffer.allocate(1024);
int noOfBytesRead = channel.read(buff, 5);
String fileContent = new String(buff.array(), StandardCharsets.UTF_8);
assertEquals("world", fileContent);
```

**我们应该注意到需要一个`Charset` 将一个字节数组解码成`String`** 。

我们指定最初用来编码字节的`Charset`。没有它`,` ,我们可能会以乱码文本结束。特别是，像`UTF-8`和`UTF-16`这样的多字节编码可能无法解码文件的任意部分，因为一些多字节字符可能不完整。

## 4.用`FileChannel`书写

### 4.1.使用`FileChannel`写入文件

让我们来探索如何使用`FileChannel`进行写作:

```java
@Test
public void whenWriteWithFileChannelUsingRandomAccessFile_thenCorrect()   
  throws IOException {
    String file = "src/test/resources/test_write_using_filechannel.txt";
    try (RandomAccessFile writer = new RandomAccessFile(file, "rw");
        FileChannel channel = writer.getChannel()){
        ByteBuffer buff = ByteBuffer.wrap("Hello world".getBytes(StandardCharsets.UTF_8));

        channel.write(buff);

     // verify
     RandomAccessFile reader = new RandomAccessFile(file, "r");
     assertEquals("Hello world", reader.readLine());
     reader.close();
    }
}
```

### 4.2.打开一个`FileChannel`

为了使用`FileChannel`写入文件，我们必须打开它。

让我们看看如何使用`RandomAccessFile`打开一个`FileChannel `:

```java
RandomAccessFile writer = new RandomAccessFile(file, "rw");
FileChannel channel = writer.getChannel();
```

**模式“rw”表示通道“开放读写”。**

让我们看看如何使用`FileOutputStream`打开一个`FileChannel`:

```java
FileOutputStream fout = new FileOutputStream(file);
FileChannel channel = fout.getChannel(); 
```

### 4.3.使用`FileChannel`写入数据

要用`FileChannel`写数据，我们可以使用`write`方法之一。

让我们看看如何编写一个字节序列，使用一个`ByteBuffer`来存储数据:

```java
ByteBuffer buff = ByteBuffer.wrap("Hello world".getBytes(StandardCharsets.UTF_8));
channel.write(buff); 
```

接下来，我们将了解如何从文件位置开始编写一个字节序列:

```java
ByteBuffer buff = ByteBuffer.wrap("Hello world".getBytes(StandardCharsets.UTF_8));
channel.write(buff, 5); 
```

## 5.当前位置

允许我们获得和改变我们阅读或书写的位置。

让我们看看如何获得当前位置:

```java
long originalPosition = channel.position();
```

接下来，我们来看看如何设置位置:

```java
channel.position(5);
assertEquals(originalPosition + 5, channel.position());
```

## 6.获取文件的大小

让我们看看如何使用`[FileChannel.size](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#size() "API usage")`方法获得文件的字节大小:

```java
@Test
public void whenGetFileSize_thenCorrect() 
  throws IOException {
    RandomAccessFile reader = new RandomAccessFile("src/test/resources/test_read.in", "r");
    FileChannel channel = reader.getChannel();

    // the original file size is 11 bytes.
    assertEquals(11, channel.size());

    channel.close();
    reader.close();
}
```

## 7.截断文件

让我们了解如何使用`[FileChannel.truncate](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#truncate(long) "API usage")`方法将文件截断到给定的字节大小:

```java
@Test
public void whenTruncateFile_thenCorrect() 
  throws IOException {
    String input = "this is a test input";

    FileOutputStream fout = new FileOutputStream("src/test/resources/test_truncate.txt");
    FileChannel channel = fout.getChannel();

    ByteBuffer buff = ByteBuffer.wrap(input.getBytes());
    channel.write(buff);
    buff.flip();

    channel = channel.truncate(5);
    assertEquals(5, channel.size());

    fout.close();
    channel.close();
} 
```

## 8.强制文件更新到存储中

出于性能原因，操作系统可能会缓存文件更改，如果系统崩溃，数据可能会丢失。要强制文件内容和元数据持续写入磁盘，我们可以使用`force`方法:

```java
channel.force(true);
```

只有当文件驻留在本地设备上时，才保证使用这种方法。

## 9.将文件的一部分装入内存

让我们看看如何使用`[FileChannel.map](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#map(java.nio.channels.FileChannel.MapMode,long,long)).` 将文件的一部分加载到内存中。我们使用 [`FileChannel.MapMode.READ_ONLY`](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.MapMode.html) 以只读模式打开文件:

```java
@Test
public void givenFile_whenReadAFileSectionIntoMemoryWithFileChannel_thenCorrect() 
  throws IOException { 
    try (RandomAccessFile reader = new RandomAccessFile("src/test/resources/test_read.in", "r");
        FileChannel channel = reader.getChannel();
        ByteArrayOutputStream out = new ByteArrayOutputStream()) {

        MappedByteBuffer buff = channel.map(FileChannel.MapMode.READ_ONLY, 6, 5);

        if(buff.hasRemaining()) {
          byte[] data = new byte[buff.remaining()];
          buff.get(data);
          assertEquals("world", new String(data, StandardCharsets.UTF_8));	
        }
    }
}
```

类似地，我们可以使用`FileChannel.MapMode.READ_WRITE` 以读写模式打开文件。

我们也可以使用`**FileChannel.MapMode.PRIVATE**` **模式，这里的修改不适用于原始文件`.`**

## 10.锁定文件的一部分

让我们了解如何使用 [`FileChannel.tryLock`](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#tryLock(long,long,boolean)) 方法锁定文件的一部分，以防止对一部分的并发访问:

```java
@Test
public void givenFile_whenWriteAFileUsingLockAFileSectionWithFileChannel_thenCorrect() 
  throws IOException { 
    try (RandomAccessFile reader = new RandomAccessFile("src/test/resources/test_read.in", "rw");
        FileChannel channel = reader.getChannel();
        FileLock fileLock = channel.tryLock(6, 5, Boolean.FALSE )){

        //do other operations...

        assertNotNull(fileLock);
    }
}
```

`tryLock`方法试图获取文件部分的锁。如果请求的文件部分已经被另一个线程阻塞，它抛出一个`[OverlappingFileLockException](https://web.archive.org/web/20221013193919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/OverlappingFileLockException.html)`异常。这个方法还需要一个`boolean`参数来请求共享锁或排他锁。

我们应该注意，一些操作系统可能不允许共享锁，而是默认为独占锁。

## 11.关闭一个`FileChannel`

最后，当我们使用完一个`FileChannel`时，我们必须关闭它。在我们的例子中，我们使用了`[try-with-resources](/web/20221013193919/https://www.baeldung.com/java-try-with-resources).`

如有必要，我们可以用`close`方法直接关闭`FileChannel `:

```java
channel.close();
```

## 12.结论

在本教程中，我们已经看到了**如何使用`FileChannel`来读写文件**。此外，我们还探索了如何读取和更改文件大小及其当前读/写位置，以及如何在并发或数据关键型应用程序中使用`FileChannels`。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)