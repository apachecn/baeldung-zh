# Java String.regionMatches()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/region-matches>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220926183110/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220926183110/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220926183110/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220926183110/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220926183110/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220926183110/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220926183110/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220926183110/https://www.baeldung.com/string/format)
[• Java String.getBytes()](/web/20220926183110/https://www.baeldung.com/string/get-bytes)
[• Java String.indexOf()](/web/20220926183110/https://www.baeldung.com/string/index-of)
[• Java String.intern()](/web/20220926183110/https://www.baeldung.com/string/intern)
[• Java String.isEmpty()](/web/20220926183110/https://www.baeldung.com/string/is-empty)
[• Java String.lastIndexOf()](/web/20220926183110/https://www.baeldung.com/string/last-index-of)
• Java String.regionMatches() (current article)[• Java String.replace()](/web/20220926183110/https://www.baeldung.com/string/replace)
[• Java String.replaceAll()](/web/20220926183110/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20220926183110/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220926183110/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220926183110/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220926183110/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220926183110/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220926183110/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220926183110/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220926183110/https://www.baeldung.com/string/value-of)

方法`regionMatches()`检查两个`String`区域是否相等。

以下是几个要点:

*   `ignoreCase`指定我们是否应该忽略两者的大小写`Strings`
*   `toffset`决定了第一个`String`的起始索引
*   `other`指定第二个`String`。
*   `ooffset`指定第二个`String`的起始索引
*   `len`指定要比较的字符数

### 可用签名

```java
boolean regionMatches(int toffset, String other, int ooffset, int len)
boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len)
```

### 例子

```java
@Test
public void whenCallRegionMatches_thenCorrect() {
    assertTrue("welcome to baeldung".regionMatches(false, 11, "baeldung", 0, 8));
}
```

Next **»**[Java String.replace()](/web/20220926183110/https://www.baeldung.com/string/replace)**«** Previous[Java String.lastIndexOf()](/web/20220926183110/https://www.baeldung.com/string/last-index-of)