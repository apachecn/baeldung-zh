# 从包含绝对文件路径的字符串中获取文件名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filename-full-path>

## 1.概观

当我们在 Java 中处理文件时，我们经常需要从给定的绝对路径中提取文件名。

在本教程中，我们将探索如何提取文件名。

## 2.问题简介

这个问题很简单。假设给我们一个绝对文件路径字符串。我们想从中提取文件名。有几个例子可以很快解释这个问题:

```java
String PATH_LINUX = "/root/with space/subDir/myFile.linux";
String EXPECTED_FILENAME_LINUX = "myFile.linux";

String PATH_WIN = "C:\\root\\with space\\subDir\\myFile.win";
String EXPECTED_FILENAME_WIN = "myFile.win";
```

正如我们所见，**不同的文件系统可能有不同的文件分隔符**。因此，在本教程中，我们将提出一些独立于平台的解决方案。换句话说，相同的实现将在*nix 和 Windows 系统上工作。

为了简单起见，我们将使用单元测试断言来验证解决方案是否按预期工作。

接下来，让我们看看他们的行动。

## 3.将绝对路径解析为字符串

首先，**文件系统不允许文件名包含文件分隔符。**因此，例如，我们不能在 Linux 的 Ext2、Ext3 或 Ext4 文件系统上创建一个文件名包含“/”的文件:

```java
$ touch "a/b.txt"
touch: cannot touch 'a/b.txt': No such file or directory
```

在上面的例子中，文件系统将“a/”视为一个目录。基于这个规则，解决问题的一个思路是**从最后一个文件分隔符开始取出子串，直到串尾。**

