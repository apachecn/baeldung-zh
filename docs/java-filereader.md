# Java FileReader 类指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filereader>

## 1.概观

顾名思义，`[FileReader](https://web.archive.org/web/20221013193920/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileReader.html)`是一个 **Java 类，它使得读取文件**的内容变得很容易。

在本教程中，我们将学习`Reader`的基本概念，以及如何使用`FileReader`类在 Java 中对字符流进行读操作。

## 2.`Reader `基础知识

如果我们看一下`FileReader`类的代码，我们会注意到这个类包含了创建一个`FileReader`对象的最少代码，没有其他方法。

这引发了诸如“谁在这个类后面做繁重的工作？”

要回答这个问题，我们必须了解 Java 中 [`Reader`](https://web.archive.org/web/20221013193920/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/Reader.html) 类的概念和层次结构。

**`Reader`是一个抽象基类，通过它的一个具体实现使得读取字符成为可能。**它定义了以下从任何介质(如内存或文件系统)读取字符的基本操作:

*   阅读单个字符
*   读取字符数组
*   标记并重置字符流中的给定位置
*   读取字符流时跳过位置
*   关闭输入流

自然，`Reader`类的所有实现都必须实现所有抽象方法，即`read()`和`close()`。此外，大多数实现还会覆盖其他继承的方法，以提供额外的功能或更好的性能。

### 2.1.何时使用`FileReader`

现在我们已经对`Reader`有了一些了解，我们准备将我们的焦点带回到`FileReader`类。

`FileReader`继承了`InputStreamReader`的功能，后者是一个`Reader`实现，用于从输入流中读取字节作为字符。

让我们看看类定义中的层次结构:

```java
public class InputStreamReader extends Reader {}

public class FileReader extends InputStreamReader {}
```

一般来说，我们可以使用一个`InputStreamReader`从任何输入源读取字符。

然而，当从文件中读取文本时，使用`InputStreamReader`就像用剑切苹果一样。当然，合适的工具是一把刀，这正是`FileReader`承诺的。

当我们想使用系统的默认字符集从文件中读取文本时，我们可以使用`FileReader`**。**对于任何其他高级功能，直接使用`InputStreamReader`类是理想的。

## 3.用`FileReader`读取文本文件

让我们通过一个使用`FileReader` 实例从`HelloWorld.txt`文件中读取字符的编码练习。

### 3.1.创建一个`FileReader`

作为一个方便的类， **`FileReader`提供了三个重载的构造函数**，它们可以用来初始化一个阅读器，该阅读器可以从作为输入源的文件中读取数据。

让我们来看看这些构造函数:

```java
public FileReader(String fileName) throws FileNotFoundException {
    super(new FileInputStream(fileName));
}

public FileReader(File file) throws FileNotFoundException {
    super(new FileInputStream(file));
}

public FileReader(FileDescriptor fd) {
    super(new FileInputStream(fd));
}
```

在我们的例子中，我们知道输入文件的文件名。因此，我们可以使用第一个构造函数来初始化读取器:

```java
FileReader fileReader = new FileReader(path);
```

### 3.2.阅读单个字符

接下来，让我们创建`readAllCharactersOneByOne()`，一种从文件中一次读取一个字符的方法:

```java
public static String readAllCharactersOneByOne(Reader reader) throws IOException {
    StringBuilder content = new StringBuilder();
    int nextChar;
    while ((nextChar = reader.read()) != -1) {
        content.append((char) nextChar);
    }
    return String.valueOf(content);
}
```

从上面的代码中我们可以看到，我们已经在一个循环中使用了****`read()`方法来逐个读取字符，直到它返回-1** ，这意味着没有更多的字符要读取。**

 **现在，让我们通过验证从文件中读取的文本与预期的文本相匹配来测试我们的代码:

```java
@Test
public void givenFileReader_whenReadAllCharacters_thenReturnsContent() throws IOException {
    String expectedText = "Hello, World!";
    File file = new File(FILE_PATH);
    try (FileReader fileReader = new FileReader(file)) {
        String content = FileReaderExample.readAllCharactersOneByOne(fileReader);
        Assert.assertEquals(expectedText, content);
    }
}
```

### 3.3.读取字符数组

我们甚至可以使用继承的`read(char cbuf[], int off, int len)`方法一次读取多个字符:

```java
public static String readMultipleCharacters(Reader reader, int length) throws IOException {
    char[] buffer = new char[length];
    int charactersRead = reader.read(buffer, 0, length);
    if (charactersRead != -1) {
        return new String(buffer, 0, charactersRead);
    } else {
        return "";
    }
}
```

在读取数组中的多个字符时，`read()`的返回值有细微的差别。这里的**返回值要么是读取的字符数，要么是-1** ，如果读取器已经到达输入流的末尾。

接下来，让我们测试代码的正确性:

```java
@Test
public void givenFileReader_whenReadMultipleCharacters_thenReturnsContent() throws IOException {
    String expectedText = "Hello";
    File file = new File(FILE_PATH);
    try (FileReader fileReader = new FileReader(file)) {
        String content = FileReaderExample.readMultipleCharacters(fileReader, 5);
        Assert.assertEquals(expectedText, content);
    }
}
```

## 4.限制

我们已经看到`FileReader`类依赖于默认的系统字符编码。

因此，对于需要为字符集、缓冲区大小或输入流使用自定义值的情况，**，我们必须**使用`InputStreamReader`** 。**

此外，我们都知道 I/O 周期非常昂贵，并且会给我们的应用程序带来延迟。因此，**通过在我们的`FileReader`对象**周围包装一个`[BufferedReader](/web/20221013193920/https://www.baeldung.com/java-buffered-reader)`来最小化 I/O 操作的数量，这是对我们最有利的:

```java
BufferedReader in = new BufferedReader(fileReader);
```

## 5.结论

在本教程中，我们通过一些例子了解了`Reader`的基本概念以及`FileReader`如何简化对文本文件的读操作。

和往常一样，教程的完整源代码可以在 [GitHub](https://web.archive.org/web/20221013193920/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis) 上获得。**