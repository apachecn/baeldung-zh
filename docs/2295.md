# 在 Java 中分割字符串并保留分隔符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-split-string-keep-delimiters>

## 1.介绍

程序员经常会遇到涉及拆分字符串的算法。在一个特殊的场景中，可能需要根据单个或多个不同的分隔符来拆分字符串，并且**也将分隔符作为拆分操作**的一部分返回。

让我们详细讨论一下这个`String`分裂问题的不同解决方案。

## 2.基本原则

Java universe 提供了相当多的库(`java.lang.String`、Guava 和 Apache Commons，仅举几个例子)来帮助在简单和相当复杂的情况下分割字符串。此外，功能丰富的[正则表达式](/web/20221128043619/https://www.baeldung.com/regular-expressions-java)为围绕特定模式匹配的拆分问题提供了额外的灵活性。

## 3.环视断言

在正则表达式中，look-around 断言表明在源字符串的当前位置，通过对另一个模式进行前视(lookahead)或后视(look ahead ),匹配是可能的。让我们用一个例子来更好地理解这一点。

前视断言`Java(?=Baeldung)` **只有在`“Java”`后面跟有`“Baeldung”`** 时才与`“Java”`匹配。

同样，负的后视断言`(?<!#)\d+`只匹配前面没有“#”的数字。

让我们使用这样的环顾断言正则表达式，并设计一个解决我们的问题。

在本文解释的所有例子中，我们将使用两个简单的`String`:

```
String text = "[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)@[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)@[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)@Program";
String textMixed = "@[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection):[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)#Java#Program";
```

## 4.使用`String.split()`

让我们从使用核心 Java 库的`String`类中的`split()`方法开始。

此外，我们将评估适当的前视断言、后视断言以及它们的组合，以便根据需要拆分字符串。

### 4.1.积极前瞻

首先，让我们使用 lookahead 断言`“(([[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))”` 并将字符串`text`拆分到其匹配项周围:

```
String[] splits = text.split("(([[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))");
```

前瞻正则表达式通过向前匹配`“@”`符号来分割字符串。结果数组的内容是:

```
[Hello, @World, @This, @Is, @A, @Java, @Program]
```

使用这个正则表达式不会在`splits`数组中单独返回分隔符。让我们试试另一种方法。

### 4.2.积极回顾

我们也可以使用肯定的后视断言`“((?<[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))”`来拆分字符串`text`:

```
String[] splits = text.split("((?<[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))");
```

但是，结果输出仍然不包含作为数组单个元素的分隔符:

```
[[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection), [[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection), [[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection), [[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection), [[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection), [[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection), Program]
```

### 4.3.正向前视或后视

我们可以用逻辑“或”来组合上述两种方法，并观察它的实际应用。

**得到的正则表达式`“(([[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection))|(?<[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))”`肯定会给我们想要的结果。**下面的代码片段演示了这一点:

```
String[] splits = text.split("(([[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection))|(?<[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))");
```

上面的正则表达式拆分字符串，得到的数组包含分隔符:

```
[Hello, @, World, @, This, @, Is, @, A, @, Java, @, Program]
```

既然我们已经理解了所需的 look-around 断言正则表达式，我们可以根据输入字符串中出现的不同类型的分隔符来修改它。

让我们尝试使用一个合适的正则表达式来分割前面定义的`textMixed `:

```
String[] splitsMixed = textMixed.split("((?=:|#|@)|(?<=:|#|@))");
```

在执行上面的代码行之后，看到下面的结果并不奇怪:

```
[@, HelloWorld, @, This, :, Is, @, A, #, Java, #, Program]
```

## 5.使用番石榴`Splitter`

考虑到现在我们已经清楚了上一节讨论的正则表达式断言，让我们深入研究 Google 提供的 Java 库。

来自 [Guava](/web/20221128043619/https://www.baeldung.com/guava-guide) 的`Splitter`类提供了方法 [`on()`](https://web.archive.org/web/20221128043619/https://guava.dev/releases/23.0/api/docs/com/google/common/base/Splitter.html#on-java.util.regex.Pattern-) 和 [`onPattern()`](https://web.archive.org/web/20221128043619/https://guava.dev/releases/23.0/api/docs/com/google/common/base/Splitter.html#onPattern-java.lang.String-) 来使用正则表达式模式作为分隔符分割字符串。

首先，让我们看看它们在包含单个分隔符`“@”`的字符串`text `上的作用:

```
List<String> splits = Splitter.onPattern("(([[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection))|(?<[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))").splitToList(text);
List<String> splits2 = Splitter.on(Pattern.compile("(([[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection))|(?<[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)))")).splitToList(text);
```

执行上面几行代码的结果与用`split `方法生成的结果非常相似，除了我们现在用`List`代替了数组。

同样，我们也可以使用这些方法来拆分包含多个不同分隔符的字符串:

```
List<String> splitsMixed = Splitter.onPattern("((?=:|#|@)|(?<=:|#|@))").splitToList(textMixed);
List<String> splitsMixed2 = Splitter.on(Pattern.compile("((?=:|#|@)|(?<=:|#|@))")).splitToList(textMixed);
```

正如我们所看到的，上述两种方法之间的差异是相当明显的。

`on()`方法接受一个参数`java.util.regex.Pattern`，而`onPattern()`方法只接受分隔符正则表达式作为`String`。

## 6.使用 Apache Commons `StringUtils`

我们还可以利用 [Apache Commons Lang](/web/20221128043619/https://www.baeldung.com/java-commons-lang-3) 项目的`StringUtils`方法`splitByCharacterType().`

值得注意的是，这个**方法的工作原理是按照 [`java.lang.Character.getType(char)`](https://web.archive.org/web/20221128043619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#getType(char)) 返回的字符类型分割输入字符串。在这里，我们不能挑选或提取我们选择的分隔符。**

此外，当源字符串始终保持大小写不变时，它会提供最佳结果:

```
String[] splits = StringUtils.splitByCharacterType("[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection);[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection);[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection);[[email protected]](/web/20221128043619/https://www.baeldung.com/cdn-cgi/l/email-protection)#10words;Java#Program");
```

上面字符串中的不同字符类型是大写和小写字母、数字和特殊字符(@；# ).

因此，得到的数组`splits,` 如预期的那样，看起来像:

```
[pg, @, no, ;, 10, @, hello, ;, world, @, this, ;, is, @, a, #, 10, words, ;, J, ava, #, P, rogram]
```

## 7.结论

在本文中，我们已经看到了如何以这样一种方式分割一个字符串，使得分隔符在结果数组中也是可用的。

首先，我们讨论了环视断言，并使用它们来获得期望的结果。后来，我们使用番石榴库提供的方法也获得了类似的结果。

最后，我们总结了 Apache Commons Lang 库，它提供了一种更加用户友好的方法来解决分割字符串的相关问题，并返回分隔符。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128043619/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)