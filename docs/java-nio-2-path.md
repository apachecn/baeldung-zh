# Java NIO2 路径 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio-2-path>

## 1。概述

在本文中，我们将学习如何在 Java 中使用新的 I/O (NIO2) `Path` API。

NIO2 中的`Path`API 构成了 Java 7 附带的主要新功能领域之一，尤其是新文件系统 API 和文件 API 的子集。

## 2。设置

NIO2 支持捆绑在`java.nio.file`包中。所以设置您的项目来使用`Path`API 只是导入这个包中的所有东西:

```java
import java.nio.file.*;
```

因为本文中的代码示例可能会在不同的环境中运行，所以让我们来了解一下用户的主目录:

```java
private static String HOME = System.getProperty("user.home");
```

该变量将指向任何环境中的有效位置。

`Paths`类是所有涉及文件系统路径的操作的主要入口点。它允许我们创建和操作文件和目录的路径。

值得注意的是，路径操作本质上主要是语法操作；它们对底层文件系统没有影响，文件系统对它们的成功或失败也没有任何影响。这意味着将不存在的路径作为路径操作的参数进行传递与操作成功与否无关。

## 3。路径操作

在本节中，我们将介绍路径操作中使用的主要语法。顾名思义，`Path`类是文件系统中路径的编程表示。

一个`Path`对象包含用于构建路径的文件名和目录列表，并用于检查、定位和操作文件。

助手类`java.nio.file.Paths`(复数形式)是创建`Path`对象的正式方式。它有两个静态方法来从路径字符串创建一个`Path`:

```java
Path path = Paths.get("path string");
```

我们在路径`String,`中使用正斜杠还是反斜杠并不重要，API 根据底层文件系统的要求解析这个参数。

从一个`java.net.URI`物体:

```java
Path path = Paths.get(URI object);
```

我们现在可以继续看这些在行动。

## 4。创建路径

要**从路径字符串创建一个`Path`** 对象:

```java
@Test
public void givenPathString_whenCreatesPathObject_thenCorrect() {
    Path p = Paths.get("/articles/baeldung");

    assertEquals("\\articles\\baeldung", p.toString());
}
```

除了第一部分(在本例中为`articles`)之外，`get` API 还可以接受路径字符串部分(在本例中为`articles`和`baeldung`)的可变参数。

如果我们提供这些部分而不是完整的路径字符串，它们将用于构造路径对象，我们不需要在变量参数部分中包含名称分隔符(斜线):

```java
@Test
public void givenPathParts_whenCreatesPathObject_thenCorrect() {
    Path p = Paths.get("/articles", "baeldung");

    assertEquals("\\articles\\baeldung", p.toString());
}
```

## 5。正在检索路径信息

您可以将 Path 对象视为一个序列的名称元素。路径`String`如`E:\baeldung\articles\java`由三个名称元素组成，即`baeldung`、`articles`和`java`。目录结构中最高的元素将位于索引 0，在这种情况下是`baeldung`。

目录结构中最低的元素将位于索引`[n-1]`，其中`n`是路径中名称元素的数量。这个最低层的元素被称为**文件名**，不管它是否是一个实际的文件:

```java
@Test
public void givenPath_whenRetrievesFileName_thenCorrect() {
    Path p = Paths.get("/articles/baeldung/logs");

    Path fileName = p.getFileName();

    assertEquals("logs", fileName.toString());
}
```

方法可用于通过索引检索单个元素:

```java
@Test
public void givenPath_whenRetrievesNameByIndex_thenCorrect() {
    Path p = Paths.get("/articles/baeldung/logs");
    Path name0 = getName(0);
    Path name1 = getName(1);
    Path name2 = getName(2);
    assertEquals("articles", name0.toString());
    assertEquals("baeldung", name1.toString());
    assertEquals("logs", name2.toString());
}
```

或者使用这些索引范围的路径的子序列:

```java
@Test
public void givenPath_whenCanRetrieveSubsequenceByIndex_thenCorrect() {
    Path p = Paths.get("/articles/baeldung/logs");

    Path subPath1 = p.subpath(0,1);
    Path subPath2 = p.subpath(0,2);

    assertEquals("articles", subPath1.toString());
    assertEquals("articles\\baeldung", subPath2.toString());
    assertEquals("articles\\baeldung\\logs", p.subpath(0, 3).toString());
    assertEquals("baeldung", p.subpath(1, 2).toString());
    assertEquals("baeldung\\logs", p.subpath(1, 3).toString());
    assertEquals("logs", p.subpath(2, 3).toString());
}
```

