# Java NIO2 文件 API 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio-2-file-api>

## 1。概述

在本文中，我们将关注 Java 平台中新的 I/O API—**nio 2——来进行基本的文件操作**。

NIO2 中的文件 API 构成了 Java 7 附带的 Java 平台的主要新功能领域之一，特别是新文件系统 API 和路径 API 的子集。

## 2。设置

设置您的项目以使用文件 API 只是进行导入的问题:

```
import java.nio.file.*;
```

由于本文中的代码示例可能会在不同的环境中运行，所以让我们来了解一下用户的主目录，这在所有操作系统中都是有效的:

```
private static String HOME = System.getProperty("user.home");
```

`Files`类是`java.nio.file`包的主要入口点之一。这个类提供了一组丰富的 API 来读取、写入和操作文件和目录。`Files`类方法作用于`Path` 对象的实例。

## 3。检查文件或目录

我们可以用一个`Path`实例来表示文件系统中的一个文件或一个目录。它所指向的文件或目录是否存在，是否可访问可以通过文件操作来确认。

为了简单起见，每当我们使用术语`file`时，我们将指文件和目录，除非另有明确说明。

为了检查文件是否存在，我们使用了`exists` API:

```
@Test
public void givenExistentPath_whenConfirmsFileExists_thenCorrect() {
    Path p = Paths.get(HOME);

    assertTrue(Files.exists(p));
}
```

为了检查文件是否存在，我们使用了`notExists` API:

```
@Test
public void givenNonexistentPath_whenConfirmsFileNotExists_thenCorrect() {
    Path p = Paths.get(HOME + "/inexistent_file.txt");

    assertTrue(Files.notExists(p));
}
```

我们还可以检查一个文件是像`myfile.txt`一样的常规文件还是仅仅是一个目录，我们使用`isRegularFile` API:

```
@Test
public void givenDirPath_whenConfirmsNotRegularFile_thenCorrect() {
    Path p = Paths.get(HOME);

    assertFalse(Files.isRegularFile(p));
}
```

还有静态方法来检查文件权限。为了检查文件是否可读，我们使用了`isReadable` API:

```
@Test
public void givenExistentDirPath_whenConfirmsReadable_thenCorrect() {
    Path p = Paths.get(HOME);

    assertTrue(Files.isReadable(p));
}
```

为了检查它是否可写，我们使用了`isWritable` API:

```
@Test
public void givenExistentDirPath_whenConfirmsWritable_thenCorrect() {
    Path p = Paths.get(HOME);

    assertTrue(Files.isWritable(p));
}
```

同样，要检查它是否可执行:

```
@Test
public void givenExistentDirPath_whenConfirmsExecutable_thenCorrect() {
    Path p = Paths.get(HOME);
    assertTrue(Files.isExecutable(p));
}
```

当我们有两个路径时，我们可以检查它们是否都指向底层文件系统上的同一个文件:

```
@Test
public void givenSameFilePaths_whenConfirmsIsSame_thenCorrect() {
    Path p1 = Paths.get(HOME);
    Path p2 = Paths.get(HOME);

    assertTrue(Files.isSameFile(p1, p2));
}
```

## 4。创建文件

文件系统 API 提供了创建文件的单行操作。为了创建一个常规文件，我们使用`createFile` API 并传递给它一个代表我们想要创建的文件的`Path`对象。

路径中的所有名称元素必须存在，除了文件名，否则，我们将得到一个`IOException:`

```
@Test
public void givenFilePath_whenCreatesNewFile_thenCorrect() {
    String fileName = "myfile_" + UUID.randomUUID().toString() + ".txt";
    Path p = Paths.get(HOME + "/" + fileName);
    assertFalse(Files.exists(p));

    Files.createFile(p);

    assertTrue(Files.exists(p));
}
```

在上面的测试中，当我们第一次检查路径时，它是不存在的，然后经过`createFile`操作后，发现它是存在的。

为了创建目录，我们使用了`createDirectory` API:

```
@Test
public void givenDirPath_whenCreatesNewDir_thenCorrect() {
    String dirName = "myDir_" + UUID.randomUUID().toString();
    Path p = Paths.get(HOME + "/" + dirName);
    assertFalse(Files.exists(p));

    Files.createDirectory(p);

    assertTrue(Files.exists(p));
    assertFalse(Files.isRegularFile(p));
    assertTrue(Files.isDirectory(p));
}
```

这个操作要求路径中的所有 name 元素都存在，如果不存在，我们还会得到一个`IOException`:

