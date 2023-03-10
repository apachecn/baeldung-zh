# 在 Java 中获取不带扩展名的文件名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filename-without-extension>

## 1.概观

当我们在 Java 中处理文件时，我们经常需要处理文件名。例如，有时我们想从给定的文件名中获取不带扩展名的名称。换句话说，我们想要删除文件的扩展名。

在本教程中，我们将讨论从文件名中删除扩展名的一般方法。

## 2.从文件名中删除扩展名的场景

当我们第一次看到它时，我们可能认为从文件名中删除扩展名是一个非常简单的问题。

然而，如果我们仔细看看这个问题，它可能比我们想象的更复杂。

首先，让我们看看文件名可以有哪些类型:

*   没有任何扩展名，例如，“`baeldung”`
*   对于单个扩展名，这是最常见的情况，例如，“`baeldung.txt`”
*   带有多个扩展名，如“`baeldung.tar.gz`”
*   不带扩展名的点文件，如“`.baeldung`”
*   具有单个扩展名的点文件，例如，“`.baeldung.conf`”
*   具有多个扩展名的点文件，例如，“`.baeldung.conf.bak`”

接下来，我们将列出删除扩展后上述示例的预期结果:

*   "`baeldung`":文件名没有扩展名。因此，文件名不应该更改，我们应该得到“`baeldung`”
*   这是一个简单的例子。正确的结果是“`baeldung`”
*   "`baeldung.tar.gz`":此文件名包含两个扩展名。如果我们只想删除一个扩展名，结果应该是“`baeldung.tar`”。但是如果我们想删除文件名中的所有扩展名，“T2”是正确的结果
*   "`.baeldung`":由于该文件名也没有任何扩展名，因此该文件名也不应更改。因此，我们期望在结果中看到“`.baeldung`
*   "`.baeldung.conf`":结果应该是"【T1 " "
*   "`.baeldung.conf.bak`":如果我们只想删除一个扩展名，结果应该是" . baeldung.conf "。否则，如果我们移除所有扩展名，那么“`.baeldung`”就是预期的输出

在本教程中，我们将测试 Guava 和 Apache Commons IO 提供的实用程序方法是否可以处理上面列出的所有情况。

此外，我们还将讨论一种通用方法来解决从给定文件名中删除扩展名的问题。

## 3.测试番石榴库