每个路径都与一个父路径相关联，如果路径没有父路径，则与`null`相关联。path 对象的父对象由路径的根组件(如果有)和路径中除文件名之外的每个元素组成。例如，`/a/b/c`的父路径是`/a/b`，而`/a`的父路径为空:

```java
@Test
public void givenPath_whenRetrievesParent_thenCorrect() {
    Path p1 = Paths.get("/articles/baeldung/logs");
    Path p2 = Paths.get("/articles/baeldung");
    Path p3 = Paths.get("/articles");
    Path p4 = Paths.get("/");

    Path parent1 = p1.getParent();
    Path parent2 = p2.getParent();
    Path parent3 = p3.getParent();
    Path parent4 = p4.getParenth();

    assertEquals("\\articles\\baeldung", parent1.toString());
    assertEquals("\\articles", parent2.toString());
    assertEquals("\\", parent3.toString());
    assertEquals(null, parent4);
}
```

我们还可以获得路径的根元素:

```java
@Test
public void givenPath_whenRetrievesRoot_thenCorrect() {
    Path p1 = Paths.get("/articles/baeldung/logs");
    Path p2 = Paths.get("c:/articles/baeldung/logs");

    Path root1 = p1.getRoot();
    Path root2 = p2.getRoot();

    assertEquals("\\", root1.toString());
    assertEquals("c:\\", root2.toString());
}
```

## 6。标准化路径

许多文件系统使用`“.”`符号表示当前目录，使用 `“..”`表示父目录。您可能会遇到路径包含冗余目录信息的情况。

例如，考虑以下路径字符串:

```java
/baeldung/./articles
/baeldung/authors/../articles
/baeldung/articles
```

它们都解析到同一个位置`/baeldung/articles`。前两个有冗余，而最后一个没有。

规范化路径包括移除路径中的冗余。为此目的提供了`Path.normalize()`操作。

这个例子现在应该是不言自明的了:

```java
@Test
public void givenPath_whenRemovesRedundancies_thenCorrect1() {
    Path p = Paths.get("/home/./baeldung/articles");

    Path cleanPath = p.normalize();

    assertEquals("\\home\\baeldung\\articles", cleanPath.toString());
}
```

这个也是:

```java
@Test
public void givenPath_whenRemovesRedundancies_thenCorrect2() {
    Path p = Paths.get("/home/baeldung/../articles");

    Path cleanPath = p.normalize();

    assertEquals("\\home\\articles", cleanPath.toString());
}
```

## 7。路径转换

有一些操作可以将路径转换为所选的表示格式。为了将任何路径转换成可以从浏览器打开的字符串，我们使用了`toUri`方法:

```java
@Test
public void givenPath_whenConvertsToBrowseablePath_thenCorrect() {
    Path p = Paths.get("/home/baeldung/articles.html");

    URI uri = p.toUri();
    assertEquals(
      "file:///E:/home/baeldung/articles.html", 
        uri.toString());
}
```

我们也可以将路径转换成它的绝对表示。`toAbsolutePath`方法解析文件系统默认目录的路径:

```java
@Test
public void givenPath_whenConvertsToAbsolutePath_thenCorrect() {
    Path p = Paths.get("/home/baeldung/articles.html");

    Path absPath = p.toAbsolutePath();

    assertEquals(
      "E:\\home\\baeldung\\articles.html", 
        absPath.toString());
}
```

但是，当检测到要解析的路径已经是绝对路径时，该方法按原样返回它:

```java
@Test
public void givenAbsolutePath_whenRetainsAsAbsolute_thenCorrect() {
    Path p = Paths.get("E:\\home\\baeldung\\articles.html");

    Path absPath = p.toAbsolutePath();

    assertEquals(
      "E:\\home\\baeldung\\articles.html", 
        absPath.toString());
}
```

我们还可以通过调用`toRealPath`方法将任何路径转换成它的实际等价物。该方法试图通过将路径的元素映射到文件系统中的实际目录和文件来解析路径。

是时候使用我们在**设置**部分创建的变量了，该变量指向登录用户在文件系统中的主位置:

```java
@Test
public void givenExistingPath_whenGetsRealPathToFile_thenCorrect() {
    Path p = Paths.get(HOME);

    Path realPath = p.toRealPath();

    assertEquals(HOME, realPath.toString());
}
```

上面的测试并没有真正告诉我们这个操作的行为。最明显的结果是，如果文件系统中不存在该路径，那么操作将抛出一个`IOException`，继续读取。

由于没有更好的方法来证明这一点，只需看看下一个测试，它试图将一个不存在的路径转换为一个真实的路径:

