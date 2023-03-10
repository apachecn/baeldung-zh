# 从 Java 字符串中移除表情符号

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-remove-emojis>

## 1。概述

如今，表情符号在短信中变得越来越受欢迎——有时我们需要清除它们和其他符号。

在本教程中，我们将讨论在 Java 中删除表情符号的不同方法。

## 2。使用表情库

首先，我们将使用表情库从我们的`String`中移除表情。

我们将在下面的例子中使用`emoji-java`，所以我们需要将这个依赖关系添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.vdurmont</groupId>
    <artifactId>emoji-java</artifactId>
    <version>4.0.0</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20221205194342/https://search.maven.org/search?q=emoji-java)

现在让我们看看如何使用`emoji-java `从我们的`String`中删除表情符号:

```java
@Test
public void whenRemoveEmojiUsingLibrary_thenSuccess() {
    String text = "la conférence, commencera à 10 heures ?";
    String result = EmojiParser.removeAllEmojis(text);

    assertEquals(result, "la conférence, commencera à 10 heures ");
}
```

在这里，我们用**调用**的`removeAllEmojis()`方法

我们也可以使用`EmojiParser`通过`parseToAliases()`方法将表情符号替换为它的别名:

```java
@Test
public void whenReplaceEmojiUsingLibrary_thenSuccess() {
    String text = "la conférence, commencera à 10 heures ?";
    String result = EmojiParser.parseToAliases(text);

    assertEquals(
      result, 
      "la conférence, commencera à 10 heures :sweat_smile:");
}
```

注意，如果我们需要用他们的别名替换表情符号，使用这个库是非常有用的。

然而，表情符号-java 库只能检测表情符号，而不能检测符号或其他特殊字符。

## 3。使用正则表达式

接下来，我们可以使用正则表达式来删除表情符号和其他符号。我们将只允许特定类型的字符:

```java
@Test
public void whenRemoveEmojiUsingMatcher_thenSuccess() {
    String text = "la conférence, commencera à 10 heures ?";
    String regex = "[^\\p{L}\\p{N}\\p{P}\\p{Z}]";
    Pattern pattern = Pattern.compile(
      regex, 
      Pattern.UNICODE_CHARACTER_CLASS);
    Matcher matcher = pattern.matcher(text);
    String result = matcher.replaceAll("");

    assertEquals(result, "la conférence, commencera à 10 heures ");
}
```

让我们分解一下我们的正则表达式:

*   `\p{L}`–允许任何语言的所有字母
*   `\p{N}`–用于数字
*   `\p{P}`–用于标点符号
*   `\p{Z}`–用于空白分隔符
*   **^** 表示否定，因此所有这些表达式都将被列入白名单

这个表达式将只保留字母、数字、标点和空格。我们可以自定义表达式，以允许或删除更多字符类型

我们也可以将`String.replaceAll()`与相同的正则表达式一起使用:

```java
@Test
public void whenRemoveEmojiUsingRegex_thenSuccess() {
    String text = "la conférence, commencera à 10 heures ?";
    String regex = "[^\\p{L}\\p{N}\\p{P}\\p{Z}]";
    String result = text.replaceAll(regex, "");

    assertEquals(result, "la conférence, commencera à 10 heures ");
}
```

## 5。使用代码点

现在，我们还将使用代码点来检测表情符号。**我们可以用 `\x{hexidecimal value}` 表达式来匹配一个特定的 Unicode 点。**

在以下示例中，我们使用表情符号的 Unicode 点移除了两个 Unicode 范围:

```java
@Test
public void whenRemoveEmojiUsingCodepoints_thenSuccess() {
    String text = "la conférence, commencera à 10 heures ?";
    String result = text.replaceAll("[\\x{0001f300}-\\x{0001f64f}]|[\\x{0001f680}-\\x{0001f6ff}]", "");

    assertEquals(result, "la conférence, commencera à 10 heures ");
}
```

当前可用表情符号及其代码点的完整列表可以在[这里](https://web.archive.org/web/20221205194342/https://unicode.org/emoji/charts/full-emoji-list.html)找到。

## 6。使用 Unicode 范围

最后，我们将再次使用 Unicode，但这次使用的是`\u`表达式。

问题是有些 Unicode 点不适合一个 16 位的 Java 字符，所以有些需要两个字符。

下面是使用`\u`的对应表达式:

```java
@Test
public void whenRemoveEmojiUsingUnicode_thenSuccess() {
    String text = "la conférence, commencera à 10 heures ?";
    String result = text.replaceAll("[\ud83c\udf00-\ud83d\ude4f]|[\ud83d\ude80-\ud83d\udeff]", "");

    assertEquals(result, "la conférence, commencera à 10 heures ");
}
```

## 7。结论

在这篇简短的文章中，我们学习了从 Java 字符串中删除表情符号的不同方法。我们使用表情库、正则表达式和 Unicode 范围。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221205194342/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)