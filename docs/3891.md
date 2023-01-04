# Java–目录大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-folder-size>

## 1。概述

在本教程中，我们将学习如何使用 Java 6、7 和新的 Java 8 以及 Guava 和 Apache Common IO 获得 Java 中文件夹的**大小。**

最后–我们还将获得一个可读的目录大小表示。

## 2。用 Java

让我们从一个简单的计算文件夹大小的例子开始—**使用其内容的总和**:

```
private long getFolderSize(File folder) {
    long length = 0;
    File[] files = folder.listFiles();

    int count = files.length;

    for (int i = 0; i < count; i++) {
        if (files[i].isFile()) {
            length += files[i].length();
        }
        else {
            length += getFolderSize(files[i]);
        }
    }
    return length;
}
```

我们可以测试我们的方法`getFolderSize()`,如下例所示:

```
@Test
public void whenGetFolderSizeRecursive_thenCorrect() {
    long expectedSize = 12607;

    File folder = new File("src/test/resources");
    long size = getFolderSize(folder);

    assertEquals(expectedSize, size);
}
```

注意:`listFiles()`用于列出给定文件夹的内容。

## 3。用 Java 7

接下来——让我们看看如何使用 **Java 7 来获得文件夹大小**。在下面的例子中——我们使用`Files.walkFileTree()`遍历文件夹中的所有文件，对它们的大小求和:

```
@Test
public void whenGetFolderSizeUsingJava7_thenCorrect() throws IOException {
    long expectedSize = 12607;

    AtomicLong size = new AtomicLong(0);
    Path folder = Paths.get("src/test/resources");

    Files.walkFileTree(folder, new SimpleFileVisitor<Path>() {
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) 
          throws IOException {
            size.addAndGet(attrs.size());
            return FileVisitResult.CONTINUE;
        }
    });

    assertEquals(expectedSize, size.longValue());
}
```

请注意我们如何利用文件系统树遍历功能，并利用访问者模式来帮助我们访问和计算每个文件和子文件夹的大小。

## 4。用 Java 8

现在——让我们看看如何使用 **Java 8、流操作和 lambdas** 获得文件夹大小。在下面的例子中——我们使用`Files.walk()`遍历文件夹中的所有文件，对它们的大小求和:

```
@Test
public void whenGetFolderSizeUsingJava8_thenCorrect() throws IOException {
    long expectedSize = 12607;

    Path folder = Paths.get("src/test/resources");
    long size = Files.walk(folder)
      .filter(p -> p.toFile().isFile())
      .mapToLong(p -> p.toFile().length())
      .sum();

    assertEquals(expectedSize, size);
}
```

注意:`mapToLong()`用于通过在每个元素中应用`length`函数来生成一个`LongStream`——之后我们可以`sum`并得到一个最终结果。

## 5。使用 Apache Commons IO

接下来，让我们看看如何使用 **Apache Commons IO** 来获取文件夹大小。在下面的例子中，我们简单地使用`FileUtils.sizeOfDirectory()`来获得文件夹的大小:

```
@Test
public void whenGetFolderSizeUsingApacheCommonsIO_thenCorrect() {
    long expectedSize = 12607;

    File folder = new File("src/test/resources");
    long size = FileUtils.sizeOfDirectory(folder);

    assertEquals(expectedSize, size);
}
```

注意，这个简单实用的方法实现了一个简单的 Java 6 解决方案。

另外，请注意，该库还提供了一个`FileUtils.sizeOfDirectoryAsBigInteger()`方法，可以更好地处理安全受限目录。

## 6。有番石榴

现在，让我们看看如何使用**番石榴**来计算一个文件夹的大小。在下面的例子中——我们使用`Files.fileTreeTraverser()`遍历文件夹中的所有文件，对它们的大小求和:

```
@Test public void whenGetFolderSizeUsingGuava_thenCorrect() { 
    long expectedSize = 12607; 
    File folder = new File("src/test/resources"); 

    Iterable<File> files = Files.fileTraverser().breadthFirst(folder);
    long size = StreamSupport.stream(files.spliterator(), false) .filter(f -> f.isFile()) 
      .mapToLong(File::length).sum(); 

    assertEquals(expectedSize, size); 
}
```

## 7。人类可读尺寸

最后，让我们看看如何获得一个更易于用户阅读的文件夹大小表示，而不仅仅是以字节为单位的大小:

```
@Test
public void whenGetReadableSize_thenCorrect() {
    File folder = new File("src/test/resources");
    long size = getFolderSize(folder);

    String[] units = new String[] { "B", "KB", "MB", "GB", "TB" };
    int unitIndex = (int) (Math.log10(size) / 3);
    double unitValue = 1 << (unitIndex * 10);

    String readableSize = new DecimalFormat("#,##0.#")
                                .format(size / unitValue) + " " 
                                + units[unitIndex];
    assertEquals("12.3 KB", readableSize);
}
```

注意:我们使用`DecimalFormat(“#,##0,#”)`将结果四舍五入到小数点后一位。

## 8。注释

以下是关于文件夹大小计算的一些注意事项:

*   如果安全管理器拒绝访问起始文件，`Files.walk()`和`Files.walkFileTree()`都将抛出 SecurityException。
*   如果文件夹包含符号链接，可能会出现无限循环。

## 9。结论

在这个快速教程中，我们举例说明了使用不同的 **Java** 版本、 **Apache Commons IO** 和 **Guava** 来计算文件系统中目录的大小。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行。