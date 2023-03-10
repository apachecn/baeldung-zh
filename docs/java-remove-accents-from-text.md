# 在 Java 中删除字符串中的重音和音调符号

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-accents-from-text>

## 1.概观

许多字母表包含重音和发音符号。为了可靠地搜索或索引数据，我们可能希望将带有发音符号的字符串转换为只包含 ASCII 字符的字符串。Unicode 定义了一个文本规范化过程来帮助实现这一点。

在本教程中，我们将了解什么是 Unicode 文本规范化，我们如何使用它来删除变音符号，以及需要注意的陷阱。然后，我们将看到一些使用 Java `Normalizer`类和 Apache Commons `StringUtils.`的例子

## 2.问题一目了然

假设我们正在处理包含要删除的发音符号范围的文本:

```java
āăąēîïĩíĝġńñšŝśûůŷ
```

读完这篇文章后，我们将知道如何去掉音调符号，最后得到:

```java
aaaeiiiiggnnsssuuy
```

## 3.Unicode 基础知识

在直接进入代码之前，让我们学习一些 Unicode 基础知识。

为了用变音符号或重音符号表示一个字符，Unicode 可以使用不同的代码点序列。原因是与旧字符集的历史兼容性。

**Unicode 规范化是使用标准**定义的等价形式分解字符 **。**

### 3.1.Unicode 等价形式

**为了比较代码点的序列，Unicode 定义了两个术语:`canonical equivalence`和`compatibility`。**

规范上等效的代码点在显示时具有相同的外观和含义。例如，字母“s”(带锐音符的拉丁文字母“s”)可以用一个码位+U015B 或两个码位+U0073(拉丁文字母“s”)和+U0301(锐音符)来表示。

另一方面，在某些情况下，相容的序列可以具有不同的外观，但具有相同的含义。例如，代码点+U013F(拉丁文连字“ŀ”)与序列+U004C(拉丁文字母“l”)和+U00B7(符号“”)兼容。此外，有些字体可以显示 L 内的中间点，有些可以显示其后的点。

规范上等价的序列是相容的，但反过来就不一定了。

### 3.2.字符分解

字符分解用基本字母的代码点替换复合字符，然后是组合字符(根据等价形式)。例如，此过程会将字母“ā”分解为字符“a”和“-”。

### 3.3.匹配发音符号和重音符号

一旦我们从变音符号中分离出基本字符，我们必须创建一个匹配不需要的字符的表达式。我们可以使用字符块或类别。

最流行的 Unicode 代码块是`Combining Diacritical Marks`。它不是很大，只包含 112 个最常见的组合字符。另一方面，我们也可以使用 Unicode 类别`Mark`。它由组合标记的代码点组成，并进一步分为三个子类别:

*   `Nonspacing_Mark`:这个类别包括 1839 个码点
*   `Enclosing_Mark`:包含 13 个码位
*   `Spacing_Combining_Mark`:包含 443 点

Unicode 字符块和类别之间的主要区别在于，字符块包含一系列连续的字符。另一方面，一个类别可以有许多字符块。例如，恰恰是`Combining Diacritical Marks`的情况:属于这个块的所有码点也包括在`Nonspacing_Mark `类别中。

## 4.算法

现在我们已经理解了基本的 Unicode 术语，我们可以设计算法从`String`中删除变音符号。

首先，我们将**使用`Normalizer`类**将基本字符与重音和发音符号分开。此外，我们将执行表示为 Java enum `NFKD`的兼容性分解。此外，我们使用兼容性分解，因为它比规范方法分解更多的连字(例如，连字“flai”)。

其次，我们将**使用`\p{M}`正则表达式**删除所有匹配 Unicode `Mark`类别的字符。我们选择这个类别，因为它提供了最广泛的标志。

## 5.使用核心 Java

让我们从一些使用核心 Java 的例子开始。

### 5.1.检查`String`是否正常

在我们执行规范化之前，我们可能想要检查一下`String`是否已经被规范化:

```java
assertFalse(Normalizer.isNormalized("āăąēîïĩíĝġńñšŝśûůŷ", Normalizer.Form.NFKD));
```

### 5.2.字符串分解

如果我们的`String`没有被规范化，我们继续下一步。为了将 ASCII 字符与变音符号分开，我们将使用兼容性分解来执行 Unicode 文本规范化:

```java
private static String normalize(String input) {
    return input == null ? null : Normalizer.normalize(input, Normalizer.Form.NFKD);
}
```

在这一步之后，字母“a”和“b”都将被简化为“a ”,后面跟着各自的变音符号。

### 5.3.删除代表变音符号和重音符号的代码点

