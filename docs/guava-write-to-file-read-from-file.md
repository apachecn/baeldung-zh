# 番石榴–写入文件，从文件中读取

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-write-to-file-read-from-file>

## 1。概述

在本教程中，我们将学习如何使用 **Guava IO** 写入文件，然后读取文件。我们将讨论如何写入文件。

## 2。用`Files`写

让我们从一个简单的例子开始，使用`Files`将`String`写入文件:

```java
@Test
public void whenWriteUsingFiles_thenWritten() throws IOException {
    String expectedValue = "Hello world";
    File file = new File("test.txt");
    Files.write(expectedValue, file, Charsets.UTF_8);
    String result = Files.toString(file, Charsets.UTF_8);
    assertEquals(expectedValue, result);
}
```

请注意，我们还可以使用`Files.append()` API 将**附加到现有文件**中。

## 3。使用`CharSink` 写入文件

接下来——让我们看看如何使用`CharSink`将`String`写入文件。在下面的例子中——我们使用`Files.asCharSink()`从一个文件中获得一个`CharSink`,然后用它来写:

```java
@Test
public void whenWriteUsingCharSink_thenWritten() throws IOException {
    String expectedValue = "Hello world";
    File file = new File("test.txt");
    CharSink sink = Files.asCharSink(file, Charsets.UTF_8);
    sink.write(expectedValue);

    String result = Files.toString(file, Charsets.UTF_8);
    assertEquals(expectedValue, result);
}
```

我们也可以使用`CharSink`向一个文件中写入多行。在下面的例子中，我们写了一个名称的`List`,并使用一个空格作为行分隔符:

```java
@Test
public void whenWriteMultipleLinesUsingCharSink_thenWritten() throws IOException {
    List<String> names = Lists.newArrayList("John", "Jane", "Adam", "Tom");
    File file = new File("test.txt");
    CharSink sink = Files.asCharSink(file, Charsets.UTF_8);
    sink.writeLines(names, " ");

    String result = Files.toString(file, Charsets.UTF_8);
    String expectedValue = Joiner.on(" ").join(names);
    assertEquals(expectedValue, result.trim());
}
```

## 4。使用`ByteSink` 写入文件

我们也可以使用`ByteSink`写入原始字节。在下面的例子中——我们使用`Files.asByteSink()`从一个文件中获得一个`ByteSink`,然后用它来写:

```java
@Test
public void whenWriteUsingByteSink_thenWritten() throws IOException {
    String expectedValue = "Hello world";
    File file = new File("test.txt");
    ByteSink sink = Files.asByteSink(file);
    sink.write(expectedValue.getBytes());

    String result = Files.toString(file, Charsets.UTF_8);
    assertEquals(expectedValue, result);
}
```

注意，我们可以通过使用简单的转换`byteSink.asCharSink()`在`ByteSink`和`CharSink`之间移动。

## 5。使用`Files` 从文件中读取

接下来，让我们讨论如何使用文件读取文件。

在下面的例子中，我们使用简单的`Files.toString():`来读取文件的所有内容

```java
@Test
public void whenReadUsingFiles_thenRead() throws IOException {
    String expectedValue = "Hello world";
    File file = new File("test.txt");
    String result = Files.toString(file, Charsets.UTF_8);

    assertEquals(expectedValue, result);
}
```

我们也可以将文件读入一个`List`行，如下例所示:

```java
@Test
public void whenReadMultipleLinesUsingFiles_thenRead() throws IOException {
    File file = new File("test.txt");
    List<String> result = Files.readLines(file, Charsets.UTF_8);

    assertThat(result, contains("John", "Jane", "Adam", "Tom"));
}
```

注意，我们可以使用`Files.readFirstLine()`只读取文件的第一行。

## 6。使用`CharSource` 从文件中读取

接下来，让我们看看如何使用 Charsource 读取文件。

在下面的例子中——我们使用`Files.asCharSource()`从一个文件中获取一个`CharSource`,然后使用`read()`用它来读取所有文件内容:

```java
@Test
public void whenReadUsingCharSource_thenRead() throws IOException {
    String expectedValue = "Hello world";
    File file = new File("test.txt");
    CharSource source = Files.asCharSource(file, Charsets.UTF_8);

    String result = source.read();
    assertEquals(expectedValue, result);
}
```

我们还可以连接两个 CharSources，并将它们用作一个`CharSource`。

在下面的例子中，我们读取两个文件，第一个包含“`Hello world`”，另一个包含“`Test`”:

```java
@Test
public void whenReadMultipleCharSources_thenRead() throws IOException {
    String expectedValue = "Hello worldTest";
    File file1 = new File("test1.txt");
    File file2 = new File("test2.txt");

    CharSource source1 = Files.asCharSource(file1, Charsets.UTF_8);
    CharSource source2 = Files.asCharSource(file2, Charsets.UTF_8);
    CharSource source = CharSource.concat(source1, source2);

    String result = source.read();
    assertEquals(expectedValue, result);
}
```

## 7。使用`CharStreams` 从文件中读取

现在——让我们看看如何使用`CharStreams`,通过中介`FileReader`将文件的内容读入`String`:

```java
@Test
public void whenReadUsingCharStream_thenRead() throws IOException {
    String expectedValue = "Hello world";
    FileReader reader = new FileReader("test.txt");
    String result = CharStreams.toString(reader);

    assertEquals(expectedValue, result);
    reader.close();
}
```

## 8。使用`ByteSource` 从文件中读取

我们可以将`ByteSource`用于原始字节格式的文件内容——如下例所示:

```java
@Test
public void whenReadUsingByteSource_thenRead() throws IOException {
    String expectedValue = "Hello world";
    File file = new File("test.txt");
    ByteSource source = Files.asByteSource(file);

    byte[] result = source.read();
    assertEquals(expectedValue, new String(result));
}
```

我们还可以使用`slice()`在特定偏移之后开始读取字节，如下例所示:

```java
@Test
public void whenReadAfterOffsetUsingByteSource_thenRead() throws IOException {
    String expectedValue = "lo world";
    File file = new File("test.txt");
    long offset = 3;
    long len = 1000;

    ByteSource source = Files.asByteSource(file).slice(offset, len);
    byte[] result = source.read();
    assertEquals(expectedValue, new String(result));
}
```

注意，我们可以使用`byteSource.asCharSource()`来获得这个`ByteSource`的`CharSource`视图。

## 9。使用`ByteStreams` 从文件中读取

接下来——让我们看看如何使用`ByteStreams`将文件内容读入原始字节数组；我们将使用中介`FileInputStream`来执行转换:

```java
@Test
public void whenReadUsingByteStream_thenRead() throws IOException {
    String expectedValue = "Hello world";
    FileInputStream reader = new FileInputStream("test.txt");
    byte[] result = ByteStreams.toByteArray(reader);
    reader.close();

    assertEquals(expectedValue, new String(result));
}
```

## 10。使用`Resources`阅读

最后——让我们看看如何读取类路径中存在的文件——使用`Resources`实用程序，如下例所示:

```java
@Test
public void whenReadUsingResources_thenRead() throws IOException {
    String expectedValue = "Hello world";
    URL url = Resources.getResource("test.txt");
    String result = Resources.toString(url, Charsets.UTF_8);

    assertEquals(expectedValue, result);
}
```

## 11。结论

在这个快速教程中，我们展示了使用 Guava IO 支持和实用程序来**读写文件的各种方法。**

所有这些示例和代码片段的实现可以在[我的番石榴 github 项目](https://web.archive.org/web/20221112110354/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-io "The Github Project with the impl of all examples using Guava Collections") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。