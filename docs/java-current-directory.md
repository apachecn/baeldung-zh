# 获取 Java 中的当前工作目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-current-directory>

## 1.概观

在 Java 中获取当前工作目录是一件容易的事情，但不幸的是，JDK 中没有直接可用的 API 来做这件事。

在本教程中，我们将学习如何用`java.lang.` `System`、`java.io.File, java.nio.file.FileSystems,` 和 `java.nio.file.Paths`在 Java 中获取当前工作目录。

## 2.`System`

让我们从使用`System#getProperty`的标准解决方案开始，假设我们当前的工作目录名在整个代码中都是`Baeldung`:

```java
static final String CURRENT_DIR = "Baeldung";
@Test
void whenUsingSystemProperties_thenReturnCurrentDirectory() {
    String userDirectory = System.getProperty("user.dir");
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

我们使用 Java 内置属性键`user.dir`从`System`的属性映射中获取当前工作目录。该解决方案适用于所有 JDK 版本。

## 3.`File`

让我们看看另一个使用`java.io.File`的解决方案:

```java
@Test
void whenUsingJavaIoFile_thenReturnCurrentDirectory() {
    String userDirectory = new File("").getAbsolutePath();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

这里，`File#getAbsolutePath`在内部使用`System#getProperty`来获取目录名，类似于我们的第一个解决方案。这是一个获取当前工作目录的非标准解决方案，适用于所有 JDK 版本。

## 4.`FileSystems`

另一个有效的替代方法是使用新的`java.nio.file.FileSystems` API:

```java
@Test
void whenUsingJavaNioFileSystems_thenReturnCurrentDirectory() {
    String userDirectory = FileSystems.getDefault()
        .getPath("")
        .toAbsolutePath()
        .toString();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

这个解决方案使用新的 [Java NIO API](/web/20220817103131/https://www.baeldung.com/java-nio-2-file-api) ，并且它只适用于 **JDK 7 或更高版本**。

## 5.`Paths`

最后，让我们看一个更简单的解决方案，使用`java.nio.file.Paths` API 获取当前目录:

```java
@Test
void whenUsingJavaNioPaths_thenReturnCurrentDirectory() {
    String userDirectory = Paths.get("")
        .toAbsolutePath()
        .toString();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

这里， `Paths#get`在内部使用`FileSystem#getPath`来获取路径。它使用新的 [Java NIO API](/web/20220817103131/https://www.baeldung.com/java-nio-2-file-api) ，所以这个解决方案只适用于 **JDK 7 或更高版本**。

## 6.结论

在本教程中，我们探讨了在 Java 中获取当前工作目录的四种不同方法。前两种解决方案适用于所有版本的 JDK，而后两种仅适用于 JDK `7`或更高版本。

我们推荐使用`System`解决方案，因为它高效且直接，我们可以通过将这个 API 调用封装在一个静态实用方法中并直接访问它来简化它。

本教程的源代码可以在 GitHub 上找到[——这是一个基于 Maven 的项目，所以应该很容易导入和运行。](https://web.archive.org/web/20220817103131/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)