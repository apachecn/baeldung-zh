# Java 正则表达式 API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/regular-expressions-java>

## 1。概述

在本文中，我们将讨论 Java Regex API 以及如何在 Java 编程语言中使用正则表达式。

在正则表达式的世界里，有许多不同的风格可供选择，比如 grep、Perl、Python、PHP、awk 等等。

这意味着在一种编程语言中有效的正则表达式在另一种语言中可能无效。Java 中的正则表达式语法与 Perl 中的非常相似。

## 2。设置

要在 Java 中使用正则表达式，我们不需要任何特殊的设置。JDK 包含一个专门用于正则表达式操作的特殊包 `java.util.regex`。我们只需要将它导入到我们的代码中。

此外，`java.lang.String`类还内置了我们在代码中常用的正则表达式支持。

## 3。Java Regex 包

`java.util.regex`包由三个类组成:`Pattern, Matcher` 和 `PatternSyntaxException:`

*   `Pattern`对象是一个已编译的正则表达式。`Pattern` 类没有提供公共构造函数。要创建一个模式，我们必须首先调用它的一个公共静态`compile` 方法，然后返回一个`Pattern` 对象。这些方法接受正则表达式作为第一个参数。
*   `Matcher`对象解释模式并对输入`String`执行匹配操作。它也没有定义公共构造函数。我们通过在一个`Pattern` 对象上调用`matcher` 方法来获得一个`Matcher` 对象。
*   `PatternSyntaxException` object 是一个未检查的异常，表示正则表达式模式中存在语法错误。

我们将详细探讨这些类；但是，我们必须首先了解正则表达式是如何在 Java 中构造的。

如果您已经熟悉了来自不同环境的 regex，您可能会发现某些差异，但它们是很小的。

## 4。简单的例子

让我们从正则表达式最简单的用例开始。正如我们前面提到的，当一个正则表达式应用于一个字符串时，它可能匹配零次或多次。

由`java.util.regex` API 支持的最基本的模式匹配形式是`String`文字的**匹配。例如，如果正则表达式是`foo` ，输入`String`是`foo`，匹配将会成功，因为`Strings`是相同的:**

```java
@Test
public void givenText_whenSimpleRegexMatches_thenCorrect() {
    Pattern pattern = Pattern.compile("foo");
    Matcher matcher = pattern.matcher("foo");

    assertTrue(matcher.find());
}
```

我们首先创建一个`Pattern`对象，方法是调用它的静态`compile`方法，并传递给它一个我们想要使用的模式。

然后我们创建一个`Matcher`对象，调用`Pattern`对象的`matcher`方法，并向其传递我们想要检查匹配的文本。

之后，我们调用 Matcher 对象中的方法`find`。

`find`方法在输入文本中不断前进，并为每个匹配返回 true，因此我们也可以用它来查找匹配计数:

```java
@Test
public void givenText_whenSimpleRegexMatchesTwice_thenCorrect() {
    Pattern pattern = Pattern.compile("foo");
    Matcher matcher = pattern.matcher("foofoo");
    int matches = 0;
    while (matcher.find()) {
        matches++;
    }

    assertEquals(matches, 2);
}
```

由于我们将运行更多的测试，我们可以在一个名为`runTest`的方法中抽象出寻找匹配数量的逻辑:

```java
public static int runTest(String regex, String text) {
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(text);
    int matches = 0;
    while (matcher.find()) {
        matches++;
    }
    return matches;
}
```

当我们得到 0 个匹配时，测试应该失败，否则，应该通过。

## 5。元字符

元字符影响模式匹配的方式，在某种程度上给搜索模式增加了逻辑。Java API 支持几个元字符，最直接的是匹配任何字符的点`“.”` :

```java
@Test
public void givenText_whenMatchesWithDotMetach_thenCorrect() {
    int matches = runTest(".", "foo");

    assertTrue(matches > 0);
}
```

考虑前面的例子，regex `foo`匹配文本`foo`和`foofoo`两次。如果我们在正则表达式中使用点元字符，在第二种情况下我们不会得到两个匹配:

```java
@Test
public void givenRepeatedText_whenMatchesOnceWithDotMetach_thenCorrect() {
    int matches= runTest("foo.", "foofoo");

    assertEquals(matches, 1);
}
```

