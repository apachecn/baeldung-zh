# Java String.format()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/format>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143229/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220829143229/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220829143229/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220829143229/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220829143229/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220829143229/https://www.baeldung.com/string/ends-with)
• Java String.format() (current article)[• Java String.getBytes()](/web/20220829143229/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220829143229/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220829143229/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220829143229/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20220829143229/https://www.baeldung.com/string/last-index-of)
[• Java String.regionMatches()](/web/20220829143229/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20220829143229/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20220829143229/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20220829143229/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220829143229/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220829143229/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220829143229/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

方法`format()`使用格式`String`和参数格式化一个`String`。例如，如果参数`arg`为空，字符‘S’和‘S’的计算结果为“空”。

如果`arg`实现了 Formattable，那么方法`Formattable,`然后方法`arg.formatTo()`被调用。否则，通过调用`arg.toString()`评估结果。

有关格式化的更多信息，请访问。

### 可用签名

```java
public static String format(String format, Object... args)
public static String format(Locale l, String format, Object... args)
```

### 例子

```java
@Test
public void whenFormat_thenCorrect() {
    String value = "Baeldung";
    String formatted = String.format("Welcome to %s!", value);

    assertEquals("Welcome to Baeldung!", formatted);
}
```

### 投掷

*   `IllegalFormatException`–如果格式`String`包含无效语法。

```java
@Test(expected = IllegalFormatException.class)
public void whenInvalidFormatSyntax_thenIllegalFormatExceptionThrown() {
    String value = "Baeldung";
    String formatted = String.format("Welcome to %x!", value);
}
```

Next **»**[Java String.getBytes()](/web/20220829143229/https://www.baeldung.com/string/get-bytes)**«** Previous[Java String.endsWith()](/web/20220829143229/https://www.baeldung.com/string/ends-with)