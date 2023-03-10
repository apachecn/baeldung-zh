# Java 中的正则表达式\s 和\s+

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regex-s-splus>

## 1.概观

字符串替换是我们在 Java 中处理字符串时的一个标准操作。

感谢 [`String`](/web/20220809232439/https://www.baeldung.com/tag/java-string/) 类中方便的`[replaceAll()](/web/20220809232439/https://www.baeldung.com/string/replace-all) `方法，我们可以轻松地用[正则表达式](/web/20220809232439/https://www.baeldung.com/regular-expressions-java)进行字符串替换。然而，有时这些表达会令人困惑，例如，`\s`和`\s+. `

在这个简短的教程中，我们将通过例子来看看这两个正则表达式之间的区别。

## 2. `\s`和`\s+`的区别

正则表达式`\s`是一个预定义的字符类。它表示一个空白字符。让我们回顾一下空白字符集:

```java
[ \t\n\x0B\f\r]
```

加号`+`是贪婪量词，表示一次或多次。例如，表达式`X+ `匹配一个或多个`X `字符。

因此，**正则表达式`\s`匹配单个空白字符，而`\` s+将匹配一个或多个空白字符。**

## 3.`replaceAll()`用非空替换

我们已经学习了正则表达式`\s` 和`\s+`的含义。

现在，让我们看看`replaceAll()`方法对这两个正则表达式的不同表现。

我们将使用一个字符串作为所有示例的输入文本:

```java
String INPUT_STR = "Text   With     Whitespaces!   ";
```

让我们尝试将`\s`作为参数传递给`replaceAll()`方法:

```java
String result = INPUT_STR.replaceAll("\\s", "_");
assertEquals("Text___With_____Whitespaces!___", result);
```

**`replaceAll()`方法查找单个空白字符并用下划线替换每个匹配。**我们在输入文本中有 11 个空白字符。因此，将发生 11 次替换。

接下来，让我们将正则表达式`\s+`传递给`replaceAll()`方法:

```java
String result = INPUT_STR.replaceAll("\\s+", "_");
assertEquals("Text_With_Whitespaces!_", result);
```

**由于贪婪的量词`+`,`replaceAll()`方法将匹配最长的连续空白字符序列，并用下划线替换每个匹配。**

在我们的输入文本中，我们有三个连续的空白字符序列。因此，这三个中的每一个都将成为下划线。

## 4.`replaceAll()`用空的替换

`replaceAll() `方法的另一个常见用法是从输入文本中删除匹配的模式。我们通常通过传递一个空字符串作为方法的替换来实现。

让我们看看，如果我们使用带有`\s` 正则表达式的`replaceAll()`方法删除空白字符，会得到什么结果:

```java
String result1 = INPUT_STR.replaceAll("\\s", "");
assertEquals("TextWithWhitespaces!", result1);
```

现在，我们将把另一个正则表达式`\s+`传递给`replaceAll()` 方法:

```java
String result2 = INPUT_STR.replaceAll("\\s+", "");
assertEquals("TextWithWhitespaces!", result2); 
```

因为替换是一个空字符串，所以两个`replaceAll()`调用产生相同的结果，即使两个正则表达式有不同的含义:

```java
assertEquals(result1, result2);
```

**如果我们比较两个`replaceAll()`调用，带`\s+`的调用效率更高。这是因为它只需要三次替换就可以完成这项工作，而使用`\s`的调用将需要十一次替换。**

## 5.结论

在这篇短文中，我们学习了正则表达式`\s`和`\s+`。

我们还看到了`replaceAll()`方法在这两个表达式中的不同表现。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220809232439/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)