注意正则表达式中`foo`后面的点。匹配器匹配前面有`foo`的每个文本，因为最后一个点表示后面的任何字符。所以在找到第一个`foo`后，剩下的就被视为任意角色。这就是为什么只有一个匹配。

API 支持其他几个元字符`<([{\^-=$!|]})?*+.>`，我们将在本文中进一步探讨。

## 6。字符分类

浏览官方的`[Pattern](https://web.archive.org/web/20220817021522/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html)` 类规范，我们会发现支持的正则表达式结构的摘要。在字符类下，我们有大约 6 个构造。

### 6.1。`OR`阶级

构造为`[abc]`。集合中的任何元素都匹配:

```java
@Test
public void givenORSet_whenMatchesAny_thenCorrect() {
    int matches = runTest("[abc]", "b");

    assertEquals(matches, 1);
}
```

如果它们都出现在文本中，则不管顺序如何，每个都会单独匹配:

```java
@Test
public void givenORSet_whenMatchesAnyAndAll_thenCorrect() {
    int matches = runTest("[abc]", "cab");

    assertEquals(matches, 3);
}
```

它们也可以作为`String`的一部分交替出现。在下面的示例中，当我们通过将首字母与集合中的每个元素交替来创建不同的单词时，它们都是匹配的:

```java
@Test
public void givenORSet_whenMatchesAllCombinations_thenCorrect() {
    int matches = runTest("[bcr]at", "bat cat rat");

    assertEquals(matches, 3);
}
```

### 6.2。`NOR`阶级

通过添加一个插入符号作为第一个元素来否定上面的集合:

```java
@Test
public void givenNORSet_whenMatchesNon_thenCorrect() {
    int matches = runTest("[^abc]", "g");

    assertTrue(matches > 0);
}
```

另一个案例:

```java
@Test
public void givenNORSet_whenMatchesAllExceptElements_thenCorrect() {
    int matches = runTest("[^bcr]at", "sat mat eat");

    assertTrue(matches > 0);
}
```

### 6.3。范围等级

我们可以定义一个类，使用连字符(-)指定匹配文本应该落入的范围，同样，我们也可以否定一个范围。

匹配大写字母:

```java
@Test
public void givenUpperCaseRange_whenMatchesUpperCase_
  thenCorrect() {
    int matches = runTest(
      "[A-Z]", "Two Uppercase alphabets 34 overall");

    assertEquals(matches, 2);
}
```

匹配小写字母:

```java
@Test
public void givenLowerCaseRange_whenMatchesLowerCase_
  thenCorrect() {
    int matches = runTest(
      "[a-z]", "Two Uppercase alphabets 34 overall");

    assertEquals(matches, 26);
}
```

匹配大写和小写字母:

```java
@Test
public void givenBothLowerAndUpperCaseRange_
  whenMatchesAllLetters_thenCorrect() {
    int matches = runTest(
      "[a-zA-Z]", "Two Uppercase alphabets 34 overall");

    assertEquals(matches, 28);
}
```

匹配给定范围的数字:

```java
@Test
public void givenNumberRange_whenMatchesAccurately_
  thenCorrect() {
    int matches = runTest(
      "[1-5]", "Two Uppercase alphabets 34 overall");

    assertEquals(matches, 2);
}
```

匹配另一组数字:

```java
@Test
public void givenNumberRange_whenMatchesAccurately_
  thenCorrect2(){
    int matches = runTest(
      "3[0-5]", "Two Uppercase alphabets 34 overall");

    assertEquals(matches, 1);
}
```

6.4。工会类

联合字符类是两个或多个字符类组合的结果:

```java
@Test
public void givenTwoSets_whenMatchesUnion_thenCorrect() {
    int matches = runTest("[1-3[7-9]]", "123456789");

    assertEquals(matches, 6);
}
```

上面的测试将只匹配 9 个整数中的 6 个，因为并集跳过了 4、5 和 6。

### 6.5。交集类

类似于 union 类，这个类是从两个或多个集合中挑选公共元素的结果。要应用交集，我们使用`&&`:

```java
@Test
public void givenTwoSets_whenMatchesIntersection_thenCorrect() {
    int matches = runTest("[1-6&&[3-9]]", "123456789");

    assertEquals(matches, 4);
}
```

我们得到 4 个匹配，因为两个集合的交集只有 4 个元素。

