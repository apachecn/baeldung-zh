# 检查字符串的第一个字母是否大写

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-first-letter-uppercase>

## 1.介绍

在这个简短的教程中，我们将熟悉在 Java 中检查字符串首字母是否大写的不同选项。

## 2.例子

首先，我们将从定义我们将在所有解决方案中使用的示例字符串开始:

```java
String example = "Katie";
```

因此，示例字符串只是一个大写的名称。现在，让我们检查检查第一个字母是否大写的选项。

## 3.核心 Java 解决方案

我们将熟悉的第一个解决方案不需要新的依赖项。我们将使用`java.lang`包中`Character`类的`isUpperCase`方法:

```java
public static boolean isUpperCase(int codePoint);
```

这个方法获取一个字符，并确定它是否是大写字符。

对于我们的例子，我们只需要提取字符串中的第一个字符。首先，我们将使用 [`charAt`](/web/20221208143926/https://www.baeldung.com/string/char-at) 方法进行提取。然后，我们将调用`isUpperCase`方法:

```java
Assertions.assertTrue(Character.isUpperCase(example.charAt(0)));
```

这个断言将通过，因为我们的示例字符串中的第一个字母是大写字符。

## 4.使用正则表达式

在输入字符串中寻找匹配时，正则表达式是一种常见的解决方案。因此，我们将使用它们来检查字符串中的第一个字符是否是大写的。

和前面的解决方案一样，这个解决方案不需要添加新的依赖项。正则表达式已经在`java.util.regex `包中可用。

下一步是定义匹配的模式。对于我们的例子，我们需要一个匹配的模式，如果一个字符串以大写字符开始，而其他字符可以是大写、小写或数字。然后，我们只需要检查模式是否与示例字符串匹配:

```java
String regEx = "[A-Z]\\w*";
Assertions.assertTrue(example.matches(regEx));
```

## 5.番石榴溶液

另一个解决方法可以在[番石榴](/web/20221208143926/https://www.baeldung.com/guava-guide)图书馆找到。 **我们将 需要使用`Ascii`类 中的`isUpperCase`方法来检查一个字符串的首字母是否大写。**

第一步是添加[番石榴](https://web.archive.org/web/20221208143926/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava)依赖关系:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

然后，我们将对示例字符串的第一个字母应用`isUpperCase` 方法:

```java
Assertions.assertTrue(Ascii.isUpperCase(example.charAt(0)));
```

这种方法实际上与第 2.1 章的核心 Java 解决方案相同。如果没有特别的原因，最好使用不需要额外依赖的解决方案。

## 6.结论

在本文中，我们考察了检查首字母是否大写的不同解决方案。

首先，我们讨论了核心 Java 中可用的解决方案。后来，我们看到了如何用正则表达式执行检查。最后，我们展示了来自番石榴图书馆的解决方案。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143926/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-3)