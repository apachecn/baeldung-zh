# 春天的溪流简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-stream-utils>

## 1。概述

在本文中，我们将看看`StreamUtils`类以及如何使用它。

简单地说，`StreamUtils`是一个 Spring 的类，包含一些处理流的实用方法——`InputStream`和`OutputStream`,它们驻留在包`java.io` 中，与 Java 8 的流 API 无关。

## 2。Maven 依赖关系

`StreamUtils`类在 spring-core 模块中可用，所以让我们将它添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

您可以在 [Maven Central Repository](https://web.archive.org/web/20221126224836/https://mvnrepository.com/artifact/org.springframework/spring-core) 找到该库的最新版本。

## 3。复制流

`StreamUtils`类包含几个名为`copy` `()`的重载方法以及其他一些变体:

*   `copyRange()`
*   `copyToByteArray()`
*   `copyString()`

我们可以在不使用任何库的情况下复制流。然而，代码将会变得很麻烦，更难阅读和理解。

注意，为了简单起见，我们省略了流的结束。

让我们看看如何将一个`InputStream`的内容复制到一个给定的`OutputStream`:

```
@Test
public void whenCopyInputStreamToOutputStream_thenCorrect() throws IOException {
    String inputFileName = "src/test/resources/input.txt";
    String outputFileName = "src/test/resources/output.txt";
    File outputFile = new File(outputFileName);
    InputStream in = new FileInputStream(inputFileName);
    OutputStream out = new FileOutputStream(outputFile);

    StreamUtils.copy(in, out);

    assertTrue(outputFile.exists());
    String inputFileContent = getStringFromInputStream(new FileInputStream(inputFileName));
    String outputFileContent = getStringFromInputStream(new FileInputStream(outputFileName));
    assertEquals(inputFileContent, outputFileContent);
}
```

创建的文件包含了*输入流*的内容。

注意，`getStringFromInputStream()`是一个接受`InputStream`并将其内容作为`String`返回的方法。完整版本的代码中提供了该方法的实现。

我们不必复制`InputStream`的全部内容，我们可以使用`copyRange()`方法将一部分内容复制到给定的`OutputStream`:

```
@Test
public void whenCopyRangeOfInputStreamToOutputStream_thenCorrect() throws IOException {
    String inputFileName = "src/test/resources/input.txt";
    String outputFileName = "src/test/resources/output.txt";
    File outputFile = new File(outputFileName);
    InputStream in = new FileInputStream(inputFileName);
    OutputStream out = new FileOutputStream(outputFileName);

    StreamUtils.copyRange(in, out, 1, 10);

    assertTrue(outputFile.exists());
    String inputFileContent = getStringFromInputStream(new FileInputStream(inputFileName));
    String outputFileContent = getStringFromInputStream(new FileInputStream(outputFileName));

    assertEquals(inputFileContent.substring(1, 11), outputFileContent);
}
```

正如我们在这里看到的，`copyRange()`有四个参数，`InputStream`、`OutputStream`、开始复制的位置和结束复制的位置。但是如果指定的范围超过了`InputStream`的长度呢？方法`copyRange()`然后复制到流的末尾。

让我们看看如何将一个`String`的内容复制到一个给定的`OutputStream`:

```
@Test
public void whenCopyStringToOutputStream_thenCorrect() throws IOException {
    String string = "Should be copied to OutputStream.";
    String outputFileName = "src/test/resources/output.txt";
    File outputFile = new File(outputFileName);
    OutputStream out = new FileOutputStream("src/test/resources/output.txt");

    StreamUtils.copy(string, StandardCharsets.UTF_8, out);

    assertTrue(outputFile.exists());

    String outputFileContent = getStringFromInputStream(new FileInputStream(outputFileName));

    assertEquals(outputFileContent, string);
}
```

方法`copy()`有三个参数——要复制的`String`、我们要用来写入文件的`Charset`,以及我们要将`String`的内容复制到的`OutputStream`。

下面是我们如何将一个给定的`InputStream`的内容复制到一个新的`String`:

```
@Test
public void whenCopyInputStreamToString_thenCorrect() throws IOException {
    String inputFileName = "src/test/resources/input.txt";
    InputStream is = new FileInputStream(inputFileName);
    String content = StreamUtils.copyToString(is, StandardCharsets.UTF_8);

    String inputFileContent = getStringFromInputStream(new FileInputStream(inputFileName));
    assertEquals(inputFileContent, content);
}
```

我们也可以将给定字节数组的内容复制到一个`OutputStream`:

```
public void whenCopyByteArrayToOutputStream_thenCorrect() throws IOException {
    String outputFileName = "src/test/resources/output.txt";
    String string = "Should be copied to OutputStream.";
    byte[] byteArray = string.getBytes();
    OutputStream out = new FileOutputStream("src/test/resources/output.txt");

    StreamUtils.copy(byteArray, out);

    String outputFileContent = getStringFromInputStream(new FileInputStream(outputFileName));

    assertEquals(outputFileContent, string);
}
```

或者，我们可以将给定的`InputStream`的内容复制到一个新的字节数组中:

```
public void whenCopyInputStreamToByteArray_thenCorrect() throws IOException {
    String inputFileName = "src/test/resources/input.txt";
    InputStream is = new FileInputStream(inputFileName);
    byte[] out = StreamUtils.copyToByteArray(is);

    String content = new String(out);
    String inputFileContent = getStringFromInputStream(new FileInputStream(inputFileName));

    assertEquals(inputFileContent, content);
}
```

## 4。其他功能

可以将一个`InputStream`作为参数传递给方法`drain()`,以移除流中所有剩余的数据:

```
StreamUtils.drain(in);
```

我们也可以使用方法`emptyInput()`来得到一个有效的空值`InputStream`:

```
public InputStream getInputStream() {
    return StreamUtils.emptyInput();
}
```

有两个名为`nonClosing()`的重载方法。可以将一个`InputStream`或一个`OutputStream`作为参数传递给这些方法，以获得一个忽略对`close()`方法调用的`InputStream`或`OutputStream`的变体:

```
public InputStream getNonClosingInputStream() throws IOException {
    InputStream in = new FileInputStream("src/test/resources/input.txt");
    return StreamUtils.nonClosing(in);
}
```

## 5。结论

在这个快速教程中，我们已经看到了`StreamUtils`是什么。我们还介绍了`StreamUtils`类的所有方法，并且已经看到了如何使用它们。

本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221126224836/https://github.com/eugenp/tutorials/tree/master/spring-core)