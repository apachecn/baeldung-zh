# Java String.concat()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/concat>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143230/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143230/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220829143230/https://www.baeldung.com/string/code-point-at)
• Java String.concat() (current article)[• Java String.contains()](/web/20220829143230/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220829143230/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220829143230/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220829143230/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220829143230/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220829143230/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220829143230/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220829143230/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20220829143230/https://www.baeldung.com/string/last-index-of)
[• Java String.regionMatches()](/web/20220829143230/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20220829143230/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20220829143230/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20220829143230/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220829143230/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220829143230/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220829143230/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220829143230/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143230/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143230/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143230/https://www.baeldung.com/string/value-of)

方法`concat()`连接两个`Strings`。简单地说，它将它们连接成一个单一的`String`。如果参数的长度为 0，那么该方法只返回`String`对象。

### 可用签名

```java
public String concat(String str)
```

### 例子

```java
@Test
public void whenCallConcat_thenCorrect() {
    assertEquals("elephant", "elep".concat("hant"));
}
```

Next **»**[Java String.contains()](/web/20220829143230/https://www.baeldung.com/string/contains)**«** Previous[Java String.codePointAt()](/web/20220829143230/https://www.baeldung.com/string/code-point-at)