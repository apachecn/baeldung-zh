# Java String.split()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/split>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143219/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143219/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220829143219/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220829143219/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220829143219/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220829143219/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220829143219/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220829143219/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220829143219/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220829143219/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220829143219/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220829143219/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20220829143219/https://www.baeldung.com/string/last-index-of)
[• Java String.regionMatches()](/web/20220829143219/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20220829143219/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20220829143219/https://www.baeldung.com/string/replace-all)
• Java String.split() (current article)[• Java String.startsWith()](/web/20220829143219/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220829143219/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220829143219/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220829143219/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143219/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143219/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143219/https://www.baeldung.com/string/value-of)

方法`split()`将一个`String`分割成多个`Strings`，给定分隔它们的分隔符。返回的对象是一个包含 split `Strings`的数组。

我们还可以对返回数组中的元素数量进行限制。如果我们传递 0 作为限制，那么该方法将表现得好像我们没有传递任何限制一样，返回一个数组，该数组包含可以使用传递的分隔符拆分的所有元素。

## 延伸阅读:

## [在 Java 中拆分字符串](/web/20220829143219/https://www.baeldung.com/java-split-string)

The article discusses several alternatives for splitting a String in Java.[Read more](/web/20220829143219/https://www.baeldung.com/java-split-string) →

## [从 Java 中的字符串获取子字符串](/web/20220829143219/https://www.baeldung.com/java-substring)

The practical ways of using the useful substring functionality in Java - from simple examples to more advanced scenarios.[Read more](/web/20220829143219/https://www.baeldung.com/java-substring) →

## Java 正则表达式 API 指南

A practical guide to Regular Expressions API in Java.[Read more](/web/20220829143219/https://www.baeldung.com/regular-expressions-java) →

### 可用签名

```
public String[] split(String regex, int limit)
public String[] split(String regex)
```

### 例子

```
@Test
public void whenSplit_thenCorrect() {
    String s = "Welcome to Baeldung";
    String[] expected1 = new String[] { "Welcome", "to", "Baeldung" };
    String[] expected2 = new String[] { "Welcome", "to Baeldung" };

    assertArrayEquals(expected1, s.split(" "));
    assertArrayEquals(expected2, s.split(" ", 2));
}
```

### 投掷

*   `PatternSyntaxException`–如果分隔符的模式无效。

```
@Test(expected = PatternSyntaxException.class)
public void whenPassInvalidParameterToSplit_thenPatternSyntaxExceptionThrown() {
    String s = "Welcome*to Baeldung";

    String[] result = s.split("*");
}
```

Next **»**[Java String.startsWith()](/web/20220829143219/https://www.baeldung.com/string/starts-with)**«** Previous[Java String.replaceAll()](/web/20220829143219/https://www.baeldung.com/string/replace-all)