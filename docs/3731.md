# 用 Java 从文件中读取给定行号的一行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-line-at-number>

## 1.概观

在这篇简短的文章中，我们将看看在文件中给定的行号处读取一行的不同方法。

## 2.输入文件

让我们首先创建一个名为`inputLines.txt`的简单文件，我们将在所有示例中使用它:

```
Line 1
Line 2
Line 3
Line 4
Line 5
```

## 3.使用`BufferedReader`

让我们看看众所周知的 [`BufferedReader`](/web/20220707094748/https://www.baeldung.com/java-buffered-reader) 类及其不将整个文件存储到内存中的优势。

**我们可以逐行读取文件，并在需要时停止:**

```
@Test
public void givenFile_whenUsingBufferedReader_thenExtractedLineIsCorrect() {
    try (BufferedReader br = Files.newBufferedReader(Paths.get(FILE_PATH))) {
        for (int i = 0; i < 3; i++) {
            br.readLine();
        }

        String extractedLine = br.readLine();
        assertEquals("Line 4", extractedLine);
    }
}
```

## 4.使用`Scanner`

我们可以采取的另一种类似的方法是使用 [`Scanner`](/web/20220707094748/https://www.baeldung.com/java-scanner) :

```
@Test
public void givenFile_whenUsingScanner_thenExtractedLineIsCorrect() {
    try (Scanner scanner = new Scanner(new File(FILE_PATH))) {
        for (int i = 0; i < 3; i++) {
            scanner.nextLine();
        }

        String extractedLine = scanner.nextLine();
        assertEquals("Line 4", extractedLine);
    }
}
```

**虽然在小文件上，`BufferedReader`和`Scanner `之间的差别可能不明显，但在大文件上，`Scanner`会慢一些，因为它也[进行解析](/web/20220707094748/https://www.baeldung.com/bufferedreader-vs-console-vs-scanner-in-java#parsingstream)，并且缓冲区较小。**

## 5.使用文件 API

### 5.1.小文件

我们可以使用来自[文件 API](/web/20220707094748/https://www.baeldung.com/java-nio-2-file-api) 的`Files.readAllLines()` 轻松地将文件内容读入内存，并提取我们想要的行:

```
@Test
public void givenSmallFile_whenUsingFilesAPI_thenExtractedLineIsCorrect() {
    String extractedLine = Files.readAllLines(Paths.get(FILE_PATH)).get(4);

    assertEquals("Line 5", extractedLine);
}
```

### 5.2.大型文件

**如果我们需要处理大文件，我们应该使用`lines`方法，它返回一个`Stream`，这样我们就可以逐行读取文件:**

```
@Test
public void givenLargeFile_whenUsingFilesAPI_thenExtractedLineIsCorrect() {
    try (Stream lines = Files.lines(Paths.get(FILE_PATH))) {
        String extractedLine = lines.skip(4).findFirst().get();

        assertEquals("Line 5", extractedLine);
    }
}
```

## 6.使用 Apache Commons IO

另一种选择是使用 [**commons-io**](/web/20220707094748/https://www.baeldung.com/apache-commons-io) 包的`FileUtils `类，该类读取整个文件并返回一列`String`行:

```
@Test
public void givenFile_whenUsingFileUtils_thenExtractedLineIsCorrect() {
    ClassLoader classLoader = getClass().getClassLoader();
    File file = new File(classLoader.getResource("linesInput.txt").getFile());

    List<String> lines = FileUtils.readLines(file, "UTF-8");

    String extractedLine = lines.get(0);
    assertEquals("Line 1", extractedLine);
}
```

**我们也可以使用`IOUtils`类来达到同样的结果，只不过这一次，整个内容作为一个`String`返回，我们自己要做拆分:**

```
@Test
public void givenFile_whenUsingIOUtils_thenExtractedLineIsCorrect() {
    String fileContent = IOUtils.toString(new FileInputStream(FILE_PATH), StandardCharsets.UTF_8);

    String extractedLine = fileContent.split(System.lineSeparator())[0];
    assertEquals("Line 1", extractedLine);
}
```

## 7.结论

在这篇简短的文章中，我们讨论了从文件中给定的行号读取一行的最常见的方法。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220707094748/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)