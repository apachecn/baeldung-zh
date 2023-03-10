# Hamcrest 文件匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-file-matchers>

## 1。概述

在本教程中，我们将讨论 Hamcrest 文件匹配器。

我们在之前的用 Hamcrest 测试的文章中讨论了 Hamcrest 匹配器。在接下来的部分中，我们将只关注*文件*匹配器。

## 2。Maven 配置

首先，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
    <scope>test</scope>
</dependency>
```

[`java-hamcrest`](https://web.archive.org/web/20221129012018/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22java-hamcrest%22) 的最新版本可以从 Maven Central 下载。

让我们继续探索 Hamcrest `File`匹配器。

## 3。文件属性

Hamcrest 提供了几个匹配器来验证常用的`File`属性。

让我们看看如何使用`aFileNamed()`和`String`匹配器来验证`File`名称:

```java
@Test
public void whenVerifyingFileName_thenCorrect() {
    File file = new File("src/test/resources/test1.in");

    assertThat(file, aFileNamed(equalToIgnoringCase("test1.in")));
}
```

我们还可以评估文件路径——再次结合使用`String`匹配器:

```java
@Test
public void whenVerifyingFilePath_thenCorrect() {
    File file = new File("src/test/resources/test1.in");

    assertThat(file, aFileWithCanonicalPath(containsString("src/test/resources")));
    assertThat(file, aFileWithAbsolutePath(containsString("src/test/resources")));
}
```

让我们看看文件的大小，以字节为单位:

```java
@Test
public void whenVerifyingFileSize_thenCorrect() {
    File file = new File("src/test/resources/test1.in");

    assertThat(file, aFileWithSize(11));
    assertThat(file, aFileWithSize(greaterThan(1L)));;
}
```

最后，我们可以检查一个`File`是否可读可写:

```java
@Test
public void whenVerifyingFileIsReadableAndWritable_thenCorrect() {
    File file = new File("src/test/resources/test1.in");

    assertThat(file, aReadableFile());
    assertThat(file, aWritableFile());        
}
```

## 4。现有文件匹配器

如果我们想验证一个`File`或目录是否存在，我们可以使用`anExistingFile()`或`anExistingDirectory()`匹配器:

```java
@Test
public void whenVerifyingFileOrDirExist_thenCorrect() {
    File file = new File("src/test/resources/test1.in");
    File dir = new File("src/test/resources");

    assertThat(file, anExistingFile());
    assertThat(dir, anExistingDirectory());
    assertThat(file, anExistingFileOrDirectory());
    assertThat(dir, anExistingFileOrDirectory());
}
```

结合两者的`anExistingFileOrDirectory()` matcher 也有。

## 5。结论

在这篇简短的文章中，我们介绍了 Hamcrest 文件匹配器及其用法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221129012018/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)