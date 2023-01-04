# 非法字符编译错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-illegal-character-error>

## 1.概观

非法字符编译错误是一个文件类型编码错误。如果我们在创建文件时使用了不正确的编码，就会产生这种错误。结果，在像 Java 这样的语言中，当我们试图编译我们的项目时，我们会得到这种类型的错误。在本教程中，我们将详细描述这个问题以及我们可能遇到的一些场景，然后，我们将给出一些如何解决它的例子。

## 2.非法字符编译错误

### 2.1.字节顺序标记

在我们进入字节顺序标记之前，我们需要快速看一下 UCS (Unicode)转换格式(UTF)。 **UTF 是一种字符编码格式，可以用 Unicode 编码所有可能的字符码点**。有几种 UTF 编码。在所有这些中，UTF-8 是使用最多的。

UTF-8 使用 8 位可变宽度编码来最大化与 ASCII 的兼容性。当我们在文件中使用这种编码时，我们可能会发现一些代表 Unicode 码位的字节。因此，我们的文件以 U+FEFF 字节顺序标记(BOM)开始。这个标记，正确使用，是看不见的。但是，在某些情况下，这可能会导致数据错误。

**在 UTF-8 编码中，BOM 的存在并不重要**。 虽然 并不重要，BOM 还是可能出现在 UTF-8 编码的文本中。可以通过编码转换或文本编辑器将内容标记为 UTF-8 来添加 BOM。

像 Windows 上的记事本这样的文本编辑器可以产生这种附加功能。因此，当我们使用类似记事本的文本编辑器创建一个代码示例并尝试运行它时，我们可能会得到一个编译错误。相比之下，现代的 ide 将创建的文件编码为没有 BOM 的 UTF-8。下一节将展示这个问题的一些例子。

### 2.2.类包含非法字符编译错误

通常，我们使用高级 ide，但有时，我们使用文本编辑器。不幸的是，正如我们所了解的，一些文本编辑器可能会产生更多的问题而不是解决方案，因为保存带有 BOM 的文件可能会导致 Java 中的编译错误。**“非法字符”错误发生在编译阶段，因此很容易检测到**。下一个例子向我们展示了它是如何工作的。

首先，让我们在文本编辑器中编写一个简单的类，比如 Notepad。这个类只是一个代表——我们可以编写任何代码来测试。接下来，我们将我们的文件与要测试的 BOM 一起保存:

```
public class TestBOM {
    public static void main(String ...args){
        System.out.println("BOM Test");
    }
}
```

现在，当我们试图使用`javac`命令编译这个文件时:

```
$ javac ./TestBOM.java
```

因此，我们得到错误消息:

```
∩╗┐public class TestBOM {
 ^
.\TestBOM.java:1: error: illegal character: '\u00bf'
∩╗┐public class TestBOM {
  ^
2 errors
```

理想情况下，要解决这个问题，唯一要做的就是将文件保存为 UTF-8 格式，不进行 BOM 编码。之后问题就解决了。我们应该经常检查我们保存的文件是否没有 BOM 。

**解决这个问题的另一种方法是使用类似 [`dos2unix`](https://web.archive.org/web/20220524061145/https://linux.die.net/man/1/dos2unix)** 的工具。该工具将删除 BOM，并处理 Windows 文本文件的其他特性。

## 3.读取文件

另外，让我们分析一些读取用 BOM 编码的文件的例子。

最初，我们需要创建一个包含 BOM 的文件，用于我们的测试。该文件包含我们的示例文本“Hello world with BOM”–这将是我们期望的字符串。接下来，我们开始测试。

### 3.1.使用 BufferedReader 读取文件

首先，我们将使用`BufferedReader` 类测试文件:

```
@Test
public void whenInputFileHasBOM_thenUseInputStream() throws IOException {
    String line;
    String actual = "";
    try (BufferedReader br = new BufferedReader(new InputStreamReader(file))) {
        while ((line = br.readLine()) != null) {
            actual += line;
        }
    }
    assertEquals(expected, actual);
}
```

在这种情况下，当我们试图断言字符串相等时，**我们得到一个错误**:

```
org.opentest4j.AssertionFailedError: expected: <Hello world with BOM.> but was: <Hello world with BOM.>
Expected :Hello world with BOM.
Actual   :Hello world with BOM.
```

实际上，如果我们浏览测试响应，两个字符串看起来似乎是相等的。即便如此，字符串的实际值包含了 BOM。结果，字符串不相等。

此外，**快速解决方案是替换 BOM 字符**:

```
@Test
public void whenInputFileHasBOM_thenUseInputStreamWithReplace() throws IOException {
    String line;
    String actual = "";
    try (BufferedReader br = new BufferedReader(new InputStreamReader(file))) {
        while ((line = br.readLine()) != null) {
            actual += line.replace("\uFEFF", "");
        }
    }
    assertEquals(expected, actual);
}
```

`replace` 方法从我们的字符串中清除 BOM，所以我们的测试通过了。我们需要小心使用`replace` 方法。要处理的大量文件可能会导致性能问题。

### 3.2.使用 Apache Commons IO 读取文件

另外， **[Apache Commons IO](/web/20220524061145/https://www.baeldung.com/apache-commons-io) 库提供了`BOMInputStream` 类**。这个类是一个包装器，包含一个编码的`ByteOrderMark`作为它的第一个字节。让我们看看它是如何工作的:

```
@Test
public void whenInputFileHasBOM_thenUseBOMInputStream() throws IOException {
    String line;
    String actual = "";
    ByteOrderMark[] byteOrderMarks = new ByteOrderMark[] { 
      ByteOrderMark.UTF_8, ByteOrderMark.UTF_16BE, ByteOrderMark.UTF_16LE, ByteOrderMark.UTF_32BE, ByteOrderMark.UTF_32LE
    };
    InputStream inputStream = new BOMInputStream(ioStream, false, byteOrderMarks);
    Reader reader = new InputStreamReader(inputStream);
    BufferedReader br = new BufferedReader(reader);
    while ((line = br.readLine()) != null) {
        actual += line;
    }
    assertEquals(expected, actual);
}
```

代码类似于前面的例子，但是我们将`BOMInputStream` 作为参数传递给`InputStreamReader`。

### 3.3.使用**谷歌数据(GData)** 读取文件

另一方面，**处理 BOM 的另一个有用的库是 Google Data (GData)** 。这是一个较旧的库，但它有助于管理文件中的 BOM。它使用 XML 作为底层格式。让我们来看看它的实际应用:

```
@Test
public void whenInputFileHasBOM_thenUseGoogleGdata() throws IOException {
    char[] actual = new char[21];
    try (Reader r = new UnicodeReader(ioStream, null)) {
        r.read(actual);
    }
    assertEquals(expected, String.valueOf(actual));
}
```

最后，正如我们在前面的例子中观察到的，从文件中删除 BOM 是很重要的。如果我们在文件中没有正确处理它，当数据被读取时，会发生意想不到的结果。这就是为什么我们需要在我们的文件中意识到这个标记的存在。

## 4.结论

在本文中，我们讨论了几个关于 Java 中非法字符编译错误的主题。首先，我们了解了什么是 UTF 以及 BOM 是如何集成到其中的。其次，我们展示了一个使用文本编辑器创建的示例类——在本例中是 Windows 记事本。生成的类引发了非法字符的编译错误。最后，我们给出了一些关于如何读取 BOM 文件的代码示例。

像往常一样，本例使用的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524061145/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java)