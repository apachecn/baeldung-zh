# 用 Java 列出目录中的文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-directory-files>

## 1.概观

在这个快速教程中，我们将学习不同的方法来列出目录中的文件。

## 2.列表

我们可以在指向一个目录的 [`java.io.File`](https://web.archive.org/web/20221007081810/https://docs.oracle.com/javase/8/docs/api/java/io/File.html) 对象上用 [`listFiles()`](https://web.archive.org/web/20221007081810/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#listFiles()) 的方法列出一个目录中的所有文件:

```java
public Set<String> listFilesUsingJavaIO(String dir) {
    return Stream.of(new File(dir).listFiles())
      .filter(file -> !file.isDirectory())
      .map(File::getName)
      .collect(Collectors.toSet());
}
```

正如我们所看到的，`listFiles()`返回一个包含目录内容的`File`对象的数组。

我们将从该数组创建一个流。然后我们将过滤掉所有不在子目录中的值。最后，我们将结果收集到一个集合中。

注意，我们选择了`Set`类型而不是`List`。事实上，不能保证`listFiles()`返回文件的顺序。

**在新实例化的`File`上使用`listFiles()`方法需要小心，**因为它可能是空的。当提供的目录无效时会发生这种情况。结果，它马上抛出一个`NullPointerException`:

```java
assertThrows(NullPointerException.class,
        () -> listFiles.listFilesUsingJavaIO(INVALID_DIRECTORY));
```

使用`listFiles()`的另一个缺点是它一次读取整个目录。因此，对于包含大量文件的文件夹来说，这可能是个问题。

所以我们来讨论一个替代方法。

## 3.`DirectoryStream`

Java 7 引入了一种替代`listFiles`的方法，叫做 [`DirectoryStream`](https://web.archive.org/web/20221007081810/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#newDirectoryStream(java.nio.file.Path)) 。**创建了一个目录流来很好地与 for-each 结构一起迭代目录的内容**。这意味着，我们不是一次读取所有内容，而是遍历目录的内容。

让我们用它来列出目录中的文件:

```java
public Set<String> listFilesUsingDirectoryStream(String dir) throws IOException {
    Set<String> fileSet = new HashSet<>();
    try (DirectoryStream<Path> stream = Files.newDirectoryStream(Paths.get(dir))) {
        for (Path path : stream) {
            if (!Files.isDirectory(path)) {
                fileSet.add(path.getFileName()
                    .toString());
            }
        }
    }
    return fileSet;
}
```

上面，我们让 Java 通过`[try-with-resources](/web/20221007081810/https://www.baeldung.com/java-try-with-resources)`构造来处理`DirectoryStream`资源的关闭。类似地，我们在过滤掉目录的文件夹中返回一组文件。

尽管名字令人困惑， **`DirectoryStream`并不是 [`Stream` API](/web/20221007081810/https://www.baeldung.com/java-8-streams)** 的一部分。

现在我们将看到如何使用 Stream API 列出文件。

## 4.Java 8 中的清单

Java 8 在`[java.nio.file.Files](https://web.archive.org/web/20221007081810/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html)`中引入了新的`[list()](https://web.archive.org/web/20221007081810/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#list(java.nio.file.Path))`方法。**`list`方法返回目录**中一个惰性填充的`Stream`条目。

### 4.1.使用`Files.list()`

让我们看一个简单的例子:

```java
public Set<String> listFilesUsingFilesList(String dir) throws IOException {
    try (Stream<Path> stream = Files.list(Paths.get(dir))) {
        return stream
          .filter(file -> !Files.isDirectory(file))
          .map(Path::getFileName)
          .map(Path::toString)
          .collect(Collectors.toSet());
    }
}
```

类似地，我们返回文件夹中包含的一组文件。尽管这可能看起来与`listFiles()`相似，但它在我们如何获得文件的`Path`上是不同的。

在这里，`list()`方法返回一个`Stream`对象，它缓慢地填充一个目录的条目。因此，我们可以更有效地处理大型文件夹。

同样，我们使用`the try-with-resources`构造来创建流，以确保在读取流之后关闭目录资源。

### 4.2.与`File.list()`的比较

我们不能混淆`Files`类提供的`list()`方法和 File 对象上的`list()`方法。后者返回一个目录中所有条目名称的`String`数组，包括文件和目录。

## 5.步行

除了列出文件之外，我们可能希望遍历目录到比它的直接文件条目更深的一层或多层。既然如此，我们可以用 [`walk()`](https://web.archive.org/web/20221007081810/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#walk(java.nio.file.Path,int,java.nio.file.FileVisitOption...)) :

```java
public Set<String> listFilesUsingFileWalk(String dir, int depth) throws IOException {
    try (Stream<Path> stream = Files.walk(Paths.get(dir), depth)) {
        return stream
          .filter(file -> !Files.isDirectory(file))
          .map(Path::getFileName)
          .map(Path::toString)
          .collect(Collectors.toSet());
    }
}
```

`walk()`方法在作为其参数提供的`depth`处遍历目录。在这里，我们遍历文件树，将所有文件的名称收集到一个`Set`中。

此外，我们可能希望在迭代每个文件时采取一些措施。在这种情况下，我们可以使用 [`walkFileTree()`](https://web.archive.org/web/20221007081810/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#walkFileTree(java.nio.file.Path,java.nio.file.FileVisitor)) 方法，提供一个[访问者](/web/20221007081810/https://www.baeldung.com/java-visitor-pattern)来描述我们想要采取的动作:

```java
public Set<String> listFilesUsingFileWalkAndVisitor(String dir) throws IOException {
    Set<String> fileList = new HashSet<>();
    Files.walkFileTree(Paths.get(dir), new SimpleFileVisitor<Path>() {
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
            if (!Files.isDirectory(file)) {
                fileList.add(file.getFileName().toString());
            }
            return FileVisitResult.CONTINUE;
        }
    });
    return fileList;
}
```

当我们想随时阅读、移动或删除文件时，这种方法非常方便。

如果我们试图传入一个有效的文件而不是一个目录，`walk()`和`walkFileTree()`方法不会抛出一个`NullPointerException`。事实上，`Stream`保证返回至少一个元素，即所提供的文件本身:

```java
Set<String> expectedFileSet = Collections.singleton("test.xml");
String filePathString = "src/test/resources/listFilesUnitTestFolder/test.xml";
assertEquals(expectedFileSet, listFiles.listFilesUsingFileWalk(filePathString, DEPTH));
```

## 6.结论

在这篇简短的文章中，我们探索了在一个目录中列出文件的不同方法。

首先，我们使用`listFiles()`获取文件夹的所有内容。然后我们使用`DirectoryStream`来延迟加载目录的内容。我们还使用了 Java 8 中引入的`list()`方法。

最后，我们演示了使用文件树的`walk()`和`walkFileTree()`方法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221007081810/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2)