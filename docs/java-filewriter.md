# Java 文件编写器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filewriter>

## 1。概述

在本教程中，我们将学习和理解`java.io`包中的 *FileWriter* 类。

## 2。`FileWriter`

***FileWriter* 是一个** **专门用于编写角色文件的**[**`OutputStreamWriter`**](/web/20221205221328/https://www.baeldung.com/java-outputstream)**。它不公开任何新的操作，而是使用从 *OutputStreamWriter* 和 *Writer* 类继承的操作。**

 **在 Java 11 之前，`FileWriter`使用默认的字符编码和默认的字节缓冲区大小。然而， **Java 11 引入了四个新的构造函数，它们接受一个`[Charset](/web/20221205221328/https://www.baeldung.com/java-char-encoding),`，从而允许用户指定的`Charset`** 。不幸的是，我们仍然不能修改字节缓冲区的大小，它被设置为 8192。

### 2.1.实例化`FileWriter`

如果我们使用的是 Java 11 之前的 Java 版本，`FileWriter`类中有五个构造函数。

让我们看一下各种构造函数:

```java
public FileWriter(String fileName) throws IOException {
    super(new FileOutputStream(fileName));
}

public FileWriter(String fileName, boolean append) throws IOException {
    super(new FileOutputStream(fileName, append));
}

public FileWriter(File file) throws IOException {
    super(new FileOutputStream(file));
}

public FileWriter(File file, boolean append) throws IOException {
    super(new FileOutputStream(file, append));
}

public FileWriter(FileDescriptor fd) {
    super(new FileOutputStream(fd));
}
```

Java 11 引入了四个额外的构造函数:

```java
public FileWriter(String fileName, Charset charset) throws IOException {
    super(new FileOutputStream(fileName), charset);
}

public FileWriter(String fileName, Charset charset, boolean append) throws IOException {
    super(new FileOutputStream(fileName, append), charset);
}

public FileWriter(File file, Charset charset) throws IOException {
    super(new FileOutputStream(file), charset);
}

public FileWriter(File file, Charset charset, boolean append) throws IOException {
    super(new FileOutputStream(file, append), charset);
}
```

### 2.2。写一个`String `到一个文件

现在让我们使用一个`FileWriter`构造函数来创建一个`FileWriter`的实例，然后写入一个文件:

```java
try (FileWriter fileWriter = new FileWriter("src/test/resources/FileWriterTest.txt")) {
    fileWriter.write("Hello Folks!");
}
```

我们使用了接受文件名的`FileWriter`的单参数构造函数。然后我们使用从`Writer `类继承的`write(String str)`操作。**由于`FileWriter`是`AutoCloseable`，我们使用了 [try-with-resources](/web/20221205221328/https://www.baeldung.com/java-try-with-resources) ，这样我们就不必显式关闭`FileWriter`**。

在执行上述代码时，`String` 将被写入指定的文件:

```java
Hello Folks!
```

**`FileWriter`不保证 FileWriterTest.txt 文件是否可用或被创建。**它依赖于底层平台。

我们还必须注意，某些平台可能只允许单个`FileWriter`实例打开文件。在这种情况下，如果涉及的文件已经打开，`FileWriter`类的其他构造函数将会失败。

### 2.3。将一个`String `附加到一个文件

我们经常需要将数据添加到文件的现有内容中。现在让我们看一个支持追加的`FileWriter`的例子:

```java
try (FileWriter fileWriter = new FileWriter("src/test/resources/FileWriterTest.txt", true)) {
    fileWriter.write("Hello Folks Again!");
}
```

正如我们所见，我们使用了双参数构造函数，它接受一个文件名和一个`boolean`标志`append`。**将标志`append`作为`true`传递会创建一个`FileWriter`，允许我们将文本添加到文件**的现有内容中。

在执行代码时，我们将把`String`附加到指定文件的现有内容中:

```java
Hello Folks!Hello Folks Again! 
```

## 3。结论

在本文中，我们学习了便利类`FileWriter` 以及创建`FileWriter`的几种方法。然后，我们用它将数据写入文件。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205221328/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)**