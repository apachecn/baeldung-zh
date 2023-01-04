# 理解模式。报价方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pattern-quote>

## 1.概观

在 Java 中使用正则表达式时，有时我们需要**匹配正则表达式模式的文字形式**–**，而不处理那些序列中出现的任何** [**元字符**](/web/20220628162615/https://www.baeldung.com/regular-expressions-java#Characters) 。

在这个快速教程中，让我们看看如何手动和使用 Java 提供的`[Pattern.quote()](https://web.archive.org/web/20220628162615/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html#quote(java.lang.String))`方法来转义正则表达式中的元字符。

## 2.不转义元字符

让我们考虑一个包含金额列表的字符串:

```java
String dollarAmounts = "$100.25, $100.50, $150.50, $100.50, $100.75";
```

现在，让我们假设我们需要在其中搜索特定金额的美元。让我们相应地初始化一个正则表达式模式字符串:

`String patternStr = "$100.50";`

首先，让我们看看**如果我们执行正则表达式搜索而不转义任何元字符**会发生什么:

```java
public void whenMetacharactersNotEscaped_thenNoMatchesFound() {
    Pattern pattern = Pattern.compile(patternStr);
    Matcher matcher = pattern.matcher(dollarAmounts);

    int matches = 0;
    while (matcher.find()) {
        matches++;
    }

    assertEquals(0, matches);
}
```

正如我们所看到的， **`matcher`甚至无法在我们的`dollarAmounts`字符串中找到`$150.50`的单个出现。这仅仅是因为 **`patternStr`以美元符号**开始，而美元符号**恰好是一个[正则表达式**元字符，指定了一行**](https://web.archive.org/web/20220628162615/https://docs.oracle.com/javase/tutorial/essential/regex/bounds.html#PageTitle) 的结束。

正如您可能已经猜到的那样，我们会在所有正则表达式元字符上面临同样的问题。我们将无法搜索包含“`5^3`”等指数的 carets (^)的数学语句，或使用反斜杠(\)的文本，如“`users\bob`”。

## 3.手动忽略元字符

其次，在执行搜索之前，让我们对正则表达式中的元字符进行转义:

```java
public void whenMetacharactersManuallyEscaped_thenMatchingSuccessful() {
    String metaEscapedPatternStr = "\\Q" + patternStr + "\\E";
    Pattern pattern = Pattern.compile(metaEscapedPatternStr);
    Matcher matcher = pattern.matcher(dollarAmounts);

    int matches = 0;
    while (matcher.find()) {
        matches++;
    }

    assertEquals(2, matches);
}
```

这一次，我们已经成功地执行了搜索。但是由于几个原因，这不是理想的解决方案:

*   **字符串串联**在转义使代码更难理解的元字符时执行。
*   **由于增加了硬编码值，干净代码**更少。

## 4.使用`Pattern.quote()`

最后，让我们看看最简单最干净的方法来忽略正则表达式中的元字符。

**Java 提供了一个** [**`quote()`方法**在它们的`Pattern`类](https://web.archive.org/web/20220628162615/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html#quote(java.lang.String))里面检索一个字符串的文字模式:

```java
public void whenMetacharactersEscapedUsingPatternQuote_thenMatchingSuccessful() {
    String literalPatternStr = Pattern.quote(patternStr);
    Pattern pattern = Pattern.compile(literalPatternStr);
    Matcher matcher = pattern.matcher(dollarAmounts);

    int matches = 0;
    while (matcher.find()) {
        matches++;
    }

    assertEquals(2, matches);
}
```

## 5.结论

在本文中，我们研究了如何处理文字形式的正则表达式模式。

我们看到了不转义正则表达式元字符如何无法提供预期的结果，以及正则表达式模式中转义元字符如何可以手动执行并使用`Pattern.quote()`方法。

这里使用的所有代码示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220628162615/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)