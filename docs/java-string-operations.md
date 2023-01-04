# Java 中常见的字符串操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-operations>

## 1.介绍

基于字符串的值和操作在日常开发中相当常见，任何 Java 开发人员都必须能够处理它们。

在本教程中，我们将提供一个常见`String`操作的快速备忘单。

此外，我们将阐明`equals`和“==”以及`StringUtils#isBlank `和# `isEmpty.`之间的区别

## 2.将字符转换为字符串

一个`char` 代表 Java 中的一个字符。但是在大多数情况下，我们需要一个`String.`

所以让我们从把`char` s 转换成 `String` s `:`开始

```
String toStringWithConcatenation(final char c) {
    return String.valueOf(c);
}
```

## 3.追加字符串

另一个经常需要的操作是给字符串附加其他值，比如一个`char`:

```
String appendWithConcatenation(final String prefix, final char c) {
    return prefix + c;
}
```

我们可以给其他基本类型加上`StringBuilder`和 **:**

```
String appendWithStringBuilder(final String prefix, final char c) {
    return new StringBuilder(prefix).append(c).toString();
}
```

## 4.通过索引获取字符

如果我们需要从字符串中提取一个字符，API 提供了我们想要的一切:

```
char getCharacterByIndex(final String text, final int index) {
    return text.charAt(index);
}
```

由于 a `String`使用一个`char[]` 作为后台数据结构，**索引从零开始**。

## 5.处理 ASCII 值

通过强制转换，我们可以很容易地在`char`和它的数字表示(ASCII)之间切换:

```
int asciiValue(final char character) {
    return (int) character;
}

char fromAsciiValue(final int value) {
    Assert.isTrue(value >= 0 && value < 65536, "value is not a valid character");
    return (char) value;
}
```

当然，由于一个`int`是 4 个无符号字节，一个`char`是 2 个无符号字节，我们需要检查以确保我们使用的是合法的字符值。

## 6.删除所有空白

有时我们需要去掉一些字符，最常见的是空格。一个**的好方法是使用带有[正则表达式](/web/20221129225350/https://www.baeldung.com/regular-expressions-java) :** 的`replaceAll`方法

```
String removeWhiteSpace(final String text) {
    return text.replaceAll("\\s+", "");
}
```

## 7.将集合连接到字符串

另一个常见的用例是当我们有某种类型的`Collection`并想用它创建一个字符串时:

```
<T> String fromCollection(final Collection<T> collection) { 
   return collection.stream().map(Objects::toString).collect(Collectors.joining(", "));
}
```

注意，`Collectors.joining`允许[指定前缀或后缀](/web/20221129225350/https://www.baeldung.com/java-8-collectors)。

## 8.拆分字符串

或者另一方面，我们可以使用`split`方法通过分隔符分割字符串:

```
String[] splitByRegExPipe(final String text) {
   return text.split("\\|");
}
```

同样，我们在这里使用了一个正则表达式，这一次是通过管道进行分割。既然我们想使用一个特殊的字符，[我们必须对它进行转义](/web/20221129225350/https://www.baeldung.com/java-regexp-escape-char)。

另一种可能是使用`Pattern`类:

```
String[] splitByPatternPipe(final String text) {
    return text.split(Pattern.quote("|"));
}
```

## 9.将所有字符作为流处理

在详细处理的情况下，我们可以将一个字符串转换成一个`[IntStream](/web/20221129225350/https://www.baeldung.com/java-string-to-stream)`:

```
IntStream getStream(final String text) {
    return text.chars();
}
```

## 10.引用相等和值相等

虽然字符串看起来像一个原始的类型，但它们不是。

因此，我们必须区分引用相等和值相等。引用相等总是意味着值相等，但通常不是相反。首先，我们用' == '操作检查，然后用`equals`方法检查:

```
@Test
public void whenUsingEquals_thenWeCheckForTheSameValue() {
    assertTrue("Values are equal", new String("Test").equals("Test"));
}

@Test
public void whenUsingEqualsSign_thenWeCheckForReferenceEquality() {
    assertFalse("References are not equal", new String("Test") == "Test");
}
```

注意文字被[保存在字符串池](/web/20221129225350/https://www.baeldung.com/java-string-pool)中。因此，编译器有时可以将它们优化为相同的引用:

```
@Test
public void whenTheCompileCanBuildUpAString_thenWeGetTheSameReference() {
    assertTrue("Literals are concatenated by the compiler", "Test" == "Te"+"st");
}
```

## 11.空白字符串与空字符串

`isBlank`和`isEmpty`有细微的区别。

如果字符串为`null`或长度为零，则该字符串为空。**而一个字符串如果为空或者只包含空白字符则为空:**

```
@Test
public void whenUsingIsEmpty_thenWeCheckForNullorLengthZero() {
    assertTrue("null is empty", isEmpty(null));
    assertTrue("nothing is empty", isEmpty(""));
    assertFalse("whitespace is not empty", isEmpty(" "));
    assertFalse("whitespace is not empty", isEmpty("\n"));
    assertFalse("whitespace is not empty", isEmpty("\t"));
    assertFalse("text is not empty", isEmpty("Anything!"));
}

@Test
public void whenUsingIsBlank_thenWeCheckForNullorOnlyContainingWhitespace() {
    assertTrue("null is blank", isBlank(null));
    assertTrue("nothing is blank", isBlank(""));
    assertTrue("whitespace is blank", isBlank("\t\t \t\n\r"));
    assertFalse("test is not blank", isBlank("Anything!"));
}
```

## 12.结论

字符串是所有应用程序的核心类型。在本教程中，我们学习了一些常见场景中的关键操作。

此外，我们给出了更详细参考资料的方向。

最后，在我们的 [GitHub 库](https://web.archive.org/web/20221129225350/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)中可以找到包含所有示例的完整代码。