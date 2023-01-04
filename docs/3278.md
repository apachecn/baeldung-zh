# Java 中不区分大小写的字符串匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-case-insensitive-string-matching>

## 1.概观

有很多方法可以检查一个 [`String`是否包含子串](/web/20220630130920/https://www.baeldung.com/java-string-contains-substring)。在本文中，我们将在 Java 中寻找`String` 中的子字符串，同时关注 [`String.contains()`](https://web.archive.org/web/20220630130920/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#contains(java.lang.CharSequence)) 的不区分大小写的变通方法。最重要的是，我们将提供如何解决这个问题的例子。

## 2.最简单的解决方案:`String.toLowerCase`

最简单的解决方法是使用 [`String.toLowerCase()`](/web/20220630130920/https://www.baeldung.com/java-string-convert-case) 。在这种情况下，我们将把两个字符串都转换成小写，然后使用`contains()`方法:

```
assertTrue(src.toLowerCase().contains(dest.toLowerCase()));
```

我们也可以使用 [`String.toUpperCase()`](/web/20220630130920/https://www.baeldung.com/java-string-convert-case) ，它会提供相同的结果。

## 3.`String.matches`用正则表达式

另一种选择是通过使用带有正则表达式的`[String.matches()](https://web.archive.org/web/20220630130920/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#matches(java.lang.String))` :

```
assertTrue(src.matches("(?i).*" + dest + ".*"));
```

`matches()`方法用一个 S `tring`来表示正则表达式。`(?i)`启用**不区分大小写**和`.*`使用除换行符以外的所有字符。

## 4.`String.regionMatches`

我们也可以用 [`String.regionMatches()`](https://web.archive.org/web/20220630130920/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#regionMatches(boolean,int,java.lang.String,int,int)) 。它检查两个`String`区域是否匹配，使用`true`作为`ignoreCase`参数:

```
public static boolean processRegionMatches(String src, String dest) {
    for (int i = src.length() - dest.length(); i >= 0; i--) 
        if (src.regionMatches(true, i, dest, 0, dest.length())) 
            return true; 
    return false;
}
```

```
assertTrue(processRegionMatches(src, dest));
```

为了提高性能，考虑到目的地`String`的长度，它开始匹配区域。然后，它减少了迭代器。

## 5.`Pattern`带`CASE_INSENSITIVE`选项

[`java.util.regex.Pattern`](https://web.archive.org/web/20220630130920/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html) 类为我们提供了一种使用`matcher()`方法匹配字符串的方法。在这种情况下，我们可以使用`quote()`方法来转义任何特殊字符，以及`CASE_INSENSITIVE`标志。让我们来看看:

```
assertTrue(Pattern.compile(Pattern.quote(dest), Pattern.CASE_INSENSITIVE)
    .matcher(src)
    .find());
```

## 6.Apache common〔t0〕

最后，我们将利用 [Apache Commons `StringUtils` class](/web/20220630130920/https://www.baeldung.com/string-processing-commons-lang) :

```
assertTrue(StringUtils.containsIgnoreCase(src, dest));
```

## 7.性能比较

在这篇关于使用`contains`方法检查子字符串的[通用文章中，我们使用开源框架](/web/20220630130920/https://www.baeldung.com/java-string-contains-substring) [Java 微基准测试工具](/web/20220630130920/https://www.baeldung.com/java-microbenchmark-harness) (JMH)来**比较这些方法在纳秒内的性能**:

1.  **`Pattern CASE_INSENSITIVE Regular Expression` : 399.387 纳秒**
2.  `String toLowerCase` : 434.064 纳秒
3.  `Apache Commons StringUtils` : 496.313 纳秒
4.  `String Region Matches` : 718.842 纳秒
5.  `String matches with Regular Expression` : 3964.346 纳秒

我们可以看到，获胜者是启用了`CASE_INSENSITIVE`标志的`Pattern`，紧随其后的是`toLowerCase()`。我们还注意到 Java 8 和 Java 11 之间的性能有了明显的提高。

## 8.结论

在本教程中，我们看了几种不同的方法来检查一个子串的`String`,同时忽略了 Java 中的大小写。

我们看了使用`String.toLowerCase()` 和`toUpperCase()`、`String.matches()`、`String.regionMatches()`、Apache Commons `StringUtils.containsIgnoreCase()`和`Pattern.matcher().find()`。

此外，我们评估了每个解决方案的性能，发现使用带有`CASE_INSENSITIVE`标志的`java.util.regex.Pattern`中的`compile()`方法表现最好`.`

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220630130920/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)