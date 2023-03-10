# Java String.codePointCount()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/code-point-count>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220926201914/https://www.baeldung.com/string/constructor)
• Java String.codePointCount() (current article)[• Java String.codePointAt()](/web/20220926201914/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220926201914/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220926201914/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220926201914/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220926201914/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220926201914/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220926201914/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220926201914/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220926201914/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220926201914/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20220926201914/https://www.baeldung.com/string/last-index-of)
[• Java String.regionMatches()](/web/20220926201914/https://www.baeldung.com/string/region-matches)
[• Java String.replace()](/web/20220926201914/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20220926201914/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20220926201914/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220926201914/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220926201914/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220926201914/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220926201914/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220926201914/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220926201914/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220926201914/https://www.baeldung.com/string/value-of)

方法`codePointCount()`返回指定范围内的 Unicode 码位数。文本范围从第一个索引开始，到第二个索引–1 结束。

### 可用签名

```java
public int codePointCount(int beginIndex, int endIndex)
```

### 例子

```java
@Test
public void whenCallCodePointCount_thenCorrect() {
    assertEquals(2, "abcd".codePointCount(0, 2));
}
```

### 投掷

*   `IndexOutOfBoundsException`–如果第一个索引为负，则第一个索引大于第二个索引，或者第二个索引不小于`String`的长度。

```java
@Test(expected = IndexOutOfBoundsException.class)
public void whenSecondIndexEqualToLengthOfString_thenExceptionThrown() {
    char character = "Paul".charAt(4);
}
```

Next **»**[Java String.codePointAt()](/web/20220926201914/https://www.baeldung.com/string/code-point-at)**«** Previous[Java String.String()](/web/20220926201914/https://www.baeldung.com/string/constructor)