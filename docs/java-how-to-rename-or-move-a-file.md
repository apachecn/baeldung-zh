# Java–重命名或移动文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-how-to-rename-or-move-a-file>

## 1。概述

在这个快速教程中，我们将学习在 Java 中重命名/移动文件。

我们将首先研究如何使用 NIO 中的`Files`和`Path`类，然后是 Java `File`类、Google Guava，最后是 Apache Commons IO 库。

这篇文章是 Baeldung 上的[**Java-回到基础**系列](/web/20221013193919/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [如何用 Java 复制文件](/web/20221013193919/https://www.baeldung.com/java-copy-file)

Take a look at some common ways of copying files in Java.[Read more](/web/20221013193919/https://www.baeldung.com/java-copy-file) →

## [Java nio 2 文件 API 介绍](/web/20221013193919/https://www.baeldung.com/java-nio-2-file-api)

A quick and practical guide to Java NIO2 File API[Read more](/web/20221013193919/https://www.baeldung.com/java-nio-2-file-api) →

## [Java 中的文件大小](/web/20221013193919/https://www.baeldung.com/java-file-size)

Examples of how to get the size of a file in Java.[Read more](/web/20221013193919/https://www.baeldung.com/java-file-size) →

## 2.设置

在示例中，我们将使用下面的设置，它由源文件名和目标文件名的两个常量以及一个能够多次运行测试的清理步骤组成:

```java
private final String FILE_TO_MOVE = "src/test/resources/originalFileToMove.txt";
private final String TARGET_FILE = "src/test/resources/targetFileToMove.txt";

@BeforeEach
public void createFileToMove() throws IOException {
    File fileToMove = new File(FILE_TO_MOVE);
    fileToMove.createNewFile();
}

@AfterEach
public void cleanUpFiles() {
    File targetFile = new File(TARGET_FILE);
    targetFile.delete();
}
```

## 3。使用 NIO `Paths`和`Files` 类

让我们从使用 Java NIO 包中的`Files.move()`方法的**开始:**

```java
@Test
public void givenUsingNio_whenMovingFile_thenCorrect() throws IOException {
    Path fileToMovePath = Paths.get(FILE_TO_MOVE);
    Path targetPath = Paths.get(TARGET_FILE);
    Files.move(fileToMovePath, targetPath);
}
```

在 JDK7 中，NIO 包进行了重大更新，并添加了`Path`类。这提供了方便操作文件系统工件的方法。

请注意，文件和目标目录都应该存在。

## 4。使用`File`类

现在让我们看看如何使用`File.renameTo()` 方法来做同样的

```java
@Test
public void givenUsingFileClass_whenMovingFile_thenCorrect() throws IOException {
    File fileToMove = new File(FILE_TO_MOVE);
    boolean isMoved = fileToMove.renameTo(new File(TARGET_FILE));
    if (!isMoved) {
        throw new FileSystemException(TARGET_FILE);
    }
}
```

在本例中，要移动的文件以及目标目录确实存在。

注意`renameTo()`只抛出两种类型的异常:

*   `SecurityException`–如果安全管理员拒绝对源或目标的写访问
*   `NullPointerException`–如果参数 target 为空

如果目标在文件系统中不存在——不会抛出异常——您必须检查该方法返回的成功标志。

## 5。使用番石榴

接下来，让我们来看看番石榴溶液，它提供了一种方便的`Files.move()`方法:

```java
@Test
public void givenUsingGuava_whenMovingFile_thenCorrect()
        throws IOException {
    File fileToMove = new File(FILE_TO_MOVE);
    File targetFile = new File(TARGET_FILE);

    com.google.common.io.Files.move(fileToMove, targetFile);
}
```

同样，在本例中，要移动的文件和目标目录需要存在。

## 6。带公共 IO

最后，让我们看看 Apache Commons IO 的解决方案——可能是最简单的一个:

```java
@Test
public void givenUsingApache_whenMovingFile_thenCorrect() throws IOException {
    FileUtils.moveFile(FileUtils.getFile(FILE_TO_MOVE), FileUtils.getFile(TARGET_FILE));
}
```

当然，这一行允许移动或重命名，这取决于目标目录是否相同。

或者，这里有一个专门用于移动的解决方案，如果目标目录不存在，我们还可以自动创建它:

```java
@Test
public void givenUsingApache_whenMovingFileApproach2_thenCorrect() throws IOException {
    FileUtils.moveFileToDirectory(
      FileUtils.getFile("src/test/resources/fileToMove.txt"), 
      FileUtils.getFile("src/main/resources/"), true);
}
```

## 6。结论

在本文中，我们研究了在 Java 中移动文件的不同解决方案。在这些代码片段中，我们着重于重命名，但是移动当然是一样的，只是目标目录需要不同。

GitHub 上的[提供了示例代码。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)**