### 6.6。减法类

我们可以使用减法来否定一个或多个字符类，例如匹配一组奇数十进制数:

```java
@Test
public void givenSetWithSubtraction_whenMatchesAccurately_thenCorrect() {
    int matches = runTest("[0-9&&[^2468]]", "123456789");

    assertEquals(matches, 5);
}
```

只会匹配`1,3,5,7,9`。

## 7。预定义的字符类别

Java regex API 也接受预定义的字符类。上面的一些字符类可以用更短的形式来表达，虽然这使得代码不太直观。这个正则表达式的 Java 版本的一个特殊方面是转义字符。

正如我们将看到的，大多数字符将以反斜杠开头，这在 Java 中有特殊的含义。为了让这些由`Pattern`类编译，必须对前导反斜杠进行转义，即`\d`变成了`\\d`。

匹配位数，相当于`[0-9]`:

```java
@Test
public void givenDigits_whenMatches_thenCorrect() {
    int matches = runTest("\\d", "123");

    assertEquals(matches, 3);
}
```

匹配非数字，相当于`[^0-9]`:

```java
@Test
public void givenNonDigits_whenMatches_thenCorrect() {
    int mathces = runTest("\\D", "a6c");

    assertEquals(matches, 2);
}
```

匹配空白:

```java
@Test
public void givenWhiteSpace_whenMatches_thenCorrect() {
    int matches = runTest("\\s", "a c");

    assertEquals(matches, 1);
}
```

匹配非空白:

```java
@Test
public void givenNonWhiteSpace_whenMatches_thenCorrect() {
    int matches = runTest("\\S", "a c");

    assertEquals(matches, 2);
}
```

匹配一个单词字符，相当于`[a-zA-Z_0-9]`:

```java
@Test
public void givenWordCharacter_whenMatches_thenCorrect() {
    int matches = runTest("\\w", "hi!");

    assertEquals(matches, 2);
}
```

匹配非单词字符:

```java
@Test
public void givenNonWordCharacter_whenMatches_thenCorrect() {
    int matches = runTest("\\W", "hi!");

    assertEquals(matches, 1);
}
```

## 8。量词

Java regex API 也允许我们使用量词。这些使我们能够通过指定匹配的出现次数来进一步调整匹配的行为。

为了零次或一次匹配一个文本，我们使用 `?`量词:

```java
@Test
public void givenZeroOrOneQuantifier_whenMatches_thenCorrect() {
    int matches = runTest("\\a?", "hi");

    assertEquals(matches, 3);
}
```

或者，我们可以使用大括号语法，Java regex API 也支持该语法:

```java
@Test
public void givenZeroOrOneQuantifier_whenMatches_thenCorrect2() {
    int matches = runTest("\\a{0,1}", "hi");

    assertEquals(matches, 3);
}
```

这个例子引入了零长度匹配的概念。碰巧的是，如果量词的匹配阈值是零，它总是匹配文本中的所有内容，包括每个输入末尾的空`String`。这意味着即使输入为空，它也将返回一个零长度匹配。

这解释了为什么我们在上面的例子中得到 3 个匹配，尽管 S `tring`的长度是 2。第三个匹配是零长度空*字符串*。

为了匹配一个文本零次或无限次，我们用*量词，它就类似于？：

```java
@Test
public void givenZeroOrManyQuantifier_whenMatches_thenCorrect() {
     int matches = runTest("\\a*", "hi");

     assertEquals(matches, 3);
}
```

支持的备选方案:

```java
@Test
public void givenZeroOrManyQuantifier_whenMatches_thenCorrect2() {
    int matches = runTest("\\a{0,}", "hi");

    assertEquals(matches, 3);
}
```

有差异的量词是+，它的匹配阈值为 1。如果所需的`String`根本没有出现，则没有匹配，甚至没有零长度的`String`:

```java
@Test
public void givenOneOrManyQuantifier_whenMatches_thenCorrect() {
    int matches = runTest("\\a+", "hi");

    assertFalse(matches);
}
```

支持的备选方案:

```java
@Test
public void givenOneOrManyQuantifier_whenMatches_thenCorrect2() {
    int matches = runTest("\\a{1,}", "hi");

    assertFalse(matches);
}
```

正如在 Perl 和其他语言中一样，大括号语法可以用来多次匹配给定的文本:

