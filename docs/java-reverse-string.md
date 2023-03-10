# 如何在 Java 中反转一个字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reverse-string>

## 1。概述

在这个快速教程中，我们将看到**如何在 Java 中反转`String`。**

我们将开始使用普通的 Java 解决方案来进行这个处理。接下来，我们将看看像 Apache Commons 这样的第三方库提供的选项。

此外，我们将演示**如何颠倒句子**中单词的顺序。

## 2。传统的`for`循环

我们知道字符串在 Java 中是[不可变的。不可变对象是这样一种对象，它的**内部状态在被完全创建后保持不变**。](/web/20220627175635/https://www.baeldung.com/java-immutable-object)

因此，我们不能通过修改来反转一个`String`。为此，我们需要创建另一个`String`。

首先，让我们看一个使用`for`循环的基本例子。我们将从最后一个元素到第一个元素迭代`String`输入，并将每个字符连接成一个新的`String`:

```java
public String reverse(String input) {

    if (input == null) {
        return input;
    }

    String output = "";

    for (int i = input.length() - 1; i >= 0; i--) {
        output = output + input.charAt(i);
    }

    return output;
}
```

正如我们所看到的，我们需要小心拐角处的情况，并分别对待它们。

为了更好地理解这个例子，我们可以构建一个单元测试:

```java
@Test
public void whenReverseIsCalled_ThenCorrectStringIsReturned() {
    String reversed = ReverseStringExamples.reverse("cat");
    String reversedNull = ReverseStringExamples.reverse(null);
    String reversedEmpty = ReverseStringExamples.reverse(StringUtils.EMPTY);

    assertEquals("tac", reversed);
    assertEquals(null, reversedNull);
    assertEquals(StringUtils.EMPTY, reversedEmpty);
}
```

## 3。`StringBuilder`

Java 也提供了一些类似于 **`StringBuilder`和`StringBuffer`的机制来创建一个[的可变字符序列](/web/20220627175635/https://www.baeldung.com/java-string-builder-string-buffer)** 。这些对象有一个`reverse()`方法，可以帮助我们获得想要的结果。

这里，我们需要从`String`输入创建一个`StringBuilder`，然后调用`reverse()`方法:

```java
public String reverseUsingStringBuilder(String input) {
    if (input == null) {
        return null;
    }

    StringBuilder output = new StringBuilder(input).reverse();
    return output.toString();
}
```

## 4 .Apache common〔t1〕

Apache Commons 是一个流行的 Java 库，有很多实用类，包括字符串操作。

像往常一样，要开始使用 Apache Commons，我们首先需要添加 [Maven 依赖项](https://web.archive.org/web/20220627175635/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav):

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

这里我们需要的是`StringUtils`类，因为它提供了类似于`StringBuilder`的`reverse()`方法。

使用这个库的一个优点是**它的实用方法执行`null`安全操作**。所以，我们不必分别处理边缘情况。

让我们创建一个方法来实现我们的目的，并使用`StringUtils`类:

```java
public String reverseUsingApacheCommons(String input) {
    return StringUtils.reverse(input);
}
```

现在，看看这三种方法，我们可以肯定地说，第三种方法是最简单、最不容易出错的方法来反转一个`String`。

## 5。颠倒句子中单词的顺序

现在，让我们假设我们有一个由空格分隔单词并且没有标点符号的句子。我们需要颠倒这个句子中单词的顺序。

**我们可以分两步解决这个问题:[用空格分隔符把句子](/web/20220627175635/https://www.baeldung.com/java-split-string)拆分，然后把单词逆序串联起来。**

首先，我们将展示一个经典的方法。为了完成问题的第一部分，我们将使用 S `tring.split()` 方法。接下来，我们将通过结果数组向后迭代，并使用`StringBuilder`连接单词。当然，我们还需要在这些词之间加一个空格:

```java
public String reverseTheOrderOfWords(String sentence) {
    if (sentence == null) {
        return null;
    }

    StringBuilder output = new StringBuilder();
    String[] words = sentence.split(" ");

    for (int i = words.length - 1; i >= 0; i--) {
        output.append(words[i]);
        output.append(" ");
    }

    return output.toString().trim();
}
```

其次，我们将考虑使用 Apache Commons 库。**再一次，它帮助我们实现一个可读性更好、更少出错的代码。**我们只需要调用`StringUtils.reverseDelimited()`方法，将输入的句子和分隔符作为参数:

```java
public String reverseTheOrderOfWordsUsingApacheCommons(String sentence) {
    return StringUtils.reverseDelimited(sentence, ' ');
}
```

## 6。结论

在本教程中，我们首先看了 Java 中反转`String`的不同方式。我们讨论了一些使用核心 Java 的例子，以及使用流行的第三方库，如 Apache Commons。

接下来，我们看到了如何通过两步来颠倒句子中单词的顺序。这些步骤也有助于实现句子的其他排列。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220627175635/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)