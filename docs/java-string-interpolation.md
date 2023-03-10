# Java 中的字符串插值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-interpolation>

## 1.概观

在本教程中，我们将讨论 Java 中的`String`插值。我们将看几个不同的例子，然后讨论一些细节。

## 2.`String`Java 中的插补

**`String`插值是一种将变量值注入字符串的直接而精确的方法`.`** 它允许用户将变量引用直接嵌入到处理过的字符串中。与像 [Scala](/web/20221130213649/https://www.baeldung.com/scala/string-interpolation) 这样的语言相比，Java 缺乏对`String`插值的本地支持。

然而，在 Java 中有一些方法可以实现这种行为。在接下来的部分中，我们将解释每一种方法。

## 3.加号运算符

首先，我们有[“+”操作符](/web/20221130213649/https://www.baeldung.com/java-strings-concatenation)。我们可以使用“+”操作符连接变量和字符串值。变量被它的值替换，所以我们实现了字符串的插值或连接。让我们来看一个代码示例:

```java
@Test
public void givenTwoString_thenInterpolateWithPlusSign() {
    String EXPECTED_STRING = "String Interpolation in Java with some Java examples.";
    String first = "Interpolation";
    String second = "Java";
    String result = "String " + first + " in " + second + " with some " + second + " examples.";
    assertEquals(EXPECTED_STRING, result);
}
```

正如我们在前面的例子中可以注意到的，使用这个操作符，得到的`String`包含了变量的值和其他字符串值。由于可以根据具体需要进行调整，这种字符串连接方法是最直接、最有价值的方法之一。使用运算符时，我们不需要将文本放在引号中。

## 4.`format()`功能

另一种方法是使用来自`String`类的`[format()](/web/20221130213649/https://www.baeldung.com/string/format) `方法。与“+”运算符相反，在这种情况下，我们需要使用占位符在`String`插值中获得预期的结果。下面  代码 片段 演示如何操作 :

```java
@Test
public void givenTwoString_thenInterpolateWithFormat() {
    String EXPECTED_STRING = "String Interpolation in Java with some Java examples.";
    String first = "Interpolation";
    String second = "Java";
    String result = String.format("String %s in %s with some %s examples.", first, second, second);
    assertEquals(EXPECTED_STRING, result);
}
```

此外，如果我们想避免在我们的`format`调用中变量的重复，我们可以引用一个特定的参数。让我们来看看这个行为:

```java
@Test
public void givenTwoString_thenInterpolateWithFormatStringReference() {
    String EXPECTED_STRING = "String Interpolation in Java with some Java examples.";
    String first = "Interpolation";
    String second = "Java";
    String result = String.format("String %1$s in %2$s with some %2$s examples.", first, second);
    assertEquals(EXPECTED_STRING, result);
}
```

现在，我们减少不必要的变量重复。相反，我们使用参数列表中参数的索引。

## 5.`StringBuilder`阶级

我们下面的做法是 [`StringBuilder`](/web/20221130213649/https://www.baeldung.com/java-string-builder-string-buffer) 类。我们实例化一个`StringBuilder`对象，然后调用`append() `函数来构建`String`。在这个过程中，我们的变量被添加到结果`String`中:

```java
@Test
public void givenTwoString_thenInterpolateWithStringBuilder() {
    String EXPECTED_STRING = "String Interpolation in Java with some Java examples.";
    String first = "Interpolation";
    String second = "Java";
    StringBuilder builder = new StringBuilder();
    builder.append("String ")
      .append(first)
      .append(" in ")
      .append(second)
      .append(" with some ")
      .append(second)
      .append(" examples.");
    String result = builder.toString();
    assertEquals(EXPECTED_STRING, result);
}
```

正如我们在上面的代码示例中可以注意到的，我们可以通过链接 append 函数来插入带有必要文本的字符串，该函数接受参数作为变量(在本例中是两个`Strings`)。

## 6.`MessageFormat`阶级

在 Java 中，使用 [`MessageFormat`](/web/20221130213649/https://www.baeldung.com/java-localization-messages-formatting) 类是一种鲜为人知的获得`String`插值的方法。有了`MessageFormat`，我们可以创建串联的消息，而不用担心底层语言。这是创建面向用户的消息的标准方法。它接受一个对象集合，格式化其中包含的字符串，并将它们插入到模式中的适当位置。

`MessageFormat`的`format`方法与`String`的`format`方法几乎相同，除了占位符是如何写的。像{0}、{1}、{2}等索引。，表示该函数中的占位符:

```java
@Test
public void givenTwoString_thenInterpolateWithMessageFormat() {
    String EXPECTED_STRING = "String Interpolation in Java with some Java examples.";
    String first = "Interpolation";
    String second = "Java";
    String result = MessageFormat.format("String {0} in {1} with some {1} examples.", first, second);
    assertEquals(EXPECTED_STRING, result);
}
```

关于性能，`StringBuilder `只将文本追加到动态缓冲区；然而，`MessageFormat` 在追加数据之前解析给定的格式。 因为这个，`StringBuilder`的效率胜过`MessageFormat`的效率。

## 7.Apache common(Apache 公共)

最后，我们有来自[阿帕奇社区](/web/20221130213649/https://www.baeldung.com/java-apache-commons-text)的`StringSubstitutor `。在这个类的上下文中，值代替了包含在字符串中的变量。这个类获取一段文本并替换所有变量。变量的默认定义是`${variableName}`。构造函数和 set 方法可以用来改变前缀和后缀。**变量值的解析通常涉及到地图的使用**。但是，我们可以通过利用系统属性或提供专门的变量解析器来解决它们:

```java
@Test
public void givenTwoString_thenInterpolateWithStringSubstitutor() {
    String EXPECTED_STRING = "String Interpolation in Java with some Java examples.";
    String baseString = "String ${first} in ${second} with some ${second} examples.";
    String first = "Interpolation";
    String second = "Java";
    Map<String, String> parameters = new HashMap<>();
    parameters.put("first", first);
    parameters.put("second", second);
    StringSubstitutor substitutor = new StringSubstitutor(parameters);
    String result = substitutor.replace(baseString);
    assertEquals(EXPECTED_STRING, result);
}
```

从我们的代码示例中，我们可以看到我们创建了一个`Map`。键名与将在`String`中替换的变量名相同。然后，我们将每个键的相应值放入`Map`。接下来，我们将它作为构造函数参数传递给`StringSubstitutor` 类。最后，实例化对象调用`replace()`函数。该函数接收带有占位符的文本作为参数。结果，我们得到一个插值文本。仅此而已，简单。

## 8.结论

总之，我们简要描述了什么是`String`插值。接下来，我们使用本地 Java 操作符，即来自`String `类的`format()`方法，用 Java 语言逐步实现这一点。此外，我们还探索了不太为人所知的选项，如来自`Apache Commons`的`MessageFormat`和`StringSubstitutor `。

像往常一样，代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221130213649/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)