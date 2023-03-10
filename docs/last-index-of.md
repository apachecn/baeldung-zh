# Java String.lastIndexOf()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/last-index-of>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220628162431/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220628162431/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220628162431/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220628162431/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220628162431/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220628162431/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220628162431/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220628162431/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220628162431/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220628162431/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220628162431/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220628162431/https://www.baeldung.com/string/is-empty)
• Java String.lastIndexOf() (current article)[• Java String.regionMatches()](/web/20220628162431/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20220628162431/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20220628162431/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20220628162431/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220628162431/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220628162431/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220628162431/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220628162431/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220628162431/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220628162431/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220628162431/https://www.baeldung.com/string/value-of)

方法`lastIndexOf()`返回一个`String`在另一个`String`中最后一次出现的索引。如果向该方法传递了一个`int`，那么该方法将搜索等价的 Unicode 字符。

我们也可以传递开始搜索的字符的索引。

### 可用签名

```java
public int lastIndexOf(int ch)
public int lastIndexOf(int ch, int fromIndex)
public int lastIndexOf(String str)
```

### 例子

```java
@Test
public void whenCallLastIndexOf_thenCorrect() {
    assertEquals(2, "foo".lastIndexOf("o"));
    assertEquals(2, "foo".lastIndexOf(111));
}
```

Next **»**[Java String.regionMatches()](/web/20220628162431/https://www.baeldung.com/string/region-matches)**«** Previous[Java String.isEmpty()](/web/20220628162431/https://www.baeldung.com/string/is-empty)