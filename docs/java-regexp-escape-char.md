# Java 正则表达式中的转义字符指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regexp-escape-char>

## 1。概述

Java 中的正则表达式 API，`java.util.regex` 广泛用于模式匹配。要了解更多，您可以关注这篇文章。

在本文中，我们将关注正则表达式中的转义字符，并展示如何在 Java 中实现。

## 2。特殊正则表达式字符

根据 Java 正则表达式 API 文档，正则表达式中有一组特殊字符，也称为元字符。

当我们想让这些字符保持原样，而不是用它们的特殊含义来解释它们时，我们需要避开它们。通过对这些字符进行转义，我们可以在将字符串与给定的正则表达式进行匹配时，强制将它们视为普通字符。

我们通常需要以这种方式转义的元字符是:

**<(【{\^-=$！|]})?*+.>**

让我们看一个简单的代码示例，其中我们将输入`String`与正则表达式中表达的模式进行匹配。

这个测试表明，对于给定的输入字符串`foof`当模式`foo`时。(`foo`以点字符结尾)匹配，则返回一个值`true`，表示匹配成功。

```java
@Test
public void givenRegexWithDot_whenMatchingStr_thenMatches() {
    String strInput = "foof";
    String strRegex = "foo.";

    assertEquals(true, strInput.matches(strRegex));
}
```

你可能会奇怪为什么没有点号的情况下匹配成功了(。)输入中出现的字符`String?`

答案很简单。圆点(。)是一个元字符——点在这里的特殊意义在于它的位置可以有“任何字符”。因此，很清楚匹配器是如何确定找到匹配的。

假设我们不想处理点(。)字有其独特的含义。相反，我们希望它被解释为一个点符号。这意味着在前面的例子中，我们不想让模式`foo.` 与输入`String.`匹配

我们如何处理这样的情况？答案是:**我们需要转义点(。)字符，这样它的特殊含义就被忽略了。**

让我们在下一节更详细地探讨它。

## 3。转义字符

根据正则表达式的 Java API 文档，有两种方法可以对有特殊含义的字符进行转义。换句话说，就是要强迫他们被当作普通人物来对待。

让我们看看它们是什么:

1.  在元字符前加一个反斜杠(\)
2.  用`\Q`和`\E`将元字符括起来

这只是意味着，在我们之前看到的例子中，如果我们想对点字符进行转义，我们需要在点字符前放一个反斜杠字符。或者，我们可以将点字符放在\Q 和\E 之间。

### 3.1。使用反斜杠进行转义

这是我们可以用来在正则表达式中转义元字符的技术之一。然而，我们知道反斜杠字符在 Java `String`文字中也是一个转义字符。因此，在使用反斜杠字符放在任何字符(包括\字符本身)之前时，我们需要将反斜杠字符加倍。

因此，在我们的示例中，我们需要更改正则表达式，如本测试所示:

```java
@Test
public void givenRegexWithDotEsc_whenMatchingStr_thenNotMatching() {
    String strInput = "foof";
    String strRegex = "foo\\.";

    assertEquals(false, strInput.matches(strRegex));
}
```

在这里，点字符被转义，因此匹配器简单地将其视为点，并试图找到以点结尾的模式(即`foo.`)。

在这种情况下，它返回`false` ,因为输入`String`中没有该模式的匹配。

### 3.2。使用\Q & \E 进行转义

或者，我们可以使用`\Q`和`\E`对特殊字符进行转义。`\Q`表示到`\E`为止的所有字符都需要转义，`\E`表示我们需要结束从`\Q`开始的转义。

这仅仅意味着在`\Q`和`\E`之间的任何东西都将被转义。

在这里显示的测试中，`String`类的`split()` 使用提供给它的正则表达式进行匹配。

我们的要求是通过竖线(|)字符将输入字符串分割成单词。因此，我们使用正则表达式模式来这样做。

管道字符是一个元字符，需要在正则表达式中进行转义。

这里，转义是通过在`\Q`和`\E`之间放置管道字符来完成的:

```java
@Test
public void givenRegexWithPipeEscaped_whenSplitStr_thenSplits() {
    String strInput = "foo|bar|hello|world";
    String strRegex = "\\Q|\\E";

    assertEquals(4, strInput.split(strRegex).length);
}
```

## 4。`Pattern.quote(String` S)法

模式。`java.util.regex.Pattern` 类中的 Quote(String S)方法将给定的正则表达式模式`String`转换成文字模式`String.`，这意味着输入`String`中的所有元字符都被视为普通字符。

使用这种方法比使用`\Q` & `\E`更方便，因为它用它们包装了给定的`String`。

让我们来看看这种方法的实际应用:

```java
@Test
public void givenRegexWithPipeEscQuoteMeth_whenSplitStr_thenSplits() {
    String strInput = "foo|bar|hello|world";
    String strRegex = "|";

    assertEquals(4,strInput.split(Pattern.quote(strRegex)).length);
}
```

在这个快速测试中，`Pattern.quote()` 方法用于对给定的 regex 模式进行转义，并将其转换为`String`文字。换句话说，它对 regex 模式中的所有元字符进行了转义。它正在做着与`\Q` & `\E`类似的工作。

管道字符由`Pattern.quote()` 方法转义，`split()` 将它解释为一个`String`文字，用它来划分输入。

正如我们所看到的，这是一个非常干净的方法，开发者也不必记住所有的转义序列。

**我们应该注意到`Pattern.quote`用一个转义序列包围了整个块。如果我们想单独转义字符，我们需要使用一个[标记替换算法](/web/20220731170519/https://www.baeldung.com/java-regex-token-replacement)。**

## 5。其他示例

我们来看看`java.util.regex.Matcher` 的`replaceAll()` 方法是如何工作的。

如果我们需要用另一个字符替换给定字符`String`的所有出现，我们可以通过向它传递一个正则表达式来使用这个方法。

假设我们有一个多次出现`$`字符的输入。我们想要得到的结果是相同的字符串，其中的`$`字符被替换为。

这个测试演示了模式`$`如何被传递而不被转义:

```java
@Test
public void givenRegexWithDollar_whenReplacing_thenNotReplace() {

    String strInput = "I gave $50 to my brother."
      + "He bought candy for $35\. Now he has $15 left.";
    String strRegex = "$";
    String strReplacement = "£";
    String output = "I gave £50 to my brother."
      + "He bought candy for £35\. Now he has £15 left.";

    Pattern p = Pattern.compile(strRegex);
    Matcher m = p.matcher(strInput);

    assertThat(output, not(equalTo(m.replaceAll(strReplacement))));
}
```

测试断言`$`没有被`£`正确替换。

现在，如果我们对 regex 模式进行转义，替换会正确发生，并且测试会通过，如下面的代码片段所示:

```java
@Test
public void givenRegexWithDollarEsc_whenReplacing_thenReplace() {

    String strInput = "I gave $50 to my brother."
      + "He bought candy for $35\. Now he has $15 left.";
    String strRegex = "\\$";
    String strReplacement = "£";
    String output = "I gave £50 to my brother."
      + "He bought candy for £35\. Now he has £15 left.";
    Pattern p = Pattern.compile(strRegex);
    Matcher m = p.matcher(strInput);

    assertEquals(output,m.replaceAll(strReplacement));
}
```

注意这里的`\\$` ,它通过转义`$`字符并成功匹配模式来完成这个任务。

## 6。结论

在本文中，我们研究了 Java 正则表达式中的转义字符。

我们讨论了为什么正则表达式需要转义，以及实现转义的不同方式。

和往常一样，与本文相关的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220731170519/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)