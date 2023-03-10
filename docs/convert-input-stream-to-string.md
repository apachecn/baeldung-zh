# Java 输入流到字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-input-stream-to-string>

## 1。概述

在本教程中，我们将看看**如何将`InputStream`转换成字符串。**

我们将从使用普通 Java 开始，包括 Java8/9 解决方案，然后研究如何使用 [Guava](https://web.archive.org/web/20221013193922/https://github.com/google/guava) 和 [Apache Commons IO](https://web.archive.org/web/20221013193922/https://commons.apache.org/proper/commons-io/) 库。

本文是 Baeldung 网站上的[“Java—回到基础”系列文章](/web/20221013193922/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [Java InputStream 到字节数组和字节缓冲区](/web/20221013193922/https://www.baeldung.com/convert-input-stream-to-array-of-bytes)

How to convert an InputStream to a byte[] using plain Java, Guava or Commons IO.[Read more](/web/20221013193922/https://www.baeldung.com/convert-input-stream-to-array-of-bytes) →

## [Java–将输入流写入文件](/web/20221013193922/https://www.baeldung.com/convert-input-stream-to-a-file)

How to write an InputStream to a File - using Java, Guava and the Commons IO library.[Read more](/web/20221013193922/https://www.baeldung.com/convert-input-stream-to-a-file) →

## [Java–阅读器的输入流](/web/20221013193922/https://www.baeldung.com/java-convert-inputstream-to-reader)

How to convert an InputStream to a Reader using Java, Guava and the Apache Commons IO library.[Read more](/web/20221013193922/https://www.baeldung.com/java-convert-inputstream-to-reader) →

## 2。用 Java 转换-`StringBuilder`

让我们看一个简单的、使用普通 Java 的低级方法，一个`InputStream`和一个简单的`StringBuilder`:

```java
@Test
public void givenUsingJava5_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    StringBuilder textBuilder = new StringBuilder();
    try (Reader reader = new BufferedReader(new InputStreamReader
      (inputStream, Charset.forName(StandardCharsets.UTF_8.name())))) {
        int c = 0;
        while ((c = reader.read()) != -1) {
            textBuilder.append((char) c);
        }
    }
    assertEquals(textBuilder.toString(), originalString);
}
```

## 3.使用 Java 8 进行转换–`BufferedReader`

Java 8 为`BufferedReader` 带来了**新的 [`lines()`](https://web.archive.org/web/20221013193922/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/BufferedReader.html#lines()) 方法。让我们看看如何利用它将一个`InputStream`转换成一个`String:`**

```java
@Test
public void givenUsingJava8_whenConvertingAnInputStreamToAString_thenCorrect() {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = new BufferedReader(
      new InputStreamReader(inputStream, StandardCharsets.UTF_8))
        .lines()
        .collect(Collectors.joining("\n"));

    assertThat(text, equalTo(originalString));
}
```

值得一提的是`lines()`在引擎盖下使用了`readLine()`的方法。`readLine()`假定一行由换行符(" \n ")、回车符(" \r ")或紧跟换行符的回车符中的任何一个结束。换句话说，它支持所有常见的`End Of Line`风格:Unix、Windows，甚至旧的 Mac OS。

另一方面，**当我们使用`Collectors.joining()`时，我们需要明确地决定我们想要为创建的`String`** 使用哪种类型的 EOL。

我们也可以使用`Collectors.joining(System.lineSeparator())`，在这种情况下，输出取决于系统设置。

## 4.使用 Java 9 进行转换–`InputStream.readAllBytes()`

如果我们在 Java 9 或更高版本上，我们可以利用添加到`InputStream:`中的一个新的`readAllBytes`方法

```java
@Test
public void givenUsingJava9_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);

    assertThat(text, equalTo(originalString));
}
```

我们需要意识到，这个简单的代码是为了方便地将所有字节读入一个字节数组的简单情况而设计的。我们不应该用它来读取包含大量数据的输入流。

## 5。用 Java 和一个`Scanner`转换成

接下来，让我们看一个使用标准文本`Scanner` : 的普通 Java 例子

```java
@Test
public void givenUsingJava7_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = null;
    try (Scanner scanner = new Scanner(inputStream, StandardCharsets.UTF_8.name())) {
        text = scanner.useDelimiter("\\A").next();
    }

    assertThat(text, equalTo(originalString));
}
```

请注意，`InputStream`将随着`Scanner`的关闭而关闭。

澄清`useDelimiter(“\\A”)`做什么也是值得的。这里我们传递了' \A '，这是一个边界标记正则表达式，表示输入的开始。本质上，这意味着`next()`调用读取整个输入流。

这是 Java 7 示例而不是 Java 5 示例的唯一原因是使用了`try-with-resources`语句。如果我们把它变成一个标准的`try-finally`块，它可以很好地编译 Java 5。

## 6。使用`ByteArrayOutputStream`转换

最后，让我们看另一个普通的 Java 例子，这次使用的是`ByteArrayOutputStream`类:

```java
@Test
public void givenUsingPlainJava_whenConvertingAnInputStreamToString_thenCorrect()
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    ByteArrayOutputStream buffer = new ByteArrayOutputStream();
    int nRead;
    byte[] data = new byte[1024];
    while ((nRead = inputStream.read(data, 0, data.length)) != -1) {
        buffer.write(data, 0, nRead);
    }

    buffer.flush();
    byte[] byteArray = buffer.toByteArray();

    String text = new String(byteArray, StandardCharsets.UTF_8);
    assertThat(text, equalTo(originalString));
}
```

在本例中，**`InputStream`通过读写字节块转换为`ByteArrayOutputStream`。然后`OutputStream`被转换成一个字节数组，用来创建一个`String`。**

## 7。用`java.nio`转换成

另一种解决方法是将**中的内容`InputStream`复制到一个文件中，然后将其转换为`String:`**

```java
@Test
public void givenUsingTempFile_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    Path tempFile = 
      Files.createTempDirectory("").resolve(UUID.randomUUID().toString() + ".tmp");
    Files.copy(inputStream, tempFile, StandardCopyOption.REPLACE_EXISTING);
    String result = new String(Files.readAllBytes(tempFile));

    assertThat(result, equalTo(originalString));
}
```

这里我们使用`java.nio.file.Files`类创建一个临时文件，并将`InputStream`的内容复制到该文件中。然后使用同一个类通过`readAllBytes()`方法将文件内容转换为`String`。

## 8。用番石榴加工

让我们从一个利用`ByteSource`功能的番石榴示例**开始:**

```java
@Test
public void givenUsingGuava_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    ByteSource byteSource = new ByteSource() {
        @Override
        public InputStream openStream() throws IOException {
            return inputStream;
        }
    };

    String text = byteSource.asCharSource(Charsets.UTF_8).read();

    assertThat(text, equalTo(originalString));
}
```

我们来复习一下步骤:

*   **首先是**——我们将`InputStream`包装成`ByteSource,`，据我们所知，这是最简单的方法。
*   **然后是**——我们将`ByteSource`视为使用 UTF8 字符集的`CharSource`。
*   **最后是**–我们使用`CharSource`将它作为一个字符串读取。

一种更简单的转换方法是用番石榴树进行转换，但是需要显式地关闭流；幸运的是，我们可以简单地使用 try-with-resources 语法来解决这个问题:

```java
@Test
public void givenUsingGuavaAndJava7_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = null;
    try (Reader reader = new InputStreamReader(inputStream)) {
        text = CharStreams.toString(reader);
    }

    assertThat(text, equalTo(originalString));
}
```

## 9。使用 Apache Commons IO 进行转换

现在让我们看看如何使用 Commons IO 库来实现这一点。

这里一个重要的警告是，与番石榴相反，这两个例子都不会结束`InputStream:`

```java
@Test
public void givenUsingCommonsIo_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = IOUtils.toString(inputStream, StandardCharsets.UTF_8.name());
    assertThat(text, equalTo(originalString));
}
```

我们也可以使用一个`StringWriter`来进行转换:

```java
@Test
public void givenUsingCommonsIoWithCopy_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    StringWriter writer = new StringWriter();
    String encoding = StandardCharsets.UTF_8.name();
    IOUtils.copy(inputStream, writer, encoding);

    assertThat(writer.toString(), equalTo(originalString));
}
```

## 10。结论

在本文中，我们学习了如何将`InputStream`转换成字符串。我们从使用普通 Java 开始，然后探索如何使用[番石榴](https://web.archive.org/web/20221013193922/https://github.com/google/guava)和 [Apache Commons IO](https://web.archive.org/web/20221013193922/https://commons.apache.org/proper/commons-io/) 库。

GitHub 上的[提供了所有这些例子和代码片段的实现。](https://web.archive.org/web/20221013193922/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2 "Conversion of an Input Stream to a String")**