# 用 Java 去除字符串中的空白

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-remove-whitespace>

## 1.概观

当我们在 Java 中操作`String`时，我们经常需要从`String`中移除空白。

在本教程中，我们将探索在 Java 中从`String`中移除空格的常见场景。

## 2.问题简介

为了更容易理解这个问题，我们先来看一个字符串示例:

```java
String myString = "   I    am a    wonderful String     !   ";
```

上面的例子显示了`myString`变量包含多个前导、尾随空格，以及中间的空白字符。

通常，当我们需要在 Java 中处理一个类似`myString`的字符串时，我们经常会面临这两个需求:

*   删除给定字符串中的所有空白字符-> `“IamawonderfulString!”`
*   用一个空格替换连续的空白字符，并删除所有前导和尾随的空白字符-> `“I am a wonderful String !”`

接下来，我们将针对每种情况提出两种方法:使用来自`String`类的便捷的`[replaceAll()](/web/20220906090139/https://www.baeldung.com/string/replace-all)`方法和来自广泛使用的 [Apache Commons Lang3](/web/20220906090139/https://www.baeldung.com/java-commons-lang-3) 库的`StringUtils`类。

为了简单起见，在本教程中，当我们谈论空白字符时，我们不包括 Unicode 字符集中的[空白字符。此外，我们将使用测试断言来验证每个解决方案。](https://web.archive.org/web/20220906090139/https://en.wikipedia.org/wiki/Template:Whitespace_(Unicode))

现在，让我们看看他们的行动。

## 3.删除字符串中的所有空格

### 3.1.使用`String.replaceAll()`

首先，让我们使用`replaceAll()`方法删除字符串中的所有空格。

`replaceAll()`使用[正则表达式](/web/20220906090139/https://www.baeldung.com/regular-expressions-java) (regex)。**我们可以使用正则表达式字符类“`\s`”来匹配一个空白字符。**我们可以用一个空字符串替换输入字符串中的每个空白字符来解决这个问题:`inputString.replaceAll(“\\s”, “”)`。

接下来，让我们创建一个测试，看看这个想法是否适用于我们的示例字符串:

```java
String result = myString.replaceAll("\\s", "");
assertThat(result).isEqualTo("IamawonderfulString!");
```

如果我们进行测试，就会通过。所以，`replaceAll() `方法解决了这个问题。接下来，让我们使用 Apache Commons Lang3 来解决这个问题。

### 3.2.使用 Apache Commons Lang3 库

Apache Commons Lang3 库附带了一个`[StringUtils](https://web.archive.org/web/20220906090139/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html)`实用程序，它允许我们方便地操作字符串。

为了开始使用 Apache Commons Lang 3，让我们添加 [Maven 依赖项](https://web.archive.org/web/20220906090139/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav):

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

如果我们检查`StringUtils`类中的方法，有一个方法叫做`deleteWhitespace()`。顾名思义，这就是我们正在寻找的方法。

接下来，让我们使用`StringUtils.deleteWhitespace()` 从字符串中移除所有空格:

```java
String result = StringUtils.deleteWhitespace(myString);
assertThat(result).isEqualTo("IamawonderfulString!");
```

如果我们执行它，测试就通过了。因此，`deleteWhitespace()`完成了这项工作。

## 4.用一个空格替换连续的空白字符

### 4.1.使用`String.replaceAll()`

现在，让我们看看另一个场景。我们可以分两步解决这个问题:

*   用一个空格替换连续的空格
*   修整第一步的结果

值得一提的是，我们也可以首先修整输入字符串，然后替换连续的空格。所以，先走哪一步并不重要。

对于第一步，我们仍然可以使用带有正则表达式的`replaceAll()`来匹配连续的空白字符，并设置一个空格作为替换。

**正则表达式“\s+”匹配一个或多个空白字符。因此，我们可以调用`replaceAll(“\\s+”, ” “)`方法来完成第一步**。然后，我们可以调用 [`String.trim()`](/web/20220906090139/https://www.baeldung.com/string/trim) 方法来应用修剪操作。

接下来，让我们创建一个测试来检查我们的想法是否能解决问题。为了清楚起见，我们为这两个步骤编写了两个断言:

```java
String result = myString.replaceAll("\\s+", " ");
assertThat(result).isEqualTo(" I am a wonderful String ! ");
assertThat(result.trim()).isEqualTo("I am a wonderful String !");
```

如果我们试一下，测试就通过了。因此，这种方法如预期的那样有效。

接下来，让我们使用 Apache Commons Lang 3 库来解决这个问题。

### 4.2.使用 Apache Commons Lang3 库

**`StringUtils.normalizeSpace()`方法修剪输入字符串，然后用一个空格替换空白字符序列。**因此，我们可以直接调用这种方法来解决问题:

```java
String result = StringUtils.normalizeSpace(myString);
assertThat(result).isEqualTo("I am a wonderful String !");
```

如果我们执行它，测试就通过了。正如我们所见，`StringUtils.normalizeSpace()`使用起来非常简单。

## 5.结论

在本文中，我们学习了如何在 Java 中删除字符串中的空白字符。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220906090139/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)