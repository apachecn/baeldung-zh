# NIO 和 NIO.2 有什么区别？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio-vs-nio-2>

## 1.介绍

在本教程中，我们将介绍 Java IO 功能，以及它们在不同的 Java 版本中是如何变化的。首先，我们将讨论最初 Java 版本的`java.io`包。接下来，我们将回顾 Java 1.4 中引入的`java.nio`包。最后，我们将讨论`java.nio.file`包，通常称为 NIO.2 包。

## 2.Java NIO 包

第一个 Java 版本是用 [`java.io`包](/web/20220528131720/https://www.baeldung.com/java-io-file)发布的，引入了`a File` 类来访问文件系统。**`File`类代表文件和目录，并在文件系统上提供有限的操作。可以创建和删除文件，检查它们是否存在，检查读/写权限等。**

它也有一些缺点:

*   **缺少复制方法**——要复制一个文件，我们需要创建两个`File` 实例，并使用一个缓冲区从一个实例读取数据，并写入另一个`File`实例。
*   **错误处理**–一些方法返回`boolean`作为操作是否成功的指示器。
*   **一组有限的文件属性**–名称、路径、读/写权限、内存大小等等。
*   **阻塞 API**–我们的线程被阻塞，直到 IO 操作完成。

要读取一个文件，我们需要一个`FileInputStream`实例从文件中读取字节:

```java
@Test
public void readFromFileUsingFileIO() throws Exception {
    File file = new File("src/test/resources/nio-vs-nio2.txt");
    FileInputStream in = new FileInputStream(file);
    StringBuilder content = new StringBuilder();
    int data = in.read();
    while (data != -1) {
        content.append((char) data);
        data = in.read();
    }
    in.close();
    assertThat(content.toString()).isEqualTo("Hello from file!");
}
```

接下来，Java 1.4 引入了捆绑在`java.nio`包中的非阻塞 IO API(nio 代表新 IO)。NIO 的引入是为了克服`java.io`包的局限性。这个包引入了三个核心类:`Channel`、`Buffer`和`Selector`。

### 2.1.`Channel`

**[Java NIO `Channel`](/web/20220528131720/https://www.baeldung.com/java-filechannel) 是一个允许我们读写缓冲区**的类。 `Channel`类类似于`Streams`(这里我们说的是 IO `Streams`，而不是 Java 1.8 `Streams`)有一些不同。 `Channel`是双向的，而`Streams`通常是单向的，它们可以异步读写。

有几个`Channel` 类的实现，包括用于文件系统读/写的`FileChannel` ，[，](/web/20220528131720/https://www.baeldung.com/java-nio-datagramchannel)用于使用 UDP 的网络读/写，以及`SocketChannel` 用于使用 TCP 的网络读/写。

### 2.2.`Buffer`

**缓冲区是一块内存，我们可以从中读取或写入数据**。NIO `Buffer`对象包装了一个内存块。 `Buffer` 类提供了一组功能来处理内存块。要使用`Buffer`对象，我们需要理解`Buffer`类的三个主要属性:容量、位置和限制。

*   容量定义了内存块的大小。当我们向缓冲区写入数据时，我们只能写入有限的长度。当缓冲区满了，我们需要读取数据或清除它。
*   位置是我们写入数据的起点。空缓冲区从 0 开始，一直到`capacity – 1`。同样，当我们读取数据时，我们从位置值开始。
*   极限意味着我们如何从缓冲区中读写。

`Buffer`类有多种变体。每个原始 Java 类型一个，不包括`Boolean`类型加上 [`MappedByteBuffer`](/web/20220528131720/https://www.baeldung.com/java-mapped-byte-buffer) 。

要使用缓冲区，我们需要知道一些重要的方法:

*   我们使用这种方法来创建一个一定大小的缓冲区。
*   `flip()`–该方法用于从写模式切换到读模式
*   `clear() –` 清除缓冲区内容的方法
*   `compact() –` 仅清除已阅读内容的方法
*   `rewind() –`将位置重置回 0，这样我们可以重新读取缓冲区中的数据

使用前面描述的概念，让我们使用`Channel`和`Buffer`类从文件中读取内容:

```java
@Test
public void readFromFileUsingFileChannel() throws Exception {
    RandomAccessFile file = new RandomAccessFile("src/test/resources/nio-vs-nio2.txt", "r");
    FileChannel channel = file.getChannel();
    StringBuilder content = new StringBuilder();
    ByteBuffer buffer = ByteBuffer.allocate(256);
    int bytesRead = channel.read(buffer);
    while (bytesRead != -1) {
        buffer.flip();
        while (buffer.hasRemaining()) {
            content.append((char) buffer.get());
        }
        buffer.clear();
        bytesRead = channel.read(buffer);
    }
    file.close();
    assertThat(content.toString()).isEqualTo("Hello from file!");
}
```

初始化所有需要的对象后，我们从通道读入缓冲区。接下来，在 while 循环中，我们使用`flip()`方法标记要读取的缓冲区，一次读取一个字节，并将其附加到结果中。最后，我们清除数据并读取另一批数据。

### 2.3.`Selector`

**[Java NIO 选择器](/web/20220528131720/https://www.baeldung.com/java-nio-selector)允许我们用单线程管理多个通道。**要使用选择器对象监控多个通道，每个通道实例必须处于非阻塞模式，并且我们必须注册它。在通道注册之后，我们得到一个`SelectionKey`对象，表示通道和选择器之间的连接。当我们有多个通道连接到一个选择器时，我们可以使用`select()`方法来检查有多少通道可供使用。在调用了`select()`方法之后，我们可以使用`selectedKeys()`方法来获取所有准备好的通道。

### 2.4.NIO 包的缺点

引入的变化`java.nio`包更多地与低级数据 IO 相关。虽然他们允许非阻塞 API，但其他方面仍然存在问题:

*   **对符号链接的有限支持**
*   **对文件属性访问的有限支持**
*   **缺少更好的文件系统管理工具**

## 3.Java NIO.2 包

Java 1.7 引入了新的`java.nio.file`包，也称为 [NIO.2 包](/web/20220528131720/https://www.baeldung.com/java-nio-2-file-api)。这个包遵循了一种异步的非阻塞 IO 方法，这种方法在 `java.nio`包中不受支持。最显著的变化与高级文件操作有关。增加了`Files, Path,` 和`Paths`两个等级。最值得注意的低级变化是增加了 [`AsynchroniousFileChannel`](/web/20220528131720/https://www.baeldung.com/java-nio2-async-file-channel) 和`AsyncroniousSocketChannel`。

**`Path`对象表示由分隔符**分隔的目录和文件名的层次序列。根组件在最左边，而文件在右边。该类提供了`getFileName()`、`getParent()`等实用方法。`Path`类还提供了`resolve`和`relativize`方法，帮助构建不同文件之间的路径。Paths 类是一组静态实用程序方法，它们接收`String`或`URI`来创建`Path`实例。

**`Files`类提供了使用前面描述的`Path`类并对文件、目录和符号链接进行操作的实用方法。**它还提供了一种使用`readAttributes()`方法读取许多文件属性的方法。

最后，让我们看看 NIO.2 在读取文件方面与以前的 IO 版本相比如何:

```java
@Test
public void readFromFileUsingNIO2() throws Exception {
    List<String> strings = Files.readAllLines(Paths.get("src/test/resources/nio-vs-nio2.txt"));
    assertThat(strings.get(0)).isEqualTo("Hello from file!");
}
```

## 4.结论

在本文中，我们介绍了`java.nio`和 `java.nio.file`包的基础知识。正如我们所看到的，NIO.2 不是 NIO 包的新版本。NIO 包为非阻塞 IO 引入了一个低级 API，而 NIO.2 引入了更好的文件管理。这两个包不是同义词，而是相互补充。和往常一样，所有的代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20220528131720/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio-2)