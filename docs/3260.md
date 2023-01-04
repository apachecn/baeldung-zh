# 检查文件或目录是否存在于 Java 中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-directory-exists>

## 1.概观

在这个快速教程中，我们将熟悉检查文件或目录是否存在的不同方法。

首先，我们将从现代 NIO APIs 开始，然后将讨论遗留 IO 方法。

## 2.使用`java.nio.file.Files`

**要检查文件或目录是否存在，我们可以利用`[Files.exists(Path)](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#exists(java.nio.file.Path,java.nio.file.LinkOption...)) `方法**。从方法签名中可以清楚地看出，我们应该首先[获得一个指向目标文件或目录的`Path`](/web/20220907165410/https://www.baeldung.com/java-nio-2-path) 。然后我们可以将那个`[Path](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html) `传递给`Files.exists(Path) `方法:

```
Path path = Paths.get("does-not-exist.txt");
assertFalse(Files.exists(path));
```

由于文件不存在，它返回`false`。值得一提的是，如果`Files.exists(Path) `方法遇到了`IOException`，它也会返回`false`。

另一方面，当给定的文件存在时，它将按预期返回`true `:

```
Path tempFile = Files.createTempFile("baeldung", "exist-article");
assertTrue(Files.exists(tempFile));
```

这里我们创建一个临时文件，然后调用`Files.exists(Path) `方法。

这甚至适用于目录:

```
Path tempDirectory = Files.createTempDirectory("baeldung-exists");
assertTrue(Files.exists(tempDirectory));
```

**如果我们特别想知道一个文件或目录是否存在，我们也可以使用`[Files.isDirectory(Path)](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#isDirectory(java.nio.file.Path,java.nio.file.LinkOption...)) `或`[Files.isRegularFile(Path)](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#isRegularFile(java.nio.file.Path,java.nio.file.LinkOption...)) `方法:**

```
assertTrue(Files.isDirectory(tempDirectory));
assertFalse(Files.isDirectory(tempFile));
assertTrue(Files.isRegularFile(tempFile));
```

如果给定的`Path `不存在，还有一个`[notExists(Path)](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#notExists(java.nio.file.Path,java.nio.file.LinkOption...)) `方法返回`true `:

```
assertFalse(Files.notExists(tempDirectory));
```

**有时`Files.exists(Path) `会返回`false `，因为我们没有所需的文件权限**。在这种情况下，我们可以使用`[Files.isReadable(Path)](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#isReadable(java.nio.file.Path)) `方法来确保文件对于当前用户是可读的:

```
assertTrue(Files.isReadable(tempFile));
assertFalse(Files.isReadable(Paths.get("/root/.bashrc")));
```

### 2.1.符号链接

**默认情况下，`Files.exists(Path) `方法跟随符号链接**。如果文件`A `有一个到文件`B`的符号链接，那么当且仅当文件`B `已经存在时，`Files.exists(A)`方法返回`true `:

```
Path target = Files.createTempFile("baeldung", "target");
Path symbol = Paths.get("test-link-" + ThreadLocalRandom.current().nextInt());
Path symbolicLink = Files.createSymbolicLink(symbol, target);
assertTrue(Files.exists(symbolicLink));
```

现在如果我们删除链接的目标，`Files.exists(Path) `将返回`false`:

```
Files.deleteIfExists(target);
assertFalse(Files.exists(symbolicLink));
```

由于链接目标已经不存在了，跟随链接不会有任何结果，`Files.exists(Path)`会返回`false`。

**通过传递一个合适的`[LinkOption](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/LinkOption.html) `作为第二个参数:**，甚至可以不遵循符号链接

```
assertTrue(Files.exists(symbolicLink, LinkOption.NOFOLLOW_LINKS));
```

因为链接本身存在，`Files.exists(Path) `方法返回`true. `同样，我们可以使用`[Files.isSymbolicLink(Path)](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#isSymbolicLink(java.nio.file.Path)) `方法检查一个`Path `是否是一个符号链接:

```
assertTrue(Files.isSymbolicLink(symbolicLink));
assertFalse(Files.isSymbolicLink(target));
```

## 3.使用`java.io.File`

**如果我们使用 Java 7 或更高版本的 Java，强烈建议使用现代的 Java NIO APIs 来满足这些需求**。

然而，为了确定文件或目录是否存在于 Java 遗留 IO 世界中，我们可以在`[File](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html) `实例上调用`[exists()](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#exists(java.nio.file.Path,java.nio.file.LinkOption...)) `方法:

```
assertFalse(new File("invalid").exists());
```

如果文件或目录已经存在，它将返回`true`:

```
Path tempFilePath = Files.createTempFile("baeldung", "exist-io");
Path tempDirectoryPath = Files.createTempDirectory("baeldung-exists-io");

File tempFile = new File(tempFilePath.toString());
File tempDirectory = new File(tempDirectoryPath.toString());

assertTrue(tempFile.exists());
assertTrue(tempDirectory.exists());
```

如上所示，**`exists() `方法不关心它是文件还是目录。因此，只要它存在，它就会返回`true`。**

然而， [`isFile() `](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#isFile()) 方法返回`true `如果给定的路径是一个已存在的文件:

```
assertTrue(tempFile.isFile());
assertFalse(tempDirectory.isFile());
```

类似地，如果给定的路径是一个现有的目录，`[isDirectory()](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#isDirectory()) `方法返回`true `:

```
assertTrue(tempDirectory.isDirectory());
assertFalse(tempFile.isDirectory());
```

最后，如果文件可读，`[canRead()](https://web.archive.org/web/20220907165410/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#canRead()) `方法返回`true `:

```
assertTrue(tempFile.canRead());
assertFalse(new File("/root/.bashrc").canRead());
```

当它返回`false`时，该文件或者不存在，或者当前用户没有该文件的读取权限。

## 4.结论

在这个简短的教程中，我们看到了如何确保文件或目录在 Java 中存在。一路上，我们讨论了现代 NIO 和遗留 IO APIs。此外，我们看到了 NIO API 如何处理符号链接。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220907165410/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)