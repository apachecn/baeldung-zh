# 将字符串转换为字符串数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-string-to-string-array>

## 1.概观

[`String`](/web/20221106080037/https://www.baeldung.com/java-string) 大概是 Java 中最常用的类型之一。

在本教程中，我们将探索如何将一个`String`转换成一个`String`数组(`String[]`)。

## 2.问题简介

将字符串转换为字符串数组可能有两种情况:

*   将字符串转换为单一数组(只有一个元素的数组)
*   按照特定规则将字符串分解成数组元素

案例 1 相对容易理解。例如，如果我们有一个字符串`“baeldung”`，我们想把它转换成`String[]{ “baeldung” }`。换句话说，**转换后的数组只有一个元素，就是输入字符串本身**。

对于第二种情况，我们需要将输入字符串分成几部分。然而，结果应该如何完全取决于需求。例如，如果我们期望最终数组中的每个元素包含来自输入字符串的两个相邻字符，给定`“baeldung”,`我们稍后将有 `String[]{ “ba”, “el”, “du”, “ng” }.`，我们将看到更多的例子。

在本教程中，我们将把这个字符串作为输入:

```java
String INPUT = "Hi there, nice to meet you!"; 
```

当然，我们将涵盖这两种转换场景。此外，为了简单起见，我们将使用单元测试断言来验证我们的解决方案是否如预期的那样工作。

## 3.转换为单一数组

由于输入字符串将是目标数组中的唯一元素，**我们可以简单地用输入字符串初始化数组来解决问题**:

```java
String[] myArray = new String[] { INPUT };
assertArrayEquals(new String[] { "Hi there, nice to meet you!" }, myArray);
```

然后，如果我们运行测试，它通过。

## 4.将输入字符串转换为数组元素

现在，让我们看看如何将输入字符串分成几段。

### 4.1.使用`String`的`split()`方法

**我们经常需要处理特定模式的输入字符串。**在这种情况下，我们可以使用正则表达式或 [regex](/web/20221106080037/https://www.baeldung.com/regular-expressions-java) 将输入分解成一个`String`数组。 **Java 的`String`类提供了 [`split()`](/web/20221106080037/https://www.baeldung.com/string/split) 方法来完成这项工作**。

接下来，我们将按照几个不同的要求把我们的输入例子分解成一个数组。

首先，假设我们想把输入的句子分成一组子句。为了解决这个问题，我们可以用标点符号分割输入字符串:

```java
String[] myArray = INPUT.split("[-,.!;?]\\s*" );
assertArrayEquals(new String[] { "Hi there", "nice to meet you" }, myArray);
```

值得一提的是**当我们需要一个 regex 的字符类包含一个破折号字符时，我们可以把它放在最开始的**。

上面的测试显示输入字符串在一个数组中被分成两个子句。

接下来，让我们将同一输入字符串中的所有单词提取到一个单词数组中。这也是我们在现实世界中可能面临的常见问题。

为了得到单词数组，我们可以通过非单词字符(`\W+`)来分割输入:

```java
String[] myArray = INPUT.split("\\W+");
assertArrayEquals(new String[] { "Hi", "there", "nice", "to", "meet", "you" }, myArray);
```

最后，让我们将输入字符串分解成字符:

```java
String[] myArray = INPUT.split("");
assertArrayEquals(new String[] {
    "H", "i", " ", "t", "h", "e", "r", "e", ",", " ",
    "n", "i", "c", "e", " ", "t", "o", " ", "m", "e", "e", "t", " ", "y", "o", "u", "!"
}, myArray);
```

如上面的代码所示，我们使用一个空字符串(零宽度)作为正则表达式。每个字符，包括输入字符串中的空格，都被提取为目标数组的一个元素。

我们应该注意到**`String.toCharArray()`也将输入转换成一个数组。但是，目标数组是一个`char`数组(`char[]`)，而不是一个`String`数组(`String[]` )** 。

这三个例子使用了`String.split()`方法将输入字符串转换成不同的字符串数组。一些流行的库，如 [Guava](/web/20221106080037/https://www.baeldung.com/guava-guide) 和 [Apache Commons](/web/20221106080037/https://www.baeldung.com/java-commons-lang-3) ，也提供了增强的字符串分割功能。我们在[的另一篇文章](/web/20221106080037/https://www.baeldung.com/java-split-string)中详细讨论过。

此外，我们还有许多其他文章讨论如何[解决不同的具体分裂问题](/web/20221106080037/https://www.baeldung.com/?s=split+string)。

### 4.2.特殊解析要求

有时我们必须遵循特定的规则来拆分输入。举个例子就能很快澄清。假设我们有这个输入字符串:

```java
String FLIGHT_INPUT = "20221018LH720FRAPEK";
```

我们期望得到这个数组的结果:

```java
{ "20221018", "LH720", "FRA", "PEK" }
```

嗯，乍一看，这个转换逻辑看起来晦涩难懂。但是，如果我们列出输入字符串的定义，我们就会明白为什么上面的数组是预期的:

```java
[date][Flight number][Airport from][Airport to]
- date: YYYY-MM-DD; length:8
- Flight number; length: variable
- Airport From: IATA airport code, length:3
- Airport To: IATA airport code, length:3 
```

正如我们所看到的，有时我们需要遵循一个非常特殊的规则来解析输入字符串。在这种情况下，**我们需要分析需求并实现一个解析器**:

```java
String dateStr = FLIGHT_INPUT.substring(0, 8);
String flightNo = FLIGHT_INPUT.substring(8, FLIGHT_INPUT.length() - 6);
int airportStart = dateStr.length() + flightNo.length();
String from = FLIGHT_INPUT.substring(airportStart, airportStart + 3);
String to = FLIGHT_INPUT.substring(airportStart + 3);

String[] myArray = new String[] { dateStr, flightNo, from, to };
assertArrayEquals(new String[] { "20221018", "LH720", "FRA", "PEK" }, myArray);
```

如上面的代码所示，我们已经使用`substring()`方法构建了一个解析器，并正确地处理了航班输入。

## 5.结论

在本文中，我们学习了如何在 Java 中将一个`String`转换成一个`String`数组。

简单地说，将一个字符串转换成一个单独的数组非常简单。如果我们需要把给定的字符串分成几段，我们可以求助于`String.split()`方法。但是，如果我们需要按照特定的规则分解输入，我们可能需要仔细分析输入格式，并实现一个解析器来解决问题。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221106080037/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)