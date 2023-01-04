# 用 Java 创建临时目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-temp-directories>

## 1.概观

当我们需要创建一组以后可以丢弃的文件时，临时目录就派上了用场。当我们创建临时目录时，我们可以委托操作系统将它们放在哪里，或者指定我们自己想要将它们放在哪里。

在这个简短的教程中，我们将学习如何使用不同的 API 和方法在 Java 中创建临时目录。本教程中的所有示例都将使用普通 Java 7+、[番石榴](https://web.archive.org/web/20221128051525/https://search.maven.org/artifact/com.google.guava/guava/29.0-jre/bundle)和 [Apache Commons IO](https://web.archive.org/web/20221128051525/https://search.maven.org/artifact/org.checkerframework.annotatedlib/commons-io/2.7/jar) 来执行。

## 2.委托给操作系统

用于创建临时目录的最流行的方法之一是将目标委托给底层操作系统。**位置由`java.io.tmpdir`属性给出，每个操作系统都有自己的结构和清理例程。**

在普通 Java 中，我们通过指定希望目录采用的前缀来创建目录:

```java
String tmpdir = Files.createTempDirectory("tmpDirPrefix").toFile().getAbsolutePath();
String tmpDirsLocation = System.getProperty("java.io.tmpdir");
assertThat(tmpdir).startsWith(tmpDirsLocation);
```

使用 Guava，过程是相似的，但是我们不能指定我们想要如何前缀我们的目录:

```java
String tmpdir = Files.createTempDir().getAbsolutePath();
String tmpDirsLocation = System.getProperty("java.io.tmpdir");
assertThat(tmpdir).startsWith(tmpDirsLocation);
```

Apache Commons IO 没有提供创建临时目录的方法。它提供了一个包装器来获取操作系统临时目录，然后，剩下的工作就由我们来做了:

```java
String tmpDirsLocation = System.getProperty("java.io.tmpdir");
Path path = Paths.get(FileUtils.getTempDirectory().getAbsolutePath(), UUID.randomUUID().toString());
String tmpdir = Files.createDirectories(path).toFile().getAbsolutePath();
assertThat(tmpdir).startsWith(tmpDirsLocation);
```

为了避免与现有目录的名称冲突，我们使用`UUID.randomUUID()`来创建一个具有随机名称的目录。

## 3.指定位置

有时我们需要指定我们想要创建临时目录的位置。一个很好的例子是在 Maven 构建期间。因为我们已经有了一个“临时”构建目录，我们可以利用这个目录来放置我们的构建可能需要的临时目录:

```java
Path tmpdir = Files.createTempDirectory(Paths.get("target"), "tmpDirPrefix");
assertThat(tmpdir.toFile().getPath()).startsWith("target");
```

Guava 和 Apache Commons IO 都缺乏在特定位置创建临时目录的方法。

值得注意的是**`target`目录可能会有所不同，这取决于构建配置**。使它防弹的一种方法是将目标目录位置传递给运行测试的 JVM。

由于操作系统不负责清理，我们可以利用`File.deleteOnExit()`:

```java
tmpdir.toFile().deleteOnExit();
```

这样，**一旦 JVM 终止，文件就被删除，但前提是终止是优雅的**。

## 4.使用不同的文件属性

像任何其他文件或目录一样，可以在创建临时目录时指定文件属性。因此，如果我们想要创建一个只能由创建它的用户读取的临时目录，我们可以指定一组属性来完成这个任务:

```java
FileAttribute<Set> attrs = PosixFilePermissions.asFileAttribute(
  PosixFilePermissions.fromString("r--------"));
Path tmpdir = Files.createTempDirectory(Paths.get("target"), "tmpDirPrefix", attrs);
assertThat(tmpdir.toFile().getPath()).startsWith("target");
assertThat(tmpdir.toFile().canWrite()).isFalse();
```

正如所料，Guava 和 Apache Commons IO 在创建临时目录时没有提供指定属性的方法。

同样值得注意的是，前面的例子假设我们在一个 Posix 兼容的文件系统下，比如 Unix 或 macOS。

关于文件属性的更多信息可以在我们的 NIO2 文件属性 API 指南中找到。

## 5.结论

在这个简短的教程中，我们探讨了如何在普通 Java 7+、Guava 和 Apache Commons IO 中创建临时目录。我们看到普通 Java 是创建临时目录最灵活的方式，因为它提供了更广泛的可能性，同时保持了最小的冗长性。

像往常一样，本教程的所有源代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128051525/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)