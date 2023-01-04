# Java String.startsWith()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/starts-with>

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
• Java String.startsWith() (current article)[• Java String.subSequence()](/web/20220829143229/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220829143229/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

方法`startsWith()`是一种方便的方法，它检查一个`String`是否以另一个`String`开始。我们也可以传递第一个字符的索引来开始检查。

### 可用签名

```
public boolean startsWith(String prefix)
public boolean startsWith(String prefix, int toffset)
```

### 例子

```
@Test
public void whenCallStartsWith_thenCorrect() {
    String str = "foo";

    assertTrue(str.startsWith("f"));
    assertTrue(str.startsWith("oo", 1));
}
```

Next **»**[Java String.subSequence()](/web/20220829143229/https://www.baeldung.com/string/sub-sequence)**«** Previous[Java String.split()](/web/20220829143229/https://www.baeldung.com/string/split)