# 统计字符串中字符的出现次数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-count-chars>

## 1。概述

在 Java 中，有很多方法可以统计一个字符在一个`String`中出现的次数。

在这个快速教程中，我们将重点介绍几个如何计算字符数的例子——首先是核心 Java 库，然后是 Spring 和 Guava 等其他库和框架。

## 延伸阅读:

## [使用 indexOf 查找一个单词在一个字符串中的所有出现](/web/20221017230852/https://www.baeldung.com/java-indexof-find-string-occurrences)

Learn how to solve the "needle in a haystack" problem by using the indexOf method to find all occurrences of a word in a larger text string.[Read more](/web/20221017230852/https://www.baeldung.com/java-indexof-find-string-occurrences) →

## [番石榴查马特](/web/20221017230852/https://www.baeldung.com/guava-string-charmatcher)

Use the Guava CharMatcher to work with Strings - remove special chars, validate, trim, collapse, replace and count among other super useful APIs.[Read more](/web/20221017230852/https://www.baeldung.com/guava-string-charmatcher) →

## [用 Apache Commons Lang 3 进行字符串处理](/web/20221017230852/https://www.baeldung.com/string-processing-commons-lang)

Quick intro to working with Strings with the Apache Commons library and StringUtils.[Read more](/web/20221017230852/https://www.baeldung.com/string-processing-commons-lang) →

## 2。使用核心 Java 库

### 2**1 .** **命令式方法**

一些开发人员可能更喜欢使用核心 Java。有许多方法可以计算一个字符在一个字符串中出现的次数。

让我们从一个简单/幼稚的方法开始:

```java
String someString = "elephant";
char someChar = 'e';
int count = 0;

for (int i = 0; i < someString.length(); i++) {
    if (someString.charAt(i) == someChar) {
        count++;
    }
}
assertEquals(2, count);
```

不出所料，这将是可行的，但有更好的方法来做到这一点。

### 2.2。使用递归

一个不太明显但仍然有趣的解决方案是使用递归:

```java
private static int countOccurences(
  String someString, char searchedChar, int index) {
    if (index >= someString.length()) {
        return 0;
    }

    int count = someString.charAt(index) == searchedChar ? 1 : 0;
    return count + countOccurences(
      someString, searchedChar, index + 1);
}
```

我们可以通过以下方式调用这个递归方法:`useRecursionToCountChars(“elephant”, ‘e', 0)`。

### 2.3。使用正则表达式

另一种方法是使用正则表达式:

```java
Pattern pattern = Pattern.compile("[^e]*e");
Matcher matcher = pattern.matcher("elephant");
int count = 0;
while (matcher.find()) {
    count++;
}

assertEquals(2, count);
```

请注意，这种解决方案在技术上是正确的，但不是最佳的，因为使用非常强大的正则表达式来解决诸如查找字符串中某个字符的出现次数这样简单的问题是多余的。

### 2.4。使用 Java 8 特性

Java 8 中的新特性在这里非常有用。

让我们使用 streams 和 lambdas 来实现计数:

```java
String someString = "elephant";
long count = someString.chars().filter(ch -> ch == 'e').count();
assertEquals(2, count);

long count2 = someString.codePoints().filter(ch -> ch == 'e').count();
assertEquals(2, count2);
```

因此，这显然是一个使用核心库的更干净、更可读的解决方案。

## 3。使用外部库

现在让我们看几个利用外部库的实用程序的解决方案。

### 3.1。使用 `StringUtils`

一般来说，使用现有的解决方案总是比发明我们自己的更好。`commons.lang.StringUtils`类为我们提供了`countMatches()`方法，该方法可用于计算给定`String`中的字符甚至子字符串。

首先，我们需要包含适当的依赖关系:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221017230852/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 上找到最新版本。

现在让我们使用`countMatches()`来计算“大象”字符串文字中`e`字符的数量:

```java
int count = StringUtils.countMatches("elephant", "e");
assertEquals(2, count);
```

### 3.2。使用番石榴

番石榴也有助于计算字符。我们需要定义依赖性:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221017230852/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 上找到最新版本。

让我们看看番石榴如何快速帮助我们计算字符:

```java
int count = CharMatcher.is('e').countIn("elephant");
assertEquals(2, count);
```

### 3.3。使用弹簧

自然，仅仅为了计算字符数而将 Spring 框架添加到我们的项目中是没有意义的。

然而，如果我们的项目中已经有了它，我们只需要使用`countOccurencesOf()`方法:

```java
int count = StringUtils.countOccurrencesOf("elephant", "e");
assertEquals(2, count);
```

## 4。结论

在本文中，我们关注了计算字符串中字符数的各种方法。其中一些是纯 Java 设计的；有些需要额外的库。

我们的建议是使用已经存在的来自`StringUtils`、番石榴或 Spring 的工具。然而，如果只使用普通 Java 是首选，本文提供了一些用 Java 8 完成它的可能性。

这些例子的完整源代码可以在 GitHub 项目中找到。