```java
@Test
public void givenBraceQuantifier_whenMatches_thenCorrect() {
    int matches = runTest("a{3}", "aaaaaa");

    assertEquals(matches, 2);
}
```

在上面的例子中，我们得到两个匹配，因为只有当`a`连续出现三次时才匹配。然而，在下一个测试中，我们不会得到一个匹配，因为文本只连续出现两次:

```java
@Test
public void givenBraceQuantifier_whenFailsToMatch_thenCorrect() {
    int matches = runTest("a{3}", "aa");

    assertFalse(matches > 0);
}
```

当我们在大括号中使用一个范围时，匹配将是贪婪的，从范围的高端匹配:

```java
@Test
public void givenBraceQuantifierWithRange_whenMatches_thenCorrect() {
    int matches = runTest("a{2,3}", "aaaa");

    assertEquals(matches, 1);
}
```

我们已经指定了至少两次出现，但不超过三次，所以我们得到一个匹配，匹配器看到一个单独的`aaa`和一个不能匹配的`a`。

然而，API 允许我们指定一种懒惰或勉强的方法，这样匹配器可以从范围的低端开始，在这种情况下匹配两个事件作为`aa`和`aa`:

```java
@Test
public void givenBraceQuantifierWithRange_whenMatchesLazily_thenCorrect() {
    int matches = runTest("a{2,3}?", "aaaa");

    assertEquals(matches, 2);
}
```

## 9。捕获组

API 还允许我们通过捕获组将多个角色作为一个单元对待。

它会将数字附加到捕获组，并允许使用这些数字进行反向引用。

在这一节中，我们将看到一些如何在 Java regex API 中使用捕获组的例子。

让我们使用仅当输入文本包含两个相邻的数字时才匹配的捕获组:

```java
@Test
public void givenCapturingGroup_whenMatches_thenCorrect() {
    int matches = runTest("(\\d\\d)", "12");

    assertEquals(matches, 1);
}
```

上面匹配的数字是`1`，使用反向引用告诉匹配器我们想要匹配文本匹配部分的另一个出现。这样，而不是:

```java
@Test
public void givenCapturingGroup_whenMatches_thenCorrect2() {
    int matches = runTest("(\\d\\d)", "1212");

    assertEquals(matches, 2);
}
```

当输入有两个单独的匹配时，我们可以有一个匹配，但是使用反向引用传播相同的正则表达式匹配以跨越输入的整个长度:

```java
@Test
public void givenCapturingGroup_whenMatchesWithBackReference_
  thenCorrect() {
    int matches = runTest("(\\d\\d)\\1", "1212");

    assertEquals(matches, 1);
}
```

在这里，我们必须重复正则表达式而不进行反向引用，以获得相同的结果:

```java
@Test
public void givenCapturingGroup_whenMatches_thenCorrect3() {
    int matches = runTest("(\\d\\d)(\\d\\d)", "1212");

    assertEquals(matches, 1);
}
```

类似地，对于任何其他数量的重复，反向引用可以使匹配器将输入视为单个匹配:

```java
@Test
public void givenCapturingGroup_whenMatchesWithBackReference_
  thenCorrect2() {
    int matches = runTest("(\\d\\d)\\1\\1\\1", "12121212");

    assertEquals(matches, 1);
}
```

但是如果你改变了最后一个数字，匹配就会失败:

```java
@Test
public void givenCapturingGroupAndWrongInput_
  whenMatchFailsWithBackReference_thenCorrect() {
    int matches = runTest("(\\d\\d)\\1", "1213");

    assertFalse(matches > 0);
}
```

重要的是不要忘记转义反斜杠，这在 Java 语法中是至关重要的。

## 10。边界匹配器

Java regex API 也支持边界匹配。如果我们关心输入文本中匹配应该出现的确切位置，那么这就是我们要寻找的。在前面的例子中，我们关心的是是否找到匹配。

为了仅在文本开头所需的正则表达式为真时进行匹配，我们使用插入符号`^.`

该测试将失败，因为可以在开头找到文本`dog`:

```java
@Test
public void givenText_whenMatchesAtBeginning_thenCorrect() {
    int matches = runTest("^dog", "dogs are friendly");

    assertTrue(matches > 0);
}
```

以下测试将失败:

