# Java 8 无符号算术支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unsigned-arithmetic>

## 1。概述

从 Java 诞生开始，所有数值数据类型都是有符号的。然而，在许多情况下，需要使用无符号值。例如，如果我们计算一个事件发生的次数，我们不希望遇到负值。

从版本 8 开始，对无符号算法的支持终于成为了 JDK 的一部分。**这种支持以无符号整数 API 的形式出现，主要包含`Integer`和`Long`类中的静态方法。**

在本教程中，我们将浏览这个 API，并给出如何正确使用无符号数的说明。

## 2。位级表示法

为了理解如何处理有符号和无符号数，让我们先来看看它们在比特级别的表示。

在 Java 中，数字使用二进制补码 T2 系统进行编码。无论操作数是有符号的还是无符号的，这种编码都以相同的方式实现了许多基本的算术运算，包括加法、减法和乘法。

有了代码示例，事情应该更清楚了。为了简单起见，我们将使用`byte`原始数据类型的变量。其他整数类型的操作类似，如`short`、`int`或`long`。

假设我们有一个值为`100`的类型`byte`。这个数字有二进制表示`0110_0100`。

让我们将这个值加倍:

```
byte b1 = 100;
byte b2 = (byte) (b1 << 1);
```

给定代码中的左移运算符将变量`b1`中的所有位向左移动一个位置，从技术上讲，使其值增加一倍。变量`b2`的二进制表示将会是`1100_1000`。

在无符号类型系统中，该值表示一个十进制数，相当于`2^7 + 2^6 + 2^3`或`200`。然而，**在有符号系统中，最左边的位作为符号位。**因此，结果是` -2^7 + 2^6 + 2^3`，或`-56`。

快速测试可以验证结果:

```
assertEquals(-56, b2);
```

我们可以看到，有符号数和无符号数的计算是相同的。只有当 JVM 将二进制表示解释为十进制数时，差异才会出现。

加法、减法和乘法运算可以处理无符号数，而不需要改变 JDK。其他操作，如比较或除法，以不同的方式处理有符号和无符号数字。

这就是无符号整数 API 发挥作用的地方。

## 3。无符号整数 API

无符号整数 API 在 Java 8 中提供了对无符号整数运算的支持。这个 API 的大多数成员都是`Integer`和`Long`类中的静态方法。

这些类中的方法工作方式类似。因此，我们将只关注`Integer`类，为了简洁起见，我们将忽略`Long`类。

### 3.1。比较

`Integer`类定义了一个名为`compareUnsigned`的方法来比较无符号数。**这种方法认为所有的二进制值都是无符号的，忽略了符号位的概念。**

让我们从位于`int`数据类型边界的两个数字开始:

```
int positive = Integer.MAX_VALUE;
int negative = Integer.MIN_VALUE;
```

如果我们将这些数字作为有符号值进行比较，`positive`显然大于`negative`:

```
int signedComparison = Integer.compare(positive, negative);
assertEquals(1, signedComparison);
```

将数字作为无符号值进行比较时，最左边的位被视为最高有效位，而不是符号位。因此，结果是不同的，`positive`小于`negative`:

```
int unsignedComparison = Integer.compareUnsigned(positive, negative);
assertEquals(-1, unsignedComparison);
```

如果我们看一下这些数字的二进制表示，应该会更清楚:

*   `MAX_VALUE`->-`0111_1111_…_1111`
*   `MIN_VALUE`->-`1000_0000_…_0000`

当最左边的位是常规值位时，`MIN_VALUE`比二进制中的`MAX_VALUE`大一个单位。该测试证实:

```
assertEquals(negative, positive + 1);
```

### 3.2。除法和模运算

就像比较运算一样，**无符号除法和模运算将所有位作为值位处理。**因此，当我们对有符号和无符号数执行这些运算时，商和余数是不同的:

```
int positive = Integer.MAX_VALUE;
int negative = Integer.MIN_VALUE;

assertEquals(-1, negative / positive);
assertEquals(1, Integer.divideUnsigned(negative, positive));

assertEquals(-1, negative % positive);
assertEquals(1, Integer.remainderUnsigned(negative, positive));
```

### 3.3。解析

当使用`parseUnsignedInt`方法解析`String`时，**文本参数可以表示一个大于`MAX_VALUE`的数字。**

像这样大的值不能用`parseInt`方法解析，它只能处理从`MIN_VALUE`到`MAX_VALUE`的数字的文本表示。

以下测试用例验证解析结果:

```
Throwable thrown = catchThrowable(() -> Integer.parseInt("2147483648"));
assertThat(thrown).isInstanceOf(NumberFormatException.class);

assertEquals(Integer.MAX_VALUE + 1, Integer.parseUnsignedInt("2147483648"));
```

请注意，`parseUnsignedInt`方法可以解析表示大于`MAX_VALUE`的数字的字符串，但是无法解析任何负的表示。

### 3.4。格式化

与解析类似，格式化数字时，无符号运算将所有位视为值位。因此，**我们可以产生一个大约是`MAX_VALUE`两倍大的数字的文本表示。**

以下测试用例在两种情况下确认了`MIN_VALUE`的格式化结果——有符号和无符号:

```
String signedString = Integer.toString(Integer.MIN_VALUE);
assertEquals("-2147483648", signedString);

String unsignedString = Integer.toUnsignedString(Integer.MIN_VALUE);
assertEquals("2147483648", unsignedString);
```

## 4。利弊

许多开发人员，尤其是那些来自支持无符号数据类型的语言(如 C)的开发人员，欢迎引入无符号算术运算。然而，这未必是一件好事。

对无符号数的需求主要有两个原因。

首先，在有些情况下，负值永远不会出现，使用无符号类型可以首先防止出现这样的值。第二，对于无符号类型，我们可以**将可用正值的范围**扩大到有符号类型的两倍。

让我们分析一下无符号数的吸引力背后的基本原理。

**当一个变量应该总是非负的时候，一个小于`0`的值可以方便地指示一个异常情况。**

例如，`String.indexOf`方法返回字符串中某个字符第一次出现的位置。索引-1 可以很容易地表示这种字符的缺失。

无符号数的另一个原因是值空间的扩展。然而，如果一个有符号类型的范围不够，那么一个双倍的范围就不太可能足够了。

如果一种数据类型不够大，我们需要使用另一种支持更大值的数据类型，比如用`long`代替`int`，或者用`BigInteger`代替`long`。

无符号整数 API 的另一个问题是，一个数字的二进制形式是相同的，不管它是有符号的还是无符号的。因此**很容易混淆有符号和无符号值，这可能会导致意想不到的结果**。

## 5。结论

Java 对无符号算法的支持是应许多人的要求而来的。然而，它带来的好处还不清楚。我们在使用这一新功能时应该小心谨慎，以避免意想不到的结果。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220712100819/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)