一旦我们分解了我们的`String`，我们想要移除不想要的代码点。因此，我们将使用 [Unicode 正则表达式](https://web.archive.org/web/20220524023511/https://www.regular-expressions.info/unicode.html) `\p{M}`:

```java
static String removeAccents(String input) {
    return normalize(input).replaceAll("\\p{M}", "");
}
```

### 5.4.试验

让我们看看我们的分解在实践中是如何工作的。首先，让我们挑选具有由 Unicode 定义的规范化形式的字符，并期望删除所有变音符号:

```java
@Test
void givenStringWithDecomposableUnicodeCharacters_whenRemoveAccents_thenReturnASCIIString() {
    assertEquals("aaaeiiiiggnnsssuuy", StringNormalizer.removeAccents("āăąēîïĩíĝġńñšŝśûůŷ"));
}
```

其次，让我们挑选几个没有分解映射的字符:

```java
@Test
void givenStringWithNondecomposableUnicodeCharacters_whenRemoveAccents_thenReturnOriginalString() {
    assertEquals("łđħœ", StringNormalizer.removeAccents("łđħœ"));
}
```

不出所料，我们的方法无法分解它们。

此外，我们可以创建一个测试来验证分解字符的十六进制代码:

```java
@Test
void givenStringWithDecomposableUnicodeCharacters_whenUnicodeValueOfNormalizedString_thenReturnUnicodeValue() {
    assertEquals("\\u0066 \\u0069", StringNormalizer.unicodeValueOfNormalizedString("ﬁ"));
    assertEquals("\\u0061 \\u0304", StringNormalizer.unicodeValueOfNormalizedString("ā"));
    assertEquals("\\u0069 \\u0308", StringNormalizer.unicodeValueOfNormalizedString("ï"));
    assertEquals("\\u006e \\u0301", StringNormalizer.unicodeValueOfNormalizedString("ń"));
}
```

### 5.5.使用`Collator`比较包含重音符号的字符串

包`java.text`包含了另一个有趣的类——`Collator`。它**使我们能够执行区域敏感的`String`比较**。一个重要的配置属性是`Collator's`强度。此属性定义了在比较过程中被视为显著的最小差异水平。

**Java 为一个`Collator`** 提供了四个力量值:

*   省略大小写和重音的比较
*   `SECONDARY`:省略大小写但包括重音和音调符号的比较
*   `TERTIARY`:包括大小写和重音在内的比较
*   所有差异都是显著的

让我们看一些例子，首先是主要力量:

```java
Collator collator = Collator.getInstance();
collator.setDecomposition(2);
collator.setStrength(0);
assertEquals(0, collator.compare("a", "a"));
assertEquals(0, collator.compare("ä", "a"));
assertEquals(0, collator.compare("A", "a"));
assertEquals(1, collator.compare("b", "a"));
```

次要强度打开重音敏感性:

```java
collator.setStrength(1);
assertEquals(1, collator.compare("ä", "a"));
assertEquals(1, collator.compare("b", "a"));
assertEquals(0, collator.compare("A", "a"));
assertEquals(0, collator.compare("a", "a"));
```

三级强度包括以下情况:

```java
collator.setStrength(2);
assertEquals(1, collator.compare("A", "a"));
assertEquals(1, collator.compare("ä", "a"));
assertEquals(1, collator.compare("b", "a"));
assertEquals(0, collator.compare("a", "a"));
assertEquals(0, collator.compare(valueOf(toChars(0x0001)), valueOf(toChars(0x0002))));
```

相同的力量使所有的差异变得重要。倒数第二个例子很有趣，因为我们可以发现 Unicode 控制代码点+U001(“标题开始”的代码)和+U002(“文本开始”)之间的区别:

```java
collator.setStrength(3);
assertEquals(1, collator.compare("A", "a"));
assertEquals(1, collator.compare("ä", "a"));
assertEquals(1, collator.compare("b", "a"));
assertEquals(-1, collator.compare(valueOf(toChars(0x0001)), valueOf(toChars(0x0002))));
assertEquals(0, collator.compare("a", "a")));
```

最后一个值得一提的例子表明，**如果字符没有定义的分解规则，它将不会被认为等同于另一个具有相同基本字母**的字符。这是因为 **`Collator`不能执行 Unicode 分解**:

```java
collator.setStrength(0);
assertEquals(1, collator.compare("ł", "l"));
assertEquals(1, collator.compare("ø", "o")); 
```

## 6.使用 Apache Commons `StringUtils`

既然我们已经看到了如何使用核心 Java 来消除重音，我们将检查一下 [Apache Commons Text](/web/20220524023511/https://www.baeldung.com/java-apache-commons-text) 提供了什么。我们很快就会知道，**更容易使用，但是我们对分解过程的控制更少。它使用具有`NFD`分解形式和正则表达式的`Normalizer.normalize()`方法:**

```java
static String removeAccentsWithApacheCommons(String input) {
    return StringUtils.stripAccents(input);
}
```

### 6.1.试验

让我们在实践中看看这个方法——首先，**只有可分解的 Unicode 字符**:

```java
@Test
void givenStringWithDecomposableUnicodeCharacters_whenRemoveAccentsWithApacheCommons_thenReturnASCIIString() {
    assertEquals("aaaeiiiiggnnsssuuy", StringNormalizer.removeAccentsWithApacheCommons("āăąēîïĩíĝġńñšŝśûůŷ"));
}
```

不出所料，我们去掉了所有的口音。

让我们尝试一个包含连字和带笔画的字母的字符串**:**

```java
@Test 
void givenStringWithNondecomposableUnicodeCharacters_whenRemoveAccentsWithApacheCommons_thenReturnModifiedString() {
    assertEquals("lđħœ", StringNormalizer.removeAccentsWithApacheCommons("łđħœ"));
}
```

正如我们所看到的，**`StringUtils.stripAccents()`方法手动定义了拉丁字符和拉丁字符的翻译规则。但是，不幸的是，它不能正常化其他的连写**。

## 7.Java 中字符分解的局限性

综上，我们看到有些角色没有定义分解规则。更具体地说， **Unicode 没有为带有笔画**的连字和字符定义分解规则。正因为如此，Java 也不能规范化它们。**如果我们想摆脱这些字符，我们必须手动定义转录映射。**

最后，值得考虑的是我们是否需要去掉重音符号和音调符号。对于某些语言来说，去掉发音符号的字母没有多大意义。在这种情况下，更好的办法是使用`Collator`类并比较两个`Strings`，包括地区信息。

## 8.结论

在本文中，我们研究了如何使用核心 Java 和流行的 Java 实用程序库 Apache Commons 来消除重音和发音符号。我们还看到了一些例子，学习了如何比较包含重音的文本，以及在处理包含重音的文本时需要注意的一些事情。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220524023511/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)