# 在 Java 中将字符串转换为双精度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-double>

## 1。概述

在本教程中，我们将介绍许多在 Java 中将`String`转换成`double`的方法。

## 2。`Double.parseDouble`

我们可以使用`Double.` `parseDouble`方法将`String`转换为`double`:

```java
assertEquals(1.23, Double.parseDouble("1.23"), 0.000001);
```

## 3。`Double.valueOf`

同样，我们可以使用`Double.valueOf`方法将`String`转换成[装箱的`Double`](/web/20221206083201/https://www.baeldung.com/java-generics#generics-primitive-data-types) :

```java
assertEquals(1.23, Double.valueOf("1.23"), 0.000001);
```

注意，`Double.valueOf`的返回值是一个装箱的`Double`。从 Java 5 开始，这个装箱的`Double`在需要的地方被编译器转换成原语`double`。

一般来说，**我们应该倾向于`Double.parseDouble`** ，因为它不需要编译器执行自动拆箱。

## 4。`DecimalFormat.parse`

当代表`double`的`String`具有更复杂的格式时，我们可以使用`DecimalFormat`。

例如，我们可以在不删除非数字符号的情况下转换基于十进制的货币值:

```java
DecimalFormat format = new DecimalFormat("\u00A4#,##0.00");
format.setParseBigDecimal(true);

BigDecimal decimal = (BigDecimal) format.parse("-$1,000.57");

assertEquals(-1000.57, decimal.doubleValue(), 0.000001);
```

类似于`Double.valueOf`,**`DecimalFormat.parse`方法返回一个`Number`** ，我们可以使用`doubleValue`方法将它转换成一个原语 `double`。此外，我们使用`setParseBigDecimal`方法来强制`DecimalFormat.parse`返回一个`BigDecimal`。

通常情况下，`DecimalFormat`比我们要求的更先进，因此，我们应该选择`Double.parseDouble`或`Double.valueOf`。

要了解更多关于`DecimalFormat`的信息，请查看[`DecimalFormat`](/web/20221206083201/https://www.baeldung.com/java-decimalformat)实用指南。

## 5。无效转换

Java 为处理无效的数字`String`提供了统一的接口。

值得注意的是， **`Double.parseDouble`，`Double.valueOf`，`DecimalFormat.parse`在我们经过`null.` 时抛出一个`NullPointerException`**

同样， **`Double.parseDouble` 和`Double.valueOf`在我们传递一个无法解析的无效字符串给一个`double`(比如`&`)时抛出一个`NumberFormatException`** 。

最后， **`DecimalFormat.parse`在我们经过一个无效的`String.`时抛出一个`ParseException`**

## 6。避免贬值转换

在 Java 9 之前，我们可以通过实例化一个`Double`从一个`String`创建一个装箱的 *Double* :

```java
new Double("1.23");
```

从版本 9 开始，Java 正式否决了这种方法。

## 7 .**。结论**

总之，Java 为我们提供了多种将`String`转换成`double`值的方法。

一般来说，我们建议使用`Double.parseDouble`，除非需要一个装箱的`Double`。

本文的源代码，包括示例，可以在 GitHub 的[中找到。](https://web.archive.org/web/20221206083201/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)