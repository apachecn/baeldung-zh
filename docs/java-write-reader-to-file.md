# Java——编写一个文件阅读器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-write-reader-to-file>

在这个快速教程中，我们将使用普通 Java，然后是 Guava，最后是 Apache Commons IO 库，将`Reader`的内容写入文件。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用 Java

让我们从简单的 Java 解决方案开始:

```java
@Test
public void givenUsingPlainJava_whenWritingReaderContentsToFile_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("Some text");

    int intValueOfChar;
    StringBuilder buffer = new StringBuilder();
    while ((intValueOfChar = initialReader.read()) != -1) {
        buffer.append((char) intValueOfChar);
    }
    initialReader.close();

    File targetFile = new File("src/test/resources/targetFile.txt");
    targetFile.createNewFile();

    Writer targetFileWriter = new FileWriter(targetFile);
    targetFileWriter.write(buffer.toString());
    targetFileWriter.close();
}
```

首先——我们将阅读器的内容读入一个字符串；然后我们简单地将字符串写入文件。

## 2。有番石榴

Guava 解决方案更简单——我们现在有 API 来处理将阅读器写入文件:

```java
@Test
public void givenUsingGuava_whenWritingReaderContentsToFile_thenCorrect() 
  throws IOException {
    Reader initialReader = new StringReader("Some text");

    File targetFile = new File("src/test/resources/targetFile.txt");
    com.google.common.io.Files.touch(targetFile);
    CharSink charSink = com.google.common.io.Files.
      asCharSink(targetFile, Charset.defaultCharset(), FileWriteMode.APPEND);
    charSink.writeFrom(initialReader);

    initialReader.close();
}
```

## 3。使用 Apache Commons IO

最后，Commons IO 解决方案——也使用更高级别的 API 从`Reader`读取数据并将数据写入文件:

```java
@Test
public void givenUsingCommonsIO_whenWritingReaderContentsToFile_thenCorrect() 
  throws IOException {
    Reader initialReader = new CharSequenceReader("CharSequenceReader extends Reader");

    File targetFile = new File("src/test/resources/targetFile.txt");
    FileUtils.touch(targetFile);
    byte[] buffer = IOUtils.toByteArray(initialReader);
    FileUtils.writeByteArrayToFile(targetFile, buffer);

    initialReader.close();
}
```

现在我们有了 3 个简单的解决方案来将阅读器的内容写到文件中。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20211023204433/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)