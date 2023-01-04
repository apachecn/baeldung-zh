# 用 Java 递归删除一个目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-delete-directory>

## 1。简介

在本文中，我们将演示如何在普通 Java 中递归删除目录。我们还将看看使用外部库删除目录的一些替代方法。

## 2。递归删除目录

Java 有一个删除目录的选项。但是，这要求目录为空。因此，我们需要使用递归来删除一个特定的非空目录:

1.  获取要删除的目录的所有内容
2.  删除所有不是目录的子目录(退出递归)
3.  对于当前目录的每个子目录，从步骤 1(递归步骤)开始
4.  删除目录

让我们实现这个简单的算法:

```java
boolean deleteDirectory(File directoryToBeDeleted) {
    File[] allContents = directoryToBeDeleted.listFiles();
    if (allContents != null) {
        for (File file : allContents) {
            deleteDirectory(file);
        }
    }
    return directoryToBeDeleted.delete();
}
```

这个方法可以用一个简单的测试用例来测试:

```java
@Test
public void givenDirectory_whenDeletedWithRecursion_thenIsGone() 
  throws IOException {

    Path pathToBeDeleted = TEMP_DIRECTORY.resolve(DIRECTORY_NAME);

    boolean result = deleteDirectory(pathToBeDeleted.toFile());

    assertTrue(result);
    assertFalse(
      "Directory still exists", 
      Files.exists(pathToBeDeleted));
}
```

我们的测试类的`@Before`方法在`pathToBeDeleted`位置创建一个包含子目录和文件的目录树，如果需要的话，`@After`方法清理目录。

接下来，让我们看看如何使用两个最常用的库来实现删除——Apache 的`commons-io`和 Spring Framework 的`spring-core.`,这两个库都允许我们只使用一行代码来删除目录。

## 3。使用`commons-io`中的`FileUtils`和

首先，我们需要向 Maven 项目添加`commons-io`依赖项:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22)找到。

现在，我们可以用一条语句使用`FileUtils`来执行任何基于文件的操作，包括`deleteDirectory()`:

```java
FileUtils.deleteDirectory(file);
```

## 4。利用`FileSystemUtils`从春天

或者，我们可以将 s `pring-core`依赖项添加到 Maven 项目中:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.3.10.RELEASE</version>
</dependency>
```

最新版本的依赖可以在这里找到[。](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-core%22)

我们可以使用`FileSystemUtils`中的`deleteRecursively()`方法来执行删除:

```java
boolean result = FileSystemUtils.deleteRecursively(file);
```

Java 的最新版本提供了执行这种 IO 操作的新方法，下面几节将对此进行描述。

## 5。在 Java 7 中使用 nio 2

Java 7 引入了一种全新的使用`Files`执行文件操作的方式。它允许我们遍历目录树，并使用回调来执行操作。

```java
public void whenDeletedWithNIO2WalkFileTree_thenIsGone() 
  throws IOException {

    Path pathToBeDeleted = TEMP_DIRECTORY.resolve(DIRECTORY_NAME);

    Files.walkFileTree(pathToBeDeleted, 
      new SimpleFileVisitor<Path>() {
        @Override
        public FileVisitResult postVisitDirectory(
          Path dir, IOException exc) throws IOException {
            Files.delete(dir);
            return FileVisitResult.CONTINUE;
        }

        @Override
        public FileVisitResult visitFile(
          Path file, BasicFileAttributes attrs) 
          throws IOException {
            Files.delete(file);
            return FileVisitResult.CONTINUE;
        }
    });

    assertFalse("Directory still exists", 
      Files.exists(pathToBeDeleted));
}
```

`Files.walkFileTree()`方法遍历文件树并发出事件。我们需要为这些事件指定回调。因此，在这种情况下，我们将定义`SimpleFileVisitor`对生成的事件采取以下操作:

1.  访问文件–删除它
2.  在处理目录条目之前访问目录–什么也不做
3.  处理完目录条目后访问目录——删除该目录，因为现在该目录中的所有条目都已被处理(或删除)
4.  无法访问文件–重新抛出导致失败的`IOException`

关于 NIO2 APIs 处理文件操作的更多细节，请参见[Java nio 2 文件 API 介绍](/web/20221208143832/https://www.baeldung.com/java-nio-2-file-api)。

## 6。使用 *NIO2 搭配 Java 8*

从 Java 8 开始，Stream API 提供了一种更好的删除目录的方法:

```java
@Test
public void whenDeletedWithFilesWalk_thenIsGone() 
  throws IOException {
    Path pathToBeDeleted = TEMP_DIRECTORY.resolve(DIRECTORY_NAME);

    Files.walk(pathToBeDeleted)
      .sorted(Comparator.reverseOrder())
      .map(Path::toFile)
      .forEach(File::delete);

    assertFalse("Directory still exists", 
      Files.exists(pathToBeDeleted));
}
```

在这里，`Files.walk()`返回一个`Path`的`Stream`，我们以相反的顺序排序。这将表示目录内容的路径放在目录本身之前。此后，它将`Path`映射到`File`，并删除每个`File.`

## 7.结论

在这个快速教程中，我们探索了删除目录的不同方法。当我们看到如何使用递归来删除时，我们也看了一些库，NIO2 利用事件和 Java 8 路径流采用函数式编程范式。

本文的所有源代码和测试用例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/libraries-4)