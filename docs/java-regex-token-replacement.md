# Java 中如何用正则表达式替换字符串中的记号

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regex-token-replacement>

## 1.概观

当我们需要在 Java 中查找或替换字符串中的值时，我们通常使用[正则表达式](/web/20220820151444/https://www.baeldung.com/regular-expressions-java)。这些允许我们确定[一个字符串的部分或全部是否匹配](/web/20220820151444/https://www.baeldung.com/java-matcher-find-vs-matches)一个模式。在`Matcher`和`String`中使用`replaceAll`方法，我们可以**轻松地** **对一个字符串中的多个令牌应用相同的替换。**

在本教程中，我们将探索如何为字符串中的每个标记应用不同的替换。这将使我们很容易满足像转义某些字符或替换占位符值这样的用例。

我们还将看看一些调整正则表达式以正确识别标记的技巧。

## 2.单独处理匹配

在我们能够构建我们的逐标记替换算法之前，我们需要理解围绕正则表达式的 Java API。让我们使用捕获组和非捕获组来解决一个棘手的匹配问题。

### 2.1.标题案例示例

假设我们想要构建一个算法来处理一个字符串中的所有标题词。这些单词以一个大写字符开始，然后以小写字符结束或继续。

我们的输入可能是:

```java
"First 3 Capital Words! then 10 TLAs, I Found"
```

从标题词的定义来看，这包含了匹配项:

*   `First`
*   `Capital`
*   `Words`
*   `I`
*   `Found`

识别这种模式的正则表达式是:

```java
"(?<=^|[^A-Za-z])([A-Z][a-z]*)(?=[^A-Za-z]|$)"
```

为了理解这一点，让我们把它分解成它的组成部分。我们从中间开始:

```java
[A-Z]
```

将识别单个大写字母。

我们允许单字符单词或后跟小写字母的单词，因此:

```java
[a-z]*
```

识别零个或多个小写字母。

在某些情况下，上述两个字符类足以识别我们的令牌。不幸的是，在我们的示例文本中，有一个以多个大写字母开头的单词。所以，**我们需要表示我们找到的单个大写字母一定是第一个出现在非字母之后的。**

类似地，当我们允许单个大写字母单词时，我们需要表达我们找到的单个大写字母不能是一个多大写字母单词的第一个。

`[^A-Za-z] `这个表达的意思是“没有字母”。在非捕获组中，我们将其中一个放在表达式的开头:

```java
(?<=^|[^A-Za-z])
```

非捕获组从`(?<=,` 开始进行**回顾，以确保匹配出现在正确的边界。**结尾的对应部分为后面的角色做同样的工作。

然而，如果单词触及字符串的开头或结尾，那么我们需要考虑这一点，这就是我们在第一个组中添加^|，使其表示“字符串或任何非字母字符的开头”，并且我们在最后一个非捕获组的末尾添加|$以允许字符串的结尾作为边界。

**当我们使用`find`时，在非捕获组中发现的字符不会出现在匹配**中。

我们应该注意，即使像这样简单的用例也可能有许多边缘情况，所以测试我们的正则表达式很重要。为此，我们可以编写单元测试，使用我们 IDE 的内置工具，或者使用像 [Regexr](https://web.archive.org/web/20220820151444/https://regexr.com/4vdaf) 这样的在线工具。

### 2.2.测试我们的示例

我们的示例文本在一个名为`EXAMPLE_INPUT`的常量中，正则表达式在一个名为`TITLE_CASE_PATTERN`的`Pattern`中，让我们使用`Matcher` 类上的`find`来提取单元测试中的所有匹配:

```java
Matcher matcher = TITLE_CASE_PATTERN.matcher(EXAMPLE_INPUT);
List<String> matches = new ArrayList<>();
while (matcher.find()) {
    matches.add(matcher.group(1));
}

assertThat(matches)
  .containsExactly("First", "Capital", "Words", "I", "Found");
```

这里我们使用`Pattern`上的`matcher`函数来产生一个`Matcher`。然后我们在一个循环中使用`find`方法，直到它停止返回`true `来迭代所有的匹配。

每次`find`返回`true`时，`Matcher`对象的状态被设置为代表当前匹配。我们可以使用`group(0)` **检查整个匹配，或者使用基于 1 的索引**检查特定的捕获组。在这种情况下，在我们想要的片段周围有一个捕获组，所以我们使用`group(1)`将匹配添加到我们的列表中。

### 2.3.再多检查一点

到目前为止，我们已经找到了想要处理的单词。

但是，如果这些单词中的每一个都是我们想要替换的标记，我们就需要更多的匹配信息来构建结果字符串。**让我们看看`Matcher`** 的一些其他属性，它们可能会对我们有所帮助:

```java
while (matcher.find()) {
    System.out.println("Match: " + matcher.group(0));
    System.out.println("Start: " + matcher.start());
    System.out.println("End: " + matcher.end());
}
```

这段代码会告诉我们每个匹配的位置。它还向我们展示了`group(0)`匹配，这是捕获的所有内容:

```java
Match: First
Start: 0
End: 5
Match: Capital
Start: 8
End: 15
Match: Words
Start: 16
End: 21
Match: I
Start: 37
End: 38
... more
```

这里我们可以看到每个匹配只包含我们期望的单词。**`start`属性显示了字符串中匹配**的从零开始的索引。`end`显示紧随其后的字符的索引。这意味着我们可以使用`substring(start, end-start)`从原始字符串中提取每个匹配。这基本上就是`group`方法为我们所做的。

既然我们可以使用`find`迭代匹配，让我们处理我们的令牌。

## 3.逐一替换火柴

让我们继续我们的例子，使用我们的算法将原始字符串中的每个标题词替换为小写的等价词。这意味着我们的测试字符串将被转换为:

```java
"first 3 capital words! then 10 TLAs, i found"
```

而`Pattern`和`Matcher`类无法为我们做到这一点，所以我们需要构造一个算法。

### 3.1.替换算法

下面是算法的伪代码:

*   从一个空输出字符串开始
*   对于每场比赛:
    *   将匹配之前和之前匹配之后的任何内容添加到输出中
    *   处理这个匹配并将其添加到输出中
    *   继续，直到处理完所有匹配
    *   将最后一次匹配后剩下的内容添加到输出中

我们应该注意到，这个算法的目的是**找到所有未匹配的区域，并将它们添加到输出**中，同时添加已处理的匹配。

### 3.2.Java 中的令牌替换器

我们想把每个单词都转换成小写，所以我们可以写一个简单的转换方法:

```java
private static String convert(String token) {
    return token.toLowerCase();
}
```

现在我们可以编写算法来迭代匹配。这可以使用一个`StringBuilder`来输出:

```java
int lastIndex = 0;
StringBuilder output = new StringBuilder();
Matcher matcher = TITLE_CASE_PATTERN.matcher(original);
while (matcher.find()) {
    output.append(original, lastIndex, matcher.start())
      .append(convert(matcher.group(1)));

    lastIndex = matcher.end();
}
if (lastIndex < original.length()) {
    output.append(original, lastIndex, original.length());
}
return output.toString();
```

我们应该注意到 **`StringBuilder`提供了一个方便的`append`版本，可以提取子字符串**。这与`Matcher`的`end`属性配合得很好，让我们挑选出自上次匹配以来所有未匹配的字符。

## 4.推广算法

既然我们已经解决了替换某些特定令牌的问题，为什么不将代码转换成一种可以用于一般情况的形式呢？不同实现之间唯一不同的是要使用的正则表达式，以及将每个匹配转换成其替换的逻辑。

### 4.1.使用函数和模式输入

我们可以使用 Java `Function<Matcher, String>`对象来允许调用者提供处理每个匹配的逻辑。我们可以接受一个名为`tokenPattern`的输入来查找所有的令牌:

```java
// same as before
while (matcher.find()) {
    output.append(original, lastIndex, matcher.start())
      .append(converter.apply(matcher));

// same as before
```

这里，正则表达式不再是硬编码的。相反，`converter`函数由调用者提供，并应用于`find`循环中的每个匹配。

### 4.2.测试通用版本

让我们看看通用方法是否和原始方法一样有效:

```java
assertThat(replaceTokens("First 3 Capital Words! then 10 TLAs, I Found",
  TITLE_CASE_PATTERN,
  match -> match.group(1).toLowerCase()))
  .isEqualTo("first 3 capital words! then 10 TLAs, i found");
```

这里我们看到调用代码很简单。转换函数很容易表示为λ。测试通过了。

现在我们有了一个令牌替换器，所以让我们尝试一些其他的用例。

## 5.一些使用案例

### 5.1.转义特殊字符

假设我们想使用正则表达式转义字符`\`来手动引用正则表达式的每个字符，而不是[使用`quote`方法](/web/20220820151444/https://www.baeldung.com/java-regexp-escape-char)。也许我们在创建传递给另一个库或服务的正则表达式时引用了一个字符串，所以用块引用表达式是不够的。

如果我们可以表达表示“正则表达式字符”的模式，就很容易使用我们的算法来避开它们:

```java
Pattern regexCharacters = Pattern.compile("[<(\\[{\\\\^\\-=$!|\\]})?*+.>]");

assertThat(replaceTokens("A regex character like [",
  regexCharacters,
  match -> "\\" + match.group()))
  .isEqualTo("A regex character like \\[");
```

对于每个匹配，我们给字符加上前缀`\`。因为`\`是 Java 字符串中的一个特殊字符，所以用另一个`\`对它进行了转义。

事实上，这个例子包含了额外的`\`字符，因为`regexCharacters`模式中的字符类必须引用许多特殊字符。这显示了正则表达式解析器，我们用它们来表示它们的文字，而不是作为正则表达式语法。

### 5.2.替换占位符

表达占位符的一种常见方式是使用类似于`${name}`的语法。让我们考虑一个用例，其中模板`“Hi ${name} at ${company}” `需要从名为`placeholderValues`的地图中填充:

```java
Map<String, String> placeholderValues = new HashMap<>();
placeholderValues.put("name", "Bill");
placeholderValues.put("company", "Baeldung");
```

**我们只需要一个好的正则表达式**来找到`${…}`令牌:

```java
"\\$\\{(?<placeholder>[A-Za-z0-9-_]+)}"
```

是一种选择。它必须引用`$`和最初的花括号，否则它们将被视为正则表达式语法。

这个模式的核心是占位符名称的捕获组。我们使用了一个允许字母数字、破折号和下划线的字符类，这应该适合大多数用例。

然而，**为了使代码更具可读性，我们将这个捕获组命名为** `placeholder`。让我们看看如何使用命名的捕获组:

```java
assertThat(replaceTokens("Hi ${name} at ${company}",
  "\\$\\{(?<placeholder>[A-Za-z0-9-_]+)}",
  match -> placeholderValues.get(match.group("placeholder"))))
  .isEqualTo("Hi Bill at Baeldung");
```

这里我们可以看到，从`Matcher`中获取命名组的值只需要使用`group `并将名称作为输入，而不是数字。

## 6.结论

在本文中，我们研究了如何使用强大的正则表达式在字符串中查找标记。我们学习了`find`方法如何与`Matcher`一起工作来显示匹配。

然后，我们创建并推广了一种算法，允许我们进行逐令牌替换。

最后，我们看了一些转义字符和填充模板的常见用例。

和往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220820151444/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex-2)