# 获取字符串的最后一个单词

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-last-word>

## 1.概观

在这个简短的教程中，我们将学习如何使用两种方法在 Java 中**获取`String`的最后一个单词。**

## 2.使用`split()` 方法

来自`String`类的 [`split()`](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#split(java.lang.String)) 实例方法根据提供的正则表达式分割字符串。这是一个重载方法，返回一个`String`数组。

我们来考虑一个输入`String`，“祝你一天无 bug”。

因为我们必须得到一个字符串的最后一个单词，我们将使用`space (” “)` 作为一个正则表达式来拆分字符串。

我们可以使用`split()`方法对 S `tring`进行标记，并获得最后一个标记，这将是我们的结果:

```java
public String getLastWordUsingSplit(String input) {
    String[] tokens = input.split(" ");
    return tokens[tokens.length - 1];
}
```

这将返回“day”，这是我们输入字符串的最后一个单词。

**注意，如果输入的`String` 只有一个单词或者没有空格，上面的方法将简单地返回相同的`String`。**

## 3.使用 substring()方法

`String `类的`[substring()](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#substring(int))`方法返回一个`String.`的子串，这是一个重载方法，其中一个重载版本接受`beginIndex` 并返回给定索引后的`String `中的所有字符。

我们还将使用来自`String `类的`[lastIndexOf()](https://web.archive.org/web/20221208143926/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#lastIndexOf(java.lang.String)) `方法。它接受一个子字符串，并返回该字符串中指定子字符串的最后一个匹配项的索引。在我们的例子中，这个指定的子串将再次成为一个 `space (” “)`。

让我们结合`substring()`和`lastIndexOf`找到一个输入的最后一个单词`String`:

```java
public String getLastWordUsingSubString(String input) {
    return input.substring(input.lastIndexOf(" ") + 1);
}
```

如果我们像以前一样传递相同的输入`String`“祝你有一个无 bug 的一天”，我们的方法将返回“day”。

**再次注意，如果输入的`String` 只有一个单词或者没有空格，上面的方法将简单地返回相同的`String`。**

## 4.结论

总之，我们已经看到了在 Java 中获取一个`String`的最后一个字的两种方法。