# 在 Java 中将字符串转换为 BigDecimal

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-bigdecimal>

## 1。概述

在本教程中，我们将介绍在 Java 中将`String`转换成 [`BigDecimal`](/web/20220626081441/https://www.baeldung.com/java-bigdecimal-biginteger) 的许多方法。

## 2.`BigDecimal`

**`BigDecimal`表示不可变的任意精度有符号十进制数**。它由两部分组成:

*   未缩放的值–任意精度的整数
*   scale–一个 32 位整数，表示小数点右边的位数

例如，`BigDecimal` 3.14 的未缩放值为 314，缩放值为 2。

如果为零或正数，则小数位数是小数点右边的位数。

如果为负，则数字的未缩放值乘以 10 的小数位数的负幂。因此，**`BigDecimal`所代表的数字的值为(`未换算值×10**(-换算)`)。

Java 中的`BigDecimal`类提供了基本算术、小数位数操作、比较、格式转换和散列的操作。

此外，**我们使用`BigDecimal`进行高精度运算，需要控制规模的计算，以及舍入行为**。一个这样的例子是涉及金融交易的计算。

我们可以使用以下方法之一将 Java 中的`String`转换成`BigDecimal`:

*   `BigDecimal(String)`构造器
*   `BigDecimal.valueOf()`方法
*   `DecimalFormat.parse()`方法

下面我们逐一讨论。

## 3.`BigDecimal(String)`

在 Java 中将`String`转换为`BigDecimal`的最简单方法是使用`BigDecimal(String)`构造函数:

```java
BigDecimal bigDecimal = new BigDecimal("123");
assertEquals(new BigDecimal(123), bigDecimal);
```

## 4.`BigDecimal.valueOf()`

我们也可以通过使用`[BigDecimal.valueOf(double)](https://web.archive.org/web/20220626081441/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html#%3Cinit%3E(double))`方法将`String`转换为`BigDecimal`。

这是一个两步过程。第一步是将`String`转换为`Double`。第二步是将`Double`转换为`BigDecimal`:

```java
BigDecimal bigDecimal = BigDecimal.valueOf(Double.valueOf("123.42"));
assertEquals(new BigDecimal(123.42).setScale(2, BigDecimal.ROUND_HALF_UP), bigDecimal);
```

必须注意的是，有些浮点数不能用一个`Double`值来精确表示。这是因为类型为`Double`的浮点数在内存中的表示。事实上，这个数字是以一种尽可能接近输入的`Double`数的有理数的形式表示的。这样一来，一些浮点数就变成了[不准确的](/web/20220626081441/https://www.baeldung.com/cs/floating-point-numbers-inaccuracy)。

## 5.`DecimalFormat.parse()`

当代表一个值的`String`具有更复杂的格式时，我们可以使用`DecimalFormat`。

例如，我们可以在不删除非数字符号的情况下转换基于十进制的长值:

```java
BigDecimal bigDecimal = new BigDecimal(10692467440017.111).setScale(3, BigDecimal.ROUND_HALF_UP);

DecimalFormatSymbols symbols = new DecimalFormatSymbols();
symbols.setGroupingSeparator(',');
symbols.setDecimalSeparator('.');
String pattern = "#,##0.0#";
DecimalFormat decimalFormat = new DecimalFormat(pattern, symbols);
decimalFormat.setParseBigDecimal(true);

// parse the string value
BigDecimal parsedStringValue = (BigDecimal) decimalFormat.parse("10,692,467,440,017.111");

assertEquals(bigDecimal, parsedStringValue);
```

**`DecimalFormat.parse`方法返回一个`Number`** ，我们使用`setParseBigDecimal(true).`将它转换成一个`BigDecimal` 数

通常情况下， [`DecimalFormat`](/web/20220626081441/https://www.baeldung.com/java-decimalformat) 比我们要求的更先进。因此，我们应该选择`new BigDecimal(String)`或者`BigDecimal.valueOf()`。

## 6.**无效转换**

Java 提供了处理无效数字`String`的通用异常。

**值得注意的是，`new BigDecimal(String), *BigDecimal.valueOf()*`，`DecimalFormat.parse` 在我们经过`null` :** 时抛出一个`NullPointerException`

```java
@Test(expected = NullPointerException.class)
public void givenNullString_WhenBigDecimalObjectWithStringParameter_ThenNullPointerExceptionIsThrown() {
    String bigDecimal = null;
    new BigDecimal(bigDecimal);
}

@Test(expected = NullPointerException.class)
public void givenNullString_WhenValueOfDoubleFromString_ThenNullPointerExceptionIsThrown() {
    BigDecimal.valueOf(Double.valueOf(null));
}

@Test(expected = NullPointerException.class)
public void givenNullString_WhenDecimalFormatOfString_ThenNullPointerExceptionIsThrown()
  throws ParseException {
    new DecimalFormat("#").parse(null);
}
```

同理，`new BigDecimal(String)` 和 *BigDecimal.valueOf()* 当我们将一个无法解析的无效`String`传递给一个`BigDecimal`(比如`&`)时抛出一个`NumberFormatException`:

```java
@Test(expected = NumberFormatException.class)
public void givenInalidString_WhenBigDecimalObjectWithStringParameter_ThenNumberFormatExceptionIsThrown() {
    new BigDecimal("&");
}

@Test(expected = NumberFormatException.class)
public void givenInalidString_WhenValueOfDoubleFromString_ThenNumberFormatExceptionIsThrown() {
    BigDecimal.valueOf(Double.valueOf("&"));
} 
```

**最后，当我们经过一个无效的`String` :** 时，`DecimalFormat.parse`抛出一个`ParseException`

```java
@Test(expected = ParseException.class)
public void givenInalidString_WhenDecimalFormatOfString_ThenNumberFormatExceptionIsThrown()
  throws ParseException {
    new DecimalFormat("#").parse("&");
}
```

## 7.结论

在本文中，我们了解到 Java 为我们提供了多种将`String`转换为`BigDecimal`值的方法。一般来说，我们推荐使用`new BigDecimal(String)` 方法来达到这个目的。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626081441/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)