# Java 中的非捕获正则表达式组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regex-non-capturing-groups>

## 1。概述

非捕获组是 [Java 正则表达式](/web/20220628053628/https://www.baeldung.com/regular-expressions-java)中的重要构造。**他们创建一个子模式，作为一个单独的单元，但不保存匹配的字符序列。** 在本教程中，我们将探讨如何在 Java 正则表达式中使用非捕获组。

## 2。正则表达式组

正则表达式组可以是两种类型之一:捕获和非捕获。 

捕获组保存匹配的字符序列。它们的值可以用作模式中的反向引用和/或稍后在代码中检索。

虽然它们不保存匹配的字符序列，但是非捕获组可以改变组内的模式匹配修饰符。一些非捕获组甚至可以在成功的子模式匹配后丢弃回溯信息。

让我们来看看一些非捕获型团队的例子。

## 3.非捕获组

操作员`(?:X)`创建一个非捕获组。`X`是 : 组的图案

```java
Pattern.compile("[^:]+://(?:[.a-z]+/?)+")
```

此模式有一个非捕获组。它将匹配一个值，如果它是类似 URL 的。URL 的完整正则表达式要复杂得多。我们使用一个简单的模式来关注非捕获组。 

模式 `“[^:]:` ”匹配协议—例如，“`http://`”。非捕获组“`(?:[.a-z]+/?)`”匹配带有可选斜线的域名。由于“`+`”操作符匹配这个模式的一个或多个实例，我们也将匹配随后的路径段。让我们在一个 URL 上测试这个模式: 

```java
Pattern simpleUrlPattern = Pattern.compile("[^:]+://(?:[.a-z]+/?)+");
Matcher urlMatcher
  = simpleUrlPattern.matcher("http://www.microsoft.com/some/other/url/path");

Assertions.assertThat(urlMatcher.matches()).isTrue(); 
```

让我们看看当我们尝试检索匹配的文本时会发生什么: 

```java
Pattern simpleUrlPattern = Pattern.compile("[^:]+://(?:[.a-z]+/?)+");
Matcher urlMatcher = simpleUrlPattern.matcher("http://www.microsoft.com/");

Assertions.assertThat(urlMatcher.matches()).isTrue();
Assertions.assertThatThrownBy(() -> urlMatcher.group(1))
  .isInstanceOf(IndexOutOfBoundsException.class);
```

正则表达式被编译成一个`java.util.Pattern` 对象。然后，我们创建一个`java.util.Matcher`来将我们的`Pattern`应用到提供的值。

接下来，我们断言 [`matches()`](/web/20220628053628/https://www.baeldung.com/java-matcher-find-vs-matches) 的结果返回`true`。

我们使用一个非捕获组来匹配 URL 中的域名。**由于非捕获组不保存匹配的文本，我们无法检索匹配的文本`“www.microsoft.com/”`。**试图检索域名将导致`IndexOutOfBoundsException`。

### 3.1.内嵌修饰符

正则表达式区分大小写。如果我们将我们的模式应用于大小写混合的 URL，匹配将会失败:

```java
Pattern simpleUrlPattern
  = Pattern.compile("[^:]+://(?:[.a-z]+/?)+");
Matcher urlMatcher
  = simpleUrlPattern.matcher("http://www.Microsoft.com/");

Assertions.assertThat(urlMatcher.matches()).isFalse();
```

在我们也想匹配大写字母的情况下，我们可以尝试几个选项。

一种选择是将大写字符范围添加到模式中:

```java
Pattern.compile("[^:]+://(?:[.a-zA-Z]+/?)+")
```

另一种选择是使用修饰符标志。因此，我们可以编译不区分大小写的正则表达式:

```java
Pattern.compile("[^:]+://(?:[.a-z]+/?)+", Pattern.CASE_INSENSITIVE)
```

非捕获组允许第三种选择:**我们可以只改变组的修改标志。**让我们将不区分大小写的修饰符标志(“`i`”)添加到组中:

```java
Pattern.compile("[^:]+://(?i:[.a-z]+/?)+");
```

既然我们已经使组不区分大小写，让我们将此模式应用于大小写混合的 URL:

```java
Pattern scopedCaseInsensitiveUrlPattern
  = Pattern.compile("[^:]+://(?i:[.a-z]+/?)+");
Matcher urlMatcher
  = scopedCaseInsensitiveUrlPattern.matcher("http://www.Microsoft.com/");

Assertions.assertThat(urlMatcher.matches()).isTrue();
```

当一个模式被编译成不区分大小写时，我们可以通过在修饰符前添加“-”操作符来关闭它。让我们将此模式应用于另一个大小写混合的 URL:

```java
Pattern scopedCaseSensitiveUrlPattern
  = Pattern.compile("[^:]+://(?-i:[.a-z]+/?)+/ending-path", Pattern.CASE_INSENSITIVE);
Matcher urlMatcher
  = scopedCaseSensitiveUrlPattern.matcher("http://www.Microsoft.com/ending-path");

Assertions.assertThat(urlMatcher.matches()).isFalse(); 
```

在本例中，最后的路径段“`/ending-path`”不区分大小写。模式的“`/ending-path`”部分将匹配大写和小写字符。

当我们在组内关闭不区分大小写选项时，非捕获组只支持小写字符。因此，混合大小写的域名不匹配。

## 4.独立的非捕获组

独立的非捕获组是正则表达式组的一种类型。这些组**在找到成功匹配**后丢弃回溯信息。当使用这种类型的组时，我们需要知道何时会发生回溯。否则，我们的模式可能不符合我们认为它们应该符合的价值观。

[回溯](/web/20220628053628/https://www.baeldung.com/java-regex-performance)是非确定性有限自动机(NFA)正则表达式引擎的一个特性。**当引擎无法匹配文本时，NFA 引擎可以在模式中寻找替代文本。**在用尽所有可用的替代方案后，引擎将使匹配失败。我们只讨论回溯，因为它与独立的非捕获组相关。

使用运算符“`(?>X)`”创建一个独立的非捕获组，其中`X`是子模式:

`Pattern.compile("[^:]+://(?>[.a-z]+/?)+/ending-path");`

我们添加了“`/ending-path`”作为恒定路径段。有了这个额外的要求，就产生了回溯的情况。域名和其他路径段可以匹配斜杠字符。为了匹配`“/ending-path”`，引擎需要回溯。通过回溯，引擎可以从组中移除斜线，并将其应用于模式的“`/ending-path`”部分。

让我们将独立的非捕获组模式应用于 URL:

```java
Pattern independentUrlPattern
  = Pattern.compile("[^:]+://(?>[.a-z]+/?)+/ending-path");
Matcher independentMatcher
  = independentUrlPattern.matcher("http://www.microsoft.com/ending-path");

Assertions.assertThat(independentMatcher.matches()).isFalse();
```

该组成功匹配域名和斜杠。所以，我们离开了独立非捕获组的范围。

这种模式要求在`“ending-path”`之前出现一个斜杠。然而，我们的独立非捕获组已经匹配了斜线。

NFA 引擎应该尝试回溯。由于组末尾的斜杠是可选的，NFA 引擎将从组中删除该斜杠并重试。独立非捕获组丢弃了回溯信息。所以，NFA 引擎不能回溯。

### 4.1.在组内回溯

回溯可以发生在独立的非捕获组中。当 NFA 引擎匹配该组时，回溯信息没有被丢弃。回溯信息直到组匹配成功后才被丢弃:

```java
Pattern independentUrlPatternWithBacktracking
  = Pattern.compile("[^:]+://(?>(?:[.a-z]+/?)+/)ending-path");
Matcher independentMatcher
  = independentUrlPatternWithBacktracking.matcher("http://www.microsoft.com/ending-path");

Assertions.assertThat(independentMatcher.matches()).isTrue();
```

现在我们在一个独立的非捕获组中有了一个非捕获组。我们还有一个回溯的情况，涉及到`“ending-path”`前面的斜杠。然而，我们已经将模式的回溯部分包含在独立的非捕获组中。**回溯会发生在独立的非捕获组内。因此，NFA 引擎有足够的信息进行回溯，并且模式与提供的 URL 匹配。**

## 5.结论

我们已经展示了非捕获组不同于捕获组。然而，它们像它们的捕获对应物一样作为一个单独的单元运行。我们还展示了**非捕获组可以启用或禁用该组的修改器，而不是作为一个整体的模式**。

类似地，我们已经展示了独立的非捕获组是如何丢弃回溯信息的。没有这些信息，NFA 引擎就无法探索替代方案来进行成功的匹配。然而，回溯可以发生在组内。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628053628/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex-2)