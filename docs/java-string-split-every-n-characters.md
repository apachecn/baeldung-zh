# 在 Java 中每 n 个字符拆分一个字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-split-every-n-characters>

## 1.概观

在本教程中，我们将阐明如何在 Java 中**每隔`n`个字符拆分一个字符串。**

首先，我们将从探索使用内置 Java 方法实现这一点的可能方式开始。然后，我们将展示如何用番石榴达到同样的目的。

## 2.使用`String#split`方法

`String`类附带了一个叫做`[split](/web/20220627173129/https://www.baeldung.com/string/split).` 的简便方法，顾名思义就是`,` ，它根据给定的分隔符或正则表达式将一个字符串分成多个部分。

让我们来看看它的实际应用:

```java
public static List<String> usingSplitMethod(String text, int n) {
    String[] results = text.split("(?<=\\G.{" + n + "})");

    return Arrays.asList(results);
}
```

正如我们所看到的，我们使用了正则表达式`(?<=\\G.{” + n + “})` ，其中`n` 是字符数。这是一个[肯定回顾](/web/20220627173129/https://www.baeldung.com/java-regex-lookahead-lookbehind#positive-lookbehind)断言**，它匹配最后一个匹配的字符串`(\G)`，后跟`n`字符**。

现在，让我们创建一个测试用例来检查一切是否按预期运行:

```java
public class SplitStringEveryNthCharUnitTest {

    public static final String TEXT = "abcdefgh123456";

    @Test
    public void givenString_whenUsingSplit_thenSplit() {
        List<String> results = SplitStringEveryNthChar.usingSplitMethod(TEXT, 3);

        assertThat(results, contains("abc", "def", "gh1", "234", "56"));
    }
}
```

## 3.使用`String#substring`方法

另一种在每第 n 个字符拆分一个`String`对象的方法是使用`substring`方法。

基本上，我们可以遍历字符串并调用`substring` 来根据指定的`n`字符将它分成多个部分:

```java
public static List<String> usingSubstringMethod(String text, int n) {
    List<String> results = new ArrayList<>();
    int length = text.length();

    for (int i = 0; i < length; i += n) {
        results.add(text.substring(i, Math.min(length, i + n)));
    }

    return results;
}
```

如上所示，`substring`方法允许我们获取当前索引`i`和`i+n.`之间的字符串部分

现在，让我们用一个测试案例来证实这一点:

```java
@Test
public void givenString_whenUsingSubstring_thenSplit() {
    List<String> results = SplitStringEveryNthChar.usingSubstringMethod(TEXT, 4);

    assertThat(results, contains("abcd", "efgh", "1234", "56"));
}
```

## 4.使用`Pattern` 类

[`Pattern`](/web/20220627173129/https://www.baeldung.com/regular-expressions-java#Package) 提供了一种简洁的方法来编译正则表达式并将其与给定的字符串进行匹配。

因此，有了正确的正则表达式，我们可以使用`Pattern` 来实现我们的目标:

```java
public static List<String> usingPattern(String text, int n) {
    return Pattern.compile(".{1," + n + "}")
        .matcher(text)
        .results()
        .map(MatchResult::group)
        .collect(Collectors.toList());
}
```

正如我们所看到的，我们使用`“.{1,n}”`作为正则表达式来创建我们的`Pattern`对象`.` **，它匹配至少一个最多`n`字符。**

最后，让我们编写一个简单的测试:

```java
@Test
public void givenString_whenUsingPattern_thenSplit() {
    List<String> results = SplitStringEveryNthChar.usingPattern(TEXT, 5);

    assertThat(results, contains("abcde", "fgh12", "3456"));
}
```

## 5.用番石榴

既然我们已经知道了如何使用核心 Java 方法将一个字符串拆分成每`n`个字符，让我们看看如何使用[番石榴](/web/20220627173129/https://www.baeldung.com/guava-guide)库做同样的事情:

```java
public static List<String> usingGuava(String text, int n) {
    Iterable<String> parts = Splitter.fixedLength(n).split(text);

    return ImmutableList.copyOf(parts);
}
```

Guava 提供了`Splitter`类来简化从字符串中提取子字符串的逻辑。`fixedLength()` 方法**将给定的字符串分割成指定长度的片段**。

让我们用一个测试案例来验证我们的方法:

```java
@Test
public void givenString_whenUsingGuava_thenSplit() {
    List<String> results = SplitStringEveryNthChar.usingGuava(TEXT, 6);

    assertThat(results, contains("abcdef", "gh1234", "56"));
}
```

## 6.结论

总而言之，我们解释了如何使用 Java 方法在每第 n 个字符处拆分一个字符串。

之后，我们展示了如何使用 Guava 库来完成相同的目标。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627173129/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)