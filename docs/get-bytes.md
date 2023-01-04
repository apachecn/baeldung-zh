# Java String.getBytes()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/get-bytes>

[This article is part of a series:](javascript:void(0);)[• Java String.String()](/web/20220829143229/https://www.baeldung.com/string/constructor)
[• Java String.codePointCount()](/web/20220829143229/https://www.baeldung.com/string/code-point-count)
[• Java String.codePointAt()](/web/20220829143229/https://www.baeldung.com/string/code-point-at)
[• Java String.concat()](/web/20220829143229/https://www.baeldung.com/string/concat)
[• Java String.contains()](/web/20220829143229/https://www.baeldung.com/string/contains)
[• Java String.copyValueOf()](/web/20220829143229/https://www.baeldung.com/string/copy-value-of)
[• Java String.endsWith()](/web/20220829143229/https://www.baeldung.com/string/ends-with)
[• Java String.format()](/web/20220829143229/https://www.baeldung.com/string/format)
• Java String.getBytes() (current article)[• Java String.indexOf()](/web/20220829143229/https://www.baeldung.com/string/index-of)
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

如果没有传递参数，方法`getBytes()`使用平台的默认字符集将`String`编码成一个字节数组。

我们可以传递一个特定的`Charset`用于编码过程，或者作为一个`String`对象或者一个`String`对象。

### 可用签名

```java
public byte[] getBytes()
public byte[] getBytes(Charset charset)
public byte[] getBytes(String charsetName)
```

### 例子

```java
@Test
public void whenGetBytes_thenCorrect() throws UnsupportedEncodingException {
    byte[] byteArray1 = "abcd".getBytes();
    byte[] byteArray2 = "efgh".getBytes(StandardCharsets.US_ASCII);
    byte[] byteArray3 = "ijkl".getBytes("UTF-8");
    byte[] expected1 = new byte[] { 97, 98, 99, 100 };
    byte[] expected2 = new byte[] { 101, 102, 103, 104 };
    byte[] expected3 = new byte[] { 105, 106, 107, 108 };

    assertArrayEquals(expected1, byteArray1);
    assertArrayEquals(expected2, byteArray2);
    assertArrayEquals(expected3, byteArray3);
}
```

Next **»**[Java String.indexOf()](/web/20220829143229/https://www.baeldung.com/string/index-of)**«** Previous[Java String.format()](/web/20220829143229/https://www.baeldung.com/string/format)