# Java 字符串. replace()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string/replace>

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
• Java String.replace() (current article)[• Java String.replaceAll()](/web/20220829143229/https://www.baeldung.com/string/replace-all)
[• Java String.split()](/web/20220829143229/https://www.baeldung.com/string/split)
[• Java String.startsWith()](/web/20220829143229/https://www.baeldung.com/string/starts-with)
[• Java String.subSequence()](/web/20220829143229/https://www.baeldung.com/string/sub-sequence)
[• Java String.substring()](/web/20220829143229/https://www.baeldung.com/string/substring)
[• Java String.toLowerCase()](/web/20220829143229/https://www.baeldung.com/string/to-lower-case)
[• Java String.toUpperCase()](/web/20220829143229/https://www.baeldung.com/string/to-upper-case)
[• Java String.trim()](/web/20220829143229/https://www.baeldung.com/string/trim)
[• Java String.valueOf()](/web/20220829143229/https://www.baeldung.com/string/value-of)

方法`replace()` 将一个`String`在另一个`String`中的所有出现或者一个`char`的所有出现替换为另一个`char`。

### 可用签名

```java
public String replace(char oldChar, char newChar)
public String replace(CharSequence target, CharSequence replacement)
```

### 例子

```java
@Test
void whenReplaceOldCharNewChar_thenCorrect() {
    String s = "wslcoms to basldung";

    assertEquals("welcome to baeldung", s.replace('s', 'e'));
}

```

```java
@Test
void whenReplaceTargetReplacement_thenCorrect() {
    String s = "welcome at baeldung, login at your course";

    assertEquals("welcome to baeldung, login to your course", s.replace("at", "to"));
}
```

Next **»**[Java String.replaceAll()](/web/20220829143229/https://www.baeldung.com/string/replace-all)**«** Previous[Java String.regionMatches()](/web/20220829143229/https://www.baeldung.com/string/region-matches)