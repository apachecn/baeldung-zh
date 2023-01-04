# Java 中的左修剪和右修剪替代方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-trim-alternatives>

## 1.概观

方法 [`String.trim()`](/web/20221007082754/https://www.baeldung.com/string/trim) 删除尾部和前导空白。但是，不支持只做 L-Trim 或 R-Trim。

在本教程中，我们将看到实现这一点的几种方法；最后，我们将比较他们的表现。

## 2.`while`循环

最简单的解决方案是使用几个`while` 循环`.` 遍历字符串

对于 L-Trim，我们将从左向右读取字符串，直到遇到一个非空白字符:

```java
int i = 0;
while (i < s.length() && Character.isWhitespace(s.charAt(i))) {
    i++;
}
String ltrim = s.substring(i); 
```

`ltrim` 是从第一个非空白字符开始的子字符串。

或者对于 R-Trim，我们将从右向左读取字符串，直到遇到非空白字符:

```java
int i = s.length()-1;
while (i >= 0 && Character.isWhitespace(s.charAt(i))) {
    i--;
}
String rtrim = s.substring(0,i+1);
```

*rtrim* 是一个子字符串，从开头开始，到第一个非空白字符结束。

## 3.`String.replaceAll` 使用正则表达式

另一个选择是使用 [`String.replaceAll()`](https://web.archive.org/web/20221007082754/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#replaceAll(java.lang.String,java.lang.String)) 和一个正则表达式:

```java
String ltrim = src.replaceAll("^\\s+", "");
String rtrim = src.replaceAll("\\s+$", "");
```

(\\s+)是匹配一个或多个空白字符的正则表达式。正则表达式开头和结尾的插入符号(^)和($)匹配一行的开头和结尾。

## 4.`Pattern.compile()` 和 `.matcher()`

我们可以重用带有 [`java.util.regex.Pattern`](https://web.archive.org/web/20221007082754/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html) 的正则表达式，也是:

```java
private static Pattern LTRIM = Pattern.compile("^\\s+");
private static Pattern RTRIM = Pattern.compile("\\s+$");

String ltrim = LTRIM.matcher(s).replaceAll("");
String rtim = RTRIM.matcher(s).replaceAll("");
```

## 5.Apache common(Apache 公共)

此外，我们可以利用 [Apache Commons `StringUtils#stripStart`和`#stripEnd`](https://web.archive.org/web/20221007082754/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#stripStart-java.lang.String-java.lang.String-) 方法来删除空白。

为此，让我们首先添加[的`commons-lang3`依赖关系](https://web.archive.org/web/20221007082754/https://mvnrepository.com/artifact/org.apache.commons/commons-lang3):

```java
<dependency> 
    <groupId>org.apache.commons</groupId> 
    <artifactId>commons-lang3</artifactId> 
    <version>3.12.0</version> 
</dependency>
```

遵循文档，我们使用`null`来去除空白:

```java
String ltrim = StringUtils.stripStart(src, null);
String rtrim = StringUtils.stripEnd(src, null);
```

## 6.番石榴

最后，我们将利用[番石榴](https://web.archive.org/web/20221007082754/https://guava.dev/releases/20.0-rc1/api/docs/com/google/common/base/CharMatcher.html#trimLeadingFrom-java.lang.CharSequence-) `[CharMatcher#trimLeadingFrom](https://web.archive.org/web/20221007082754/https://guava.dev/releases/20.0-rc1/api/docs/com/google/common/base/CharMatcher.html#trimLeadingFrom-java.lang.CharSequence-)` [和](https://web.archive.org/web/20221007082754/https://guava.dev/releases/20.0-rc1/api/docs/com/google/common/base/CharMatcher.html#trimLeadingFrom-java.lang.CharSequence-) `[#trimTrailingFrom](https://web.archive.org/web/20221007082754/https://guava.dev/releases/20.0-rc1/api/docs/com/google/common/base/CharMatcher.html#trimLeadingFrom-java.lang.CharSequence-)`方法获得相同的结果。

再次，让我们添加适当的 Maven 依赖项，这次它的 [`guava`](https://web.archive.org/web/20221007082754/https://mvnrepository.com/artifact/com.google.guava/guava) :

```java
<dependency> 
    <groupId>com.google.guava</groupId> 
    <artifactId>guava</artifactId> 
    <version>31.0.1-jre</version> 
</dependency>
```

在 Guava 中，这与 Apache Commons 中的做法非常相似，只是方法更有针对性:

```java
String ltrim = CharMatcher.whitespace().trimLeadingFrom(s); 
String rtrim = CharMatcher.whitespace().trimTrailingFrom(s);
```

## 7.性能比较

让我们看看这些方法的性能。像往常一样，我们将利用开源框架 [Java 微基准测试工具](/web/20221007082754/https://www.baeldung.com/java-microbenchmark-harness) (JMH)来比较纳秒级的不同选择。

### 7.1.基准设置

对于基准的初始配置，我们使用了五个分支和以纳秒为单位的平均时间计算时间:

```java
@Fork(5)
@State(Scope.Benchmark)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
```

在 setup 方法中，我们初始化原始消息字段和结果字符串，以便与以下内容进行比较:

```java
@Setup
public void setup() {
    src = "       White spaces left and right          ";
    ltrimResult = "White spaces left and right          ";
    rtrimResult = "       White spaces left and right";
}
```

所有的基准首先删除左边的空白，然后删除右边的空白，最后将结果与预期的字符串进行比较。

### 7.2.`while`循环

对于我们的第一个基准测试，让我们使用`while`循环方法:

```java
@Benchmark
public boolean whileCharacters() {
    String ltrim = whileLtrim(src);
    String rtrim = whileRtrim(src);
    return checkStrings(ltrim, rtrim);
}
```

### 7.3.`String.replaceAll() `用正则表达式

那么，我们来试试`String.replaceAll()`:

```java
@Benchmark
public boolean replaceAllRegularExpression() {
    String ltrim = src.replaceAll("^\\s+", "");
    String rtrim = src.replaceAll("\\s+$", "");
    return checkStrings(ltrim, rtrim);
}
```

### 7.4.`Pattern.compile().matches()`

之后是`Pattern.compile().matches()`:

```java
@Benchmark
public boolean patternMatchesLTtrimRTrim() {
    String ltrim = patternLtrim(src);
    String rtrim = patternRtrim(src);
    return checkStrings(ltrim, rtrim);
}
```

### 7.5 秒。Apache common(Apache 公共)

第四，Apache Commons:

```java
@Benchmark
public boolean apacheCommonsStringUtils() {
    String ltrim = StringUtils.stripStart(src, " ");
    String rtrim = StringUtils.stripEnd(src, " ");
    return checkStrings(ltrim, rtrim);
}
```

### 7.6.番石榴

最后，让我们用番石榴:

```java
@Benchmark
public boolean guavaCharMatcher() {
    String ltrim = CharMatcher.whitespace().trimLeadingFrom(src);
    String rtrim = CharMatcher.whitespace().trimTrailingFrom(src);
    return checkStrings(ltrim, rtrim);
}
```

### 7.7.结果分析

我们应该会得到类似如下的结果:

```java
# Run complete. Total time: 00:16:57

Benchmark                               Mode  Cnt     Score    Error  Units
LTrimRTrim.apacheCommonsStringUtils     avgt  100   108,718 ±  4,503  ns/op
LTrimRTrim.guavaCharMatcher             avgt  100   113,601 ±  5,563  ns/op
LTrimRTrim.patternMatchesLTtrimRTrim    avgt  100   850,085 ± 17,578  ns/op
LTrimRTrim.replaceAllRegularExpression  avgt  100  1046,660 ±  7,151  ns/op
LTrimRTrim.whileCharacters              avgt  100   110,379 ±  1,032  ns/op
```

看起来我们的赢家是 loop、Apache Commons 和 Guava！

## 8.结论

在本教程中，我们看了几种不同的方法来删除`String`开头和结尾的空白字符。

我们使用了`while` loop、`String.replaceAll(),` `Pattern.matcher().replaceAll(),` Apache Commons 和 Guava 来获得这个结果。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221007082754/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)