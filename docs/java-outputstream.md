# Java 输出流指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-outputstream>

## 1.概观

在本教程中，我们将探索 Java 类`OutputStream`的细节。`O` `utputStream`是一个抽象类。这充当代表输出字节流的所有类的超类。

随着我们的深入，我们将更详细地研究“输出”和“流”这些词的含义。

## 2.Java IO 简介

**OutputStream 是 Java IO API** 的一部分，它定义了在 Java 中执行 I/O 操作所需的类。这些都打包在`java.io`名称空间中。这是从 1.0 版本开始 Java 中可用的核心包之一。

从 Java 1.4 开始，我们还在名称空间`java.nio`中打包了 Java NIO，这支持非阻塞的输入和输出操作。然而，本文的重点是作为 Java IO 一部分的`ObjectStream`。

Java IO 和 Java NIO 的相关细节可以在[这里](https://web.archive.org/web/20221013193919/https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html)找到。

### 2.1.输入和输出

Java IO 基本上为**提供了一种从源读取数据和向目的地写入数据的机制**。在这里，输入代表源，而输出代表目的。

这些源和目的地可以是任何东西，从文件、管道到网络连接。

### 2.2.流

Java IO 提供了**流的概念，它基本上代表了连续的数据流**。流可以支持许多不同类型的数据，如字节、字符、对象等。

此外，到源或目的地的连接是流所代表的。因此，它们分别以`InputStream`或`OutputStream`的形式出现。

## 3.`OutputStream`的接口

实现了一些接口，这些接口为它的子类提供了一些独特的特征。让我们快速浏览一遍。

### 3.1.`Closeable`

接口`Closeable`提供了一个名为`close() `的方法，由**处理关闭数据源或数据目的地。**`OutputStream`的每个实现都必须提供这个方法的一个实现。在这里，他们可以执行释放资源的操作。

### 3.2.`AutoCloseable`

接口`AutoCloseable`还提供了一个名为`close()`的方法，其行为类似于`Closeable`中的方法。然而，在这种情况下，当退出 try-with-resource 块时，会自动调用**方法`close()`。**

关于 try-with-resource 的更多细节可以在[这里](/web/20221013193919/https://www.baeldung.com/java-try-with-resources)找到。

### 3.3.`Flushable`

接口`Flushable`提供了一个名为`flush()`的方法，用于将数据刷新到目的地。

`OutputStream`的特定实现可能会选择缓冲先前写入的字节来进行优化，但是**对`flush()`的调用会使其立即写入目的地**。

## 4.输出流中的方法

有几个方法，每个实现类必须为它们各自的数据类型实现这些方法。

除了从`Closeable`和`Flushable`接口继承的`close()`和`flush()`方法。

### 4.1.`write(int b)`

我们可以用这种方法将**中的一个特定字节写入`OutputStream`** 。因为参数“int”由四个字节组成，所以作为 par 契约，只写入第一个低位字节，其余三个高位字节被忽略:

```java
public static void fileOutputStreamByteSingle(String file, String data) throws IOException {
    byte[] bytes = data.getBytes();
    try (OutputStream out = new FileOutputStream(file)) {
        out.write(bytes[6]);
    }
}
```

如果我们用数据调用这个方法为“Hello World！”，我们得到的结果是一个包含以下文本的文件:

```java
W
```

正如我们所看到的，这是索引为第六的字符串的第七个字符。

### 4.2.`write(byte[] b, int off, int length)`

这个重载版本的`write()`方法是为了让**将字节数组的一个子序列写到`OutputStream`** 。

它可以从字节数组中写入“长度”数量的字节，如参数所指定，从“off”确定的偏移量开始写入`OutputStream:`

```java
public static void fileOutputStreamByteSubSequence(
  String file, String data) throws IOException {
    byte[] bytes = data.getBytes();
    try (OutputStream out = new FileOutputStream(file)) {
        out.write(bytes, 6, 5);
    }
}
```

如果我们现在使用与以前相同的数据调用此方法，我们会在输出文件中得到以下文本:

```java
World
```

这是我们的数据的子串，从索引 5 开始，由五个字符组成。

### 4.3.`write(byte[] b)`

这是`write()`方法的另一个重载版本，它可以按照`OutputStream`的参数所指定的那样**写一个完整的字节数组**。

这与调用`write(b, 0, b.lengh)`具有相同的效果:

```java
public static void fileOutputStreamByteSequence(String file, String data) throws IOException {
    byte[] bytes = data.getBytes();
    try (OutputStream out = new FileOutputStream(file)) {
        out.write(bytes);
    }
}
```

当我们现在用相同的数据调用这个方法时，我们在输出文件中有完整的`String`:

```java
Hello World!
```

## 5.`OutputStream`的直接子类

现在我们将讨论一些直接已知的`OutputStream`的子类，它们分别代表它们定义的`OutputStream`的特定数据类型。

除了实现从`OutputStream`继承的方法之外，它们还定义了自己的方法。

我们不会深入这些子类的细节。

### 5.1.`FileOutputStream`

顾名思义，一个`FileOutputStream`就是**一个`OutputStream`向一个`File`T6 写入数据。`FileOutputStream`和其他`OutputStream`一样，可以写一个原始字节流。**

作为上一节的一部分，我们已经在`FileOutputStream`中检查了不同的方法。

### 5.2.`ByteArrayOutputStream`

`ByteArrayOutputStream`是`OutputStream`的**实现，可以将数据写入字节数组**。随着`ByteArrayOutputStream`向缓冲区写入数据，缓冲区不断增长。

我们可以将缓冲区的默认初始大小保持为 32 字节，或者使用一个可用的构造函数来设置特定的大小。

这里需要注意的重要一点是，方法`close()`实际上没有任何效果。即使在调用了`close()`之后，也可以安全地调用`ByteArrayOutputStream`中的其他方法。

### 5.3.`FilterOutputStream`

OutputStream 主要将字节流写入目标，但它也可以在这样做之前转换数据。`FilterOutputStream`表示执行特定数据转换的所有此类的**超类。`FilterOutputStream`总是用现有的`OutputStream`构造。**

`FilterOutputStream`的一些例子有`BufferedOutputStream`、`CheckedOutputStream`、`CipherOutputStream`、`DataOutputStream`、`DeflaterOutputStream`、`DigestOutputStream`、`InflaterOutputStream`、`PrintStream`。

### 5.4.`ObjectOutputStream`

`ObjectOutputStream`能不能**把 Java 对象的原始数据类型和图形**写到目的地。我们可以使用现有的`OutputStream`构造一个`ObjectOutputStream`来写入特定的目的地，比如 File。

请注意，`ObjectOutputStream`需要对象实现`Serializable`才能将它们写入目的地。你可以在这里找到更多关于 Java 序列化[的细节。](/web/20221013193919/https://www.baeldung.com/java-serialization)

### 5.5.`PipedOutputStream`

一个`PipedOutputStream`是**用于创建一个通信管道**。`PipedOutputStream`可以写入连接的`PipedInputStream`可以读取的数据。

`PipedOutputStream`的特点是用一个构造函数把它和一个`PipedInputStream`连接起来。或者，我们可以稍后通过使用在`PipedOutputStream`中提供的一个叫做`connect()`的方法来做这件事。

## 6.`OutputStream`缓冲

输入和输出操作通常涉及相对昂贵的操作，如磁盘访问、网络活动等。经常这样做会降低程序的效率。

我们在 Java 中有“缓冲数据流”来处理这些情况。`BufferedOutputStream` **将数据写入缓冲区，当缓冲区变满或调用方法`flush()`时，该缓冲区不经常刷新到目的地**。

`BufferedOutputStream`扩展了前面讨论的`FilterOutputStream`,并包装了现有的`OutputStream `,以写入目的地:

```java
public static void bufferedOutputStream(
  String file, String ...data) throws IOException {

    try (BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(file))) {
        for(String s : data) {
            out.write(s.getBytes());
            out.write(" ".getBytes());
        }
    }
}
```

要注意的关键点是，每个数据参数对`write()`的每次调用都只写入缓冲区，不会导致对文件的潜在开销调用。

在上面的案例中，如果我们用数据调用这个方法为“Hello”，“World！”，这将仅导致当代码从 try-with-resources 块退出时，数据被写入文件，该块调用`BufferedOutputStream`上的方法`close()`。

这会产生一个包含以下文本的输出文件:

```java
Hello World!
```

## 7.使用`OutputStreamWriter`编写文本

如前所述，字节流代表可能是一串文本字符的原始数据。现在我们可以自己获取字符数组并执行到字节数组的转换:

```java
byte[] bytes = data.getBytes();
```

Java 提供了方便的类来弥合这一差距。对于`OutputStream`的情况，这个类就是`OutputStreamWriter`。 **`OutputStreamWriter`包裹一个`OutputStream`，可以直接将字符写到想要的目的地**。

我们也可以选择为`OutputStreamWriter `提供一个字符集来进行编码:

```java
public static void outputStreamWriter(String file, String data) throws IOException {
    try (OutputStream out = new FileOutputStream(file); 
        Writer writer = new OutputStreamWriter(out,"UTF-8")) {
        writer.write(data);
    }
}
```

现在我们可以看到，在使用`FileOutputStream.` `OutputStreamWriter`之前，我们不必执行字符数组到字节数组的转换，这对于我们来说很方便`.`

当我们用类似“Hello World！”的数据调用上述方法时，这并不奇怪，这将生成一个文本如下的文件:

```java
Hello World!
```

## 8.结论

在本文中，我们讨论了 Java 抽象类`OutputStream`。我们浏览了它实现的接口和提供的方法。

然后我们讨论了 Java 中可用的`OutputStream`的一些子类。我们最后谈到了缓冲和字符流。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)