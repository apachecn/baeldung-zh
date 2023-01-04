# Java String.replaceAll()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/replace-all>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220707182030/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220707182030/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220707182030/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220707182030/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220707182030/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220707182030/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220707182030/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220707182030/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220707182030/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220707182030/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220707182030/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220707182030/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20220707182030/https://www.baeldung.com/string/last-index-of)
[• Java String.regionMatches()](/web/20220707182030/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20220707182030/https://www.baeldung.com/string/replace)
• Java String.replaceAll() (current article)[• Java String.split()](/web/20220707182030/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220707182030/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220707182030/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220707182030/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220707182030/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220707182030/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220707182030/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220707182030/https://www.baeldung.com/string/value-of)

方法`replaceAll()`替换由`regex.`匹配的另一个`String `中`String`的所有出现

这类似于`[replace()](/web/20220707182030/https://www.baeldung.com/string/replace)` 函数，唯一不同的是，在`replaceAll()`中被替换的字符串是一个`regex`，而在 [`replace()`](/web/20220707182030/https://www.baeldung.com/string/replace) 中是一个`String.`

### 可用签名

```java
public String replaceAll(String regex, String replacement)
```

### 例子

```java
@Test
void whenReplaceAll_thenCorrect() {
    String s = "my url with spaces";

    assertEquals("my-url-with-spaces", s.replaceAll("\\s+", "-"));
    assertEquals("your url with spaces", s.replaceAll("my", "your"));
}
```

### 投掷

*   `PatternSyntaxException`–如果正则表达式无效

```java
@Test(expected = PatternSyntaxException.class)
void whenInvalidRegex_thenPatternSyntaxExceptionThrown() {
    String s = "my url with spaces";

    s.replaceAll("\\s+\\", "-");
}
```

Next **»**[Java String.split()](/web/20220707182030/https://www.baeldung.com/string/split)**«** Previous[Java String.replace()](/web/20220707182030/https://www.baeldung.com/string/replace)