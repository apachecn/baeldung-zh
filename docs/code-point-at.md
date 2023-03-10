# java string . codepointat()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/code-point-at>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143229/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)
• Java String.codePointAt() (current article)[• Java String.concat()](/web/20220829143229/https://www.baeldung.com/string/concat)
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
[• Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

方法`codePointAt()`将一个`int`作为参数，并返回指定索引处的代码点。码位是字符在 Unicode 标准中给定的十进制值。

### 可用签名

```java
public int codePointAt(int index)
```

### 例子

```java
@Test
public void whenCallCodePointAt_thenDecimalUnicodeReturned() {
    assertEquals(97, "abcd".codePointAt(0));
}
```

### 投掷

*   `StringIndexOutOfBoundsException`–如果向方法传递了不存在的索引。

```java
@Test(expected = StringIndexOutOfBoundsException.class)
public void whenPassNonExistingIndex_thenStringIndexOutOfBoundsExceptionThrown() {
    int a = "abcd".codePointAt(4);
}
```

Next **»**[Java String.concat()](/web/20220829143229/https://www.baeldung.com/string/concat)**«** Previous[Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)