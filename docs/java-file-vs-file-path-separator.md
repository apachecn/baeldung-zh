# Java 文件分隔符与文件路径分隔符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-vs-file-path-separator>

## 1.概观

不同的操作系统使用不同的字符作为文件和路径分隔符。当我们的应用程序必须在多个平台上运行时，我们需要正确地处理这些问题。

Java 帮助我们选择一个合适的分隔符，并提供一些函数来帮助我们创建在主机操作系统上工作的路径。

在这个简短的教程中，我们将了解如何编写代码来使用正确的文件和路径分隔符。

## 2。文件分隔符

文件分隔符是用于分隔构成特定位置路径的目录名的字符。

### 2.1。获取文件分隔符

在 Java 中有几种方法可以得到文件分隔符。

我们可以使用`File.separator`得到分隔符作为`String`:

```
String fileSeparator = File.separator;
```

我们也可以将这个分隔符作为带有`File.separatorChar`的`char` :

```
char fileSeparatorChar = File.separatorChar;
```

从 Java 7 开始，我们也可以使用`FileSystems`:

```
String fileSeparator = FileSystems.getDefault().getSeparator();
```

输出将取决于主机操作系统。文件分隔符在 Windows 上是`\`，在 macOS 和基于 Unix 的操作系统上是`/`。

### 2.2。构建一个文件路径

Java 提供了两种从目录列表中构造文件路径的方法。

这里，我们使用的是`Paths`类:

```
Path path = Paths.get("dir1", "dir2");
```

让我们在 Microsoft Windows 上测试一下:

```
assertEquals("dir1\\dir2", path.toString());
```

同样，我们可以在 Linux 或 Mac 上测试它:

```
assertEquals("dir1/dir2", path.toString()); 
```

我们也可以使用`File` 类:

```
File file = new File("file1", "file2");
```

让我们在 Microsoft Windows 上测试一下:

```
assertEquals("file1\\file2", file.toString());
```

同样，我们可以在 Linux 或 Mac 上测试它:

```
assertEquals("file1/file2", file.toString());
```

正如我们所看到的，我们可以只提供路径字符串来构造文件路径——我们不需要显式地包含文件分隔符。

## 3。路径分隔符

路径分隔符是操作系统通常用来分隔路径列表中各个路径的字符。

### 3.1。获取路径分隔符

我们可以使用`File.pathSeparator`获得路径分隔符作为`String`:

```
String pathSeparator = File.pathSeparator;
```

我们也可以将路径分隔符作为一个 `char`:

```
char pathSeparatorChar = File.pathSeparatorChar;
```

两个示例都返回路径分隔符。它是分号(；`)`在基于 Windows 和冒号(:)的 Mac 和 Unix 操作系统上。

### 3.2。构建一个文件路径

我们可以使用分隔符作为分隔符来构造一个文件路径作为`String`。

**让我们试试 `String.join` 的方法:**

```
String[] pathNames = { "path1", "path2", "path3" };
String path = String.join(File.pathSeparator, pathNames);
```

这里我们在 Windows 上测试我们的代码:

```
assertEquals("path1;path2;path3", path);
```

和文件路径在 Linux 或 Mac 上会有所不同:

```
assertEquals("path1:path2:path3", path);
```

**同样，我们可以使用`StringJoiner`类:**

```
public static StringJoiner buildPathUsingStringJoiner(String path1, String path2) {
    StringJoiner joiner = new StringJoiner(File.pathSeparator);
    joiner.add(path1);
    joiner.add(path2);
    return joiner;
}
```

让我们在 Microsoft Windows 上测试我们的代码:

```
assertEquals("path1;path2", buildPathUsingStringJoiner("path1", "path2"));
```

它在 Mac 或 Unix 上的行为会有所不同:

```
assertEquals("path1:path2", buildPathUsingStringJoiner("path1", "path2"));
```

## 4。结论

在这篇短文中，我们学习了如何使用特定于系统的文件分隔符来构造路径，以便我们的代码可以在多个操作系统上工作。

我们看到了如何使用内置类`Path`和`File` 来构造文件路径，以及如何获得必要的分隔符来与`String`连接实用程序一起使用。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)