从 14.0 版本开始，[芭乐](https://web.archive.org/web/20220707143820/https://github.com/google/guava)引入了`[Files.getNameWithoutExtension()](https://web.archive.org/web/20220707143820/https://guava.dev/releases/30.0-jre/api/docs/com/google/common/io/Files.html#getNameWithoutExtension(java.lang.String))`的方法。它允许我们轻松地删除给定文件名的扩展名。

要使用实用程序方法，我们需要将 Guava 库添加到我们的类路径中。例如，如果我们使用 Maven 作为构建工具，我们可以将[番石榴依赖项](https://web.archive.org/web/20220707143820/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency> 
```

首先，让我们看看这个方法的实现:

```java
public static String getNameWithoutExtension(String file) {
   ...
   int dotIndex = fileName.lastIndexOf('.');
   return (dotIndex == -1) ? fileName : fileName.substring(0, dotIndex);
 }
```

实现非常简单。如果文件名包含点，该方法将从最后一个点开始剪切到文件名的末尾。否则，如果文件名不包含点，将返回原始文件名，不做任何更改。

因此， **Guava 的`getNameWithoutExtension() `方法对没有扩展名的 dotfiles 不起作用。**让我们写一个测试来证明:

```java
@Test
public void givenDotFileWithoutExt_whenCallGuavaMethod_thenCannotGetDesiredResult() {
    //negative assertion
    assertNotEquals(".baeldung", Files.getNameWithoutExtension(".baeldung"));
} 
```

当我们处理一个有多个扩展名的文件名时，**这个方法不提供从文件名中删除所有扩展名的选项:**

```java
@Test
public void givenFileWithoutMultipleExt_whenCallGuavaMethod_thenCannotRemoveAllExtensions() {
    //negative assertion
    assertNotEquals("baeldung", Files.getNameWithoutExtension("baeldung.tar.gz"));
} 
```

## 4.测试 Apache Commons IO 库

像番石榴库一样，流行的 [Apache Commons IO](https://web.archive.org/web/20220707143820/https://commons.apache.org/proper/commons-io/) 库在`FilenameUtils`类中提供了一个`[removeExtension()](https://web.archive.org/web/20220707143820/https://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/FilenameUtils.html#removeExtension-java.lang.String-)`方法来快速删除文件名的扩展名。

在我们看这个方法之前，让我们将 [Apache Commons IO 依赖关系](https://web.archive.org/web/20220707143820/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22)添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.8.0</version>
</dependency> 
```

实现类似于 Guava 的`getNameWithoutExtension()`方法:

```java
public static String removeExtension(final String filename) {
    ...
    final int index = indexOfExtension(filename); //used the String.lastIndexOf() method
    if (index == NOT_FOUND) {
  	return filename;
    } else {
	return filename.substring(0, index);
    }
} 
```

因此，**Apache Commons IO 的方法也不能用于点文件**:

```java
@Test
public void givenDotFileWithoutExt_whenCallApacheCommonsMethod_thenCannotGetDesiredResult() {
    //negative assertion
    assertNotEquals(".baeldung", FilenameUtils.removeExtension(".baeldung"));
} 
```

**如果一个文件名有多个扩展名，`removeExtension()`方法不能删除所有扩展名:**

```java
@Test
public void givenFileWithoutMultipleExt_whenCallApacheCommonsMethod_thenCannotRemoveAllExtensions() {
    //negative assertion
    assertNotEquals("baeldung", FilenameUtils.removeExtension("baeldung.tar.gz"));
} 
```

## 5.从文件名中删除扩展名

到目前为止，我们已经看到了两个广泛使用的库中删除文件名扩展名的实用方法。这两种方法都非常方便，适用于大多数常见的情况。

然而，另一方面，它们也有一些缺点:

*   它们不适用于点文件，例如，“`.baeldung`”
*   当文件名有多个扩展名时，它们不提供仅删除最后一个扩展名或删除所有扩展名的选项

接下来，让我们构建一个涵盖所有情况的方法:

```java
public static String removeFileExtension(String filename, boolean removeAllExtensions) {
    if (filename == null || filename.isEmpty()) {
        return filename;
    }

    String extPattern = "(?<!^)[.]" + (removeAllExtensions ? ".*" : "[^.]*$");
    return filename.replaceAll(extPattern, "");
} 
```

我们添加了一个`boolean`参数`removeAllExtensions`,以提供从文件名中删除所有扩展名或仅删除最后一个扩展名的选项。

这种方法的核心部分是 [`regex`](/web/20220707143820/https://www.baeldung.com/regular-expressions-java) 模式。那么让我们来理解这个`regex`模式是做什么的:

*   `“(?<!^)[.]”`–我们在这个`regex`中使用了一个[负向回顾](https://web.archive.org/web/20220707143820/https://www.regular-expressions.info/lookaround.html)。它匹配一个不在文件名开头的点“`.`
*   "`(?<!^)[.].*`"–如果设置了`removeAllExtensions`选项，它将匹配第一个匹配的点，直到文件名结束
*   "`(?<!^)[.][^.]*$`"–此模式仅匹配最后一个扩展名

最后，让我们编写一些测试方法来验证我们的方法是否适用于所有不同的情况:

```java
@Test
public void givenFilenameNoExt_whenCallFilenameUtilMethod_thenGetExpectedFilename() {
    assertEquals("baeldung", MyFilenameUtil.removeFileExtension("baeldung", true));
    assertEquals("baeldung", MyFilenameUtil.removeFileExtension("baeldung", false));
}

@Test
public void givenSingleExt_whenCallFilenameUtilMethod_thenGetExpectedFilename() {
    assertEquals("baeldung", MyFilenameUtil.removeFileExtension("baeldung.txt", true));
    assertEquals("baeldung", MyFilenameUtil.removeFileExtension("baeldung.txt", false));
}

@Test
public void givenDotFile_whenCallFilenameUtilMethod_thenGetExpectedFilename() {
    assertEquals(".baeldung", MyFilenameUtil.removeFileExtension(".baeldung", true));
    assertEquals(".baeldung", MyFilenameUtil.removeFileExtension(".baeldung", false));
}

@Test
public void givenDotFileWithExt_whenCallFilenameUtilMethod_thenGetExpectedFilename() {
    assertEquals(".baeldung", MyFilenameUtil.removeFileExtension(".baeldung.conf", true));
    assertEquals(".baeldung", MyFilenameUtil.removeFileExtension(".baeldung.conf", false));
}

@Test
public void givenDoubleExt_whenCallFilenameUtilMethod_thenGetExpectedFilename() {
    assertEquals("baeldung", MyFilenameUtil.removeFileExtension("baeldung.tar.gz", true));
    assertEquals("baeldung.tar", MyFilenameUtil.removeFileExtension("baeldung.tar.gz", false));
}

@Test
public void givenDotFileWithDoubleExt_whenCallFilenameUtilMethod_thenGetExpectedFilename() {
    assertEquals(".baeldung", MyFilenameUtil.removeFileExtension(".baeldung.conf.bak", true));
    assertEquals(".baeldung.conf", MyFilenameUtil.removeFileExtension(".baeldung.conf.bak", false));
} 
```

## 6.结论

在本文中，我们讨论了如何删除给定文件名的扩展名。

首先，我们讨论了删除扩展的不同场景。

接下来，我们介绍了两个广泛使用的库提供的方法:Guava 和 Apache Commons IO。它们非常方便，适用于常见情况，但不适用于点文件。此外，它们不提供删除单个扩展或所有扩展的选项。

最后，我们构建了一个覆盖所有需求的方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220707143820/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)