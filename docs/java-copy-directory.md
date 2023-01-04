# 用 Java 复制一个目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-directory>

## 1.介绍

在这个简短的教程中，我们将看到如何用 Java 复制一个目录，包括它的所有文件和子目录。这可以通过使用核心 Java 特性或第三方库来实现。

## 2.使用`java.nio` API

[Java `NIO`](https://web.archive.org/web/20220627080539/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/package-summary.html) 从 Java 1.4 开始就有了。Java 7 引入了 [`NIO 2`](/web/20220627080539/https://www.baeldung.com/java-nio-2-file-api) 带来了很多有用的特性，比如更好地支持处理符号链接、文件属性访问。它还为我们提供了诸如`[Path](/web/20220627080539/https://www.baeldung.com/java-nio-2-path)`、`Paths`和`Files`之类的类，使得文件系统操作变得更加容易。

让我们演示一下这种方法:

```java
public static void copyDirectory(String sourceDirectoryLocation, String destinationDirectoryLocation) 
  throws IOException {
    Files.walk(Paths.get(sourceDirectoryLocation))
      .forEach(source -> {
          Path destination = Paths.get(destinationDirectoryLocation, source.toString()
            .substring(sourceDirectoryLocation.length()));
          try {
              Files.copy(source, destination);
          } catch (IOException e) {
              e.printStackTrace();
          }
      });
}
```

在这个例子中，**我们使用`Files.walk()`遍历以给定源目录为根的文件树，并为我们在源目录中找到的每个文件或目录**调用`Files.copy()`。

## 3.使用`java.io` API

从文件系统管理的角度来看，Java 7 是一个转折点，因为它引入了许多方便的新特性。

然而，如果我们想与旧的 Java 版本保持兼容，我们可以使用递归复制一个目录**和 [`java.io.File`](https://web.archive.org/web/20220627080539/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html) 特性**:

```java
private static void copyDirectory(File sourceDirectory, File destinationDirectory) throws IOException {
    if (!destinationDirectory.exists()) {
        destinationDirectory.mkdir();
    }
    for (String f : sourceDirectory.list()) {
        copyDirectoryCompatibityMode(new File(sourceDirectory, f), new File(destinationDirectory, f));
    }
}
```

在本例中，**我们将在目标目录中为源目录树**中的每个目录创建一个目录。然后我们将调用`copyDirectoryCompatibityMode()`方法:

```java
public static void copyDirectoryCompatibityMode(File source, File destination) throws IOException {
    if (source.isDirectory()) {
        copyDirectory(source, destination);
    } else {
        copyFile(source, destination);
    }
} 
```

同样，**让我们看看如何使用 [`FileInputStream`](https://web.archive.org/web/20220627080539/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileInputStream.html) 和 [`FileOutputStream`](https://web.archive.org/web/20220627080539/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileOutputStream.html)** 来复制一个文件:

```java
private static void copyFile(File sourceFile, File destinationFile) 
  throws IOException {
    try (InputStream in = new FileInputStream(sourceFile); 
      OutputStream out = new FileOutputStream(destinationFile)) {
        byte[] buf = new byte[1024];
        int length;
        while ((length = in.read(buf)) > 0) {
            out.write(buf, 0, length);
        }
    }
} 
```

## 4.使用 Apache Commons IO

[Apache Commons IO](https://web.archive.org/web/20220627080539/https://search.maven.org/artifact/org.apache.commons/commons-io) 有很多有用的特性，比如实用程序类、文件过滤器和文件比较器**。**这里我们将使用`FileUtils`，它提供了简单的文件和目录操作方法，例如，读取、移动、复制。

让我们将 [commons-io](https://web.archive.org/web/20220627080539/https://search.maven.org/search?q=a:commons-io%20g:commons-io) 添加到我们的 `pom.xml`文件中:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

最后，让我们使用这种方法复制一个目录:

```java
public static void copyDirectory(String sourceDirectoryLocation, String destinationDirectoryLocation) throws IOException {
    File sourceDirectory = new File(sourceDirectoryLocation);
    File destinationDirectory = new File(destinationDirectoryLocation);
    FileUtils.copyDirectory(sourceDirectory, destinationDirectory);
}
```

如前面的例子所示，Apache Commons IO 使这一切变得更加容易，因为**我们只需要调用`FileUtils.copyDirectory()`方法**。

## 5.结论

本文演示了如何用 Java 复制一个目录。GitHub 上的[提供了完整的代码示例。](https://web.archive.org/web/20220627080539/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)