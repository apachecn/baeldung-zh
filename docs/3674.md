# Java–创建一个文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-how-to-create-a-file>

## 1。概述

在这个快速教程中，我们将学习如何用 Java 创建一个新文件——首先使用 NIO 的`Files`和`Path`类，然后是 Java 的`File`和`FileOutputStream`类，[谷歌番石榴](https://web.archive.org/web/20220817183523/https://github.com/google/guava)，最后是 [Apache Commons IO](https://web.archive.org/web/20220817183523/https://commons.apache.org/proper/commons-io/) 库。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 2.设置

在示例中，我们将为文件名定义一个常数:

```
private final String FILE_NAME = "src/test/resources/fileToCreate.txt";
```

我们还将添加一个清理步骤，以确保该文件在每次测试之前不存在，并在每次测试运行之后删除它:

```
@AfterEach
@BeforeEach
public void cleanUpFiles() {
    File targetFile = new File(FILE_NAME);
    targetFile.delete();
}
```

## 3。使用 NIO `Files.createFile()`

让我们从使用 Java NIO 包中的`Files.createFile()`方法的**开始:**

```
@Test
public void givenUsingNio_whenCreatingFile_thenCorrect() throws IOException {
    Path newFilePath = Paths.get(FILE_NAME);
    Files.createFile(newFilePath);
}
```

如你所见，代码仍然非常简单；我们现在使用新的`Path`接口，而不是旧的`File`。

这里需要注意的一点是，新的 API 很好地利用了异常。如果文件已经存在，我们不再需要检查返回代码。相反，我们将得到一个`FileAlreadyExistsException`:

```
java.nio.file.FileAlreadyExistsException: <code class="language-java">src/test/resources/fileToCreate.txt at sun.n.f.WindowsException.translateToIOException(WindowsException.java:81)
```

## 4。 **利用`File.createNewFile()`**

现在让我们看看如何使用`java.io.File`类来做同样的

```
@Test
public void givenUsingFile_whenCreatingFile_thenCorrect() throws IOException {
    File newFile = new File(FILE_NAME);
    boolean success = newFile.createNewFile();
    assertTrue(success);
}
```

请注意，该文件必须不存在，此操作才能成功。如果文件确实存在，那么`createNewFile()`操作将返回 false。

## 5.使用`FileOutputStream`

另一种创建新文件的方法是使用`java.io.FileOutputStream`**:**

```
@Test
public void givenUsingFileOutputStream_whenCreatingFile_thenCorrect() throws IOException {
    try(FileOutputStream fileOutputStream = new FileOutputStream(FILE_NAME)){
    }
}
```

在这种情况下，当我们实例化`FileOutputStream`对象时，会创建一个新文件。如果具有给定名称**的文件已经存在，它将被覆盖**。然而，如果现有文件是一个目录，或者由于某种原因无法创建新文件，那么我们将得到一个`FileNotFoundException`。

此外，注意我们使用了一个`try-with-resources`语句——以确保流被正确关闭。

## 6。使用番石榴

创建新文件的 Guava 解决方案也是一个快速的一行程序:

```
@Test
public void givenUsingGuava_whenCreatingFile_thenCorrect() throws IOException {
    com.google.common.io.Files.touch(new File(FILE_NAME));
}
```

## 7。使用 Apache Commons IO

**Apache Commons 库提供了`FileUtils.touch()`方法，该方法实现了与 Linux 中的`touch`实用程序相同的行为。**

因此，它会在文件系统中创建一个新的空文件，甚至是一个文件及其完整路径:

```
@Test
public void givenUsingCommonsIo_whenCreatingFile_thenCorrect() throws IOException {
    FileUtils.touch(new File(FILE_NAME));
}
```

请注意，这与前面的示例略有不同:如果文件已经存在，操作不会失败，它只是不做任何事情。

现在我们有了 4 种在 Java 中创建新文件的快速方法。

## 8。结论

在本文中，我们研究了用 Java 创建文件的不同解决方案。我们使用了属于 JDK 和外部库的类。

GitHub 上的[提供了示例代码。](https://web.archive.org/web/20220817183523/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)**