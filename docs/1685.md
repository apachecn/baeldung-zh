# Java 中的有损转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lossy-conversion>

## 1.概观

在这个快速教程中，我们将讨论 Java 中有损转换的概念及其背后的原因。

同时，我们将探索一些简便的转换技术来避免这种错误。

## 2.有损转换

有损转换就是在处理数据时丢失信息。 ****

在 Java 中，它对应于在将从一种类型转换为另一种类型时**丢失变量的值或精度的可能性。**

当我们试图将一个**大尺寸类型的变量赋给一个更小尺寸类型的**时，Java 会在编译代码时产生一个错误`incompatible types: possible lossy conversion**,**` 。

例如，让我们尝试将一个`long`分配给一个`int`:

```
long longNum = 10;
int intNum = longNum;
```

编译这段代码时，Java 将发出一个错误:

```
incompatible types: possible lossy conversion from long to int
```

这里，Java 会发现`long`和`int`不兼容，导致有损转换错误。因为可以有在`int`范围之外的`long`值-2，147，483，648 到 2，147，483，647。

类似地，让我们尝试将一个`float`分配给一个 `long`:

```
float floatNum = 10.12f;
long longNum = floatNum;
```

```
incompatible types: possible lossy conversion from float to long
```

因为`float`可以有没有相应`long`值的十进制值。因此，我们会收到同样的错误。

类似地，给`int` 分配一个`double`号也会导致同样的错误:

```
double doubleNum = 1.2;
int intNum = doubleNum;
```

```
incompatible types: possible lossy conversion from double to int
```

`double`值对于`int`可能太大或太小，十进制值将在转换中丢失。因此，这是一个潜在的有损转换。

此外，在执行简单计算时，我们可能会遇到以下错误:

```
int fahrenheit = 100;
int celcius = (fahrenheit - 32) * 5.0 / 9.0;
```

当一个`double`乘以一个`int`时，我们在一个`double`中得到结果。因此，这也是一个潜在的有损转换。

因此，**有损转换中不兼容的类型既可以有不同的大小，也可以有不同的类型**(整数或小数)。【T2

## 3.原始数据类型

在 Java 中，有许多[原始数据类型](/web/20221129012658/https://www.baeldung.com/java-primitives)和它们对应的[包装类](/web/20221129012658/https://www.baeldung.com/java-wrapper-classes)可用。

接下来，让我们汇编一份 Java 中所有可能的有损转换的列表:

*   `short`到`byte`或`char`
*   `char`到`byte`或`short`
*   `int`至`byte`、`short`或`char`
*   `long`到`byte`、`short`、`char`或`int`
*   `float`至`byte`、`short`、`char`、`int`或`long`
*   `double`至`byte`、`short`、`char`、`int`、`long`或`float`

注意，尽管`short`和`char`具有相同的尺寸。然而，**从`short`到`char`的转换是有损耗的，因为`char`是无符号数据类型**。

## 4.转换技术

### 4.1.在原始类型之间转换

转换原语以避免有损转换的简单方法是通过向下转换；换句话说，将较大尺寸的类型铸造成较小尺寸的类型。因此，它也被称为收缩原语转换。

例如，让我们使用向下转换`:`将`a long` 数字转换为 `short`

```
long longNum = 24;
short shortNum = (short) longNum;
assertEquals(24, shortNum);
```

同样，让我们将一个`double`转换成一个`int`:

```
double doubleNum = 15.6;
int integerNum = (int) doubleNum;
assertEquals(15, integerNum);
```

但是，我们应该注意，通过向下转换将值过大或过小的大型类型转换为较小的类型可能会导致意外的值。

让我们转换`short`范围之外的`long`值:

```
long largeLongNum = 32768; 
short minShortNum = (short) largeLongNum;
assertEquals(-32768, minShortNum);

long smallLongNum = -32769;
short maxShortNum = (short) smallLongNum;
assertEquals(32767, maxShortNum);
```

如果我们仔细分析转换，我们会发现这些不是预期值。

换句话说，当 Java 在从大尺寸类型转换时达到小尺寸类型的最高值时，**下一个数字是小尺寸类型的最低值**，反之亦然。

让我们通过例子来理解这个。当值为 32768 的`largeLongNum` 转换为`short`时，`shortNum1`的值为-32768 `.`，因为`short`的最大值为 32767，所以 Java 会取下一个`short.`的最小值

同理，当`smallLongNum`转换为`short`时。`shortNum2`的值是 32767，因为 Java 会取`short`的下一个最大值。

同样，让我们看看当我们将一个`long`的最大值和最小值转换成一个`int`时会发生什么:

```
long maxLong = Long.MAX_VALUE; 
int minInt = (int) maxLong;
assertEquals(-1, minInt);

long minLong = Long.MIN_VALUE;
int maxInt = (int) minLong;
assertEquals(0, maxInt);
```

### 4.2.在包装对象和基本类型之间转换

为了直接将包装对象转换成原语，我们可以在包装类中使用各种方法，比如`intValue()`、`shortValue()`和`longValue()`。这叫`unboxing`。

例如，让我们将一个`Float`对象转换成一个`long`:

```
Float floatNum = 17.564f;
long longNum = floatNum.longValue();
assertEquals(17, longNum);
```

同样，如果我们看看`longValue` 或类似方法的实现，我们会发现收缩原语转换的使用:

```
public long longValue() {
    return (long) value;
}
```

但是，有时应该避免收缩原语转换，以保存有价值的信息:

```
Double doubleNum = 15.9999;
long longNum = doubleNum.longValue();
assertEquals(15, longNum); 
```

转换后，`longNum` 的值将为 15。但是`doubleNum`是 15.9999，非常接近 16。

相反，我们可以使用`Math.round()`转换成最接近的整数:

```
Double doubleNum = 15.9999;
long longNum = Math.round(doubleNum);

assertEquals(16, longNum);
```

### 4.3.在包装对象之间转换

为此，让我们使用已经讨论过的转换技术。

首先，我们将把**包装器对象转换成一个原始值，向下转换并转换成另一个包装器对象**。换句话说，我们将执行取消装箱、向下转换和装箱技术。

例如，让我们将一个`Double`对象转换成一个`Integer` 对象:

```
Double doubleNum = 10.3;
double dbl = doubleNum.doubleValue(); // unboxing
int intgr = (int) dbl; // downcasting
Integer intNum = Integer.valueOf(intgr);
assertEquals(Integer.valueOf(10), intNum); 
```

最后，我们使用`Integer`。`valueOf()`将原始类型`int`转换成一个`Integer`对象。这种类型的转换被称为`boxing`。

## 5.结论

在本文中，我们借助一些例子探讨了 Java 中有损转换的概念。此外，我们还编制了一份所有可能的有损转换列表。

在这一过程中，我们已经认识到收缩原语转换是一种转换原语数字并避免有损转换错误的简单技术。

同时，我们还探索了在 Java 中进行数值转换的其他简便技术。

本文的代码实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221129012658/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-2)