# Java String.endsWith()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/ends-with>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143229/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220829143229/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220829143229/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220829143229/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220829143229/https://www.baeldung.com/string/copy-value-of)
• Java String.endsWith() (current article)[• Java String.format()](/web/20220829143229/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220829143229/https://www.baeldung.com/string/get-bytes)
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

方法`endsWith()`是一个方便的方法，它检查一个`String`是否以另一个给定的`String`结束。如果参数是空的`String`，那么方法返回 true。

### 可用签名

```java
public boolean endsWith(String suffix)
```

### 例子

```java
@Test
public void whenCallEndsWith_thenCorrect() {
    String s1 = "test";

    assertTrue(s1.endsWith("t"));
}
```

Next **»**[Java String.format()](/web/20220829143229/https://www.baeldung.com/string/format)**«** Previous[Java String.copyValueOf()](/web/20220829143229/https://www.baeldung.com/string/copy-value-of)