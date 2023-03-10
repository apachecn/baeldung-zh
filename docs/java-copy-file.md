# 如何用 Java 复制文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-file>

## 1。概述

在本文中，我们将介绍在 Java 中复制文件的常用方法。

首先，我们将使用标准的`IO`和`NIO.2`API，以及两个外部库: [commons-io](https://web.archive.org/web/20220626204937/https://commons.apache.org/proper/commons-io/) 和 [guava](https://web.archive.org/web/20220626204937/https://github.com/google/guava) 。

## 2。`IO`API(JDK 7 之前)

首先，**到** **用`java.io` API 复制一个文件，我们需要打开一个流，循环遍历内容并写出到另一个流:**

```java
@Test
public void givenIoAPI_whenCopied_thenCopyExistsWithSameContents() 
  throws IOException {

    File copied = new File("src/test/resources/copiedWithIo.txt");
    try (
      InputStream in = new BufferedInputStream(
        new FileInputStream(original));
      OutputStream out = new BufferedOutputStream(
        new FileOutputStream(copied))) {

        byte[] buffer = new byte[1024];
        int lengthRead;
        while ((lengthRead = in.read(buffer)) > 0) {
            out.write(buffer, 0, lengthRead);
            out.flush();
        }
    }

    assertThat(copied).exists();
    assertThat(Files.readAllLines(original.toPath())
      .equals(Files.readAllLines(copied.toPath())));
}
```

实现这样的基本功能需要做大量的工作。

对我们来说幸运的是， **Java 已经改进了它的核心 API，我们有了一种更简单的使用`NIO.2` API** 复制文件的方法。

## 3。`NIO.2` API (JDK7)

**使用 [`NIO.2`](/web/20220626204937/https://www.baeldung.com/java-nio-2-file-api) 可以显著提高文件复制性能，因为`NIO.2` 利用了较低级别的系统入口点。**

让我们仔细看看这些文件。`copy()`方法奏效。

`copy()`方法让我们能够指定一个表示复制选项的可选参数。**默认情况下，复制文件和目录不会覆盖现有文件和目录，也不会复制文件属性。**

可以使用以下复制选项来更改此行为:

*   如果文件存在，替换它
*   `COPY_ATTRIBUTES –`将元数据复制到新文件
*   *no follow _ LINKS—*不应该跟随符号链接

`NIO.2 Files` 类提供了一组重载的`copy()`方法，用于在文件系统中复制文件和目录。

让我们看一个使用带有两个`Path`参数的`copy()`的例子:

```java
@Test
public void givenNIO2_whenCopied_thenCopyExistsWithSameContents() 
  throws IOException {

    Path copied = Paths.get("src/test/resources/copiedWithNio.txt");
    Path originalPath = original.toPath();
    Files.copy(originalPath, copied, StandardCopyOption.REPLACE_EXISTING);

    assertThat(copied).exists();
    assertThat(Files.readAllLines(originalPath)
      .equals(Files.readAllLines(copied)));
}
```

请注意，**目录副本是浅层的**，这意味着目录中的文件和子目录不会被复制。

## 4 .Apache common me〔t1〕

用 Java 复制文件的另一种常见方法是使用`commons-io`库。

首先，我们需要添加依赖关系:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

最新版本可以从 [Maven Central](https://web.archive.org/web/20220626204937/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22) 下载。

然后，要复制一个文件，我们只需要使用`FileUtils`类中定义的 **`copyFile()` 方法。**该方法接受一个源文件和一个目标文件。

让我们看看使用`copyFile()`方法的 JUnit 测试:

```java
@Test
public void givenCommonsIoAPI_whenCopied_thenCopyExistsWithSameContents() 
  throws IOException {

    File copied = new File(
      "src/test/resources/copiedWithApacheCommons.txt");
    FileUtils.copyFile(original, copied);

    assertThat(copied).exists();
    assertThat(Files.readAllLines(original.toPath())
      .equals(Files.readAllLines(copied.toPath())));
}
```

## 5。番石榴

最后，我们来看看谷歌的番石榴库。

同样，如果我们想使用番石榴`,` ,我们需要包括依赖性:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

最新版本可以在 Maven Central 上找到[。](https://web.archive.org/web/20220626204937/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22guava%22%20AND%20g%3A%22com.google.guava%22)

这是番石榴复制文件的方法:

```java
@Test
public void givenGuava_whenCopied_thenCopyExistsWithSameContents() 
  throws IOException {

    File copied = new File("src/test/resources/copiedWithGuava.txt");
    com.google.common.io.Files.copy(original, copied);

    assertThat(copied).exists();
    assertThat(Files.readAllLines(original.toPath())
      .equals(Files.readAllLines(copied.toPath())));
}
```

## 6。结论

在本文中，我们探索了用 Java 复制文件的最常见的方法。

这篇文章的完整实现可以在 Github 上找到。