# 芭乐 CharMatcher

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-string-charmatcher>

在这个快速教程中，我们将探索 Guava 中的`CharMatcher`实用程序类。

## 1。从`String`中删除特殊字符

让我们从删除 a `String`中的所有特殊字符开始。

在下面的例子中，我们使用`retainFrom()`删除了所有不是数字或字母的字符:

```java
@Test
public void whenRemoveSpecialCharacters_thenRemoved(){
    String input = "H*el.lo,}12";
    CharMatcher matcher = CharMatcher.javaLetterOrDigit();
    String result = matcher.retainFrom(input);

    assertEquals("Hello12", result);
}
```

## 2。从字符串中删除非 ASCII 字符

我们也可以使用`CharMatcher`从`String`中删除非 ASCII 字符，如下例所示:

```java
@Test
public void whenRemoveNonASCIIChars_thenRemoved() {
    String input = "あhello₤";

    String result = CharMatcher.ascii().retainFrom(input);
    assertEquals("hello", result);

    result = CharMatcher.inRange('0', 'z').retainFrom(input);
    assertEquals("hello", result);
}
```

## 3。删除`Charset`中没有的字符

现在，让我们看看如何删除不属于给定`Charset`的字符。在以下示例中，我们将删除不属于“CP 437”`Charset`的字符:

```java
@Test
public void whenRemoveCharsNotInCharset_thenRemoved() {
    Charset charset = Charset.forName("cp437");
    CharsetEncoder encoder = charset.newEncoder();

    Predicate<Character> inRange = new Predicate<Character>() {
        @Override
        public boolean apply(Character c) {
            return encoder.canEncode(c);
        }
    };

    String result = CharMatcher.forPredicate(inRange)
                               .retainFrom("helloは");
    assertEquals("hello", result);
}
```

注意:我们使用了`CharsetEncoder`来创建一个`Predicate`来检查给定的`Character`是否可以被编码成给定的`Charset`。

## 4。验证`String`

接下来，让我们看看如何使用`CharMatcher`来验证`String`。

我们可以使用`matchesAllOf()`来检查是否所有的字符都匹配一个条件。我们可以利用`matchesNoneOf()`来检查某个条件是否不适用于任何一个`String`字符。

在下面的例子中，我们检查我们的`String`是否是小写的，是否包含至少一个“`e`”字符并且不包含任何数字:

```java
@Test
public void whenValidateString_thenValid(){
    String input = "hello";

    boolean result = CharMatcher.javaLowerCase().matchesAllOf(input);
    assertTrue(result);

    result = CharMatcher.is('e').matchesAnyOf(input);
    assertTrue(result);

    result = CharMatcher.javaDigit().matchesNoneOf(input);
    assertTrue(result);
}
```

## 5。修剪`String`

现在，让我们看看如何使用`CharMatcher`来修剪`String`。

在下面的例子中，我们使用`trimLeading()`、`trimTrailing`和`trimFrom()`来修剪我们的`String`:

```java
@Test
public void whenTrimString_thenTrimmed() {
    String input = "---hello,,,";

    String result = CharMatcher.is('-').trimLeadingFrom(input);
    assertEquals("hello,,,", result);

    result = CharMatcher.is(',').trimTrailingFrom(input);
    assertEquals("---hello", result);

    result = CharMatcher.anyOf("-,").trimFrom(input);
    assertEquals("hello", result);
}
```

## 6。`String`崩一个

接下来——让我们看看如何使用`CharMatcher`折叠`String`。

在下面的例子中，我们使用`collapseFrom()`将连续的空格替换为“`–`”:

```java
@Test
public void whenCollapseFromString_thenCollapsed() {
    String input = "       hel    lo      ";

    String result = CharMatcher.is(' ').collapseFrom(input, '-');
    assertEquals("-hel-lo-", result);

    result = CharMatcher.is(' ').trimAndCollapseFrom(input, '-');
    assertEquals("hel-lo", result);
}
```

## 7。替换自`String`

我们可以使用`CharMatcher`来替换`String`中的特定字符，如下例所示:

```java
@Test
public void whenReplaceFromString_thenReplaced() {
    String input = "apple-banana.";

    String result = CharMatcher.anyOf("-.").replaceFrom(input, '!');
    assertEquals("apple!banana!", result);

    result = CharMatcher.is('-').replaceFrom(input, " and ");
    assertEquals("apple and banana.", result);
}
```

## 8。统计字符出现次数

最后，让我们看看如何使用`CharMatcher`来计算字符的出现次数。

在下面的例子中，我们计算'`a`':'【T1]'之间的逗号和字符:

```java
@Test
public void whenCountCharInString_thenCorrect() {
    String input = "a, c, z, 1, 2";

    int result = CharMatcher.is(',').countIn(input);
    assertEquals(4, result);

    result = CharMatcher.inRange('a', 'h').countIn(input);
    assertEquals(2, result);
}
```

## 9。结论

在本文中，我们举例说明了一些更有用的 API 和将 Guava 用于字符串的真实使用示例。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220812052239/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-core)