```java
@Test
public void givenTextAndWrongInput_whenMatchFailsAtBeginning_
  thenCorrect() {
    int matches = runTest("^dog", "are dogs are friendly?");

    assertFalse(matches > 0);
}
```

为了仅在文本末尾所需的正则表达式为真时进行匹配，我们使用美元字符`$.` 在以下情况下将找到匹配:

```java
@Test
public void givenText_whenMatchesAtEnd_thenCorrect() {
    int matches = runTest("dog$", "Man's best friend is a dog");

    assertTrue(matches > 0);
}
```

在这里找不到匹配:

```java
@Test
public void givenTextAndWrongInput_whenMatchFailsAtEnd_thenCorrect() {
    int matches = runTest("dog$", "is a dog man's best friend?");

    assertFalse(matches > 0);
}
```

如果我们只希望在单词边界找到所需文本时进行匹配，我们在正则表达式的开头和结尾使用`\\b`正则表达式:

空格是一个单词的边界:

```java
@Test
public void givenText_whenMatchesAtWordBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\b", "a dog is friendly");

    assertTrue(matches > 0);
}
```

行首的空字符串也是单词边界:

```java
@Test
public void givenText_whenMatchesAtWordBoundary_thenCorrect2() {
    int matches = runTest("\\bdog\\b", "dog is man's best friend");

    assertTrue(matches > 0);
}
```

这些测试通过是因为一个*字符串*的开头，以及一个文本和另一个文本之间的空格，标记了一个单词边界，然而，下面的测试显示了相反的情况:

```java
@Test
public void givenWrongText_whenMatchFailsAtWordBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\b", "snoop dogg is a rapper");

    assertFalse(matches > 0);
}
```

出现在一行中的两个单词的字符不标记单词边界，但是我们可以通过改变正则表达式的结尾来寻找非单词边界，从而使它通过:

```java
@Test
public void givenText_whenMatchesAtWordAndNonBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\B", "snoop dogg is a rapper");
    assertTrue(matches > 0);
}
```

## 11。模式类方法

之前，我们只以基本的方式创建了`Pattern`对象。然而，这个类有另一个`compile`方法的变体，它接受一组标志以及影响模式匹配方式的 regex 参数。

这些标志只是抽象的整数值。让我们在测试类中重载`runTest`方法，这样它就可以将一个标志作为第三个参数:

```java
public static int runTest(String regex, String text, int flags) {
    pattern = Pattern.compile(regex, flags);
    matcher = pattern.matcher(text);
    int matches = 0;
    while (matcher.find()){
        matches++;
    }
    return matches;
}
```

在本节中，我们将了解不同的支持标志以及它们的用法。

**T2`Pattern.CANON_EQ`**

该标志启用规范等价。当指定时，当且仅当两个字符的完全规范分解匹配时，才认为这两个字符匹配。

考虑带重音的 Unicode 字符`é`。它的复合码点是`u00E9`。但是，Unicode 对于其组成字符`e`、`u0065`和重音符号`u0301`也有单独的代码点。在这种情况下，复合字符`u00E9`与两个字符序列`u0065 u` `0301`无法区分。

默认情况下，匹配不考虑规范等价:

```java
@Test
public void givenRegexWithoutCanonEq_whenMatchFailsOnEquivalentUnicode_thenCorrect() {
    int matches = runTest("\u00E9", "\u0065\u0301");

    assertFalse(matches > 0);
}
```

但是如果我们添加了标志，那么测试将会通过:

```java
@Test
public void givenRegexWithCanonEq_whenMatchesOnEquivalentUnicode_thenCorrect() {
    int matches = runTest("\u00E9", "\u0065\u0301", Pattern.CANON_EQ);

    assertTrue(matches > 0);
}
```

`**Pattern.CASE_INSENSITIVE**`

此标志启用匹配，不考虑大小写。默认情况下，匹配会考虑大小写:

```java
@Test
public void givenRegexWithDefaultMatcher_whenMatchFailsOnDifferentCases_thenCorrect() {
    int matches = runTest("dog", "This is a Dog");

    assertFalse(matches > 0);
}
```

因此，使用此标志，我们可以更改默认行为:

