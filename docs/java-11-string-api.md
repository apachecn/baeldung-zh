# Java 11 字符串 API 附加

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-11-string-api>

## 1。简介

Java 11 给常用的 [`String`类](/web/20220706122849/https://www.baeldung.com/java-string)增加了几个有用的 API。在本教程中，我们将探索和使用这些新的 API。

## 2。`repeat()`

顾名思义，`repeat()`实例方法重复字符串内容。

**它返回一个字符串，其值是重复`n`次的字符串的串联，其中`n`作为参数**传递:

```java
@Test
public void whenRepeatStringTwice_thenGetStringTwice() {
    String output = "La ".repeat(2) + "Land";

    is(output).equals("La La Land");
}
```

另外，如果字符串为空或者计数为零，`repeat()`将返回一个空字符串。

## 3。`strip*()`

**`strip()`实例方法返回一个去掉了所有前导和尾随空格的字符串**:

```java
@Test
public void whenStripString_thenReturnStringWithoutWhitespaces() {
    is("\n\t  hello   \u2005".strip()).equals("hello");
}
```

Java 11 还增加了方法`stripLeading()`和`stripTrailing()`，分别处理前导和尾随空格。

### 3.1。`strip()`与`trim()`和的区别

`strip*()`根据`Character.isWhitespace()`判断字符是否为空白。换句话说，**能够识别 Unicode 空白字符**。

这与 [`trim()`](/web/20220706122849/https://www.baeldung.com/string/trim) 不同，后者将空格定义为小于或等于 Unicode 空格字符(U+0020)的任何字符。如果我们在前面的例子中使用`trim()`，我们将得到不同的结果:

```java
@Test
public void whenTrimAdvanceString_thenReturnStringWithWhitespaces() {
    is("\n\t  hello   \u2005".trim()).equals("hello   \u2005");
}
```

注意`trim()`如何能够修剪前导空白，但是它没有修剪尾部空白。这是因为 **`trim()`** 不识别 Unicode 空白字符，因此不认为‘`\u2005′`是空白字符。

## 4。`isBlank()`

**如果字符串为空或者只包含空白，`isBlank()`实例方法返回`true`。否则，它返回`false`** :

```java
@Test
public void whenBlankString_thenReturnTrue() {
    assertTrue("\n\t\u2005  ".isBlank());
}
```

类似地，`isBlank()`方法可以识别 Unicode 空白字符，就像`strip()`一样。

## 5。`lines()`

**`lines()`实例方法返回一个从字符串中提取的 [`Stream`](/web/20220706122849/https://www.baeldung.com/java-8-streams-introduction) 行，由行终止符**分隔:

```java
@Test
public void whenMultilineString_thenReturnNonEmptyLineCount() {
    String multilineStr = "This is\n \n a multiline\n string.";

    long lineCount = multilineStr.lines()
      .filter(String::isBlank)
      .count();

    is(lineCount).equals(3L);
}
```

行终止符是下列之一:`“\n”,` `“\r”,`或`“\r\n”`。

**流中包含的行按照它们出现的顺序排列。从每一行中删除行终止符。**

这种方法比`split()`更可取，因为它在断开多行输入时提供了更好的性能。

## 6。结论

在这篇简短的文章中，我们探索了 Java 11 中新的字符串 API。

最后，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220706122849/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)