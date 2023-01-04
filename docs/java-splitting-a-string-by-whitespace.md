# Java 中用空格分割字符串的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-splitting-a-string-by-whitespace>

## 1.概观

在 Java 中，`String`可以看作是多个子字符串的串联。此外，通常使用空白作为分隔符，将子字符串集合构建和存储到单个字符串中。

在本教程中，我们将**学习如何通过空白字符分割`String`，例如空格、制表符或换行符**。

## 2.`String`样品

首先，我们需要构建几个`String`样本，我们可以使用它们作为按空白分割的输入。所以，让我们从**将一些空白字符定义为`String`常量**开始，这样我们可以方便地重用它们:

```java
String SPACE = " ";
String TAB = "	";
String NEW_LINE = "\n";
```

接下来，让我们使用这些作为分隔符来定义包含不同水果名称的`String`样本:

```java
String FRUITS_TAB_SEPARATED = "Apple" + TAB + "Banana" + TAB + "Mango" + TAB + "Orange";
String FRUITS_SPACE_SEPARATED = "Apple" + SPACE + "Banana" + SPACE + "Mango" + SPACE + "Orange";
String FRUITS_NEWLINE_SEPARATED = "Apple" + NEW_LINE + "Banana" + NEW_LINE + "Mango" + NEW_LINE + "Orange";
```

最后，让我们也**编写`verifySplit()`方法，我们将重用它来验证通过空白字符**分割这些字符串的预期结果:

```java
private void verifySplit(String[] fruitArray) {
    assertEquals(4, fruitArray.length);
    assertEquals("Apple", fruitArray[0]);
    assertEquals("Banana", fruitArray[1]);
    assertEquals("Mango", fruitArray[2]);
    assertEquals("Orange", fruitArray[3]);
}
```

现在我们已经构建了输入字符串，我们准备探索不同的策略来分割这些字符串并验证分割。

## 3.使用分隔符正则表达式拆分

`String`类的 **`split()`方法是拆分字符串**的事实上的标准。它接受一个定界符 regex，并生成一个由`String`组成的数组:

```java
String[] split(String regex);
```

首先，让我们用一个空格字符分割`FRUITS_SPACE_SEPARATED` `String`:

```java
@Test
public void givenSpaceSeparatedString_whenSplitUsingSpace_shouldGetExpectedResult() {
    String fruits = FRUITS_SPACE_SEPARATED;
    String[] fruitArray = fruits.split(SPACE);
    verifySplit(fruitArray);
}
```

类似地，我们可以通过分别使用`TAB`和`NEW_LINE`作为分隔符 regex 来拆分`FRUITS_TAB_SEPARATED`和`FRUITS_NEWLINE_SEPARATED`。

接下来，让我们尝试对空格、制表符和换行符使用更通用的正则表达式，并用相同的正则表达式分割所有的字符串样本:

```java
@Test
public void givenWhiteSpaceSeparatedString_whenSplitUsingWhiteSpaceRegex_shouldGetExpectedResult() {
    String whitespaceRegex = SPACE + "|" + TAB + "|" + NEW_LINE;
    String[] allSamples = new String[] { FRUITS_SPACE_SEPARATED, FRUITS_TAB_SEPARATED, FRUITS_NEWLINE_SEPARATED };
    for (String fruits : allSamples) {
        String[] fruitArray = fruits.split(whitespaceRegex);
        verifySplit(fruitArray);
    }
}
```

到目前为止，看起来我们做对了！

最后，让我们通过使用空白元字符(`\s` ) 来简化我们的方法，空白元字符本身代表所有类型的空白字符:

```java
@Test
public void givenNewlineSeparatedString_whenSplitUsingWhiteSpaceMetaChar_shouldGetExpectedResult() {
    String whitespaceMetaChar = "\\s";
    String[] allSamples = new String[] { FRUITS_SPACE_SEPARATED, FRUITS_TAB_SEPARATED, FRUITS_NEWLINE_SEPARATED };
    for (String fruits : allSamples) {
        String[] fruitArray = fruits.split(whitespaceMetaChar);
        verifySplit(fruitArray);
    }
}
```

我们应该注意，使用元字符`\s`比创建空白的自定义正则表达式更方便、更可靠。此外，如果我们的输入字符串可以有一个以上的空白字符作为分隔符，那么我们可以在`\\s` 上使用 [`\\s+`，而不改变其余的代码。](/web/20221220042827/https://www.baeldung.com/java-regex-s-splus#diff)

## 4.使用`StringTokenizer`分割

用空格分割字符串是一个非常常见的用例，许多 Java 库都公开了一个接口来实现这一点，而没有明确指定分隔符。在本节中，让我们学习如何使用`StringTokenizer`来解决这个用例:

```java
@Test
public void givenSpaceSeparatedString_whenSplitUsingStringTokenizer_shouldGetExpectedResult() {
    String fruits = FRUITS_SPACE_SEPARATED;
    StringTokenizer tokenizer = new StringTokenizer(fruits);
    String[] fruitArray = new String[tokenizer.countTokens()];
    int index = 0;
    while (tokenizer.hasMoreTokens()) {
        fruitArray[index++] = tokenizer.nextToken();
    }
    verifySplit(fruitArray);
}
```

我们可以看到我们没有提供任何分隔符，因为默认情况下`StringTokenizer`使用空白分隔符。同样，**代码遵循[迭代器设计模式](/web/20221220042827/https://www.baeldung.com/java-iterator)，其中`hasMoreTokens()`方法决定循环终止条件，`nextToken()`给出下一个拆分**。

此外，我们应该注意，我们使用了`countTokens()`方法来预先确定分割的数量。但是，如果我们想在一个序列中一次使用一个结果分割，这不是必需的。一般来说，**当输入字符串很长时，我们应该使用这种方法，我们希望得到下一个分割，而不需要等待整个分割过程完成**。

## 5.使用 Apache Commons 进行分割

`org.apache.commons.lang3`包的 **`StringUtils`类提供了一个`split()`实用方法来拆分一个`String`** 。像`StringTokenizer`类一样，它使用空白作为分割字符串的默认分隔符:

```java
public static String[] split(String str);
```

让我们从在项目的`pom.xml`文件中添加 [`commons-lang3`](https://web.archive.org/web/20221220042827/https://search.maven.org/search?q=g:%20org.apache.commons%20AND%20a:commons-lang3) 依赖项开始:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
</dependency>
```

接下来，让我们通过分割`String`样本来看看这一过程:

```java
@Test
public void givenWhiteSpaceSeparatedString_whenSplitUsingStringUtils_shouldGetExpectedResult() {
    String[] allSamples = new String[] { FRUITS_SPACE_SEPARATED, FRUITS_TAB_SEPARATED, FRUITS_NEWLINE_SEPARATED };
    for (String fruits : allSamples) {
        String[] fruitArray = StringUtils.split(fruits);
        verifySplit(fruitArray);
    }
}
```

**使用`StringUtils`类的`split()`实用方法的一个优点是调用者不必显式地执行空检查**。这是因为`split()`方法很好地处理了这个问题。让我们继续来看看它的实际应用:

```java
@Test
public void givenNullString_whenSplitUsingStringUtils_shouldReturnNull() {
    String fruits = null;
    String[] fruitArray = StringUtils.split(fruits);
    assertNull(fruitArray);
}
```

正如所料，该方法为`null`输入返回一个`null`值。

## 6.结论

在本教程中，**我们学习了用空格分割字符串的多种方法**。此外，我们还注意到与一些战略相关的优势和建议的最佳做法。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221220042827/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)