```java
@Test
public void givenRegexWithCaseInsensitiveMatcher
  _whenMatchesOnDifferentCases_thenCorrect() {
    int matches = runTest(
      "dog", "This is a Dog", Pattern.CASE_INSENSITIVE);

    assertTrue(matches > 0);
}
```

我们还可以使用等效的嵌入式标志表达式来实现相同的结果:

```java
@Test
public void givenRegexWithEmbeddedCaseInsensitiveMatcher
  _whenMatchesOnDifferentCases_thenCorrect() {
    int matches = runTest("(?i)dog", "This is a Dog");

    assertTrue(matches > 0);
}
```

`**Pattern.COMMENTS**`

Java API 允许在正则表达式中使用#包含注释。这有助于记录复杂的正则表达式，对于另一个程序员来说可能不是很明显。

注释标志使匹配器忽略正则表达式中的任何空白或注释，只考虑模式。在默认匹配模式下，以下测试会失败:

```java
@Test
public void givenRegexWithComments_whenMatchFailsWithoutFlag_thenCorrect() {
    int matches = runTest(
      "dog$  #check for word dog at end of text", "This is a dog");

    assertFalse(matches > 0);
}
```

这是因为匹配器将在输入文本中查找整个正则表达式，包括空格和#字符。但是当我们使用标志时，它将忽略多余的空格，并且以#开头的每个文本都将被视为每行都将被忽略的注释:

```java
@Test
public void givenRegexWithComments_whenMatchesWithFlag_thenCorrect() {
    int matches = runTest(
      "dog$  #check end of text","This is a dog", Pattern.COMMENTS);

    assertTrue(matches > 0);
}
```

还有一个替代的嵌入式标志表达式:

```java
@Test
public void givenRegexWithComments_whenMatchesWithEmbeddedFlag_thenCorrect() {
    int matches = runTest(
      "(?x)dog$  #check end of text", "This is a dog");

    assertTrue(matches > 0);
}
```

`**Pattern.DOTALL**`

默认情况下，当我们使用点“.”时表达式，我们匹配输入中的每个字符`String`，直到我们遇到一个新的行字符。

使用这个标志，匹配也将包括行结束符。下面的例子会让我们更好的理解。这些例子会有一点不同。由于我们对断言匹配的`String`感兴趣，我们将使用`matcher`的`group`方法，它返回前一个匹配。

首先，我们将看到默认行为:

```java
@Test
public void givenRegexWithLineTerminator_whenMatchFails_thenCorrect() {
    Pattern pattern = Pattern.compile("(.*)");
    Matcher matcher = pattern.matcher(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line");
    matcher.find();

    assertEquals("this is a text", matcher.group(1));
}
```

正如我们所看到的，只有行结束符之前的输入的第一部分被匹配。

现在在`dotall`模式下，包括行结束符在内的整个文本将被匹配:

```java
@Test
public void givenRegexWithLineTerminator_whenMatchesWithDotall_thenCorrect() {
    Pattern pattern = Pattern.compile("(.*)", Pattern.DOTALL);
    Matcher matcher = pattern.matcher(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line");
    matcher.find();
    assertEquals(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line", matcher.group(1));
}
```

我们还可以使用嵌入的标志表达式来启用`dotall`模式:

```java
@Test
public void givenRegexWithLineTerminator_whenMatchesWithEmbeddedDotall
  _thenCorrect() {

    Pattern pattern = Pattern.compile("(?s)(.*)");
    Matcher matcher = pattern.matcher(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line");
    matcher.find();

    assertEquals(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line", matcher.group(1));
}
```

`**Pattern.LITERAL**`

在这种模式下，matcher 不会给任何元字符、转义字符或正则表达式语法赋予特殊的含义。如果没有这个标志，匹配器将根据任何输入匹配下面的正则表达式`String`:

```java
@Test
public void givenRegex_whenMatchesWithoutLiteralFlag_thenCorrect() {
    int matches = runTest("(.*)", "text");

    assertTrue(matches > 0);
}
```

这是我们在所有例子中看到的默认行为。然而，有了这个标志，就找不到匹配，因为匹配器将寻找`(.*)`而不是解释它:

```java
@Test
public void givenRegex_whenMatchFailsWithLiteralFlag_thenCorrect() {
    int matches = runTest("(.*)", "text", Pattern.LITERAL);

    assertFalse(matches > 0);
}
```

现在，如果我们添加所需的字符串，测试将通过:

