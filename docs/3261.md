# 在 Java 中检查目录是否为空

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-empty-directory>

## 1.概观

在这个快速教程中，我们将熟悉几种确定目录是否为空的方法。

## 2.使用`Files.newDirectoryStream`

**从 Java 7 开始，`[Files.newDirectoryStream](https://web.archive.org/web/20221126230413/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#newDirectoryStream(java.nio.file.Path)) `方法返回一个`[DirectoryStream](https://web.archive.org/web/20221126230413/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/DirectoryStream.html)<Path>` 来迭代目录**中的所有条目。所以我们可以使用这个 API 来检查给定的目录是否为空:

```
public boolean isEmpty(Path path) throws IOException {
    if (Files.isDirectory(path)) {
        try (DirectoryStream<Path> directory = Files.newDirectoryStream(path)) {
            return !directory.iterator().hasNext();
        }
    }

    return false;
}
```

对于非目录输入，我们将返回`false `，甚至不尝试加载目录条目:

```
Path aFile = Paths.get(getClass().getResource("/notDir.txt").toURI());
assertThat(isEmpty(aFile)).isFalse();
```

另一方面，如果输入是一个目录，我们将尝试打开一个指向该目录的`DirectoryStream `。**那么我们将认为目录为空当且仅当第一个`hasNext() `方法调用返回`false`** 。否则，它不是空的:

```
Path currentDir = new File("").toPath().toAbsolutePath();
assertThat(isEmpty(currentDir)).isFalse();
```

`DirectoryStream `是一个`Closeable `资源，所以我们将它包装在一个 [try-with-resources 块](/web/20221126230413/https://www.baeldung.com/java-try-with-resources)中。正如我们所料，`isEmpty `方法返回空目录的`true `:

```
Path path = Files.createTempDirectory("baeldung-empty");
assertThat(isEmpty(path)).isTrue();
```

这里我们使用`[Files.createTempDirectory](/web/20221126230413/https://www.baeldung.com/java-nio-2-file-api#creating-temporary-files) `来创建一个空的临时目录。

## 3.使用`Files.list`

**从 JDK 8 开始，`[Files.list](https://web.archive.org/web/20221126230413/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#list(java.nio.file.Path)) `方法在内部使用`Files.newDirectoryStream` API 来公开一个`Stream<Path>`T5。每个`Path `是给定父目录中的一个条目。因此，我们也可以将这个 API 用于相同的目的:**

```
public boolean isEmpty(Path path) throws IOException {
    if (Files.isDirectory(path)) {
        try (Stream<Path> entries = Files.list(path)) {
            return !entries.findFirst().isPresent();
        }
    }

    return false;
}
```

同样，我们使用`findFirst `方法只接触第一个条目。如果返回的`[Optional](/web/20221126230413/https://www.baeldung.com/java-optional) `是空的，那么目录也是空的。

[`Stream `由 I/O 资源](/web/20221126230413/https://www.baeldung.com/java-stream-close)支持，因此我们确保使用 try-with-resources 块适当地释放它。

## 4.低效的解决方案

**`Files.list `和`Files.newDirectoryStream `都将缓慢地迭代目录条目。因此，他们将非常有效地处理巨大的目录**。然而，像这样的解决方案在这种情况下并不好用:

```
public boolean isEmpty(Path path) {
    return path.toFile().listFiles().length == 0;
}
```

这将急切地加载目录中的所有条目，在处理大型目录时，这将是非常低效的。

## 5.结论

在这个简短的教程中，我们熟悉了一些检查目录是否为空的有效方法。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221126230413/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)