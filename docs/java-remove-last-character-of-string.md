# 如何去掉一个字符串的最后一个字符？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-last-character-of-string>

## 1。概述

在这个快速教程中，我们将探索删除`String.`最后一个字符的不同技术

## 2。使用`String.substring()`

最简单的方法是使用`String`类的内置`substring()`方法**。**

为了删除给定`String,`的最后一个字符，我们必须使用两个参数:`0`作为起始索引，以及倒数第二个字符的索引。我们可以通过调用`String`的`length()` 方法，并从结果中减去`1`来实现。

然而，**这个方法不是空安全的，**如果我们使用空字符串，这将会失败。

为了克服 null 和空字符串的问题，我们可以将该方法包装在一个助手类中:

```java
public static String removeLastChar(String s) {
    return (s == null || s.length() == 0)
      ? null 
      : (s.substring(0, s.length() - 1));
}
```

我们可以重构代码，使用 Java 8:

```java
public static String removeLastCharOptional(String s) {
    return Optional.ofNullable(s)
      .filter(str -> str.length() != 0)
      .map(str -> str.substring(0, str.length() - 1))
      .orElse(s);
    }
```

## 3。使用`StringUtils.substring()`

不用重新发明轮子，我们可以使用 Apache Commons Lang3 库中的 **`StringUtils`类，它提供了有用的`String`操作。其中一个是处理异常的**空安全`substring()`方法**。**

为了包含`StringUtils,`，我们必须更新我们的`pom.xml`文件:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

`StringUtils.substring()`需要三个参数:一个给定的`String,`的第一个字符的索引(在我们的例子中它总是 0)，和倒数第二个字符的索引。同样，我们可以简单地使用`length()`方法减去`1:`

```java
String TEST_STRING = "abcdef";

StringUtils.substring(TEST_STRING, 0, TEST_STRING.length() - 1);
```

同样，这个操作也不是空安全的。不过空的时候也能很好地工作。

## 4。使用`StringUtils.chop()`

**`StringUtils`类提供了`chop()`方法，适用于所有边缘场景:空字符串**。

它非常容易使用，并且只需要一个参数:`String.`它唯一的目的是删除最后一个字符，不多也不少:

```java
StringUtils.chop(TEST_STRING);
```

## 5。使用正则表达式

我们还可以通过充分利用正则表达式来删除`String`中的最后一个字符(或任意数量的字符)。

例如，我们可以使用`String`类本身的 **`replaceAll()`方法，**采用两个参数:正则表达式和替换`String`:

```java
TEST_STRING.replaceAll(".$", "");
```

注意，因为我们在调用`String,`上的方法，所以操作**不是空安全的**。

另外，`replaceAll()`和正则表达式乍一看可能很复杂。我们可以在这里阅读更多关于正则表达式[的内容，但是为了使逻辑更加用户友好，我们可以将它包装在一个助手类中:](/web/20220925222917/https://www.baeldung.com/regular-expressions-java)

```java
public static String removeLastCharRegex(String s) {
    return (s == null) ? null : s.replaceAll(".$", "");
}
```

注意，如果一个`String`以换行符结束，那么上面的方法将失败，因为 regex 中的`“.”`匹配除了行结束符之外的任何字符。

最后，让我们用 Java 8 重写**的实现:**

```java
public static String removeLastCharRegexOptional(String s) {
    return Optional.ofNullable(s)
      .map(str -> str.replaceAll(".$", ""))
      .orElse(s);
}
```

## 6。结论

在这篇简短的文章中，我们讨论了仅删除一个`String,`的最后一个字符的不同方法，有些是手动的，有些是现成的。

如果我们需要更多的灵活性，我们需要删除更多的字符，我们可以使用更高级的正则表达式解决方案。

和往常一样，整篇文章中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220925222917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)