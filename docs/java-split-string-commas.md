# 拆分逗号分隔的字符串时忽略引号中的逗号

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-split-string-commas>

## 1.概观

当处理包含逗号分隔值的文本时，可能需要忽略出现在带引号的子字符串中的逗号。

在本教程中，我们将探索在拆分逗号分隔的`String`时忽略引号内逗号的不同方法。

## 2.问题陈述

假设我们需要分割以下逗号分隔的输入:

```java
String input = "baeldung,tutorial,splitting,text,\"ignoring this comma,\"";
```

在分割这个输入并打印结果之后，我们期望得到以下输出:

```java
baeldung
tutorial
splitting
text
"ignoring this comma,"
```

换句话说，我们不能认为所有的逗号字符都是分隔符。我们必须忽略引用的子字符串中出现的逗号。

## 3.实现一个简单的解析器

让我们创建一个简单的解析算法:

```java
List<String> tokens = new ArrayList<String>();
int startPosition = 0;
boolean isInQuotes = false;
for (int currentPosition = 0; currentPosition < input.length(); currentPosition++) {
    if (input.charAt(currentPosition) == '\"') {
        isInQuotes = !isInQuotes;
    }
    else if (input.charAt(currentPosition) == ',' && !isInQuotes) {
        tokens.add(input.substring(startPosition, currentPosition));
        startPosition = currentPosition + 1;
    }
}

String lastToken = input.substring(startPosition);
if (lastToken.equals(",")) {
    tokens.add("");
} else {
    tokens.add(lastToken);
}
```

这里，我们首先定义一个名为`tokens`的`List`，它负责存储所有逗号分隔的值。

接下来，我们迭代输入`String`中的字符。

**在每次循环迭代中，我们需要检查当前字符是否是双引号**。当发现双引号时，我们使用`isInQuotes`标志来表示双引号后面的所有逗号都应该被忽略。当我们发现双引号时,`isInQuotes`标志将被设置为 false。

**当`isInQuotes`是`false,`并且我们发现一个逗号字符时，一个新的令牌将被添加到`tokens`列表中。**新令牌将包含从`startPosition`到逗号字符前最后一个位置的字符。

然后，新的`startPosition`将是逗号字符之后的位置。

最后，在循环之后，我们仍然拥有从`startPosition`到输入的最后一个位置的最后一个令牌。因此，我们使用`substring()`方法来获取它。如果这最后一个标记只是一个逗号，则意味着最后一个标记应该是一个空字符串。否则，我们将最后一个令牌添加到`tokens`列表中。

**现在，让我们测试解析代码:**

```java
String input = "baeldung,tutorial,splitting,text,\"ignoring this comma,\"";
var matcher = contains("baeldung", "tutorial", "splitting", "text", "\"ignoring this comma,\"");
assertThat(splitWithParser(input), matcher);
```

这里，我们在一个名为`splitWithParser`的静态方法中实现了解析代码。然后，在我们的测试中，我们定义了一个简单的测试`input`，它包含一个用双引号括起来的逗号。接下来，我们使用 [hamcrest 测试框架](/web/20220525132445/https://www.baeldung.com/java-junit-hamcrest-guide)为预期输出创建一个`contains` `matcher`。最后，我们使用`assertThat`测试方法来检查我们的解析器是否返回了预期的输出。

在实际场景中，我们应该创建更多的单元测试，用其他可能的输入来验证我们算法的行为。

## 4.应用正则表达式

实现解析器是一种有效的方法。然而，由此产生的算法相对较大且复杂。因此，作为替代，我们可以使用正则表达式。

接下来，我们将讨论依赖正则表达式的两种可能的实现。然而，应谨慎使用它们，因为与前一种方法相比，它们的处理时间较长。因此，在处理大量输入数据时，禁止在这种情况下使用正则表达式。

### 4.1.`String split()`方法

在第一个正则表达式选项中，我们将使用来自`String`类的[方法`split()`。**该方法在给定正则表达式:**的匹配周围拆分`String`](/web/20220525132445/https://www.baeldung.com/java-split-string)

```java
String[] tokens = input.split(",(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)", -1);
```

乍一看，正则表达式似乎非常复杂。然而，它的功能相对简单。

简而言之，**使用[正向前瞻](/web/20220525132445/https://www.baeldung.com/java-regex-lookahead-lookbehind)，告诉仅当逗号前面没有双引号或者有偶数个双引号时，才在逗号周围拆分。**

`split()`方法的最后一个参数是极限。当我们提供一个负的限制时，这个模式被应用尽可能多的次数，得到的令牌数组可以有任意长度。

### 4.2.番石榴的`Splitter`课

另一种基于正则表达式的方法是使用 Guava 库中的`Splitter`类:

```java
Pattern pattern = Pattern.compile(",(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)");
Splitter splitter = Splitter.on(pattern);
List<String> tokens = splitter.splitToList(input);
```

这里，我们基于与之前相同的正则表达式模式创建一个`splitter`对象。在创建了`splitter`之后，我们使用`splitToList()`方法，该方法在拆分输入`String`之后返回一个`List`令牌。

## 5.使用 CSV 库

尽管提出的替代方案很有趣，但是可能有必要使用一个 CSV 解析库，比如 T2 的 OpenCSV。

使用 CSV 库的优点是需要更少的努力，因为我们不需要编写解析器或复杂的正则表达式。结果，我们的代码不容易出错，也更容易维护。

此外，当我们不确定输入的形状时，CSV 库可能是最好的方法。例如，输入可能有转义引号，这是以前的方法无法正确处理的。

要使用 OpenCSV，我们需要将它作为一个依赖项包含进来。在一个 Maven 项目中，我们包含了 [opencsv 依赖关系](https://web.archive.org/web/20220525132445/https://search.maven.org/artifact/com.opencsv/opencsv):

```java
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>4.1</version>
</dependency>
```

然后，我们可以如下使用 OpenCSV:

```java
CSVParser parser = new CSVParserBuilder()
  .withSeparator(',')
  .build();

CSVReader reader = new CSVReaderBuilder(new StringReader(input))
  .withCSVParser(parser)
  .build();

List<String[]> lines = new ArrayList<>();
lines = reader.readAll();
reader.close();
```

使用`CSVParserBuilder`类，我们首先创建一个带有逗号分隔符的解析器。然后，我们使用`CSVReaderBuilder`来创建一个基于逗号解析器的 CSV 阅读器。

在我们的例子中，我们提供了一个`StringReader`作为`CSVReaderBuilder`构造函数的参数。但是，如果需要，我们可以使用不同的阅读器(例如，文件阅读器)。

最后，我们从我们的`reader`对象调用`readAll()`方法来获得`String`数组的`List`。因为 OpenCSV 是为处理多行输入而设计的，所以`lines`列表中的每个位置都对应于我们输入的一行。因此，对于每一行，我们都有一个包含相应逗号分隔值的`String`数组。

与以前的方法不同，使用 OpenCSV，双引号从生成的输出中删除。

## 6.结论

在本文中，我们探索了在拆分逗号分隔的`String`时忽略引号中的逗号的多种替代方法。除了学习如何实现我们自己的解析器，我们还探索了正则表达式和 OpenCSV 库的使用。

和往常一样，本教程中使用的代码示例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220525132445/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)