String 的 [`lastIndexOf()`](/web/20221010153619/https://www.baeldung.com/string/last-index-of) 方法返回子字符串在该字符串中的最后一个索引。然后，我们可以通过调用`absolutePath.substring(lastIndex+1)`简单地获得文件名。

正如我们所见，实现非常简单。但是，我们应该注意，为了使我们的解决方案独立于系统，我们不应该将文件分隔符硬编码为“\”用于 Windows，或者“/”用于*nix 系统。相反，**让我们在代码中使用`[File.separator](/web/20221010153619/https://www.baeldung.com/java-file-vs-file-path-separator)`,这样我们的程序就能自动适应运行它的系统:**

```java
int index = PATH_LINUX.lastIndexOf(File.separator);
String filenameLinux = PATH_LINUX.substring(index + 1);
assertEquals(EXPECTED_FILENAME_LINUX, filenameLinux);
```

如果我们在 Linux 机器上运行，上面的测试就通过了。类似地，下面的测试在 Windows 机器上通过:

```java
int index = PATH_WIN.lastIndexOf(File.pathSeparator);
String filenameWin = PATH_WIN.substring(index + 1);
assertEquals(EXPECTED_FILENAME_WIN, filenameWin);
```

正如我们所看到的，相同的实现在两个系统上都有效。

除了将绝对路径解析为字符串，我们可以使用标准的`[File](/web/20221010153619/https://www.baeldung.com/java-io-file)`类来解决这个问题。

## 4.使用`File.getName()`方法

`File`类提供了直接获取文件名的 [`getName()`](/web/20221010153619/https://www.baeldung.com/java-io-file#2-getting-metadata-about-file-instances) 方法。此外，我们可以从给定的绝对路径字符串中构造一个`File`对象。

让我们首先在 Linux 系统上测试它:

```java
File fileLinux = new File(PATH_LINUX);
assertEquals(EXPECTED_FILENAME_LINUX, fileLinux.getName());
```

如果我们试一试，测试就会通过。由于 **`File`在内部**使用`File.separator`，如果我们在 Windows 系统上测试相同的解决方案，它也通过了:

```java
File fileWin = new File(PATH_WIN);
assertEquals(EXPECTED_FILENAME_WIN, fileWin.getName());
```

## 5.使用`Path.getFileName()`方法

`File`是来自`java.io`包的标准类。从 Java 1.7 开始，更新的 [`java.nio`](/web/20221010153619/https://www.baeldung.com/java-io-vs-nio) 库带有`[Path](/web/20221010153619/https://www.baeldung.com/java-path-vs-file)`接口。

一旦我们有了一个`Path`对象，就可以通过调用 [`Path.getFileName()`](https://web.archive.org/web/20221010153619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html#getFileName()) 方法得到文件名。与`File`类不同，**我们可以使用静态的 [`Paths.get()`](https://web.archive.org/web/20221010153619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Paths.html#get(java.lang.String,java.lang.String...)) 方法创建一个`Path `实例。**

接下来，让我们从给定的`PATH_LINUX`字符串创建一个`Path`实例，并在 Linux 上测试该解决方案:

```java
Path pathLinux = Paths.get(PATH_LINUX);
assertEquals(EXPECTED_FILENAME_LINUX, pathLinux.getFileName().toString());
```

当我们执行测试时，它通过了。值得一提的是 **`Path.getFileName()`返回一个`Path`对象**。因此，我们显式调用`toString() `方法将其转换为字符串。

同样的实现也适用于使用`PATH_WIN`作为路径字符串的 Windows 系统。这是因为`Path`可以检测到它正在运行的电流`[FileSystem](https://web.archive.org/web/20221010153619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/FileSystem.html)`:

```java
Path pathWin = Paths.get(PATH_WIN);
assertEquals(EXPECTED_FILENAME_WIN, pathWin.getFileName().toString());
```

## 6.使用 Apache Commons IO 中的`FilenameUtils.getName()`

到目前为止，我们已经提出了三种从绝对路径中提取文件名的解决方案。正如我们提到的，它们是独立于平台的。然而，只有当给定的绝对路径与运行程序的系统相匹配时，这三种解决方案才能正确工作。例如，如果我们的程序运行在 Windows 上，它只能处理 Windows 路径。

### 6.1.智能`FilenameUtils.getName()`方法

实际上，解析不同系统的路径格式的可能性相对较低。但是， **[Apache Commons IO](/web/20221010153619/https://www.baeldung.com/apache-commons-io) 的 [`FilenameUtils`](/web/20221010153619/https://www.baeldung.com/apache-commons-io#2-filenameutils) 类可以从不同的路径格式**中“智能”提取文件名。因此，如果我们的程序在 Windows 上运行，它也可以为 Linux 文件路径工作，反之亦然。

接下来，让我们创建一个测试:

```java
String filenameLinux = FilenameUtils.getName(PATH_LINUX);
assertEquals(EXPECTED_FILENAME_LINUX, filenameLinux);

String filenameWin = FilenameUtils.getName(PATH_WIN);
assertEquals(EXPECTED_FILENAME_WIN, filenameWin);
```

正如我们所看到的，上面的测试解析了`PATH_LINUX`和`PATH_WIN`。无论我们在 Linux 还是 Windows 上运行，测试都通过了。

所以接下来，我们可能想知道`FilenameUtils`如何自动处理不同系统的路径。

### 6.2.`FilenameUtils.getName()`如何工作

如果我们看看`FilenameUtils.getName()`的实现，它的逻辑类似于我们的“lastIndexOf”文件分隔符方法。不同的是，`FilenameUtils`调用了两次 `lastIndexOf()` 方法，一次用*nix 分隔符(/)，然后用 Windows 文件分隔符(\)。最后，它将较大的索引作为“lastIndex”:

```java
...
final int lastUnixPos = fileName.lastIndexOf(UNIX_SEPARATOR); // UNIX_SEPARATOR = '/'
final int lastWindowsPos = fileName.lastIndexOf(WINDOWS_SEPARATOR); // WINDOWS_SEPARATOR = '\\'
return Math.max(lastUnixPos, lastWindowsPos);
```

因此， **`FilenameUtils.getName()`不检查当前文件系统或者系统的文件分隔符**。相反，它会找到最后一个文件分隔符的索引，不管它属于哪个系统，然后从这个索引中提取子串，直到字符串的末尾，作为最终结果。

### 6.3.使`FilenameUtils.getName()`失败的边缘情况

现在我们明白了`FilenameUtils.getName()`是如何工作的。这确实是一个聪明的解决方案，而且在大多数情况下都有效。然而，**许多 Linux 支持的文件系统允许文件名包含反斜杠(' \')** :

```java
$ echo 'Hi there!' > 'my\file.txt'

$ ls -l my*
-rw-r--r-- 1 kent kent 10 Sep 13 23:55 'my\file.txt'

$ cat 'my\file.txt'
Hi there!
```

如果给定 Linux 文件路径中的文件名包含反斜杠，`FilenameUtils.getName()`将会失败。一项测试可以清楚地解释这一点:

```java
String filenameToBreak = FilenameUtils.getName("/root/somedir/magic\\file.txt");
assertNotEquals("magic\\file.txt", filenameToBreak); // <-- filenameToBreak = "file.txt", but we expect: magic\file.txt
```

当我们使用这种方法时，我们应该记住这种情况。

## 7.结论

在本文中，我们学习了如何从给定的绝对路径字符串中提取文件名。

和往常一样，这个例子的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221010153619/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)