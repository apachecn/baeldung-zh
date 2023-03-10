# Java 字符串。字符串()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/constructor>

[This article is part of a series:](javascript:void(0);)• Java String.String() (current article)[• Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)
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
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

`String`对象可以通过使用文字创建:

```java
String s = "a string";
```

或者通过调用其中一个构造函数:

```java
String s = new String("a string");
```

如果我们使用`String`文字，它将尝试重用`String`常量池中已经存在的对象。

另一方面，**当使用构造函数实例化一个`String`时，一个新的对象将被创建**

这个构造函数接受多种类型的参数，并用它们来创建一个新的`String`对象。

### 可用签名

```java
public String()
public String(byte[] bytes)
public String(byte[] bytes, Charset charset)
public String(byte[] bytes, int offset, int length)
public String(byte[] bytes, int offset, int length, Charset charset)
public String(byte[] bytes, int offset, int length, String charsetName)
public String(byte[] bytes, String charsetName)
public String(char[] value)
public String(char[] value, int offset, int count)
public String(int[] codePoints, int offset, int count)
public String(String original)
public String(StringBuffer buffer)
public String(StringBuilder builder)
```

### 例子

```java
@Test
public void whenCreateStringUsingByteArray_thenCorrect() {
    byte[] array = new byte[] { 97, 98, 99, 100 };
    String s = new String(array);

    assertEquals("abcd", s);
}
```

Next **»**[Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)