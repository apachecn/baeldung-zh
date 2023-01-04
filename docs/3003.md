# 用 Java 查找目录中最后修改的文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-last-modified-file>

## 1.概观

在这个快速教程中，我们将仔细研究如何在 Java 中找到特定目录中最后修改的文件。

首先，我们将从[传统 IO](/web/20221208143828/https://www.baeldung.com/java-io-vs-nio#1-io---javaio) 和[现代 NIO](/web/20221208143828/https://www.baeldung.com/java-io-vs-nio#2-nio---javanio)API 开始。然后，我们将看到如何使用 [Apache Commons IO](https://web.archive.org/web/20221208143828/https://commons.apache.org/proper/commons-io/) 库来完成同样的事情。

## 2。使用 `java.io` **API**

遗留的`java.io` 包提供了`File`类来封装文件和目录路径名的抽象表示。

幸运的是，`File`类附带了一个叫做`lastModified().` **的简便方法。这个方法返回由抽象路径名**表示的文件的最后修改时间。

现在让我们看看如何使用`java.io.File`类来达到预期的目的:

```
public static File findUsingIOApi(String sdir) {
    File dir = new File(sdir);
    if (dir.isDirectory()) {
        Optional<File> opFile = Arrays.stream(dir.listFiles(File::isFile))
          .max((f1, f2) -> Long.compare(f1.lastModified(), f2.lastModified()));

        if (opFile.isPresent()){
            return opFile.get();
        }
    }

    return null;
}
```

正如我们所见，我们使用 [Java 8 Stream API](/web/20221208143828/https://www.baeldung.com/java-8-streams) 来遍历一组文件。然后，我们调用`max()` 操作来获取带有最近修改的文件`.`

注意，我们使用了一个 [`Optional`实例](/web/20221208143828/https://www.baeldung.com/java-optional)来封装最后修改的文件。

请记住，这种方法被认为是过时的。然而，如果我们想与 Java 遗留 IO 世界保持兼容，我们可以使用它。

## 3。使用`java.nio` **API**

NIO API 的引入是文件系统管理的一个转折点。Java 7 中发布的新版本 NIO.2 带有一组增强特性，可以更好地管理和操作文件。

事实上，在 Java 中操作文件和目录时，`java.nio.file.Files` 类提供了很大的灵活性。

因此，让我们看看如何利用`Files` 类来获取目录中最后修改的文件:

```
public static Path findUsingNIOApi(String sdir) throws IOException {
    Path dir = Paths.get(sdir);
    if (Files.isDirectory(dir)) {
        Optional<Path> opPath = Files.list(dir)
          .filter(p -> !Files.isDirectory(p))
          .sorted((p1, p2)-> Long.valueOf(p2.toFile().lastModified())
            .compareTo(p1.toFile().lastModified()))
          .findFirst();

        if (opPath.isPresent()){
            return opPath.get();
        }
    }

    return null;
}
```

与第一个例子类似，我们依靠 Steam API 只获取文件。然后，我们借助于一个[λ表达式](/web/20221208143828/https://www.baeldung.com/java-8-lambda-expressions-tips) ，根据最后修改时间对文件进行排序。

## 4.**使用 Apache Commons IO**

Apache Commons IO 将文件系统管理提升到了一个新的层次。它提供了一组方便的类、文件比较器、文件过滤器等等。

对我们来说幸运的是，这个库提供了 **`LastModifiedFileComparator` 类，它可以作为一个** **比较器，根据文件的最后修改时间**对一组文件进行排序。

首先，我们需要在我们的`pom.xml`中添加 [`commons-io`依赖关系](https://web.archive.org/web/20221208143828/https://search.maven.org/classic/#artifactdetails%7Ccommons-io%7Ccommons-io%7C2.7%7Cjar):

```
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

最后，让我们展示如何使用 Apache Commons IO 在文件夹中找到最后修改的文件:

```
public static File findUsingCommonsIO(String sdir) {
    File dir = new File(sdir);
    if (dir.isDirectory()) {
        File[] dirFiles = dir.listFiles((FileFilter)FileFilterUtils.fileFileFilter());
        if (dirFiles != null && dirFiles.length > 0) {
            Arrays.sort(dirFiles, LastModifiedFileComparator.LASTMODIFIED_REVERSE);
            return dirFiles[0];
        }
     }

    return null;
}
```

**如上图所示，我们使用** **单例实例`LASTMODIFIED_REVERSE`对我们的文件数组进行逆序排序**。

因为数组是反向排序的，所以最后修改的文件是数组的第一个元素。

## 5.结论

在本教程中，我们探索了在特定目录中查找最后修改的文件的不同方法。在这个过程中，我们使用了属于 JDK 和 Apache Commons IO 外部库的 API。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221208143828/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)