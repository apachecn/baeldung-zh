# Java 正则表达式中的前视和后视

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regex-lookahead-lookbehind>

## 1.概观

有时，我们可能会在将一个字符串与一个 [正则表达式](/web/20221020160451/https://www.baeldung.com/regular-expressions-java) 匹配时遇到困难。例如，我们可能不知道我们想要精确匹配的是什么，但是我们可以意识到它的周围环境，比如它前面直接出现了什么或者它后面缺少了什么。在这些情况下，我们可以使用 lookaround 断言。这些表达式被称为断言，因为它们只表明某个东西是否匹配，但不包括在结果中。

在本教程中，我们将看看如何使用四种类型的 regex lookaround 断言。

## 2.积极前瞻

假设我们想要分析 java 文件的导入。首先，让我们通过检查`static`关键字是否跟在`import`关键字之后来寻找`static`的导入语句。

让我们在表达式中使用带有`(?=criteria)`语法的肯定前瞻断言来匹配主表达式`import`之后的字符组`static`:

```
Pattern pattern = Pattern.compile("import (?=static).+");

Matcher matcher = pattern
  .matcher("import static org.junit.jupiter.api.Assertions.assertEquals;");
assertTrue(matcher.find());
assertEquals("import static org.junit.jupiter.api.Assertions.assertEquals;", matcher.group());

assertFalse(pattern.matcher("import java.util.regex.Matcher;").find());
```

## 3.消极前瞻

接下来，让我们做与上一个例子相反的事情，寻找不是`static`的 import 语句。让我们通过检查`static`关键字没有跟在`import`关键字后面来实现这一点。

让我们在表达式中使用带有`(?!criteria)`语法的否定前瞻断言来确保字符组`static`不能在我们的主表达式`import`之后匹配:

```
Pattern pattern = Pattern.compile("import (?!static).+");

Matcher matcher = pattern.matcher("import java.util.regex.Matcher;");
assertTrue(matcher.find());
assertEquals("import java.util.regex.Matcher;", matcher.group());

assertFalse(pattern
  .matcher("import static org.junit.jupiter.api.Assertions.assertEquals;").find());
```

## 4.Java 中 look back 的局限性

在 Java 8 之前，我们可能会遇到这样的限制，即像`+`和`*`这样的未绑定量词不允许出现在后视断言中。也就是说，举个例子，下面的断言会把`PatternSyntaxException`一直扔到 Java 8:

*   `(?<!fo+)bar` ，如果`fo`有一个或多个`o`字符在之前，我们不想匹配`bar`
*   `(?<!fo*)bar` ，如果`bar`前面是一个`f`字符，后面是零个或多个`o`字符，我们不想匹配它
*   `(?<!fo{2,})bar`，其中如果有两个或更多`o`字符的`foo`在之前，我们不想匹配`bar`

作为一种变通方法，我们可以使用一个带有指定上限的花括号量词，例如`(?<!fo{2,4})bar`，其中我们将跟在`f`字符后面的`o`字符的数量最大化为 4。

从 Java 9 开始，我们可以在 lookbehinds 中使用未绑定的量词。然而，由于 regex 实现的内存消耗，仍然建议在 lookbehinds 中只使用有合理上限的量词，例如用`(?<!fo{2,20})bar`代替`(?<!fo{2,2000})bar`。

## 5.积极回顾

假设我们希望在分析中区分 JUnit 4 和 JUnit 5 的导入。首先，让我们检查一下`assertEquals`方法的导入语句是否来自`jupiter`包。

让我们在表达式中使用带有`(?<=criteria)`语法的肯定后视断言来匹配主表达式之前的字符组`jupiter`。* `assertEquals`:

```
Pattern pattern = Pattern.compile(".*(?<=jupiter).*assertEquals;");

Matcher matcher = pattern
  .matcher("import static org.junit.jupiter.api.Assertions.assertEquals;");
assertTrue(matcher.find());
assertEquals("import static org.junit.jupiter.api.Assertions.assertEquals;", matcher.group());

assertFalse(pattern.matcher("import static org.junit.Assert.assertEquals;").find());
```

## 6.消极回顾

接下来，让我们做与上一个例子相反的事情，寻找不是来自`jupiter`包的导入语句。

为此，让我们在表达式中使用带有`(?<!criteria)`语法的负后视断言，以确保字符组 `jupiter.{0,30}`不能在我们的主表达式`assertEquals`之前匹配:

```
Pattern pattern = Pattern.compile(".*(?<!jupiter.{0,30})assertEquals;");

Matcher matcher = pattern.matcher("import static org.junit.Assert.assertEquals;");
assertTrue(matcher.find());
assertEquals("import static org.junit.Assert.assertEquals;", matcher.group());

assertFalse(pattern
  .matcher("import static org.junit.jupiter.api.Assertions.assertEquals;").find());
```

## 7.结论

在本文中，我们看到了如何使用四种类型的正则表达式查找来解决一些用正则表达式匹配字符串的困难情况。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221020160451/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex-2)