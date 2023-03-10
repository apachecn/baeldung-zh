# 在 Java 中将字符串转换为字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-string-to-char>

## 1.概观

[`String`](/web/20221031080025/https://www.baeldung.com/java-string) 是常见类型，`char`是 Java 中的[原语](/web/20221031080025/https://www.baeldung.com/java-primitives)。

在本教程中，我们将探索如何在 Java 中将一个`String`对象转换成`char`。

## 2.问题简介

我们知道**一个`char`只能包含一个单字符**。然而，一个`String`对象可以包含多个字符。

因此，我们的教程将涵盖两种情况:

*   源是一个单字符字符串。
*   源是多字符字符串。

对于第一种情况，我们可以很容易地得到单个字符作为`char`。例如，假设这是我们的输入:

```java
String STRING_b = "b";
```

转换后，我们希望有一个字符“`b`”。

对于第二种情况，如果源`String`是一个多字符的字符串，我们仍然希望得到一个单个的`char`作为结果，我们必须分析选择所需字符的需求，比如第一个、最后一个或第 n 个字符。

在本教程中，我们将提出一个更通用的解决方案。**我们将把源`String`转换成保存字符串中每个字符的`char`数组**。这样，我们可以根据需求选择任何元素。

我们将使用`STRING_Baeldung`作为输入示例:

```java
String STRING_Baeldung = "Baeldung";
```

接下来，让我们看看他们的行动。

## 3.单字符串

**Java 的`String`类提供了 [`charAt()`](/web/20221031080025/https://www.baeldung.com/string/char-at) 从输入字符串中获取第 n 个字符(从 0 开始)作为** `**char**.`因此，我们可以直接调用方法`getChar(0)`将单个字符`String`转换为`char`:

```java
assertEquals('b', STRING_b.charAt(0));
```

但是，我们应该注意到，**如果输入的是空字符串，`charAt()`方法调用抛出`StringIndexOutOfBoundsException`** :

```java
assertThrows(StringIndexOutOfBoundsException.class, () -> "".charAt(0));
```

因此，在调用`charAt()`方法之前，我们应该检查输入字符串是否为空。

## 4.多字符字符串

我们已经学会了使用`charAt(0)`将单个字符`String`转换成`char`。如果输入是一个多字符`String`，并且我们确切地知道我们想要将哪个字符转换成`char`，我们仍然可以使用`charAt()`方法。例如，我们可以从输入字符串“`Baeldung`”中获得第四个字符(“`l`”):

```java
assertEquals('l', STRING_Baeldung.charAt(3));
```

此外，我们可以使用`String.toCharArray()`获得一个包含所有字符的`char[]`数组:

```java
assertArrayEquals(new char[] { 'B', 'a', 'e', 'l', 'd', 'u', 'n', 'g' }, STRING_Baeldung.toCharArray());
```

值得一提的是**`toCharArray()`方法也适用于空字符串输入**。它返回一个空的`char`数组作为结果:

```java
assertArrayEquals(new char[] {}, "".toCharArray());
```

除了`toCharArray()`， **[`String.getChars()`](https://web.archive.org/web/20221031080025/https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#getChars-int-int-char:A-int-) 可以从给定的`String`中提取连续的字符到一个`char`数组**。该方法接收四个参数:

*   `srcBegin`–要获取的字符串中第一个字符的索引，包括
*   `srcEnd`–要复制的字符串中最后一个字符的索引，不包括
*   `dst`–目标数组，这是我们的结果
*   `dstBegin`–目标数组中的起始偏移量。我们用一个例子来讨论这个问题。

首先，我们从字符串“`Baeldung`”中提取“`aeld`”，并将其填充到一个预定义的`char`数组中:

```java
char[] aeld = new char[4];
STRING_Baeldung.getChars(1, 5, aeld, 0);
assertArrayEquals(new char[] { 'a', 'e', 'l', 'd' }, aeld);
```

正如上面的测试所示，要调用`getChars()`，我们首先应该有一个`char`数组来保存结果。

在这个例子中，当我们调用`getChars()`时，我们为`dstBegin`传递`0`。这是因为我们希望转换后的结果从数组`aeld`中的第一个元素开始。

当然，有时候，**我们希望结果覆盖数组的中间部分。然后我们可以将`dstBegin`设置为期望值**。

接下来，让我们看另一个例子，将"`aeld`"转换为 chars 并从第二个(index=1)元素开始覆盖目标数组:

```java
char[] anotherArray = new char[] { '#', '#', '#', '#', '#', '#' };
STRING_Baeldung.getChars(1, 5, anotherArray, 1);
assertArrayEquals(new char[] { '#', 'a', 'e', 'l', 'd', '#' }, anotherArray);
```

因此，正如我们所看到的，我们将`dstBegin=1`传递给该方法，并获得预期的结果。

## 5.结论

在本文中，我们学习了如何在 Java 中将一个`String`转换成一个`char`。

和往常一样，本文中使用的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221031080025/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)