# 用 Java 中的 Grep 进行模式搜索

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/grep-in-java>

## 1。概述

在本教程中，我们将学习如何使用 Java 和第三方库，如 [Unix4J](https://web.archive.org/web/20220815040658/https://github.com/tools4j/unix4j) 和 [Grep4J](https://web.archive.org/web/20220815040658/https://code.google.com/archive/p/grep4j/) ，在给定的文件中**搜索模式。**

## 2。背景

Unix 有一个强大的命令叫做`grep`——代表“ `global regular expression print` ”。它在给定的一组文件中搜索模式或正则表达式。

可以使用零个或多个选项以及 grep 命令来丰富搜索结果，我们将在接下来的部分中详细讨论。

如果你使用的是 Windows，你可以在这里安装 bash。

## 3。带 Unix4j 库

首先，让我们看看如何使用 Unix4J 库来 grep 文件中的模式。

在下面的例子中，我们将看看如何用 Java 翻译 Unix grep 命令。

### 3.1。构建配置

在您的`pom.xml`或`build.gradle`上添加以下依赖项:

```java
<dependency>
    <groupId>org.unix4j</groupId>
    <artifactId>unix4j-command</artifactId>
    <version>0.4</version>
</dependency>
```

### 3.2。Grep 的例子

Unix 中的 grep 示例:

```java
grep "NINETEEN" dictionary.txt 
```

Java 中的对等用法是:

```java
@Test 
public void whenGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 4;
    File file = new File("dictionary.txt");
    List<Line> lines = Unix4j.grep("NINETEEN", file).toLineList(); 

    assertEquals(expectedLineCount, lines.size());
} 
```

另一个例子是我们可以在文件中使用逆向文本搜索。下面是相同的 Unix 版本:

```java
grep -v "NINETEEN" dictionary.txt 
```

以下是上述命令的 Java 版本:

```java
@Test
public void whenInverseGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 178687;
    File file = new File("dictionary.txt");
    List<Line> lines 
      = Unix4j.grep(Grep.Options.v, "NINETEEN", file). toLineList();

    assertEquals(expectedLineCount, lines.size()); 
} 
```

让我们看看，如何使用正则表达式在文件中搜索模式。下面是计算整个文件中所有正则表达式模式的 Unix 版本:

```java
grep -c ".*?NINE.*?" dictionary.txt 
```

以下是上述命令的 Java 版本:

```java
@Test
public void whenGrepWithRegex_thenCorrect() {
    int expectedLineCount = 151;
    File file = new File("dictionary.txt");
    String patternCount = Unix4j.grep(Grep.Options.c, ".*?NINE.*?", file).
                          cut(CutOption.fields, ":", 1).toStringResult();

    assertEquals(expectedLineCount, patternCount); 
}
```

## 4。带 Grep4J

接下来——让我们看看如何使用 Grep4J 库来 Grep 驻留在本地或远程位置的文件中的模式。

在下面的例子中，我们将看看如何用 Java 翻译 Unix grep 命令。

### 4.1。构建配置

在您的`pom.xml`或`build.gradle`上添加以下依赖项:

```java
<dependency>
    <groupId>com.googlecode.grep4j</groupId>
    <artifactId>grep4j</artifactId>
    <version>1.8.7</version>
</dependency>
```

### 4.2。Grep 示例

Java 中的 grep 示例，相当于:

```java
grep "NINETEEN" dictionary.txt 
```

下面是 Java 版本的命令:

```java
@Test 
public void givenLocalFile_whenGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 4;
    Profile localProfile = ProfileBuilder.newBuilder().
                           name("dictionary.txt").filePath(".").
                           onLocalhost().build();
    GrepResults results 
      = Grep4j.grep(Grep4j.constantExpression("NINETEEN"), localProfile);

    assertEquals(expectedLineCount, results.totalLines());
} 
```

另一个例子是我们可以在文件中使用逆向文本搜索。下面是相同的 Unix 版本:

```java
grep -v "NINETEEN" dictionary.txt 
```

这是 Java 版本:

```java
@Test
public void givenRemoteFile_whenInverseGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 178687;
    Profile remoteProfile = ProfileBuilder.newBuilder().
                            name("dictionary.txt").filePath(".").
                            filePath("/tmp/dictionary.txt").
                            onRemotehost("172.168.192.1").
                            credentials("user", "pass").build();
    GrepResults results = Grep4j.grep(
      Grep4j.constantExpression("NINETEEN"), remoteProfile, Option.invertMatch());

    assertEquals(expectedLineCount, results.totalLines()); 
} 
```

让我们看看，如何使用正则表达式在文件中搜索模式。下面是计算整个文件中所有正则表达式模式的 Unix 版本:

```java
grep -c ".*?NINE.*?" dictionary.txt 
```

下面是 Java 版本:

```java
@Test
public void givenLocalFile_whenGrepWithRegex_thenCorrect() {
    int expectedLineCount = 151;
    Profile localProfile = ProfileBuilder.newBuilder().
                           name("dictionary.txt").filePath(".").
                           onLocalhost().build();
    GrepResults results = Grep4j.grep(
      Grep4j.regularExpression(".*?NINE.*?"), localProfile, Option.countMatches());

    assertEquals(expectedLineCount, results.totalLines()); 
}
```

## 5。结论

在这个快速教程中，我们展示了使用 **Grep4j** 和 **Unix4J** 在给定文件中搜索模式。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行。

最后，您也可以使用 JDK 中的 regex 功能自然地完成一些类似 grep 的基本功能。