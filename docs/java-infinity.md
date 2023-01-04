# Java 中的无限

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-infinity>

## 1.概观

在本教程中，我们将看看 Java 中的无限概念，以及如何使用它。

## 2.Java 中的数字简介

在数学中，我们有一组实数和一组整数。显然，这两个集合都是无限的，并且都包含正负无穷大。

在计算机世界中，我们需要一个内存位置来存储这些集合的值，这个位置必须是有限的，因为计算机的内存是有限的。

**对于 Java 中的`int`类型，不涵盖无穷大的概念。我们只能存储适合我们选择的内存位置的整数。**

**对于实数，我们也有无穷大的概念，可以是正数，也可以是负数。**32 位`float`类型和 64 位`double`类型在 Java 中都支持这一点。接下来，我们将使用`double`类型作为示例，因为它也是 Java 中最常用的实数类型，因为它具有更好的精度。

## 3.正无穷大

常数`Double.POSITIVE_INFINITY`保存类型`double`的正无穷大值。这个值是通过将`1.0`除以`0.0`得到的。它的`String`代表是`Infinity`。这个值是约定，它的十六进制表示是`7FF0000000000000`。每个具有这个位值的`double`变量都包含正无穷大。

## 4.负无穷大

常数`Double.NEGATIVE_INFINITY`保存类型`double`的负无穷大值。该值通过将`-1.0`除以`0.0\.` 得到，其`String`表示为`-Infinity`。这个值也是约定俗成，十六进制表示是`FFF0000000000000`。每个具有这个位值的`double`变量都包含负无穷大。

## 5.无限运算

让我们声明一个名为`positiveInfinity`的`double`变量，并给它赋值`Double.POSITIVE_INFINITY`和另一个`double`变量`negativeInfinity`，并给它赋值`Double.NEGATIVE_INFINITY`。然后，我们得到以下运算结果:

```java
Double positiveInfinity = Double.POSITIVE_INFINITY;
Double negativeInfinity = Double.NEGATIVE_INFINITY;

assertEquals(Double.NaN, (positiveInfinity + negativeInfinity));
assertEquals(Double.NaN, (positiveInfinity / negativeInfinity));
assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity - negativeInfinity));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity - positiveInfinity));
assertEquals(Double.NEGATIVE_INFINITY, (positiveInfinity * negativeInfinity)); 
```

这里，常数`Double.NaN`表示不是数字的结果。

让我们来看看无穷大和正数的数学运算:

```java
Double positiveInfinity = Double.POSITIVE_INFINITY;
Double negativeInfinity = Double.NEGATIVE_INFINITY;
double positiveNumber = 10.0; 

assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity + positiveNumber));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity + positiveNumber));

assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity - positiveNumber));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity - positiveNumber));

assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity * positiveNumber));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity * positiveNumber));

assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity / positiveNumber));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity / positiveNumber));

assertEquals(Double.NEGATIVE_INFINITY, (positiveNumber - positiveInfinity));
assertEquals(Double.POSITIVE_INFINITY, (positiveNumber - negativeInfinity));

assertEquals(0.0, (positiveNumber / positiveInfinity));
assertEquals(-0.0, (positiveNumber / negativeInfinity)); 
```

现在，让我们来看看无穷大和负数的数学运算:

```java
Double positiveInfinity = Double.POSITIVE_INFINITY;
Double negativeInfinity = Double.NEGATIVE_INFINITY;
double negativeNumber = -10.0; 

assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity + negativeNumber));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity + negativeNumber));

assertEquals(Double.POSITIVE_INFINITY, (positiveInfinity - negativeNumber));
assertEquals(Double.NEGATIVE_INFINITY, (negativeInfinity - negativeNumber));

assertEquals(Double.NEGATIVE_INFINITY, (positiveInfinity * negativeNumber));
assertEquals(Double.POSITIVE_INFINITY, (negativeInfinity * negativeNumber));

assertEquals(Double.NEGATIVE_INFINITY, (positiveInfinity / negativeNumber));
assertEquals(Double.POSITIVE_INFINITY, (negativeInfinity / negativeNumber));

assertEquals(Double.NEGATIVE_INFINITY, (negativeNumber - positiveInfinity));
assertEquals(Double.POSITIVE_INFINITY, (negativeNumber - negativeInfinity));

assertEquals(-0.0, (negativeNumber / positiveInfinity));
assertEquals(0.0, (negativeNumber / negativeInfinity)); 
```

有一些经验法则可以更好地记住这些操作:

*   分别用`Infinity`和`-Infinity,`代替正负不定式，先进行符号运算
*   对于非零和无穷大之间的任何运算，你将得到无穷大的结果
*   当我们将正负无穷大相加或相除时，我们得到`not a number`的结果

## 6.被零除

被零除是除法的一种特殊情况，因为它产生负的和正的无穷大值。

举例来说，让我们取一个`double`值`d`,并用零检查以下除法的结果:

```java
double d = 1.0;

assertEquals(Double.POSITIVE_INFINITY, (d / 0.0));
assertEquals(Double.NEGATIVE_INFINITY, (d / -0.0));
assertEquals(Double.NEGATIVE_INFINITY, (-d / 0.0));
assertEquals(Double.POSITIVE_INFINITY, (-d / -0.0)); 
```

## 7.结论

在本文中，我们探讨了 Java 中正负无穷大的概念和用法。实现和代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221025070137/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5 "Implementing Infinity in Java")