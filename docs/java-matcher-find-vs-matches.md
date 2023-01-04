# Java Matcher find()和 matches()的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-matcher-find-vs-matches>

## 1.概观

在 Java 中使用正则表达式时，我们通常希望在字符序列中搜索给定的*模式*。 为了方便起见， [Java 正则表达式 API](/web/20221206190645/https://www.baeldung.com/regular-expressions-java) 提供了 *Matcher* 类，我们可以用它来匹配给定的正则表达式和文本。

**作为一般规则，我们几乎总是想要使用`Matcher` 类**的两个流行方法之一:

*   `find()`
*   `matches()`

在这个快速教程中，我们将通过一组简单的例子来了解这些方法之间的区别。

## 2.`find()`法

**简单地说，`find()` 方法试图在给定的字符串**中找到正则表达式模式的出现。如果在字符串中找到多个匹配项，那么对`find()`的第一次调用将跳转到第一个匹配项。此后，对`find()` 方法的每个后续调用将一个接一个地进行到下一个匹配的出现。

假设我们想在提供的字符串`“goodbye 2019 and welcome 2020”` 中只搜索四位数。

为此，我们将使用模式`“\\d\\d\\d\\d”` :

```java
@Test
public void whenFindFourDigitWorks_thenCorrect() {
    Pattern stringPattern = Pattern.compile("\\d\\d\\d\\d");
    Matcher m = stringPattern.matcher("goodbye 2019 and welcome 2020");

    assertTrue(m.find());
    assertEquals(8, m.start());
    assertEquals("2019", m.group());
    assertEquals(12, m.end());

    assertTrue(m.find());
    assertEquals(25, m.start());
    assertEquals("2020", m.group());
    assertEquals(29, m.end());

    assertFalse(m.find());
}
```

因为我们在这个例子中有两个实例——`2019`和`2020`—`find()` 方法将返回`true`两次，一旦它到达匹配区域的末尾，它将返回`false`。

一旦我们找到任何匹配，我们就可以使用像 **`start()`、`group()`和`end()`这样的方法来获得关于匹配**的更多细节，如上所示。

`start()` 方法将给出匹配的开始索引，`end()`将返回匹配结束后字符的最后一个索引， **`group()`将返回匹配的实际值**。

## 3.`find(int)`法

我们还有 find 方法的重载版本— `find(int)`。它将起始索引作为一个参数，并且**将起始索引视为在字符串**中查找出现的起点。

让我们看看如何在前面的例子中使用这个方法:

```java
@Test
public void givenStartIndex_whenFindFourDigitWorks_thenCorrect() {
    Pattern stringPattern = Pattern.compile("\\d\\d\\d\\d");
    Matcher m = stringPattern.matcher("goodbye 2019 and welcome 2020");

    assertTrue(m.find(20));
    assertEquals(25, m.start());
    assertEquals("2020", m.group());
    assertEquals(29, m.end());  
}
```

由于我们已经提供了一个起始索引`20`，我们可以看到现在只找到了一个事件——`2020,` ，它按照预期出现在这个索引`.` 之后，和`find()`的情况一样，我们可以使用像`start()`、`group()`和`end()`这样的方法来提取关于匹配的更多细节。

## 4.`matches()`法

另一方面，****`matches()` 方法试图根据模式**匹配整个字符串。**

 **同样的例子，`matches()`将返回`false`:

```java
@Test
public void whenMatchFourDigitWorks_thenFail() {
    Pattern stringPattern = Pattern.compile("\\d\\d\\d\\d");
    Matcher m = stringPattern.matcher("goodbye 2019 and welcome 2020");

    assertFalse(m.matches());
} 
```

这是因为它会尝试将`“\\d\\d\\d\\d”` 与整个字符串“`goodbye 2019 and welcome 2020”` — **进行匹配，这与`find()`和`find(int)`方法不同，这两种方法都会在字符串**中的任何地方找到模式的出现。

如果我们将字符串改为四位数的数字`“2019”`，那么`matches()`将返回 `true`:

```java
@Test
public void whenMatchFourDigitWorks_thenCorrect() {
    Pattern stringPattern = Pattern.compile("\\d\\d\\d\\d");
    Matcher m = stringPattern.matcher("2019");

    assertTrue(m.matches());
    assertEquals(0, m.start());
    assertEquals("2019", m.group());
    assertEquals(4, m.end());
    assertTrue(m.matches());
}
```

如上所示，我们还可以使用像`start()`、`group()`和`end()`这样的方法来收集关于比赛的更多细节。**值得注意的一点是，多次调用`find()`可能会在调用这些方法后返回不同的输出，正如我们在第一个例子中看到的，但是`matches()` 将总是返回相同的值。**

## 5.`matcher()`和`Pattern.matches()`的区别

正如我们在上一节中看到的,`matcher()`方法返回一个`Matcher`,它将给定的输入与模式进行匹配。

另一方面，`Pattern.matches()` 是一个**静态方法，它编译一个正则表达式，** **将整个输入与它的** `.`相匹配

让我们创建测试用例来突出区别:

```java
@Test
public void whenUsingMatcher_thenReturnTrue() {
    Pattern pattern = Pattern.compile(REGEX);
    Matcher matcher = pattern.matcher(STRING_INPUT);

    assertTrue(matcher.find());
}
```

简而言之，当我们使用`matcher()`时，我们会问这样的问题:**字符串是否包含模式？**

而对于`Pattern.matches()`，我们在问:**字符串是模式吗？**

让我们来看看它的实际应用:

```java
@Test
public void whenUsingMatches_thenReturnFalse() {
    assertFalse(Pattern.matches(REGEX, STRING_INPUT));
}
```

因为`Pattern.matches()`试图匹配整个字符串，所以它返回`false`。

## 6.结论

在本文中，我们已经通过一个实例看到了`find()`、`find(int)`和`matches()`之间的区别。我们还看到了各种方法，如`start()`、`group()`和`end()`如何帮助我们提取关于给定比赛的更多细节`.`

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221206190645/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)**