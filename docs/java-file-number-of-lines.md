# 使用 Java 查找文件中的行数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-number-of-lines>

## 1.概观

在本教程中，我们将学习**如何在标准 Java IO APIs、 [`Google Guav` a](/web/20221129014325/https://www.baeldung.com/category/guava/) 和`[Apache Commons IO](/web/20221129014325/https://www.baeldung.com/apache-commons-io)`库的帮助下使用 Java** 找到文件中的行数。

## 2.nio2`Files`的缩写形式

请注意，在本教程中，我们将使用以下示例值作为输入文件名和总行数:

```java
static final String INPUT_FILE_NAME = "src/main/resources/input.txt";
static final int NO_OF_LINES = 45; 
```

Java 7 对现有的 IO 库进行了许多改进，并打包在 [NIO2:](/web/20221129014325/https://www.baeldung.com/java-nio-2-file-api) 下

让我们从`Files`开始，看看我们如何使用它的 API 来计算行数:

```java
@Test
public void whenUsingNIOFiles_thenReturnTotalNumberOfLines() throws IOException {
    try (Stream<String> fileStream = Files.lines(Paths.get(INPUT_FILE_NAME))) {
        int noOfLines = (int) fileStream.count();
        assertEquals(NO_OF_LINES, noOfLines);
    }
}
```

或者通过简单地使用`Files#readAllLines`方法:

```java
@Test
public void whenUsingNIOFilesReadAllLines_thenReturnTotalNumberOfLines() throws IOException {
    List<String> fileStream = Files.readAllLines(Paths.get(INPUT_FILE_NAME));
    int noOfLines = fileStream.size();
    assertEquals(NO_OF_LINES, noOfLines);
}
```

## 3.九`FileChannel`

现在让我们检查一下`FileChannel,` 一个高性能的 Java NIO 替代品来读取行数:

```java
@Test
public void whenUsingNIOFileChannel_thenReturnTotalNumberOfLines() throws IOException {
    int noOfLines = 1;
    try (FileChannel channel = FileChannel.open(Paths.get(INPUT_FILE_NAME), StandardOpenOption.READ)) {
        ByteBuffer byteBuffer = channel.map(MapMode.READ_ONLY, 0, channel.size());
        while (byteBuffer.hasRemaining()) {
            byte currentByte = byteBuffer.get();
            if (currentByte == '\n')
                noOfLines++;
       }
    }
    assertEquals(NO_OF_LINES, noOfLines);
}
```

虽然`FileChannel`是在 JDK 4 中引入的，**上述解决方案只适用于 JDK 7 或更高版本的**。

## 4.谷歌番石榴`Files`

另一个第三方库是 Google Guava `Files` class。这个类也可以用来计算总行数，就像我们在`Files#readAllLines`中看到的那样。

让我们从在我们的`pom.xml`中添加[的`guava` 依赖关系](https://web.archive.org/web/20221129014325/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

然后我们可以使用`readLines `来获得文件行的`List`:

```java
@Test
public void whenUsingGoogleGuava_thenReturnTotalNumberOfLines() throws IOException {
    List<String> lineItems = Files.readLines(Paths.get(INPUT_FILE_NAME)
      .toFile(), Charset.defaultCharset());
    int noOfLines = lineItems.size();
    assertEquals(NO_OF_LINES, noOfLines);
}
```

## 5.Apache commons me〔t0〕

现在，让我们来看一下 [Apache Commons IO](/web/20221129014325/https://www.baeldung.com/apache-commons-io) `FileUtils` API，一个与 Guava 并行的解决方案。

要使用这个库，我们必须在`pom.xml`中包含[commons-io 依赖关系](https://web.archive.org/web/20221129014325/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22):

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

此时，我们可以使用 Apache Commons IO 的`FileUtils#lineIterator`，它为我们清理了一些文件处理:

```java
@Test
public void whenUsingApacheCommonsIO_thenReturnTotalNumberOfLines() throws IOException {
    int noOfLines = 0;
    LineIterator lineIterator = FileUtils.lineIterator(new File(INPUT_FILE_NAME));
    while (lineIterator.hasNext()) {
        lineIterator.nextLine();
        noOfLines++;
    }
    assertEquals(NO_OF_LINES, noOfLines);
}
```

正如我们所见，这比谷歌番石榴解决方案更冗长。

## 6.`BufferedReader`

那么，老派的方法呢？如果我们不在 JDK 7 上并且我们不能使用第三方库，我们有 [`BufferedReader`](/web/20221129014325/https://www.baeldung.com/java-buffered-reader) :

```java
@Test
public void whenUsingBufferedReader_thenReturnTotalNumberOfLines() throws IOException {
    int noOfLines = 0;
    try (BufferedReader reader = new BufferedReader(new FileReader(INPUT_FILE_NAME))) {
        while (reader.readLine() != null) {
            noOfLines++;
        }
    }
    assertEquals(NO_OF_LINES, noOfLines);
}
```

## 7.`LineNumberReader`

或者，我们可以使用`[BufferedReader](/web/20221129014325/https://www.baeldung.com/java-buffered-reader)`的直接子类`LineNumberReader,` ,这稍微简单一些:

```java
@Test
public void whenUsingLineNumberReader_thenReturnTotalNumberOfLines() throws IOException {
    try (LineNumberReader reader = new LineNumberReader(new FileReader(INPUT_FILE_NAME))) {
        reader.skip(Integer.MAX_VALUE);
        int noOfLines = reader.getLineNumber() + 1;
        assertEquals(NO_OF_LINES, noOfLines);
    }
}
```

这里我们是**调用`skip`方法**来到达文件的末尾，并且**我们正在给从 0 开始的行数**加 1。

## 8.`Scanner`

最后，如果我们已经在使用`[Scanner](/web/20221129014325/https://www.baeldung.com/java-scanner) `作为一个更大的解决方案的一部分，它也可以为我们解决问题:

```java
@Test
public void whenUsingScanner_thenReturnTotalNumberOfLines() throws IOException {
    try (Scanner scanner = new Scanner(new FileReader(INPUT_FILE_NAME))) {
        int noOfLines = 0;
        while (scanner.hasNextLine()) {
            scanner.nextLine();
            noOfLines++;
        }
        assertEquals(NO_OF_LINES, noOfLines);
    }
}
```

## 9.结论

在本教程中，我们探索了使用 Java 查找文件行数的不同方法。因为所有这些 API 的主要目的不是计算文件中的行数，所以建议根据我们的需要选择正确的解决方案。

和往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129014325/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)