# Java String.substring()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/substring>

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
• Java String.substring() (current article)[• Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

方法`substring()`有两个签名。如果我们将 beginIndex 和 endIndex 传递给该方法，那么在给定起始索引和结果长度的情况下，它将获得一个`String`的一部分。

我们也可以只传递 beginIndex，获取从 beginIndex 到`String`结尾的那部分`String`。

### 可用签名

```java
public String substring(int beginIndex)
public String substring(int beginIndex, int endIndex)
```

### 例子

```java
@Test
public void whenCallSubstring_thenCorrect() {
    String s = "Welcome to Baeldung";

    assertEquals("Welcome", s.substring(0, 7));
}
```

### 投掷

*   `IndexOutOfBoundsException –`如果第一个索引为负，则第一个索引大于第二个索引或者第二个索引大于`String`的长度

```java
@Test(expected = IndexOutOfBoundsException.class)
public void whenSecondIndexEqualToLengthOfString_thenCorrect() {
    String s = "Welcome to Baeldung";

    String sub = s.substring(0, 20);
}
```

Next **»**[Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)**«** Previous[Java String.subSequence()](/web/20220829143229/https://www.baeldung.com/string/sub-sequence)