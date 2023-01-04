# Java–删除文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/how-to-delete-a-file-in-java>

这篇简短的文章展示了如何用 Java 删除文件——首先使用 JDK 6，然后是 JDK 7，最后是 Apache Commons IO 库。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用 Java——JDK 6

让我们从标准的 Java 6 解决方案开始:

```java
@Test
public void givenUsingJDK6_whenDeletingAFile_thenCorrect() throws IOException {
    new File("src/test/resources/fileToDelete_jdk6.txt").createNewFile();

    File fileToDelete = new File("src/test/resources/fileToDelete_jdk6.txt");
    boolean success = fileToDelete.delete();

    assertTrue(success);
}
```

如您所见-**文件必须在删除操作**之前存在；如果没有，API 将不会抛出任何异常，而是返回 false。

## 2。用 Java——JDK 7

让我们来看看 JDK 7 解决方案:

```java
@Test
public void givenUsingJDK7nio2_whenDeletingAFile_thenCorrect() throws IOException {
    Files.createFile(Paths.get("src/test/resources/fileToDelete_jdk7.txt"));

    Path fileToDeletePath = Paths.get("src/test/resources/fileToDelete_jdk7.txt");
    Files.delete(fileToDeletePath);
}
```

现在——这将更好地利用异常。如果文件在删除操作被触发时不存在，API 将抛出一个`NoSuchFileException`:

```java
java.nio.file.NoSuchFileException: srctestresourcesfileToDelete_jdk7.txt
    at s.n.f.WindowsException.translateToIOException(WindowsException.java:79)
```

## 3。带公共 IO

Commons IO 允许我们在删除文件时控制异常行为。对于隐藏任何可能异常的安静删除:

```java
@Test
public void givenUsingCommonsIo_whenDeletingAFileV1_thenCorrect() throws IOException {
    FileUtils.touch(new File("src/test/resources/fileToDelete_commonsIo.txt"));
    File fileToDelete = FileUtils.getFile("src/test/resources/fileToDelete_commonsIo.txt");
    boolean success = FileUtils.deleteQuietly(fileToDelete);

    assertTrue(success);
}
```

注意，我们仍然可以通过简单地检查 delete 方法的返回值来确定操作是否成功。

现在——如果我们想抛出一个异常:

```java
@Test
public void givenUsingCommonsIo_whenDeletingAFileV2_thenCorrect() throws IOException {
    FileUtils.touch(new File("src/test/resources/fileToDelete.txt"));

    FileUtils.forceDelete(FileUtils.getFile("src/test/resources/fileToDelete.txt"));
}
```

如果文件系统中不存在要删除的文件，API 将抛出一个标准的`FileNotFoundException`:

```java
java.io.FileNotFoundException: File does not exist: srctestresourcesfileToDelete.txt
    at org.apache.commons.io.FileUtils.forceDelete(FileUtils.java:2275)
```

现在你知道了——用 Java 删除文件的 4 种简单方法。