```java
@Test
public void givenRegex_whenMatchesWithLiteralFlag_thenCorrect() {
    int matches = runTest("(.*)", "text(.*)", Pattern.LITERAL);

    assertTrue(matches > 0);
}
```

没有用于启用文字分析的嵌入标志字符。

`**Pattern.MULTILINE**`

默认情况下，`^`和`$`元字符分别在整个输入`String`的开头和结尾完全匹配。匹配器忽略任何行终止符:

```java
@Test
public void givenRegex_whenMatchFailsWithoutMultilineFlag_thenCorrect() {
    int matches = runTest(
      "dog$", "This is a dog" + System.getProperty("line.separator") 
      + "this is a fox");

    assertFalse(matches > 0);
}
```

匹配失败是因为匹配器在整个`String`的末尾搜索`dog`，但是`dog`出现在字符串的第一行的末尾。

但是，有了标志，同样的测试也将通过，因为匹配器现在考虑了行终止符。因此，就在该行结束之前找到了字符串`dog`，因此成功了:

```java
@Test
public void givenRegex_whenMatchesWithMultilineFlag_thenCorrect() {
    int matches = runTest(
      "dog$", "This is a dog" + System.getProperty("line.separator") 
      + "this is a fox", Pattern.MULTILINE);

    assertTrue(matches > 0);
}
```

下面是嵌入的标志版本:

```java
@Test
public void givenRegex_whenMatchesWithEmbeddedMultilineFlag_
  thenCorrect() {
    int matches = runTest(
      "(?m)dog$", "This is a dog" + System.getProperty("line.separator") 
      + "this is a fox");

    assertTrue(matches > 0);
}
```

## 12。匹配器类方法

在这一节中，我们将看看`Matcher`类的一些有用的方法。为了清楚起见，我们将根据功能对它们进行分组。

### 12.1。索引方法

索引方法提供了有用的索引值，可以精确地显示在输入`String`中找到匹配的位置。在下面的测试中，我们将确认输入`String`中`dog`匹配的开始和结束索引:

```java
@Test
public void givenMatch_whenGetsIndices_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher("This dog is mine");
    matcher.find();

    assertEquals(5, matcher.start());
    assertEquals(8, matcher.end());
}
```

### 12.2。学习方法

学习方法检查输入`String`并返回一个布尔值，表明是否找到了模式。常用的有`matches`和`lookingAt`方法。

`matches`和`lookingAt`方法都试图将一个输入序列与一个模式进行匹配。不同之处在于，`matches`要求匹配整个输入序列，而`lookingAt`不要求。

两种方法都从输入`String`的开头开始:

```java
@Test
public void whenStudyMethodsWork_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher("dogs are friendly");

    assertTrue(matcher.lookingAt());
    assertFalse(matcher.matches());
}
```

在如下情况下，matches 方法将返回 true:

```java
@Test
public void whenMatchesStudyMethodWorks_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher("dog");

    assertTrue(matcher.matches());
}
```

### 12.3。更换方法

替换方法对于替换输入字符串中的文本很有用。常见的有`replaceFirst`和`replaceAll`。

`replaceFirst`和`replaceAll`方法替换匹配给定正则表达式的文本。顾名思义，`replaceFirst`替换第一个出现，`replaceAll`替换所有出现:

```java
@Test
public void whenReplaceFirstWorks_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher(
      "dogs are domestic animals, dogs are friendly");
    String newStr = matcher.replaceFirst("cat");

    assertEquals(
      "cats are domestic animals, dogs are friendly", newStr);
}
```

替换所有事件:

```java
@Test
public void whenReplaceAllWorks_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher(
      "dogs are domestic animals, dogs are friendly");
    String newStr = matcher.replaceAll("cat");

    assertEquals("cats are domestic animals, cats are friendly", newStr);
}
```

`replaceAll`方法允许我们用相同的替换来替换所有匹配。如果我们想要逐个替换匹配，我们需要一种[令牌替换技术](/web/20220817021522/https://www.baeldung.com/java-regex-token-replacement)。

## 13。结论

在本文中，我们学习了如何在 Java 中使用正则表达式，并探索了`java.util.regex`包最重要的特性。

该项目的完整源代码，包括这里使用的所有代码样本，可以在 GitHub 项目中找到。