```
@Test(expected = NoSuchFileException.class)
public void givenDirPath_whenFailsToCreateRecursively_thenCorrect() {
    String dirName = "myDir_" + UUID.randomUUID().toString() + "/subdir";
    Path p = Paths.get(HOME + "/" + dirName);
    assertFalse(Files.exists(p));

    Files.createDirectory(p);
}
```

然而，如果我们希望通过一次调用创建一个目录层次结构，我们使用`createDirectories`方法。与前面的操作不同，当它在路径中遇到任何缺少的 name 元素时，它不会抛出一个`IOException`，而是递归地创建它们，直到最后一个元素:

```
@Test
public void givenDirPath_whenCreatesRecursively_thenCorrect() {
    Path dir = Paths.get(
      HOME + "/myDir_" + UUID.randomUUID().toString());
    Path subdir = dir.resolve("subdir");
    assertFalse(Files.exists(dir));
    assertFalse(Files.exists(subdir));

    Files.createDirectories(subdir);

    assertTrue(Files.exists(dir));
    assertTrue(Files.exists(subdir));
}
```

## 5。创建临时文件

许多应用程序在运行时会在文件系统中创建一系列临时文件。因此，大多数文件系统都有一个专用目录来存储此类应用程序生成的临时文件。

新的文件系统 API 为此提供了特定的操作。`createTempFile` API 执行这个操作。它采用一个路径对象、一个文件前缀和一个文件后缀:

```
@Test
public void givenFilePath_whenCreatesTempFile_thenCorrect() {
    String prefix = "log_";
    String suffix = ".txt";
    Path p = Paths.get(HOME + "/");

    Files.createTempFile(p, prefix, suffix);

    assertTrue(Files.exists(p));
}
```

这些参数足以满足需要此操作的要求。但是，如果需要指定文件的特定属性，还有第四个变量 arguments 参数。

上面的测试在`HOME`目录中创建了一个临时文件，分别预先挂起和附加了提供的前缀和后缀字符串。我们将以类似于`log_8821081429012075286.txt`的文件名结束。长数字字符串是系统生成的。

但是，如果我们不提供前缀和后缀，那么文件名将只包括长数字字符串和默认的`.tmp`扩展名:

```
@Test
public void givenPath_whenCreatesTempFileWithDefaults_thenCorrect() {
    Path p = Paths.get(HOME + "/");

    Files.createTempFile(p, null, null);

    assertTrue(Files.exists(p));
}
```

上面的操作创建了一个文件名类似于`8600179353689423985.tmp`的文件。

最后，如果我们既不提供路径，也不提供前缀或后缀，那么操作将始终使用默认值。所创建文件的默认位置将是文件系统提供的临时文件目录:

```
@Test
public void givenNoFilePath_whenCreatesTempFileInTempDir_thenCorrect() {
    Path p = Files.createTempFile(null, null);

    assertTrue(Files.exists(p));
}
```

在 windows 上，这将默认为类似于`C:\Users\user\AppData\Local\Temp\6100927974988978748.tmp`的东西。

以上所有操作都可以通过使用`createTempDirectory`而不是`createTempFile`来创建目录而不是常规文件。

## 6。删除文件

要删除一个文件，我们使用`delete` API。为了清楚起见，下面的测试首先确保该文件不存在，然后创建它并确认它现在存在，最后删除它并确认它不再存在:

```
@Test
public void givenPath_whenDeletes_thenCorrect() {
    Path p = Paths.get(HOME + "/fileToDelete.txt");
    assertFalse(Files.exists(p));
    Files.createFile(p);
    assertTrue(Files.exists(p));

    Files.delete(p);

    assertFalse(Files.exists(p));
}
```

但是，如果文件系统中不存在某个文件，删除操作将失败，并显示`IOException`:

```
@Test(expected = NoSuchFileException.class)
public void givenInexistentFile_whenDeleteFails_thenCorrect() {
    Path p = Paths.get(HOME + "/inexistentFile.txt");
    assertFalse(Files.exists(p));

    Files.delete(p);
}
```

我们可以通过使用`deleteIfExists`来避免这种情况，如果文件不存在，它会自动失败。当多个线程正在执行该操作时，这一点很重要，我们不希望仅仅因为一个线程比当前失败的线程更早执行该操作而出现失败消息:

```
@Test
public void givenInexistentFile_whenDeleteIfExistsWorks_thenCorrect() {
    Path p = Paths.get(HOME + "/inexistentFile.txt");
    assertFalse(Files.exists(p));

    Files.deleteIfExists(p);
}
```

当处理目录而不是常规文件时，我们应该记住删除操作在默认情况下不是递归的。因此，如果一个目录不为空，它将失败并显示一个`IOException`:

```
@Test(expected = DirectoryNotEmptyException.class)
public void givenPath_whenFailsToDeleteNonEmptyDir_thenCorrect() {
    Path dir = Paths.get(
      HOME + "/emptyDir" + UUID.randomUUID().toString());
    Files.createDirectory(dir);
    assertTrue(Files.exists(dir));

    Path file = dir.resolve("file.txt");
    Files.createFile(file);

    Files.delete(dir);

    assertTrue(Files.exists(dir));
}
```

## 7。复制文件

您可以使用`copy` API 复制文件或目录:

```
@Test
public void givenFilePath_whenCopiesToNewLocation_thenCorrect() {
    Path dir1 = Paths.get(
      HOME + "/firstdir_" + UUID.randomUUID().toString());
    Path dir2 = Paths.get(
      HOME + "/otherdir_" + UUID.randomUUID().toString());

    Files.createDirectory(dir1);
    Files.createDirectory(dir2);

    Path file1 = dir1.resolve("filetocopy.txt");
    Path file2 = dir2.resolve("filetocopy.txt");

    Files.createFile(file1);

    assertTrue(Files.exists(file1));
    assertFalse(Files.exists(file2));

    Files.copy(file1, file2);

    assertTrue(Files.exists(file2));
}
```

如果目标文件存在，除非指定了`REPLACE_EXISTING`选项，否则复制会失败:

```
@Test(expected = FileAlreadyExistsException.class)
public void givenPath_whenCopyFailsDueToExistingFile_thenCorrect() {
    Path dir1 = Paths.get(
      HOME + "/firstdir_" + UUID.randomUUID().toString());
    Path dir2 = Paths.get(
      HOME + "/otherdir_" + UUID.randomUUID().toString());

    Files.createDirectory(dir1);
    Files.createDirectory(dir2);

    Path file1 = dir1.resolve("filetocopy.txt");
    Path file2 = dir2.resolve("filetocopy.txt");

    Files.createFile(file1);
    Files.createFile(file2);

    assertTrue(Files.exists(file1));
    assertTrue(Files.exists(file2));

    Files.copy(file1, file2);

    Files.copy(file1, file2, StandardCopyOption.REPLACE_EXISTING);
}
```

但是，在复制目录时，不会递归复制内容。这意味着如果`/baeldung`包含`/articles.db`和`/authors.db`文件，将`/baeldung`复制到新位置将创建一个空目录。

## 8。移动文件

您可以使用`move` API 来移动文件或目录。它在大多数方面类似于`copy`操作。如果复制操作类似于基于 GUI 的系统中的`copy and paste`操作，那么`move`类似于`cut and paste`操作:

```
@Test
public void givenFilePath_whenMovesToNewLocation_thenCorrect() {
    Path dir1 = Paths.get(
      HOME + "/firstdir_" + UUID.randomUUID().toString());
    Path dir2 = Paths.get(
      HOME + "/otherdir_" + UUID.randomUUID().toString());

    Files.createDirectory(dir1);
    Files.createDirectory(dir2);

    Path file1 = dir1.resolve("filetocopy.txt");
    Path file2 = dir2.resolve("filetocopy.txt");
    Files.createFile(file1);

    assertTrue(Files.exists(file1));
    assertFalse(Files.exists(file2));

    Files.move(file1, file2);

    assertTrue(Files.exists(file2));
    assertFalse(Files.exists(file1));
}
```

如果目标文件存在，则`move`操作失败，除非指定了`REPLACE_EXISTING`选项，就像我们对`copy`操作所做的那样:

```
@Test(expected = FileAlreadyExistsException.class)
public void givenFilePath_whenMoveFailsDueToExistingFile_thenCorrect() {
    Path dir1 = Paths.get(
      HOME + "/firstdir_" + UUID.randomUUID().toString());
    Path dir2 = Paths.get(
      HOME + "/otherdir_" + UUID.randomUUID().toString());

    Files.createDirectory(dir1);
    Files.createDirectory(dir2);

    Path file1 = dir1.resolve("filetocopy.txt");
    Path file2 = dir2.resolve("filetocopy.txt");

    Files.createFile(file1);
    Files.createFile(file2);

    assertTrue(Files.exists(file1));
    assertTrue(Files.exists(file2));

    Files.move(file1, file2);

    Files.move(file1, file2, StandardCopyOption.REPLACE_EXISTING);

    assertTrue(Files.exists(file2));
    assertFalse(Files.exists(file1));
}
```

## 9。结论

在本文中，我们了解了作为 Java 7 的一部分提供的新文件系统 API (NIO2)中的文件 API，并看到了大多数重要的文件操作。

本文中使用的代码示例可以在文章的 [Github 项目](https://web.archive.org/web/20220627174946/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)中找到。