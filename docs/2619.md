# Java 中的字符串等于()与内容等于()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-equals-vs-contentequals>

## 1.介绍

Java 中的 [`String`类](/web/20221208143814/https://www.baeldung.com/java-string)的`equals()`和 `contentEquals()`方法用于执行`String`比较。然而，这两种方法的功能之间存在特定的差异。

在本教程中，我们将通过实际例子快速了解这两种方法之间的区别。

## 2.`equals()`法

[`equals()`](/web/20221208143814/https://www.baeldung.com/java-compare-strings#equals-string) 方法是 Java `String`类的`public`方法。它覆盖了来自`Object`类的最初的`equals()`方法。这种方法的特征是:

```
public boolean equals(Object anObject)
```

该方法通过检查两个不同的中的单个字符来比较两个不同的。**然而，该方法不仅检查内容，还检查对象是否是`String`的实例。**因此，如果所有这些条件都满足，该方法只返回`true`:

*   参数对象不是`null`
*   这是一个`String`物体
*   字符序列是相同的

## 3.`contentEquals()`法

与 `equals()`方法类似， [`contentEquals()`](https://web.archive.org/web/20221208143814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#contentEquals(java.lang.CharSequence)) 方法也用于比较`String's`内容。**然而，与`equals()`方法不同，`contentEquals()`将`CharSequence`接口的任何实现作为参数。**那就是说`String`、`StringBuffer`、`StringBuilder`、`CharBuffer,`或者`Segment`可以比较。

这种方法的特征是:

```
public boolean contentEquals(StringBuffer sb)
public boolean contentEquals(CharSequence cs)
```

**因此，`contentEquals()`方法只关心字符串**的内容。如果参数是一个`String`对象，则调用`equals()`方法进行比较。另一方面，如果提供了通用字符序列，则该方法比较相似位置的单个字符。

如果给定参数中的字符序列与原始的`String`匹配，则该方法返回`true`。与`equals()`方法不同，如果将一个`null`参数传递给`contentEquals()`方法，它会抛出一个`NullPointerException`。

## 4.例子

让我们通过编写简单的测试用例来看看这两种方法的实际应用。为了简单起见，我们在代码中使用“Baeldung”这个词。

首先，我们将选取两个相同的`String`对象并检查它们。在这种情况下，两种方法都将返回一个`true`值:

```
String actualString = "baeldung";
String identicalString = "baeldung";

assertTrue(actualString.equals(identicalString));
assertTrue(actualString.contentEquals(identicalString));
```

接下来，我们采用内容相同的`CharSequence`的两种不同实现。对于第一个实现，我们将用一个`String`实例化`CharSequence`。在这种情况下，两种方法都应该返回`true`，因为内容和类型是相同的:

```
CharSequence identicalStringInstance = "baeldung";

assertTrue(actualString.equals(identicalStringInstance));
assertTrue(actualString.contentEquals(identicalStringInstance));
```

对于下一个例子，我们将采用一个`StringBuffer`实现。由于`contentEquals()`方法只检查内容，它应该返回`true`。但是，`equals()`方法应该`false`:

```
CharSequence identicalStringBufferInstance = new StringBuffer("baeldung");

assertFalse(actualString.equals(identicalStringBufferInstance));
assertTrue(actualString.contentEquals(identicalStringBufferInstance));
```

## 5.结论

在本文中，我们快速浏览了一下`String`类的两个方法。虽然`equals()`方法只比较`String`的实例，但是`contentEquals()`方法可以比较`CharSequence`的任何实现。

总之，当我们只关心对象的内容时，我们应该使用`contentEquals()`。另一方面，有时检查对象的类型可能很重要。在这种情况下，我们应该使用`equals()`方法，它给了我们更严格的检查条件。

和往常一样，代码片段可以在 GitHub 上[找到。](https://web.archive.org/web/20221208143814/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)