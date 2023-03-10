# NIO2 异步文件通道指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio2-async-file-channel>

## 1。概述

在本文中，我们将探索 Java 7 中新 I/O (NIO2)的关键附加 API 之一，异步文件通道 API。

如果你是异步通道 API 的新手，我们在这个网站上有一篇介绍性的文章，你可以在继续之前点击[这个链接](/web/20221208143830/https://www.baeldung.com/java-nio-2-async-channels)来阅读。

您还可以阅读更多关于 NIO.2 [文件操作](/web/20221208143830/https://www.baeldung.com/java-nio-2-file-api)和[路径操作](/web/20221208143830/https://www.baeldung.com/java-nio-2-path)的内容——理解这些将使本文更容易理解。

为了在我们的项目中使用 NIO2 异步文件通道，我们必须导入 `java.nio.channels`包，因为它捆绑了所有必需的类:

```java
import java.nio.channels.*;
```

## 2。`AsynchronousFileChannel`

在这一节中，我们将探索如何使用使我们能够对文件执行异步操作的主类，即`AsynchronousFileChannel`类。为了创建它的实例，我们调用静态的`open`方法:

```java
Path filePath = Paths.get("/path/to/file");

AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
  filePath, READ, WRITE, CREATE, DELETE_ON_CLOSE); 
```

所有枚举值都来自 Sta `ndardOpenOption`。

open API 的第一个参数是一个代表文件位置的`Path`对象。要阅读更多关于 NIO2 中路径操作的内容，请点击[链接](/web/20221208143830/https://www.baeldung.com/java-nio-2-path)。其他参数组成了一个集合，指定应该对返回的文件通道可用的选项。

我们创建的异步文件通道可用于对文件执行所有已知的操作。为了只执行操作的子集，我们将只为这些操作指定选项。例如，只读:

```java
Path filePath = Paths.get("/path/to/file");

AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
  filePath, StandardOpenOption.READ);
```

## 3。从文件中读取

就像 NIO2 中的所有异步操作一样，读取文件内容有两种方式。使用`Future`和使用`CompletionHandler`。在每种情况下，我们都使用返回通道的`read` API。

在 maven 的 test resources 文件夹中，或者如果没有使用 maven，在源目录中，让我们创建一个名为`file.txt`的文件，其开头只有文本`baeldung.com`。我们现在将演示如何阅读这些内容。

### 3.1。未来的方法

首先，我们将看到如何使用`Future`类异步读取文件:

```java
@Test
public void givenFilePath_whenReadsContentWithFuture_thenCorrect() {
    Path path = Paths.get(
      URI.create(
        this.getClass().getResource("/file.txt").toString()));
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
      path, StandardOpenOption.READ);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    Future<Integer> operation = fileChannel.read(buffer, 0);

    // run other code as operation continues in background
    operation.get();

    String fileContent = new String(buffer.array()).trim();
    buffer.clear();

    assertEquals(fileContent, "baeldung.com");
}
```

在上面的代码中，在创建了一个文件通道之后，我们使用了`read`API——它使用一个`ByteBuffer`来存储从通道中读取的内容作为它的第一个参数。

第二个参数是一个 long 类型的值，表示文件中开始读取的位置。

无论文件是否被读取，该方法都会立即返回。

接下来，随着操作在后台继续，我们可以执行任何其他代码。当我们执行完其他代码时，我们可以调用`get()` API，如果操作已经完成，它会立即返回，否则它会阻塞，直到操作完成。

我们的断言确实证明了文件中的内容已经被读取。

如果我们将`read` API 调用中的 position 参数从零更改为其他值，我们也会看到效果。例如，字符串`baeldung.com`中的第七个字符是`g`。因此，将位置参数更改为 7 将导致缓冲区包含字符串`g.com`。

### 3.2。`CompletionHandler`接近

接下来，我们将看到如何使用`CompletionHandler`实例读取文件内容:

```java
@Test
public void 
  givenPath_whenReadsContentWithCompletionHandler_thenCorrect() {

    Path path = Paths.get(
      URI.create( this.getClass().getResource("/file.txt").toString()));
    AsynchronousFileChannel fileChannel 
      = AsynchronousFileChannel.open(path, StandardOpenOption.READ);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    fileChannel.read(
      buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {

        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            // result is number of bytes read
            // attachment is the buffer containing content
        }
        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {

        }
    });
}
```

在上面的代码中，我们使用了第二种类型的`read` API。它仍然将一个`ByteBuffer`和`read` 操作的开始位置分别作为第一个和第二个参数。第三个参数是`CompletionHandler`实例。

完成处理程序的第一个通用类型是操作的返回类型，在本例中，是一个表示读取的字节数的整数。

第二是附件的类型。我们选择附加缓冲区，这样当`read`完成时，我们可以使用`completed`回调 API 中的文件内容。

从语义上讲，这不是真正有效的单元测试，因为我们不能在`completed`回调方法中做断言。然而，我们这样做是为了一致性，因为我们希望我们的代码尽可能地具有`copy-paste-run-`能力。

## 4。写入文件

Java NIO2 还允许我们对文件执行写操作。正如我们对其他操作所做的那样，我们可以用两种方式写入文件。使用`Future`和使用`CompletionHandler`。在每种情况下，我们都使用返回通道的`write` API。

创建一个用于写入文件的`AsynchronousFileChannel`可以像这样完成:

```java
AsynchronousFileChannel fileChannel
  = AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);
```

### 4.1。特殊注意事项

注意传递给`open` API 的选项。如果我们希望创建一个由`path`表示的文件，以防它不存在，我们还可以添加另一个选项`StandardOpenOption.CREATE`。另一个常见的选项是`StandardOpenOption.APPEND`，它不会覆盖文件中的现有内容。

出于测试目的，我们将使用以下代码行来创建文件通道:

```java
AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
  path, WRITE, CREATE, DELETE_ON_CLOSE);
```

这样，我们将提供任意路径，并确保文件将被创建。测试退出后，创建的文件将被删除。为了确保创建的文件在测试退出后不会被删除，您可以删除最后一个选项。

为了运行断言，我们需要在写入之后尽可能地读取文件内容。让我们将读取逻辑隐藏在一个单独的方法中，以避免冗余:

```java
public static String readContent(Path file) {
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
      file, StandardOpenOption.READ);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    Future<Integer> operation = fileChannel.read(buffer, 0);

    // run other code as operation continues in background
    operation.get();     

    String fileContent = new String(buffer.array()).trim();
    buffer.clear();
    return fileContent;
}
```

### 4.2。`Future`接近

使用`Future`类异步写入文件:

```java
@Test
public void 
  givenPathAndContent_whenWritesToFileWithFuture_thenCorrect() {

    String fileName = UUID.randomUUID().toString();
    Path path = Paths.get(fileName);
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
      path, WRITE, CREATE, DELETE_ON_CLOSE);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    buffer.put("hello world".getBytes());
    buffer.flip();

    Future<Integer> operation = fileChannel.write(buffer, 0);
    buffer.clear();

    //run other code as operation continues in background
    operation.get();

    String content = readContent(path);
    assertEquals("hello world", content);
}
```

让我们检查一下上面的代码中发生了什么。我们创建一个随机文件名，并用它来获得一个`Path`对象。我们使用这个路径打开一个带有前面提到的选项的异步文件通道。

然后，我们将想要写入文件的内容放在一个缓冲区中，并执行`write`。我们使用我们的 helper 方法来读取文件的内容，并确认这就是我们所期望的。

### 4.3。`CompletionHandler`接近

我们还可以使用完成处理程序，这样我们就不必在 while 循环中等待操作完成:

```java
@Test
public void 
  givenPathAndContent_whenWritesToFileWithHandler_thenCorrect() {

    String fileName = UUID.randomUUID().toString();
    Path path = Paths.get(fileName);
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
      path, WRITE, CREATE, DELETE_ON_CLOSE);

    ByteBuffer buffer = ByteBuffer.allocate(1024);
    buffer.put("hello world".getBytes());
    buffer.flip();

    fileChannel.write(
      buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {

        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            // result is number of bytes written
            // attachment is the buffer
        }
        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {

        }
    });
}
```

当我们这次调用 write API 时，唯一的新东西是第三个参数，在这里我们传递了一个类型为`CompletionHandler`的匿名内部类。

当操作完成时，该类调用它的 completed 方法，在该方法中我们可以定义应该发生什么。

## 5。结论

在本文中，我们探索了 Java NIO2 的异步文件通道 API 的一些最重要的特性。

要获得本文的所有代码片段和完整源代码，可以访问 [Github 项目](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)。