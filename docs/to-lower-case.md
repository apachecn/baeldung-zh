# Java String.toLowerCase()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/to-lower-case>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143229/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220829143229/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220829143229/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220829143229/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220829143229/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220829143229/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220829143229/https://www.baeldung.com/string/format)
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
• Java String.toLowerCase() (current article)[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

方法`toLowerCase()`将一个`String`的所有字符转换成小写。如果没有向该方法传递`Locale`，那么它将使用默认的`Locale`。

然而，如果它运行在默认`Locale`不同的系统上，可能会产生意想不到的结果。为了避免这种情况，我们可以简单地将`Locale`传递给方法。

### 可用签名

```java
public String toLowerCase(Locale locale)
public String toLowerCase()
```

### 例子

```java
@Test
public void whenConvertToLowerCase_thenCorrect() {
    String s = "WELCOME to BAELDUNG!";

    assertEquals("welcome to baeldung!", s.toLowerCase());
}
```

Next **»**[Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)**«** Previous[Java String.substring()](/web/20220829143229/https://www.baeldung.com/string/substring)