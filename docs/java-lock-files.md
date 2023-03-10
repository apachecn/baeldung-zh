# 如何在 Java 中锁定文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lock-files>

## 1.概观

当读取或写入文件时，我们需要确保有适当的文件锁定机制。这确保了基于并发 I/O 的应用程序中的数据完整性。

在本教程中，我们将看看使用 [Java NIO](/web/20220628082254/https://www.baeldung.com/java-nio-2-file-api) 库实现这一点的各种方法。

## 2.文件锁简介

**一般来说，有两种类型的锁**:

简而言之，在写操作完成时，排他锁会阻止所有其他操作，包括读操作。

相比之下，共享锁允许多个进程同时读取。读锁的作用是防止另一个进程获得写锁。通常，处于一致状态的文件确实应该可以被任何进程读取。

在下一节中，我们将看到 Java 如何处理这些类型的锁。

## 3.Java 中的文件锁

Java NIO 库支持在操作系统级别锁定文件。一个`[FileChannel](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html)`的`[lock()](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#lock())`和 [`tryLock()`](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#tryLock()) 方法就是为了这个目的。

我们可以通过`[FileInputStream](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileInputStream.html#getChannel())`、`[FileOutputStream](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileOutputStream.html#getChannel())`或`[RandomAccessFile](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/RandomAccessFile.html#getChannel())`创建一个`FileChannel`。这三个都有一个返回`FileChannel`的`getChannel()`方法。

或者，我们可以通过静态的`open` 方法直接创建一个`FileChannel`:

```java
try (FileChannel channel = FileChannel.open(path, openOptions)) {
  // write to the channel
}
```

接下来，我们将回顾在 Java 中获得独占锁和共享锁的不同选项。要了解更多关于文件通道的信息，请查看我们的 Java 文件通道指南教程。

## 4.独占锁

正如我们已经知道的，在写入文件时，**我们可以通过使用一个排他锁**来阻止其他进程读取或写入文件。

我们通过调用`FileChannel`类上的`lock()`或`tryLock()`来获得独占锁。我们也可以使用它们的重载方法:

*   `lock(long position, long size, boolean shared)`
*   `tryLock(long position, long size, boolean shared)`

在这些情况下，`shared`参数必须设置为`false`。

要获得一个排他锁，我们必须使用一个可写的`FileChannel`。我们可以通过`FileOutputStream`或`RandomAccessFile`的`getChannel()`方法来创建它。或者，如前所述，我们可以使用`FileChannel`类的静态`open`方法。我们只需要将第二个参数设置为`StandardOpenOption.APPEND`:

```java
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.APPEND)) { 
    // write to channel
}
```

### 4.1.使用`FileOutputStream`的排他锁

从`FileOutputStream`创建的`FileChannel`是可写的。因此，我们可以获得一个独占锁:

```java
try (FileOutputStream fileOutputStream = new FileOutputStream("/tmp/testfile.txt");
     FileChannel channel = fileOutputStream.getChannel();
     FileLock lock = channel.lock()) { 
    // write to the channel
}
```

这里，`channel.lock()`要么阻塞直到获得锁，要么抛出异常。例如，如果指定的区域已经被锁定，就会抛出一个`OverlappingFileLockException` 。参见 [Javadoc](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#lock()) 以获得可能的异常的完整列表。

我们还可以使用`channel.tryLock()`执行非阻塞锁。如果因为另一个程序持有一个重叠的锁而未能获得锁，那么它返回`null`。如果由于任何其他原因未能这样做，则会引发适当的异常。

### 4.2.使用`RandomAccessFile`的排他锁

有了`RandomAccessFile`，我们需要在[构造函数](https://web.archive.org/web/20220628082254/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/RandomAccessFile.html#%3Cinit%3E(java.io.File,java.lang.String))的第二个参数上设置标志。

在这里，我们将使用读写权限打开文件:

```java
try (RandomAccessFile file = new RandomAccessFile("/tmp/testfile.txt", "rw");
      FileChannel channel = file.getChannel();
      FileLock lock = channel.lock()) {
    // write to the channel
} 
```

如果我们以只读模式打开文件并试图写入它的通道，它将抛出一个`NonWritableChannelException`。

### 4.3.排他锁需要一个可写的`FileChannel`

如前所述，独占锁需要一个可写通道。因此，我们不能通过从`FileInputStream`创建的`FileChannel`来获得排他锁:

```java
Path path = Files.createTempFile("foo","txt");
Logger log = LoggerFactory.getLogger(this.getClass());
try (FileInputStream fis = new FileInputStream(path.toFile()); 
    FileLock lock = fis.getChannel().lock()) {
    // unreachable code
} catch (NonWritableChannelException e) {
    // handle exception
}
```

在上面的例子中，`lock()`方法将抛出一个`NonWritableChannelException`。事实上，这是因为我们在`FileInputStream`上调用了`getChannel`，这创建了一个只读通道。

这个例子只是为了说明我们不能写一个不可写的通道。在真实的场景中，我们不会捕捉并重新抛出异常。

## 5.共享锁

记住，共享锁也叫做`read`锁。因此，要获得读锁，我们必须使用可读的`FileChannel`。

这样的`FileChannel`可以通过在`FileInputStream`或`RandomAccessFile`上调用`getChannel()`方法来获得。同样，另一种选择是使用`FileChannel` 类的静态`open`方法。在这种情况下，我们将第二个参数设置为`StandardOpenOption.READ`:

```java
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ);
    FileLock lock = channel.lock(0, Long.MAX_VALUE, true)) {
    // read from the channel
}
```

这里需要注意的一点是，我们选择通过调用`lock(0, Long.MAX_VALUE, true)`来锁定整个文件。我们也可以通过将前两个参数更改为不同的值，只锁定文件的特定区域。在共享锁的情况下，第三个参数必须设置为`true`。

为了简单起见，在下面的所有例子中，我们将锁定整个文件，但是请记住，我们总是可以锁定文件的特定区域。

### 5.1.使用`FileInputStream`的共享锁

从`FileInputStream`获得的`FileChannel`是可读的。因此，我们可以获得一个共享锁:

```java
try (FileInputStream fileInputStream = new FileInputStream("/tmp/testfile.txt");
    FileChannel channel = fileInputStream.getChannel();
    FileLock lock = channel.lock(0, Long.MAX_VALUE, true)) {
    // read from the channel
}
```

在上面的代码片段中，对通道上的`lock()`的调用将会成功。这是因为共享锁只要求通道可读。自从我们从一个`FileInputStream`创建它以来，情况就是这样。

### 5.2.使用`RandomAccessFile`的共享锁

这一次，我们可以只使用**读**权限打开文件:

```java
try (RandomAccessFile file = new RandomAccessFile("/tmp/testfile.txt", "r"); 
     FileChannel channel = file.getChannel();
     FileLock lock = channel.lock(0, Long.MAX_VALUE, true)) {
     // read from the channel
}
```

在这个例子中，我们创建了一个具有读权限的`RandomAccessFile`。我们可以从中创建一个可读的通道，从而创建一个共享锁。

### 5.3.共享锁需要可读的`FileChannel`

因此，我们不能通过从`FileOutputStream`创建的`FileChannel`获取共享锁:

```java
Path path = Files.createTempFile("foo","txt");
try (FileOutputStream fis = new FileOutputStream(path.toFile()); 
    FileLock lock = fis.getChannel().lock(0, Long.MAX_VALUE, true)) {
    // unreachable code
} catch (NonWritableChannelException e) { 
    // handle exception
} 
```

在这个例子中，对`lock()`的调用试图获得从`FileOutputStream`创建的通道上的共享锁。这样的通道是只写的。它不能满足通道必须可读的要求。这将引发一场`NonWritableChannelException`。

同样，这个片段只是为了证明我们不能从不可读的通道中读取。

## 6.需要考虑的事项

实际上，使用文件锁是困难的；锁定装置不是便携式的。考虑到这一点，我们需要精心设计我们的锁定逻辑。

在 POSIX 系统中，锁是建议性的。读取或写入给定文件的不同进程必须在锁定协议上达成一致。这将确保文件的完整性。操作系统本身不会强制任何锁定。

在 Windows 上，除非允许共享，否则锁将是独占的。讨论特定于操作系统的机制的优缺点超出了本文的范围。然而，在实现锁定机制时，了解这些细微差别很重要。

## 7.结论

在本教程中，我们回顾了在 Java 中获取文件锁的几种不同的方法。

首先，我们从理解两种主要的锁定机制以及 Java NIO 库如何帮助锁定文件开始。然后，我们通过一系列简单的例子展示了我们可以在应用程序中获得独占和共享锁。我们还研究了使用文件锁时可能遇到的典型异常的类型。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220628082254/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio-2)