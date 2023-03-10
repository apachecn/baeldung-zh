# Java String.charAt()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/char-at>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20221126234046/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20221126234046/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20221126234046/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20221126234046/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20221126234046/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20221126234046/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20221126234046/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20221126234046/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20221126234046/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20221126234046/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20221126234046/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20221126234046/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20221126234046/https://www.baeldung.com/string/last-index-of)
[• Java String.regionMatches()](/web/20221126234046/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20221126234046/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20221126234046/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20221126234046/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20221126234046/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20221126234046/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20221126234046/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20221126234046/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20221126234046/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20221126234046/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20221126234046/https://www.baeldung.com/string/value-of)

方法`charAt()`返回指定索引处的字符。索引值必须介于 0 和`String.length() – 1`之间。

### 可用签名

```java
public char charAt(int index)
```

### 例子

```java
@Test
public void whenCallCharAt_thenCorrect() {
    assertEquals('P', "Paul".charAt(0));
}
```

### 投掷

*   `IndexOutOfBoundsException`–如果向方法传递了不存在的或负的索引

```java
@Test(expected = IndexOutOfBoundsException.class)
public void whenCharAtOnNonExistingIndex_thenIndexOutOfBoundsExceptionThrown() {
    int character = "Paul".charAt(4);
}
```