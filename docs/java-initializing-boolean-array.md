# 在 Java 中初始化布尔数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-initializing-boolean-array>

## 1.概观

Boolean 是 Java 中的一种基本数据类型。通常，它只能有两个值，`true`或`false`。

在本教程中，我们将讨论如何初始化一个布尔值数组。

## 2.问题简介

这个问题很简单。简单地说，我们希望用相同的默认值初始化一个布尔变量数组。

然而， **Java 有两种“不同”的布尔类型，[原语](/web/20221208143917/https://www.baeldung.com/java-primitives-vs-objects) `boolean`和[装箱](/web/20221208143917/https://www.baeldung.com/java-wrapper-classes) `Boolean`。因此，在本教程中，我们将涵盖这两种情况，并说明如何初始化一个由`boolean`和`Boolean`组成的数组。**

此外，为了简单起见，我们将使用单元测试断言来验证我们的数组初始化是否按预期工作。

那么接下来，让我们从原语`boolean`类型开始。

## 3.初始化原始`boolean`数组

在 Java 中，一个原始变量有一个默认值。例如，一个原语`int`变量的默认值是 0，一个原语`boolean`变量将默认保存`false`。

因此，**如果我们想用所有的`false`初始化一个`boolean`数组，我们可以简单地创建数组而不用设置值。**

接下来，让我们创建一个测试来验证它:

```java
boolean[] expected = { false, false, false, false, false };
boolean[] myArray = new boolean[5];
assertArrayEquals(expected, myArray);
```

如果我们运行上面的测试，它就通过了。正如我们所见，`boolean[] myArray = new boolean[5];`初始化了`boolean`数组中的五个`false`元素。

值得一提的是，当我们希望[根据两个数组](/web/20221208143917/https://www.baeldung.com/java-comparing-arrays)、**的值来比较它们时，我们应该使用`assertArrayEquals()`方法，而不是`assertEquals()`、**。这是因为数组的`equals()`方法检查两个数组的引用是否相同。

因此，初始化原语数组`false`很容易，因为我们可以利用`boolean`的默认值。然而，有时候，我们可能想要创建一个`true`值的数组。如果是这种情况，我们必须以某种方式将值设置为`true`。当然，我们可以单独设置它们，也可以在一个循环中设置它们。但是 [`Arrays.fill()`](/web/20221208143917/https://www.baeldung.com/java-initialize-array#using-arraysfill) 的方法让工作变得更容易:

```java
boolean[] expected = { true, true, true, true, true };
boolean[] myArray = new boolean[5];
Arrays.fill(myArray, true);
assertArrayEquals(expected, myArray);
```

同样，如果我们试一试，测试就会通过。所以，我们有一个数组`true`。

## 4.初始化装箱的`Boolean`数组

到目前为止，我们已经学习了如何用`true`或`false.`初始化原语`boolean `的数组。现在，让我们看看装箱`Boolean`端的相同操作。

首先，与原始的`boolean`变量不同，装箱的`Boolean`变量没有默认值。如果我们不给它设定一个值，它的值就是`null`。因此，**如果我们创建一个没有设置元素值的`Boolean `数组，所有元素都是`null`。**让我们创建一个测试来验证它:

```java
Boolean[] expectedAllNull = { null, null, null, null, null };
Boolean[] myNullArray = new Boolean[5];
assertArrayEquals(expectedAllNull, myNullArray);
```

如果我们想用`true`或`false`初始化`Boolean`数组，我们仍然可以使用`Arrays.fill()`方法:

```java
Boolean[] expectedAllFalse = { false, false, false, false, false };
Boolean[] myFalseArray = new Boolean[5];
Arrays.fill(myFalseArray, false);
assertArrayEquals(expectedAllFalse, myFalseArray);

Boolean[] expectedAllTrue = { true, true, true, true, true };
Boolean[] myTrueArray = new Boolean[5];
Arrays.fill(myTrueArray, true);
assertArrayEquals(expectedAllTrue, myTrueArray);
```

当我们运行上面的测试时，它通过了。

## 5.结论

在这篇短文中，我们学习了如何在 Java 中初始化一个`boolean`或`Boolean`数组。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-basic)