# 在 Java 中检查一个字符串是否以某种模式结尾

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-ends-pattern>

## 1.概观

在这个简短的教程中，我们将深入探讨如何在 Java 中检查一个字符串是否以某种模式结尾。

首先，我们将从考虑使用核心 Java 的解决方案开始。然后，我们将展示如何使用外部库完成同样的事情。

## 2.使用`String`类

简单地说， [`String`](https://web.archive.org/web/20221208143814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) 提供了多个方便的选项来验证给定的字符串是否以特定的子字符串结尾。

让我们仔细看看每个选项。

### 2.1.`String#` `endsWith` 法

这种方法通常就是为此目的而引入的。**它提供了最直接的方法来检查一个`String`对象是否以另一个字符串**结束。

所以，让我们来看看它的作用:

```java
public static boolean usingStringEndsWithMethod(String text, String suffix) {
    if (text == null || suffix == null) {
        return false;
    }
    return text.endsWith(suffix);
}
```

请注意`endsWith`不是空安全的。所以，我们首先需要确定`text`和`suffix`不是`null`来避免 [`NullPointerException`](/web/20221208143814/https://www.baeldung.com/java-illegalargumentexception-or-nullpointerexception#nullpointerexception) 。

### 2.2.`String#` `matches` 法

是我们可以用来实现目标的另一个好方法。它只是检查一个字符串是否匹配给定的[正则表达式](/web/20221208143814/https://www.baeldung.com/regular-expressions-java)。

基本上，我们需要做的就是指定适合我们用例的正则表达式:

```java
public static boolean usingStringMatchesMethod(String text, String suffix) {
    if (text == null || suffix == null) {
        return false;
    }
    String regex = ".*" + suffix + "$";
    return text.matches(regex);
}
```

正如我们所看到的，我们使用了一个正则表达式来匹配字符串`text`末尾的`suffix`。然后，我们将正则表达式传递给`matches`方法。

### 2.3.`String#` `regionMatches` 法

同样，我们可以用 [`regionMatches`](/web/20221208143814/https://www.baeldung.com/string/region-matches) 的方法来解决我们的中心问题。**如果字符串部分与指定字符串完全匹配，则返回`true` ，否则返回`false`**。

现在，让我们用一个例子来说明这一点:

```java
public static boolean usingStringRegionMatchesMethod(String text, String suffix) {
    if (text == null || suffix == null) {
        return false;
    }
    int toffset = text.length() - suffix.length();
    return text.regionMatches(toffset, suffix, 0, suffix.length());
}
```

`toffset` 表示字符串中子区域的起始偏移量。所以，为了检查`text`是否以指定的`suffix`结束，`toffset`应该等于`text`的长度减去`suffix`的长度。

## 3.使用`Pattern`类

或者，我们可以使用`Pattern` 类来编译一个正则表达式，检查字符串是否以模式`.`结束

事不宜迟，让我们重用在上一节中指定的相同正则表达式:

```java
public static boolean usingPatternClass(String text, String suffix) {
    if (text == null || suffix == null) {
        return false;
    }
    Pattern pattern = Pattern.compile(".*" + suffix + "$");
    return pattern.matcher(text).find();
}
```

如上所示，`Pattern` 编译前面的 regex，它表示一个字符串的结尾，并试图将它与我们的字符串`text`进行匹配。

## 4.使用 Apache Commons Lang

[Apache Commons Lang](/web/20221208143814/https://www.baeldung.com/java-commons-lang-3) 为字符串操作提供了一组现成的实用程序类。在这些类中，我们找到了`StringUtils`。

这个实用程序类有一个有趣的方法叫做`endsWith.` 它**检查一个字符序列是否以一种空安全的方式**以一个后缀结尾。

现在，让我们举例说明`StringUtils.endsWith` 方法的使用:

```java
public static boolean usingApacheCommonsLang(String text, String suffix) {
    return StringUtils.endsWith(text, suffix);
}
```

## 5.结论

在本文中，我们探索了检查字符串是否以特定模式结尾的不同方法。

首先，我们看到了使用内置 Java 类实现这一点的几种方法。然后，我们解释了如何使用 Apache Commons Lang 库做同样的事情。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143814/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)