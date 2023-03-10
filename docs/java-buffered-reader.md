# BufferedReader 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-buffered-reader>

## 1。概述

是一个简化从字符输入流中读取文本的类。它缓冲字符，以便能够有效地读取文本数据。

在本教程中，我们将看看如何使用`BufferedReader` 类`.`

## 2。何时使用`BufferedReader`

一般来说，如果我们想从任何类型的输入源读取文本，不管是文件、套接字还是其他，那么`BufferedReader`就很方便。

简单地说，它使我们能够通过读取字符块并将它们存储在内部缓冲区中来最小化 I/O 操作的数量。当缓冲区有数据时，读取器将从其中读取数据，而不是直接从底层流中读取。

### 2.1。缓冲另一个阅读器

像大多数 Java I/O 类一样， **`BufferedReader `实现了** `**Decorator pattern,**` **，这意味着它期望在其构造函数中有一个`Reader`。**通过这种方式，它使我们能够灵活地扩展具有缓冲功能的`Reader`实现的实例:

```java
BufferedReader reader = 
  new BufferedReader(new FileReader("src/main/resources/input.txt"));
```

但是，如果缓冲对我们来说无关紧要，我们可以直接使用`FileReader `:

```java
FileReader reader = 
  new FileReader("src/main/resources/input.txt");
```

除了缓冲， **`BufferedReader `还提供了一些很好的助手函数来逐行读取文件**。因此，尽管直接使用`FileReader `可能看起来更简单，但是`BufferedReader` 可能是一个很大的帮助。

### 2.2。缓冲流

一般来说，**我们可以将`BufferedReader` 配置为将任何类型的输入流作为底层源**。我们可以使用`InputStreamReader`并将其包装在构造函数中:

```java
BufferedReader reader = 
  new BufferedReader(new InputStreamReader(System.in));
```

在上面的例子中，我们从`System.in `开始读取，这通常对应于来自键盘的输入。类似地，我们可以传递一个输入流来读取套接字、文件或任何可以想象的文本输入类型。唯一的先决条件是有一个合适的`InputStream` 实现。

### 2.3。BufferedReader vs 扫描仪

作为替代，我们可以使用`Scanner`类来实现与`BufferedReader.`相同的功能

然而，这两个类之间存在显著的差异，这使得它们对我们来说更方便或更不方便，这取决于我们的用例:

*   `BufferedReader`是同步的(线程安全的),而扫描器不是
*   `Scanner `可以使用正则表达式解析原始类型和字符串
*   `BufferedReader` 允许在扫描仪缓冲区大小固定的情况下更改缓冲区的大小
*   `BufferedReader`具有较大的默认缓冲区大小
*   `Scanner`隐藏`IOException`，而`BufferedReader`强迫我们处理它
*   `BufferedReader`通常比`Scanner`快，因为它只读取数据而不解析数据

考虑到这些，**如果我们解析文件中的单个令牌，那么`Scanner`会比`BufferedReader. `感觉更自然一些，但是，一次只读取一行是`BufferedReader `的亮点。**

