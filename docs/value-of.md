# Java 字符串. valueOf()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/value-of>

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
[• Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
• Java String.valueOf() (current article)

方法`valueOf()`有几个重载，它们接受一个不同类型的参数，并将它们转换成一个`String`。例子包括`boolean, char, char` ar `ray, double, int`和`long`。我们也可以通过传递以下命令将一个`char`数组的一部分转换成一个`String`:

*   offset–开始转换的字符的索引
*   count–要转换的字符数

### 可用签名

```java
public static String valueOf(boolean b)
public static String valueOf(char c)
public static String valueOf(char[] data)
public static String valueOf(char[] data, int offset, int count)
public static String valueOf(double d)
public static String valueOf(float f)
public static String valueOf(int i)
public static String valueOf(long l)
public static String valueOf(Object obj)
```

### 例子

```java
@Test
public void whenCallValueOf_thenCorrect() {
    long l = 200L;

    assertEquals("200", String.valueOf(l));
}
```

**«** Previous[Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)