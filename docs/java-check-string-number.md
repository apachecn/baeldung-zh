# 在 Java 中检查一个字符串是否是数字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-string-number>

## 1。简介

通常在操作`String` s 时，我们需要弄清楚`String`是否是一个有效的数字。

**在本教程中，我们将探索多种方法来检测给定的`String`是否是数字**，首先使用普通 Java，然后使用正则表达式，最后使用外部库。

一旦我们讨论完各种实现，我们将使用基准来了解哪些方法是最佳的。

## 延伸阅读:

## [Java 字符串转换](/web/20221017232038/https://www.baeldung.com/java-string-conversions)

Quick and practical examples focused on converting String objects to different data types in Java.[Read more](/web/20221017232038/https://www.baeldung.com/java-string-conversions) →

## Java 正则表达式 API 指南

A practical guide to Regular Expressions API in Java.[Read more](/web/20221017232038/https://www.baeldung.com/regular-expressions-java) →

## [理解 Java 中的 NumberFormatException](/web/20221017232038/https://www.baeldung.com/java-number-format-exception)

Learn the various causes of NumberFormatException in Java and some best practices for avoiding it.[Read more](/web/20221017232038/https://www.baeldung.com/java-number-format-exception) →

## 2。先决条件

在进入主要内容之前，让我们先了解一些先决条件。

在本文的后半部分，我们将使用 Apache Commons 外部库在我们的`pom.xml`中添加它的依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

这个库的最新版本可以在 [Maven Central](https://web.archive.org/web/20221017232038/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 上找到。

## 3。使用普通 Java

也许检查`String`是否为数字的最简单和最可靠的方法是使用 Java 的内置方法解析它:

1.  `Integer.parseInt(String)`
2.  `Float.parseFloat(String)`
3.  `Double.parseDouble(String)`
4.  `Long.parseLong(String)`
5.  `new BigInteger(String)`

如果这些方法没有抛出任何`[NumberFormatException](https://web.archive.org/web/20221017232038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NumberFormatException.html "class in java.lang")`，那么这意味着解析成功并且`String`是数字:

```java
public static boolean isNumeric(String strNum) {
    if (strNum == null) {
        return false;
    }
    try {
        double d = Double.parseDouble(strNum);
    } catch (NumberFormatException nfe) {
        return false;
    }
    return true;
}
```

让我们来看看这种方法的实际应用:

```java
assertThat(isNumeric("22")).isTrue();
assertThat(isNumeric("5.05")).isTrue();
assertThat(isNumeric("-200")).isTrue(); 
assertThat(isNumeric("10.0d")).isTrue();
assertThat(isNumeric("   22   ")).isTrue();

assertThat(isNumeric(null)).isFalse();
assertThat(isNumeric("")).isFalse();
assertThat(isNumeric("abc")).isFalse();
```

在我们的`isNumeric()`方法中，我们只是检查类型为`Double`的值；然而，我们也可以修改这个方法，通过使用我们之前使用的任何解析方法来检查`Integer`、`Float`、`Long`和大数。

这些方法在 [Java 字符串转换](/web/20221017232038/https://www.baeldung.com/java-string-conversions)一文中也有讨论。

## 4。使用正则表达式

现在让我们使用 regex `-?\d+(\.\d+)?`来匹配由正整数或负整数和浮点数组成的数字`Strings`。

不用说，我们肯定可以修改这个正则表达式来识别和处理各种各样的规则。在这里，我们将保持简单。

让我们分解这个正则表达式，看看它是如何工作的:

*   `-?`–该部分标识给定的数字是否为负数，破折号“`–`”按字面意思搜索破折号，问号“`?`”标记其为可选数字
*   `\d+`–搜索一个或多个数字
*   `(\.\d+)?`–正则表达式的这一部分用于识别浮点数。在这里，我们搜索一个或多个数字，后跟一个句点。最后，问号表示这个完整的组是可选的。

正则表达式是一个非常广泛的话题。为了获得一个简要的概述，请查看我们关于 [Java 正则表达式 API](https://web.archive.org/web/20221017232038/https://baeldung.com/regular-expressions-java) 的教程。

现在，让我们使用上面的正则表达式创建一个方法:

```java
private Pattern pattern = Pattern.compile("-?\\d+(\\.\\d+)?");

public boolean isNumeric(String strNum) {
    if (strNum == null) {
        return false; 
    }
    return pattern.matcher(strNum).matches();
}
```

现在让我们来看看上面方法的一些断言:

```java
assertThat(isNumeric("22")).isTrue();
assertThat(isNumeric("5.05")).isTrue();
assertThat(isNumeric("-200")).isTrue();

assertThat(isNumeric(null)).isFalse();
assertThat(isNumeric("abc")).isFalse();
```

## 5。使用 Apache Commons

在这一节中，我们将讨论 Apache Commons 库中可用的各种方法。

### 5.1。`NumberUtils.isCreatable(String)`

Apache Commons 的`[NumberUtils](https://web.archive.org/web/20221017232038/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/NumberUtils.html)`提供了一个静态方法`[NumberUtils.isCreatable(String)](https://web.archive.org/web/20221017232038/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/NumberUtils.html#isCreatable-java.lang.String-),`，它检查一个`String`是否是一个有效的 Java 号码。

该方法接受:

1.  以 0x 或 0X 开头的十六进制数字
2.  以 0 开头的八进制数
3.  科学记数法(例如 1.05e-10)
4.  用类型限定符标记的数字(例如 1L 或 2.2d)

如果提供的字符串是`null`或`empty/blank`，那么它不被认为是一个数字，该方法将返回`false`。

让我们使用这种方法运行一些测试:

```java
assertThat(NumberUtils.isCreatable("22")).isTrue();
assertThat(NumberUtils.isCreatable("5.05")).isTrue();
assertThat(NumberUtils.isCreatable("-200")).isTrue();
assertThat(NumberUtils.isCreatable("10.0d")).isTrue();
assertThat(NumberUtils.isCreatable("1000L")).isTrue();
assertThat(NumberUtils.isCreatable("0xFF")).isTrue();
assertThat(NumberUtils.isCreatable("07")).isTrue();
assertThat(NumberUtils.isCreatable("2.99e+8")).isTrue();

assertThat(NumberUtils.isCreatable(null)).isFalse();
assertThat(NumberUtils.isCreatable("")).isFalse();
assertThat(NumberUtils.isCreatable("abc")).isFalse();
assertThat(NumberUtils.isCreatable(" 22 ")).isFalse();
assertThat(NumberUtils.isCreatable("09")).isFalse();
```

注意，我们在第 6、7 和 8 行中分别得到了十六进制数、八进制数和科学符号的`true`断言。

同样，在第 14 行，字符串`“09”`返回`false`，因为前面的`“0”`表示这是一个八进制数，而`“09”`不是一个有效的八进制数。

对于用这种方法返回`true`的每个输入，我们可以使用 [`NumberUtils.createNumber(String)`](https://web.archive.org/web/20221017232038/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/NumberUtils.html#createNumber-java.lang.String-) ，这将给出有效的数字。

### 5.2。`NumberUtils.isParsable(String)`

`[NumberUtils.isParsable(String)](https://web.archive.org/web/20221017232038/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/NumberUtils.html#isParsable-java.lang.String-)`方法检查给定的`String`是否可解析。

**可解析的数字是指通过`Integer.parseInt(String)`、`Long.parseLong(String)`、`Float.parseFloat(String)`或`Double.parseDouble(String)`等任何解析方法解析成功的数字。**

与`NumberUtils.isCreatable()`不同，该方法不接受十六进制数字、科学符号或以任何类型的限定符如`‘f', ‘F', ‘d' ,'D' ,'l'`或`‘L'`结尾的字符串。

让我们来看看一些肯定的说法:

```java
assertThat(NumberUtils.isParsable("22")).isTrue();
assertThat(NumberUtils.isParsable("-23")).isTrue();
assertThat(NumberUtils.isParsable("2.2")).isTrue();
assertThat(NumberUtils.isParsable("09")).isTrue();

assertThat(NumberUtils.isParsable(null)).isFalse();
assertThat(NumberUtils.isParsable("")).isFalse();
assertThat(NumberUtils.isParsable("6.2f")).isFalse();
assertThat(NumberUtils.isParsable("9.8d")).isFalse();
assertThat(NumberUtils.isParsable("22L")).isFalse();
assertThat(NumberUtils.isParsable("0xFF")).isFalse();
assertThat(NumberUtils.isParsable("2.99e+8")).isFalse();
```

在第 4 行，与`NumberUtils.isCreatable()`不同，以字符串`“0”`开头的数字不是八进制数，而是普通的十进制数，因此它返回 true。

我们可以用这个方法代替我们在第 3 节中所做的，在第 3 节中，我们试图解析一个数字并检查错误。

### 5.3。`StringUtils.isNumeric(CharSequence`T3****

方法`[StringUtils.isNumeric(CharSequence)](https://web.archive.org/web/20221017232038/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isNumeric-java.lang.CharSequence-)`严格检查 Unicode 数字。这意味着:

1.  任何语言的 Unicode 数字都是可以接受的
2.  因为小数点不被认为是 Unicode 数字，所以它是无效的
3.  前导符号(正或负)也是不可接受的

现在让我们来看看这个方法的实际应用:

```java
assertThat(StringUtils.isNumeric("123")).isTrue();
assertThat(StringUtils.isNumeric("١٢٣")).isTrue();
assertThat(StringUtils.isNumeric("१२३")).isTrue();

assertThat(StringUtils.isNumeric(null)).isFalse();
assertThat(StringUtils.isNumeric("")).isFalse();
assertThat(StringUtils.isNumeric("  ")).isFalse();
assertThat(StringUtils.isNumeric("12 3")).isFalse();
assertThat(StringUtils.isNumeric("ab2c")).isFalse();
assertThat(StringUtils.isNumeric("12.3")).isFalse();
assertThat(StringUtils.isNumeric("-123")).isFalse();
```

注意，第 2 行和第 3 行中的输入参数分别用阿拉伯语和梵文表示数字`123`。因为它们是有效的 Unicode 数字，所以这个方法对它们返回`true`。

### 5.4。`StringUtils.isNumericSpace(CharSequence)`

[`StringUtils.isNumericSpace(CharSequence)`](https://web.archive.org/web/20221017232038/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isNumericSpace-java.lang.CharSequence-) 严格检查 Unicode 数字和/或空格。这与`StringUtils.isNumeric()` 相同，除了它也接受空格，不仅是前导空格和尾随空格，而且如果它们在数字之间:

```java
assertThat(StringUtils.isNumericSpace("123")).isTrue();
assertThat(StringUtils.isNumericSpace("١٢٣")).isTrue();
assertThat(StringUtils.isNumericSpace("")).isTrue();
assertThat(StringUtils.isNumericSpace("  ")).isTrue();
assertThat(StringUtils.isNumericSpace("12 3")).isTrue();

assertThat(StringUtils.isNumericSpace(null)).isFalse();
assertThat(StringUtils.isNumericSpace("ab2c")).isFalse();
assertThat(StringUtils.isNumericSpace("12.3")).isFalse();
assertThat(StringUtils.isNumericSpace("-123")).isFalse();
```

## 6。基准测试

在结束本文之前，让我们看一些基准测试结果，以帮助我们分析上面提到的哪些方法最适合我们的用例。

### 6.1。简单基准测试

首先，我们采取一个简单的方法。我们选择一个字符串值——对于我们的测试，我们使用`Integer.MAX_VALUE`。该价值将在我们的所有实施中进行测试:

```java
Benchmark                                     Mode  Cnt    Score   Error  Units
Benchmarking.usingCoreJava                    avgt   20   57.241 ± 0.792  ns/op
Benchmarking.usingNumberUtils_isCreatable     avgt   20   26.711 ± 1.110  ns/op
Benchmarking.usingNumberUtils_isParsable      avgt   20   46.577 ± 1.973  ns/op
Benchmarking.usingRegularExpressions          avgt   20  101.580 ± 4.244  ns/op
Benchmarking.usingStringUtils_isNumeric       avgt   20   35.885 ± 1.691  ns/op
Benchmarking.usingStringUtils_isNumericSpace  avgt   20   31.979 ± 1.393  ns/op
```

正如我们所看到的，开销最大的操作是正则表达式。之后是我们基于 Java 的核心解决方案。

此外，注意使用 Apache Commons 库的操作大体上是相同的。

### 6.2。增强型基准测试

让我们使用一组更多样化的测试来获得更具代表性的基准:

*   95 个值是数字(0-94 和`Integer.MAX_VALUE`)
*   3 包含数字，但格式仍然不正确—“`x0`”、“`0.` .005”和“【T2”)
*   1 只包含文本
*   1 是一个`null`

执行相同的测试后，我们将看到结果:

```java
Benchmark                                     Mode  Cnt      Score     Error  Units
Benchmarking.usingCoreJava                    avgt   20  10162.872 ± 798.387  ns/op
Benchmarking.usingNumberUtils_isCreatable     avgt   20   1703.243 ± 108.244  ns/op
Benchmarking.usingNumberUtils_isParsable      avgt   20   1589.915 ± 203.052  ns/op
Benchmarking.usingRegularExpressions          avgt   20   7168.761 ± 344.597  ns/op
Benchmarking.usingStringUtils_isNumeric       avgt   20   1071.753 ±   8.657  ns/op
Benchmarking.usingStringUtils_isNumericSpace  avgt   20   1157.722 ±  24.139  ns/op
```

最重要的区别是我们的两个测试，正则表达式解决方案和基于 Java 的核心解决方案，交换了位置。

从这个结果中，我们了解到`NumberFormatException`的投掷和处理(只发生在 5%的情况下)对整体性能有相对较大的影响。所以我们可以得出结论，最优解取决于我们期望的输入。

此外，我们可以有把握地得出结论，为了获得最佳性能，我们应该使用 Commons 库中的方法或类似实现的方法。

## 7。结论

在本文中，我们探索了不同的方法来确定`String`是否是数字。我们研究了两种解决方案——内置方法和外部库。

与往常一样，上面给出的所有示例和代码片段的实现，包括用于执行基准测试的代码，都可以在 GitHub 上找到[。](https://web.archive.org/web/20221017232038/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)