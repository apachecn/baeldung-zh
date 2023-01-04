# Hamcrest 文本匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-text-matchers>

## 1。概述

在本教程中，我们将探索 Hamcrest 文本匹配器。

我们之前在用 Hamcrest 进行的[测试中讨论了 Hamcrest 匹配器，在本教程中我们将只关注`Text`匹配器。](/web/20221208143845/https://www.baeldung.com/java-junit-hamcrest-guide)

## 2。Maven 配置

首先，我们需要向我们的`pom.xml`添加以下依赖项:

```
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
    <scope>test</scope>
</dependency>
```

[`java-hamcrest`](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22java-hamcrest%22) 的最新版本可以从 Maven Central 下载。

现在，我们将深入研究 Hamcrest 文本匹配器。

## 3。文本相等匹配器

当然，我们可以使用标准的`isEqual()`匹配器来检查两个字符串是否相等。

此外，我们有两个特定于`String`类型的匹配器:`equalToIgnoringCase()`和`equalToIgnoringWhiteSpace().`

让我们检查两个`Strings`是否相等——忽略大小写:

```
@Test
public void whenTwoStringsAreEqual_thenCorrect() {
    String first = "hello";
    String second = "Hello";

    assertThat(first, equalToIgnoringCase(second));
}
```

我们还可以检查两个`Strings`是否相等——忽略前导和尾随空格:

```
@Test
public void whenTwoStringsAreEqualWithWhiteSpace_thenCorrect() {
    String first = "hello";
    String second = "   Hello   ";

    assertThat(first, equalToIgnoringWhiteSpace(second));
}
```

## 4。空文本匹配器

我们可以通过使用`blankString()`和`blankOrNullString()`匹配器来检查`String`是否为空，这意味着它只包含空白:

```
@Test
public void whenStringIsBlank_thenCorrect() {
    String first = "  ";
    String second = null;

    assertThat(first, blankString());
    assertThat(first, blankOrNullString());
    assertThat(second, blankOrNullString());
}
```

另一方面，如果我们想验证一个`String`是否为空，我们可以使用`emptyString()`匹配器:

```
@Test
public void whenStringIsEmpty_thenCorrect() {
    String first = "";
    String second = null;

    assertThat(first, emptyString());
    assertThat(first, emptyOrNullString());
    assertThat(second, emptyOrNullString());
}
```

## 5。模式匹配器

我们还可以使用`matchesPattern()`函数检查给定文本是否匹配正则表达式:

```
@Test
public void whenStringMatchPattern_thenCorrect() {
    String first = "hello";

    assertThat(first, matchesPattern("[a-z]+"));
}
```

## 6。子字符串匹配器

我们可以通过使用`containsString()`函数或`containsStringIgnoringCase():`来确定一个文本是否包含另一个子文本

```
@Test
public void whenVerifyStringContains_thenCorrect() {
    String first = "hello";

    assertThat(first, containsString("lo"));
    assertThat(first, containsStringIgnoringCase("EL"));
}
```

如果我们希望子字符串按照特定的顺序排列，我们可以调用`stringContainsInOrder()`匹配器:

```
@Test
public void whenVerifyStringContainsInOrder_thenCorrect() {
    String first = "hello";

    assertThat(first, stringContainsInOrder("e","l","o"));
}
```

接下来，让我们看看如何检查一个`String`以一个给定的`String`开始:

```
@Test
public void whenVerifyStringStartsWith_thenCorrect() {
    String first = "hello";

    assertThat(first, startsWith("he"));
    assertThat(first, startsWithIgnoringCase("HEL"));
}
```

最后，我们可以检查一个`String`是否以指定的`String`结束:

```
@Test
public void whenVerifyStringEndsWith_thenCorrect() {
    String first = "hello";

    assertThat(first, endsWith("lo"));
    assertThat(first, endsWithIgnoringCase("LO"));
}
```

## 7。结论

在这个快速教程中，我们探索了 Hamcrest 文本匹配器。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)