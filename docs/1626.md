# 检查 Java 中的空字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-blank-empty-strings>

## 1.介绍

在本教程中，我们将讨论一些在 Java 中检查空字符串的方法。有一些本地语言的方法，以及一些库。

## 2.空对空白

当然，知道一个字符串何时为空是很常见的，但是让我们确保我们的定义是一致的。

如果一个字符串是`null`或者没有`any`长度，我们认为它是`empty`。如果一个字符串只包含空白，那么我们称之为`blank`。

对于 Java，空白是字符，就像空格、制表符等等。我们可以举出 [`Character.isWhitespace`](https://web.archive.org/web/20220816003101/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#isWhitespace(char)) 为例。

## 3.空字符串

### 3.1.使用 Java 6 及以上版本

如果我们至少在 Java 6 上，那么检查一个`empty`字符串最简单的方法是 [`String#isEmpty`](https://web.archive.org/web/20220816003101/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#isEmpty()) :

```
boolean isEmptyString(String string) {
    return string.isEmpty();
}
```

为了使它也是空安全的，我们需要添加一个额外的检查:

```
boolean isEmptyString(String string) {
    return string == null || string.isEmpty();
}
```

### 3.2.使用 Java 5 和更低版本

`String#isEmpty`随 Java 6 推出。对于 Java 5 及以下版本，我们可以使用`String#length`来代替:

```
boolean isEmptyString(String string) {
    return string == null || string.length() == 0;
}
```

其实， **`String#isEmpty`只是`String#length`的一个快捷方式。**

## 4.空白字符串

`String#isEmpty`和`String#length`都可以用来检查`empty`字符串。

如果我们还想检测`blank`字符串，我们可以借助`String#trim`来实现。在执行检查之前，**会删除所有的前导和尾随空格:**

```
boolean isBlankString(String string) {
    return string == null || string.trim().isEmpty();
}
```

准确地说，`String#trim`将删除所有带有小于或等于 U+0020 的 [Unicode 代码的前导和尾随字符。](https://web.archive.org/web/20220816003101/https://en.wikipedia.org/wiki/List_of_Unicode_characters#Control_codes)

另外，记住*字符串*是不可变的，所以调用`trim `实际上不会改变底层的字符串。

除了上面的方法，**从 Java 11 开始，我们还可以使用`[isBlank()](https://web.archive.org/web/20220816003101/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#isBlank())`方法来代替修剪**:

```
boolean isBlankString(String string) {
    return string == null || string.isBlank();
}
```

`isBlank()`方法也更有效，因为它不会在堆上创建新的`String`。因此，如果我们使用 Java 11 或更高版本，这是首选方法。

## 5.Bean 验证

检查`blank`字符串的另一种方法是正则表达式。例如，这在 [Java Bean 验证](/web/20220816003101/https://www.baeldung.com/javax-validation)中派上了用场:

```
@Pattern(regexp = "\\A(?!\\s*\\Z).+")
String someString;
```

给定的正则表达式确保空字符串不会被验证。

## 6.使用 Apache Commons

如果可以添加依赖项，我们可以使用 [Apache Commons Lang](https://web.archive.org/web/20220816003101/https://commons.apache.org/proper/commons-lang/) 。它有许多 Java 助手。

如果我们使用 Maven，我们需要将[的`commons-lang3` 依赖项](https://web.archive.org/web/20220816003101/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3)添加到 pom 中:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
</dependency>
```

除了别的以外，这给了我们 [`StringUtils`](https://web.archive.org/web/20220816003101/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html) 。

这个类带有类似 [`isEmpty`](https://web.archive.org/web/20220816003101/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isEmpty-java.lang.CharSequence-) 、 [`isBlank`](https://web.archive.org/web/20220816003101/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isBlank-java.lang.CharSequence-) 等方法:

```
StringUtils.isBlank(string)
```

这个调用和我们自己的`isBlankString`方法做的一样。它是空安全的，也检查空白。

## 7.用番石榴

另一个带来某些字符串相关实用程序的著名库是 Google 的 [Guava](https://web.archive.org/web/20220816003101/https://github.com/google/guava) 。从 23.1 版本开始，番石榴有两种口味:`android`和`jre`。Android 版面向 Android 和 Java 7，而 JRE 版面向 Java 8。

如果我们不针对 Android，我们可以将 JRE 风格的添加到我们的 pom 中:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

芭乐琴弦类自带 [`Strings.isNullOrEmpty`](https://web.archive.org/web/20220816003101/https://google.github.io/guava/releases/27.1-jre/api/docs/com/google/common/base/Strings.html#isNullOrEmpty-java.lang.String-) 方法:

```
Strings.isNullOrEmpty(string)
```

它检查给定的字符串是 null 还是空的，**，但是它不会检查只有空白的字符串**。

## 8.结论

有几种方法可以检查一个字符串是否为空。通常，我们还希望检查一个字符串是否为空，这意味着它只包含空白字符。

最方便的方法是使用 Apache Commons Lang，它提供了像`StringUtils.isBlank`这样的助手。如果我们想坚持使用普通 Java，我们可以使用`String#trim`与`String#isEmpty`或`String#length`的组合。对于 Bean 验证，可以使用正则表达式。

请务必在 GitHub 上查看所有这些示例[。](https://web.archive.org/web/20220816003101/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)