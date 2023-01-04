# 如何用 Java 高效读取大文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-lines-large-file>

## 1。概述

本教程将向**展示如何以高效的方式从 Java** 的一个大文件中读取所有的行。

这篇文章是贝尔登上的[`**Java – Back to Basic**`教程的一部分。](/web/20221013193919/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")

## 延伸阅读:

## [Java–将输入流写入文件](/web/20221013193919/https://www.baeldung.com/convert-input-stream-to-a-file)

How to write an InputStream to a File - using Java, Guava and the Commons IO library.[Read more](/web/20221013193919/https://www.baeldung.com/convert-input-stream-to-a-file) →

## [Java–将文件转换为输入流](/web/20221013193919/https://www.baeldung.com/convert-file-to-input-stream)

How to open an InputStream from a Java File - using plain Java, Guava and the Apache Commons IO library.[Read more](/web/20221013193919/https://www.baeldung.com/convert-file-to-input-stream) →

## 2。在存储器中读取

读取文件行的标准方法是在内存中 Guava 和 Apache Commons IO 都提供了一种快速的方法:

```java
Files.readLines(new File(path), Charsets.UTF_8);
```

```java
FileUtils.readLines(new File(path));
```

这种方法的问题是所有的文件行都保存在内存中——如果文件足够大，这将很快导致`OutOfMemoryError`。

例如-**读取大约 1Gb 的文件**:

```java
@Test
public void givenUsingGuava_whenIteratingAFile_thenWorks() throws IOException {
    String path = ...
    Files.readLines(new File(path), Charsets.UTF_8);
}
```

这从消耗少量内存开始:`(~0 Mb consumed)`

```java
[main] INFO  org.baeldung.java.CoreJavaIoUnitTest - Total Memory: 128 Mb
[main] INFO  org.baeldung.java.CoreJavaIoUnitTest - Free Memory: 116 Mb
```

然而，**在整个文件被处理之后**，我们在最后:`(~2 Gb consumed)`

```java
[main] INFO  org.baeldung.java.CoreJavaIoUnitTest - Total Memory: 2666 Mb
[main] INFO  org.baeldung.java.CoreJavaIoUnitTest - Free Memory: 490 Mb
```

这意味着该进程消耗了大约 2.1 Gb 的内存，原因很简单，文件的所有行现在都存储在内存中。

很明显，**将文件内容保存在内存中会很快耗尽可用内存**——不管实际有多少。

更重要的是，**我们通常不需要一次将文件中的所有行都存储在内存中**——相反，我们只需要能够遍历每一行，进行一些处理并将其丢弃。所以，这正是我们要做的——遍历这些行，而不是将它们都保存在内存中。

## 3。流过文件

现在让我们来看一个解决方案——我们将使用一个`java.util.Scanner`遍历文件的内容，并一行一行地连续检索:

```java
FileInputStream inputStream = null;
Scanner sc = null;
try {
    inputStream = new FileInputStream(path);
    sc = new Scanner(inputStream, "UTF-8");
    while (sc.hasNextLine()) {
        String line = sc.nextLine();
        // System.out.println(line);
    }
    // note that Scanner suppresses exceptions
    if (sc.ioException() != null) {
        throw sc.ioException();
    }
} finally {
    if (inputStream != null) {
        inputStream.close();
    }
    if (sc != null) {
        sc.close();
    }
}
```

这个解决方案将遍历文件中的所有行——允许处理每一行——不保留对它们的引用——最后，**不将它们保存在内存中** : `(~150 Mb consumed)`

```java
[main] INFO  org.baeldung.java.CoreJavaIoUnitTest - Total Memory: 763 Mb
[main] INFO  org.baeldung.java.CoreJavaIoUnitTest - Free Memory: 605 Mb
```

## 4。使用 Apache Commons IO 进行流式传输

使用 Commons IO 库也可以达到同样的效果，通过使用库提供的**自定义`LineIterator`** :

```java
LineIterator it = FileUtils.lineIterator(theFile, "UTF-8");
try {
    while (it.hasNext()) {
        String line = it.nextLine();
        // do something with line
    }
} finally {
    LineIterator.closeQuietly(it);
}
```

因为整个文件没有完全在内存中——这也会导致**非常保守的内存消耗数字** : `(~150 Mb consumed)`

```java
[main] INFO  o.b.java.CoreJavaIoIntegrationTest - Total Memory: 752 Mb
[main] INFO  o.b.java.CoreJavaIoIntegrationTest - Free Memory: 564 Mb
```

## 5。结论

这篇简短的文章展示了如何**处理一个大文件中的行，而不需要迭代，也不需要耗尽可用的内存**——这在处理这些大文件时非常有用。

所有这些例子和代码片段**的实现可以在我们的 [GitHub 项目](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2 "Example of processing lines in a large file efficiently")** 中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。