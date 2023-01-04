# 如何计算正则表达式的匹配数？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-count-regex-matches>

## 1.概观

[正则表达式](/web/20220626203042/https://www.baeldung.com/regular-expressions-java)可用于各种文本处理任务，如字数统计算法或文本输入验证。

在本教程中，我们将看看如何使用正则表达式来计算文本中的匹配数。

## 2.用例

让我们开发一个算法，能够计算一封有效邮件在一个字符串中出现的次数。

为了检测电子邮件地址，我们将使用一个简单的正则表达式模式:

```
([a-z0-9_.-]+)@([a-z0-9_.-]+[a-z])
```

请注意，这只是一个用于演示目的的简单模式，因为用于匹配有效电子邮件地址的实际正则表达式相当复杂。

我们需要在一个`Pattern`对象中使用这个正则表达式，这样我们就可以使用它:

```
Pattern EMAIL_ADDRESS_PATTERN = 
  Pattern.compile("([a-z0-9_.-]+)@([a-z0-9_.-]+[a-z])");
```

我们将研究两种主要方法，其中一种依赖于使用 Java 9 或更高版本。

对于我们的示例文本，我们将尝试在字符串中找到三封电子邮件:

```
"You can contact me through [[email protected]](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection), [[email protected]](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection), and [[email protected]](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection)"
```

## 3.计算 Java 8 和更早版本的匹配数

首先，让我们看看如何使用 Java 8 或更早版本来计算匹配数。

计算匹配数的一个简单方法是迭代`Matcher`类的 [`find`](https://web.archive.org/web/20220626203042/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Matcher.html#find()) 方法。这个方法试图**找到匹配模式**的输入序列的下一个子序列:

```
Matcher countEmailMatcher = EMAIL_ADDRESS_PATTERN.matcher(TEXT_CONTAINING_EMAIL_ADDRESSES);

int count = 0;
while (countEmailMatcher.find()) {
    count++;
}
```

使用这种方法，我们将找到三个匹配项，正如我们所料:

```
assertEquals(3, count);
```

请注意，`find`方法不会在每次找到匹配后重置`Matcher`——它会从上一个匹配序列结束后的字符开始恢复，因此它无法找到重叠的电子邮件地址。

例如，让我们考虑这个例子:

```
String OVERLAPPING_EMAIL_ADDRESSES = "Try to contact us at [[email protected]](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection)@baeldung.com, [[email protected]](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection)";

Matcher countOverlappingEmailsMatcher = EMAIL_ADDRESS_PATTERN.matcher(OVERLAPPING_EMAIL_ADDRESSES);

int count = 0;
while (countOverlappingEmailsMatcher.find()) {
    count++;
}

assertEquals(2, count);
```

当正则表达式试图在给定的`String, `中查找匹配项时，它首先会找到“[【电子邮件保护】](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection)”作为匹配项。因为@前面没有域部分，所以标记不会被重置，第二个`“@baeldung.com”`会被忽略。接下来，它还会将“[【电子邮件保护】](/web/20220626203042/https://www.baeldung.com/cdn-cgi/l/email-protection)”视为第二个匹配:

[![](img/dbab1334670a1f78b167205cbbed039b.png)](/web/20220626203042/https://www.baeldung.com/wp-content/uploads/2020/07/match-regex.png)

如上所示，在重叠的电子邮件示例中，我们只有两个匹配项。

## 4.Java 9 和更高版本的匹配计数

然而，如果我们有新版本的 Java 可用，我们可以使用`Matcher`类的 [`results​`](https://web.archive.org/web/20220626203042/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Matcher.html#results()) 方法。Java 9 中添加的这个方法返回一个连续的匹配结果流，使我们能够更容易地计算匹配数:

```
long count = countEmailMatcher.results()
  .count();

assertEquals(3, count);
```

就像我们看到的`find`一样，在处理来自`results`方法的流时，`Matcher`不会被重置。类似地，`results`方法也不能找到重叠的匹配。

## 5.结论

在这篇短文中，我们学习了如何计算正则表达式的匹配次数。

首先，我们学习了如何在一个`while` 循环中使用`find`方法。然后，我们看到了新的 Java 9 流方法如何让我们用更少的代码做到这一点。

与往常一样，代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220626203042/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)