# 如何用 Java 读取文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reading-file-in-java>

## 1。概述

在本教程中，我们将探索用 Java 从文件中读取数据的不同方法。

首先，我们将学习如何使用标准 Java 类从类路径、URL 或 JAR 文件加载文件。

第二，我们来看看如何用`BufferedReader`、`Scanner`、`StreamTokenizer`、`DataInputStream`、`SequenceInputStream,`、`FileChannel`来读内容。我们还将讨论如何阅读 UTF-8 编码的文件。

最后，我们将探索在 Java 7 和 Java 8 中加载和读取文件的新技术。

本文是 Baeldung 上的[“Java—回到基础”系列](/web/20220929055251/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [Java–创建一个文件](/web/20220929055251/https://www.baeldung.com/java-how-to-create-a-file)

How to create a File in Java using JDK 6, JDK 7 with NIO or Commons IO.[Read more](/web/20220929055251/https://www.baeldung.com/java-how-to-create-a-file) →

## [Java–写入文件](/web/20220929055251/https://www.baeldung.com/java-write-to-file)

The many ways to write data to File using Java.[Read more](/web/20220929055251/https://www.baeldung.com/java-write-to-file) →

## 2。设置

### 2.1。输入文件

在本文的大多数例子中，我们将读取一个文件名为`fileTest.txt`的文本文件，其中包含一行:

```
Hello, world!
```

举几个例子，我们将使用不同的文件；在这些情况下，我们将明确提到文件及其内容。

### 2.2。助手方法

我们将使用一组只包含核心 Java 类的测试示例，在测试中，我们将使用带有 [Hamcrest](/web/20220929055251/https://www.baeldung.com/hamcrest-collections-arrays) 匹配器的断言。

测试将共享一个公共的*readfrompinputstream*方法，该方法将一个*InputStream*转换为 *字符串* 以便于断言结果:

```
private String readFromInputStream(InputStream inputStream)
  throws IOException {
    StringBuilder resultStringBuilder = new StringBuilder();
    try (BufferedReader br
      = new BufferedReader(new InputStreamReader(inputStream))) {
        String line;
        while ((line = br.readLine()) != null) {
            resultStringBuilder.append(line).append("\n");
        }
    }
  return resultStringBuilder.toString();
}
```

请注意，还有其他方法可以达到同样的效果。我们可以参考 [这篇](/web/20220929055251/https://www.baeldung.com/convert-input-stream-to-a-file) 来获得一些替代方案。

## 3。从类路径中读取文件

### 3.1。使用标准 Java

本节解释了如何读取类路径中的文件。我们先来读一下“*filetest . txt*”下可用*src/main/resources*:

```
@Test
public void givenFileNameAsAbsolutePath_whenUsingClasspath_thenFileData() {
    String expectedData = "Hello, world!";

    Class clazz = FileOperationsTest.class;
    InputStream inputStream = clazz.getResourceAsStream("/fileTest.txt");
    String data = readFromInputStream(inputStream);

    Assert.assertThat(data, containsString(expectedData));
}
```

**在上面的代码片段中，我们使用当前类通过 *getResourceAsStream* 方法加载了一个文件，并传递了要加载的文件的绝对路径。**

同样的方法也适用于 *类加载器* 实例:

```
ClassLoader classLoader = getClass().getClassLoader();
InputStream inputStream = classLoader.getResourceAsStream("fileTest.txt");
String data = readFromInputStream(inputStream);
```

我们使用 *getClass()获得当前类的 *类加载器* 。getClassLoader()* 。

主要区别在于，当在*class loader*实例上使用*getResourceAsStream*时，路径被视为从类路径的根开始的绝对路径。

当用于对抗一个 *类* 实例 *，* 时，路径可以是相对于包的路径，也可以是绝对路径，这由前导斜杠来暗示。

**当然，注意在实践中，开放的流应该总是关闭的**，比如我们例子中的`InputStream`:

```
InputStream inputStream = null;
try {
    File file = new File(classLoader.getResource("fileTest.txt").getFile());
    inputStream = new FileInputStream(file);

    //...
}     
finally {
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.2。使用`commons-io`库

**另一个常见的选项是使用 [`commons-io`](/web/20220929055251/https://www.baeldung.com/apache-commons-io) 包的 *FileUtils* 类:**

```
@Test
public void givenFileName_whenUsingFileUtils_thenFileData() {
    String expectedData = "Hello, world!";

    ClassLoader classLoader = getClass().getClassLoader();
    File file = new File(classLoader.getResource("fileTest.txt").getFile());
    String data = FileUtils.readFileToString(file, "UTF-8");

    assertEquals(expectedData, data.trim());
}
```

这里我们将`File`对象传递给`FileUtils`类的方法`readFileToString()`。这个实用程序类设法加载内容，而不需要编写任何样板代码来创建一个`InputStream` 实例和读取数据。

**同一个库还提供了`IOUtils` 类:**

```
@Test
public void givenFileName_whenUsingIOUtils_thenFileData() {
    String expectedData = "Hello, world!";

    FileInputStream fis = new FileInputStream("src/test/resources/fileTest.txt");
    String data = IOUtils.toString(fis, "UTF-8");

    assertEquals(expectedData, data.trim());
}
```

这里我们将`FileInputStream`对象传递给`IOUtils`类的方法`toString()`。为了创建一个`InputStream` 实例并读取数据，这个实用程序类的行为与前一个相同。

## 4。`BufferedReader`读书用

现在让我们来关注解析文件内容的不同方法。

**我们将从使用`BufferedReader:`** 读取文件的简单方法开始

```
@Test
public void whenReadWithBufferedReader_thenCorrect()
  throws IOException {
     String expected_value = "Hello, world!";
     String file ="src/test/resources/fileTest.txt";

     BufferedReader reader = new BufferedReader(new FileReader(file));
     String currentLine = reader.readLine();
     reader.close();

    assertEquals(expected_value, currentLine);
}
```

注意，当到达文件末尾时，`readLine()`将返回`null`。

## 5。使用 Java NIO 从文件中读取

在 JDK7 中，NIO 包进行了重大更新。

**让我们看一个使用 *Files* 类和 *readAllLines* 方法**的例子。 *readAllLines* 方法接受一个*路径。*

*路径*级可以被认为是`java.io.File`级的升级版，增加了一些操作。

### 5.1。读取小文件

下面的代码展示了如何使用新的`Files`类读取一个小文件:

```
@Test
public void whenReadSmallFileJava7_thenCorrect()
  throws IOException {
    String expected_value = "Hello, world!";

    Path path = Paths.get("src/test/resources/fileTest.txt");

    String read = Files.readAllLines(path).get(0);
    assertEquals(expected_value, read);
}
```

注意，如果我们需要二进制数据，我们也可以使用`readAllBytes()`方法。

### 5.2。读取大文件

如果我们想用`Files`类读取一个大文件，我们可以使用`BufferedReader.`

下面的代码使用新的`Files`类和`BufferedReader`读取文件:

```
@Test
public void whenReadLargeFileJava7_thenCorrect()
  throws IOException {
    String expected_value = "Hello, world!";

    Path path = Paths.get("src/test/resources/fileTest.txt");

    BufferedReader reader = Files.newBufferedReader(path);
    String line = reader.readLine();
    assertEquals(expected_value, line);
}
```

### 5.3。使用`Files.lines()` 读取文件

**JDK8 在*文件*类中提供了 *lines()* 方法。它返回一个字符串元素的`Stream`。**

让我们来看一个例子，如何使用 UTF-8 字符集将数据读入字节并解码。

以下代码使用新的`Files.lines()`读取文件:

```
@Test
public void givenFilePath_whenUsingFilesLines_thenFileData() {
    String expectedData = "Hello, world!";

    Path path = Paths.get(getClass().getClassLoader()
      .getResource("fileTest.txt").toURI());

    Stream<String> lines = Files.lines(path);
    String data = lines.collect(Collectors.joining("\n"));
    lines.close();

    Assert.assertEquals(expectedData, data.trim());
}
```

像文件操作一样使用带有 IO 通道的流，我们需要使用`close()`方法显式地关闭流。

正如我们所见，`Files` API 提供了另一种将文件内容读入`String.`的简单方法

在接下来的几节中，我们将看看其他不太常用的读取文件的方法，这些方法在某些情况下可能是合适的。

## 6。`Scanner`读书用

接下来让我们使用一个`Scanner`来读取文件。这里我们将使用空白作为分隔符:

```
@Test
public void whenReadWithScanner_thenCorrect()
  throws IOException {
    String file = "src/test/resources/fileTest.txt";
    Scanner scanner = new Scanner(new File(file));
    scanner.useDelimiter(" ");

    assertTrue(scanner.hasNext());
    assertEquals("Hello,", scanner.next());
    assertEquals("world!", scanner.next());

    scanner.close();
}
```

注意，默认的分隔符是空白，但是多个分隔符可以和一个`Scanner`一起使用。

**当从控制台读取内容时，或者当内容包含原始值**并带有已知的分隔符(例如:由空格分隔的整数列表)时， [Scanner](/web/20220929055251/https://www.baeldung.com/java-scanner) 类非常有用。

## 7。`StreamTokenizer`读书用

现在让我们使用 [`StreamTokenizer`](/web/20220929055251/https://www.baeldung.com/java-streamtokenizer) 将文本文件读入令牌。

记号赋予器首先计算出下一个记号是什么，字符串还是数字。我们通过查看`tokenizer.ttype `字段来做到这一点。

然后我们将根据这个类型读取实际的令牌:

*   `tokenizer.nval`–如果类型是数字
*   `tokenizer.sval`–如果类型是字符串

在本例中，我们将使用一个不同的输入文件，它只包含:

```
Hello 1
```

以下代码从文件中读取字符串和数字:

```
@Test
public void whenReadWithStreamTokenizer_thenCorrectTokens()
  throws IOException {
    String file = "src/test/resources/fileTestTokenizer.txt";
   FileReader reader = new FileReader(file);
    StreamTokenizer tokenizer = new StreamTokenizer(reader);

    // token 1
    tokenizer.nextToken();
    assertEquals(StreamTokenizer.TT_WORD, tokenizer.ttype);
    assertEquals("Hello", tokenizer.sval);

    // token 2    
    tokenizer.nextToken();
    assertEquals(StreamTokenizer.TT_NUMBER, tokenizer.ttype);
    assertEquals(1, tokenizer.nval, 0.0000001);

    // token 3
    tokenizer.nextToken();
    assertEquals(StreamTokenizer.TT_EOF, tokenizer.ttype);
    reader.close();
}
```

请注意文件结束标记是如何在末尾使用的。

这种方法对于将输入流解析成令牌很有用。

## 8。`DataInputStream`读书用

**我们可以使用`DataInputStream`从文件中读取二进制或原始数据类型。**

以下测试使用`DataInputStream`读取文件:

```
@Test
public void whenReadWithDataInputStream_thenCorrect() throws IOException {
    String expectedValue = "Hello, world!";
    String file ="src/test/resources/fileTest.txt";
    String result = null;

    DataInputStream reader = new DataInputStream(new FileInputStream(file));
    int nBytesToRead = reader.available();
    if(nBytesToRead > 0) {
        byte[] bytes = new byte[nBytesToRead];
        reader.read(bytes);
        result = new String(bytes);
    }

    assertEquals(expectedValue, result);
}
```

## 9。`FileChannel`读书用

**如果我们正在读取一个大文件，`FileChannel`可以比标准 IO 更快。**

以下代码使用`FileChannel`和`RandomAccessFile`从文件中读取数据字节:

```
@Test
public void whenReadWithFileChannel_thenCorrect()
  throws IOException {
    String expected_value = "Hello, world!";
    String file = "src/test/resources/fileTest.txt";
    RandomAccessFile reader = new RandomAccessFile(file, "r");
    FileChannel channel = reader.getChannel();

    int bufferSize = 1024;
    if (bufferSize > channel.size()) {
        bufferSize = (int) channel.size();
    }
    ByteBuffer buff = ByteBuffer.allocate(bufferSize);
    channel.read(buff);
    buff.flip();

    assertEquals(expected_value, new String(buff.array()));
    channel.close();
    reader.close();
}
```

## 10。读取 UTF-8 编码文件

现在让我们看看如何使用`BufferedReader.` 读取 UTF-8 编码的文件。在本例中，我们将读取一个包含中文字符的文件:

```
@Test
public void whenReadUTFEncodedFile_thenCorrect()
  throws IOException {
    String expected_value = "青空";
    String file = "src/test/resources/fileTestUtf8.txt";
    BufferedReader reader = new BufferedReader
      (new InputStreamReader(new FileInputStream(file), "UTF-8"));
    String currentLine = reader.readLine();
    reader.close();

    assertEquals(expected_value, currentLine);
}
```

## 11。正在从 URL 读取内容

要从 URL 读取内容，我们将在示例中使用“`/`”URL:

```
@Test
public void givenURLName_whenUsingURL_thenFileData() {
    String expectedData = "Baeldung";

    URL urlObject = new URL("/");
    URLConnection urlConnection = urlObject.openConnection();
    InputStream inputStream = urlConnection.getInputStream();
    String data = readFromInputStream(inputStream);

    Assert.assertThat(data, containsString(expectedData));
}
```

还有其他连接到 URL 的方法。这里我们使用了标准 SDK 中可用的 *URL* 和*URL connection*类。

## 12。从 JAR 中读取文件

要读取位于 JAR 文件中的文件，我们需要一个里面有文件的 JAR。对于我们的示例，我们将从"`hamcrest-library-1.3.jar`"文件中读取"`LICENSE.txt`:

```
@Test
public void givenFileName_whenUsingJarFile_thenFileData() {
    String expectedData = "BSD License";

    Class clazz = Matchers.class;
    InputStream inputStream = clazz.getResourceAsStream("/LICENSE.txt");
    String data = readFromInputStream(inputStream);

    Assert.assertThat(data, containsString(expectedData));
}
```

这里我们想要加载驻留在 Hamcrest 库中的`LICENSE.txt`，所以我们将使用帮助获取资源的`Matcher's`类。也可以使用类加载器加载相同的文件。

## 13。结论

正如我们所看到的，使用普通 Java 加载文件和从中读取数据有很多可能性。

我们可以从不同的位置加载文件，比如类路径、URL 或 jar 文件。

然后，我们可以使用`BufferedReader`逐行读取，`Scanner`使用不同的分隔符读取，`StreamTokenizer`将文件读取为令牌，`DataInputStream`读取二进制数据和原始数据类型，`SequenceInput Stream`将多个文件链接为一个流，`FileChannel`从大文件读取速度更快，等等。

我们可以在下面的[GitHub repo](https://web.archive.org/web/20220929055251/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)中找到这篇文章的源代码。