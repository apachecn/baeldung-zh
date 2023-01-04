# Java String.intern()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/intern>

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
• Java String.intern() (current article)[• Java String.isEmpty()](/web/20220829143229/https://www.baeldung.com/string/is-empty)
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

方法`intern()`在堆内存中创建一个`String`对象的精确副本，并将其存储在`String`常量池中。

注意，如果另一个具有相同内容的`String`存在于`String`常量池中，那么不会创建新的对象，新的引用将指向另一个`String.`

### 可用签名

```java
public String intern()
```

### 例子

```java
@Test
public void whenIntern_thenCorrect() {
    String s1 = "abc";
    String s2 = new String("abc");
    String s3 = new String("foo");
    String s4 = s1.intern();
    String s5 = s2.intern();

    assertFalse(s3 == s4);
    assertTrue(s1 == s5);
}
```

Next **»**[Java String.isEmpty()](/web/20220829143229/https://www.baeldung.com/string/is-empty)**«** Previous[Java String.indexOf()](/web/20220829143229/https://www.baeldung.com/string/index-of)