如果需要，我们也有[关于`Scanner`](/web/20221121020730/https://www.baeldung.com/java-scanner) 的指南。

## 3。`BufferedReader`用阅读文字

让我们来看看构建、使用和销毁一个`BufferReader `来读取文本文件的整个过程。

### 3.1。初始化`BufferedReader`

首先，**让我们用它的`BufferedReader(Reader)`构造器**创建一个`BufferedReader`:

```java
BufferedReader reader = 
  new BufferedReader(new FileReader("src/main/resources/input.txt"));
```

像这样包装`FileReader`是向其他读者添加缓冲功能的好方法。

默认情况下，这将使用 8 KB 的缓冲区。然而，如果我们想要缓冲更小或更大的块，我们可以使用`BufferedReader(Reader, int)`构造函数:

```java
BufferedReader reader = 
  new BufferedReader(new FileReader("src/main/resources/input.txt")), 16384);
```

这将把缓冲区大小设置为 16384 字节(16 KB)。

最佳缓冲区大小取决于输入流的类型和运行代码的硬件等因素。为此，要达到理想的缓冲区大小，我们必须自己通过实验来找到它。

最好使用 2 的幂作为缓冲区大小，因为大多数硬件设备的块大小都是 2 的幂。

最后，**还有一种更方便的方法来创建一个`BufferedReader`，从`java.nio` `API:`中使用 [`Files`](https://web.archive.org/web/20221121020730/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html) 助手类**

```java
BufferedReader reader = 
  Files.newBufferedReader(Paths.get("src/main/resources/input.txt"))
```

如果我们想读取一个文件，像这样创建它是一个很好的缓冲方式，因为我们不必先手动创建一个`FileReader `然后再包装它。

### 3.2。逐行读取

接下来，让我们使用`readLine`方法读取文件的内容:

```java
public String readAllLines(BufferedReader reader) throws IOException {
    StringBuilder content = new StringBuilder();
    String line;

    while ((line = reader.readLine()) != null) {
        content.append(line);
        content.append(System.lineSeparator());
    }

    return content.toString();
}
```

**我们可以使用 Java 8** 中引入的`lines `方法做同样的事情，简单一点:

```java
public String readAllLinesWithStream(BufferedReader reader) {
    return reader.lines()
      .collect(Collectors.joining(System.lineSeparator()));
}
```

### 3.3。关闭流

在使用了`BufferedReader`之后，我们必须调用它的`close()`方法来释放与之相关的任何系统资源。如果我们使用`try-with-resources`块，这是自动完成的:

```java
try (BufferedReader reader = 
       new BufferedReader(new FileReader("src/main/resources/input.txt"))) {
    return readAllLines(reader);
}
```

## 4。其他有用的方法

现在让我们关注`BufferedReader.`中可用的各种有用的方法

### 4.1。读取单个字符

我们可以使用`read() `方法来读取单个字符。让我们一个字符一个字符地阅读整个内容，直到流的结尾:

```java
public String readAllCharsOneByOne(BufferedReader reader) throws IOException {
    StringBuilder content = new StringBuilder();

    int value;
    while ((value = reader.read()) != -1) {
        content.append((char) value);
    }

    return content.toString();
}
```

这将读取字符(作为 ASCII 值返回)，将它们转换为`char` 并将它们附加到结果中。我们重复这个过程，直到流结束，这由来自`read()`方法的响应值-1 表示。

### 4.2。读取多个字符

如果我们想一次读取多个字符，我们可以使用方法`read(char[] cbuf, int off, int len)`:

```java
public String readMultipleChars(BufferedReader reader) throws IOException {
    int length;
    char[] chars = new char[length];
    int charsRead = reader.read(chars, 0, length);

    String result;
    if (charsRead != -1) {
        result = new String(chars, 0, charsRead);
    } else {
        result = "";
    }

    return result;
}
```

在上面的代码示例中，我们将最多读取 5 个字符到一个 char 数组中，并从中构造一个字符串。如果在我们的读取尝试中没有读取到字符(即我们已经到达了流的末尾)，我们将简单地返回一个空字符串。

### 4.3。跳过字符

我们也可以通过调用`skip(long n)`方法跳过给定数量的字符:

```java
@Test
public void givenBufferedReader_whensSkipChars_thenOk() throws IOException {
    StringBuilder result = new StringBuilder();

    try (BufferedReader reader = 
           new BufferedReader(new StringReader("1__2__3__4__5"))) {
        int value;
        while ((value = reader.read()) != -1) {
            result.append((char) value);
            reader.skip(2L);
        }
    }

    assertEquals("12345", result);
}
```

在上面的例子中，我们从一个包含由两个下划线分隔的数字的输入字符串中读取。为了构造一个只包含数字的字符串，我们通过调用`skip `方法跳过了下划线。

### 4.4。`mark`和`reset`

我们可以使用`mark(int readAheadLimit)`和`reset() `方法来标记流中的某个位置，稍后再返回。作为一个有点做作的例子，让我们使用`mark()`和`reset()`来忽略流开头的所有空白:

```java
@Test
public void givenBufferedReader_whenSkipsWhitespacesAtBeginning_thenOk() 
  throws IOException {
    String result;

    try (BufferedReader reader = 
           new BufferedReader(new StringReader("    Lorem ipsum dolor sit amet."))) {
        do {
            reader.mark(1);
        } while(Character.isWhitespace(reader.read()))

        reader.reset();
        result = reader.readLine();
    }

    assertEquals("Lorem ipsum dolor sit amet.", result);
}
```

在上面的例子中，我们用`mark() `的方法来标记我们刚刚读到的位置。给它一个值 1 意味着只有代码会记住向前一个字符的标记。**这在这里很方便，因为一旦我们看到第一个非空白字符，我们就可以返回并重新读取该字符，而无需重新处理整个流。如果没有标记，我们会在最后一串中失去`L`。**

请注意，因为`mark()`可以抛出一个`UnsupportedOperationException`，所以将`markSupported()` 与调用`mark` `(). `的代码联系起来是很常见的，尽管在这里我们实际上并不需要它。**那是因为`markSupported() `总是为`BufferedReader`返回 true。**

当然，我们也许可以用其他方式更优雅地完成上面的工作，实际上`mark `和`reset`并不是非常典型的方法。**不过，当需要展望未来的时候，它们肯定会派上用场。**

## 5。结论

在这个快速教程中，我们通过一个实际例子学习了如何使用`BufferedReader`读取字符输入流。

最后，这些例子的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20221121020730/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)