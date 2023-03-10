# Java–将数据追加到文件中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-append-to-file>

## 1。简介

在这个快速教程中，我们将看到如何使用 Java 将数据添加到文件内容中——通过一些简单的方法。

让我们从如何使用核心 Java 的`FileWriter.`开始

## 2。使用`FileWriter`

这里有一个简单的测试——读取一个现有文件，附加一些文本，然后确保附加正确:

```java
@Test
public void whenAppendToFileUsingFileWriter_thenCorrect()
  throws IOException {

    FileWriter fw = new FileWriter(fileName, true);
    BufferedWriter bw = new BufferedWriter(fw);
    bw.write("Spain");
    bw.newLine();
    bw.close();

    assertThat(getStringFromInputStream(
      new FileInputStream(fileName)))
      .isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

请注意，如果我们想要将数据追加到现有文件中，那么`FileWriter's`构造函数接受一个`boolean`标记。

**如果我们将其设置为`false,`，那么现有内容将被替换。**

## 3。使用`FileOutputStream`

接下来，让我们看看如何使用`FileOutputStream`进行同样的操作:

```java
@Test
public void whenAppendToFileUsingFileOutputStream_thenCorrect()
 throws Exception {

    FileOutputStream fos = new FileOutputStream(fileName, true);
    fos.write("Spain\r\n".getBytes());
    fos.close();

    assertThat(StreamUtils.getStringFromInputStream(
      new FileInputStream(fileName)))
      .isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

类似地，`FileOutputStream`构造函数接受一个应该设置为 true 的布尔值，以标记我们想要将数据追加到现有文件中。

## 4。使用`java.nio.file`

接下来，我们还可以使用`java.nio.file`中的功能将内容添加到文件中，该功能是在 JDK 7 中引入的:

```java
@Test
public void whenAppendToFileUsingFiles_thenCorrect() 
 throws IOException {

    String contentToAppend = "Spain\r\n";
    Files.write(
      Paths.get(fileName), 
      contentToAppend.getBytes(), 
      StandardOpenOption.APPEND);

    assertThat(StreamUtils.getStringFromInputStream(
      new FileInputStream(fileName)))
      .isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

## 5。使用番石榴

要开始使用番石榴，我们需要将它的依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在，让我们看看如何开始使用 Guava 向现有文件添加内容:

```java
@Test
public void whenAppendToFileUsingFileWriter_thenCorrect()
 throws IOException {

    File file = new File(fileName);
    CharSink chs = Files.asCharSink(
      file, Charsets.UTF_8, FileWriteMode.APPEND);
    chs.write("Spain\r\n");

    assertThat(StreamUtils.getStringFromInputStream(
      new FileInputStream(fileName)))
      .isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

## 6。使用 Apache Commons IO `FileUtils`

最后，让我们看看如何使用 Apache Commons IO `FileUtils`将内容添加到现有文件中。

首先，让我们将 Apache Commons IO 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

现在，让我们看一个简单的例子，演示如何使用`FileUtils`将内容添加到现有文件中:

```java
@Test
public void whenAppendToFileUsingFiles_thenCorrect()
 throws IOException {
    File file = new File(fileName);
    FileUtils.writeStringToFile(
      file, "Spain\r\n", StandardCharsets.UTF_8, true);

    assertThat(StreamUtils.getStringFromInputStream(
      new FileInputStream(fileName)))
      .isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

## 7。结论

在本文中，我们看到了如何以多种方式添加内容。

本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220626072806/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2)