```java
@Test(expected = NoSuchFileException.class)
public void givenInExistentPath_whenFailsToConvert_thenCorrect() {
    Path p = Paths.get("E:\\home\\baeldung\\articles.html");

    p.toRealPath();
}
```

当我们捕捉到一个`IOException`时，测试成功。该操作抛出的`IOException`的实际子类是`NoSuchFileException`。

## 8。加入路径

使用`resolve`方法可以连接任意两条路径。

简单地说，我们可以在任何一个`Path`上调用`resolve`方法，并传入一个`partial path`作为参数。该部分路径会附加到原始路径:

```java
@Test
public void givenTwoPaths_whenJoinsAndResolves_thenCorrect() {
    Path p = Paths.get("/baeldung/articles");

    Path p2 = p.resolve("java");

    assertEquals("\\baeldung\\articles\\java", p2.toString());
}
```

然而，当传递给`resolve`方法的路径字符串不是一个`partial path;` 时，最明显的是一个绝对路径，那么传入的路径被返回:

```java
@Test
public void givenAbsolutePath_whenResolutionRetainsIt_thenCorrect() {
    Path p = Paths.get("/baeldung/articles");

    Path p2 = p.resolve("C:\\baeldung\\articles\java");

    assertEquals("C:\\baeldung\\articles\\java", p2.toString());
}
```

任何有根元素的路径都会发生同样的事情。路径字符串`“java”`没有根元素，而路径字符串`“/java”`有根元素。因此，当您传入带有根元素的路径时，它将按原样返回:

```java
@Test
public void givenPathWithRoot_whenResolutionRetainsIt_thenCorrect2() {
    Path p = Paths.get("/baeldung/articles");

    Path p2 = p.resolve("/java");

    assertEquals("\\java", p2.toString());
}
```

## 9。 `Relativizing`小路

术语`relativizing`仅仅意味着在两条已知路径之间创建一条直接路径。例如，如果我们有一个目录`/baeldung`，在它里面，我们有另外两个目录，这样`/baeldung/authors`和`/baeldung/articles`就是有效路径。

相对于`authors`到`articles`的路径将被描述为`“move one level up in the directory hierarchy then into articles directory”`或`..\articles:`

```java
@Test
public void givenSiblingPaths_whenCreatesPathToOther_thenCorrect() {
    Path p1 = Paths.get("articles");
    Path p2 = Paths.get("authors");

    Path p1_rel_p2 = p1.relativize(p2);
    Path p2_rel_p1 = p2.relativize(p1);

    assertEquals("..\\authors", p1_rel_p2.toString());
    assertEquals("..\\articles", p2_rel_p1.toString());
}
```

假设我们将`articles`目录移动到`authors`文件夹，这样它们就不再是兄弟了。以下相对化操作包括在`baeldung`和`articles`之间创建一条路径，反之亦然:

```java
@Test
public void givenNonSiblingPaths_whenCreatesPathToOther_thenCorrect() {
    Path p1 = Paths.get("/baeldung");
    Path p2 = Paths.get("/baeldung/authors/articles");

    Path p1_rel_p2 = p1.relativize(p2);
    Path p2_rel_p1 = p2.relativize(p1);

    assertEquals("authors\\articles", p1_rel_p2.toString());
    assertEquals("..\\..", p2_rel_p1.toString());
}
```

## 10。比较路径

`Path`类有一个`equals`方法的直观实现，它使我们能够比较两条路径是否相等:

```java
@Test
public void givenTwoPaths_whenTestsEquality_thenCorrect() {
    Path p1 = Paths.get("/baeldung/articles");
    Path p2 = Paths.get("/baeldung/articles");
    Path p3 = Paths.get("/baeldung/authors");

    assertTrue(p1.equals(p2));
    assertFalse(p1.equals(p3));
}
```

您还可以检查路径是否以给定的字符串开头:

```java
@Test
public void givenPath_whenInspectsStart_thenCorrect() {
    Path p1 = Paths.get("/baeldung/articles");

    assertTrue(p1.startsWith("/baeldung"));
}
```

或者以其他字符串结尾:

```java
@Test
public void givenPath_whenInspectsEnd_thenCorrect() {
    Path p1 = Paths.get("/baeldung/articles");

    assertTrue(p1.endsWith("articles"));
}
```

## 11。结论

在本文中，我们展示了作为 Java 7 的一部分提供的新文件系统 API (NIO2)中的路径操作，并看到了其中的大部分操作。

本文中使用的代码示例可以在文章的 [Github 项目](https://web.archive.org/web/20220629010244/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)中找到。