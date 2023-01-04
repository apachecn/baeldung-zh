# Java 中的溢出和下溢

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-overflow-underflow>

## 1.介绍

在本教程中，我们将看看 Java 中数值数据类型的溢出和下溢。

我们不会深入到更理论的方面——我们只关注它在 Java 中的发生情况。

首先，我们来看看整数数据类型，然后看看浮点数据类型。对于这两种情况，我们还将看到如何检测何时发生上溢或下溢。

## 2.溢出和下溢

简单地说，当我们给一个超出变量声明的数据类型范围的值赋值时，就会发生上溢和下溢。

如果(绝对)值太大，我们称之为溢出，如果值太小，我们称之为下溢。

让我们看一个例子，在这个例子中，我们试图将值`10^(1000)`(带有`1000`零的`1`)赋给类型`int`或`double`的变量。对于 Java 中的一个`int`或`double`变量来说值太大，会有溢出。

作为第二个例子，假设我们试图将值`10^(-1000)`(非常接近 0)赋给类型`double`的变量。这个值对于 Java 中的`double`变量来说太小了，会出现下溢。

让我们更详细地看看在这些情况下 Java 会发生什么。

## 3.整数数据类型

Java 中的整数数据类型有`byte` (8 位)`short` (16 位)`int` (32 位)`long` (64 位)。

**这里，我们将重点关注`int`数据类型。同样的行为也适用于其他数据类型，只是最小值和最大值不同。**

在 Java 中，`int`类型的整数可以是负数也可以是正数，这意味着用它的 32 位，我们可以在`-2^(31) ` ( `-2147483648`)和`2^(31)-1` ( `2147483647`)之间赋值。

包装类`Integer`定义了两个保存这些值的常量:`Integer.MIN_VALUE`和`Integer.MAX_VALUE`。

### 3.1.例子

如果我们定义了一个类型为`int`的变量`m`，并试图赋一个太大的值(例如`21474836478 = MAX_VALUE + 1)?`，会发生什么

这种赋值的一个可能结果是`m`的值将是未定义的或者将会有一个错误。

两者都是有效的结果；但是在 Java 中，`m`的值会是`-2147483648`(最小值)。另一方面，如果我们试图赋值-2147483649 ( `= MIN_VALUE – 1`)，`m`将是`2147483647` (最大值)。**这种行为称为整数回绕。**

让我们考虑下面的代码片段来更好地说明这种行为:

```java
int value = Integer.MAX_VALUE-1;
for(int i = 0; i < 4; i++, value++) {
    System.out.println(value);
}
```

我们将得到下面的输出，它演示了溢出:

```java
2147483646
2147483647
-2147483648
-2147483647 
```

## 4.处理整数数据类型的下溢和上溢

**发生溢出时，Java 不会抛出异常；这就是为什么很难发现溢出导致的错误。**我们也不能直接访问溢出标志，大多数 CPU 都有。

然而，有各种方法来处理可能的溢出。让我们来看看这些可能性中的几种。

### 4.1。使用不同的数据类型

如果我们想要允许大于`2147483647`(或小于`-2147483648`)的值，我们可以简单地使用`long`数据类型或一个`BigInteger`来代替。

虽然类型为`long`的变量也可能溢出，但是最小值和最大值要大得多，在大多数情况下可能已经足够了。

除了 JVM 可用的内存量之外，`BigInteger`的取值范围不受限制。

让我们看看如何用`BigInteger`重写上面的例子:

```java
BigInteger largeValue = new BigInteger(Integer.MAX_VALUE + "");
for(int i = 0; i < 4; i++) {
    System.out.println(largeValue);
    largeValue = largeValue.add(BigInteger.ONE);
}
```

我们将看到以下输出:

```java
2147483647
2147483648
2147483649
2147483650
```

正如我们在输出中看到的，这里没有溢出。我们的文章 Java 中的 [`BigDecimal`和`BigInteger`更详细地涵盖了`BigInteger`。](/web/20221208143830/https://www.baeldung.com/java-bigdecimal-biginteger)

### 4.2。抛出异常

有些情况下，我们不希望允许更大的值，也不希望发生溢出，而是希望抛出一个异常。

从 Java 8 开始，我们可以使用精确算术运算的方法。我们先来看一个例子:

```java
int value = Integer.MAX_VALUE-1;
for(int i = 0; i < 4; i++) {
    System.out.println(value);
    value = Math.addExact(value, 1);
}
```

静态方法`addExact()`执行正常的加法，但是如果操作导致溢出或下溢，则抛出异常:

```java
2147483646
2147483647
Exception in thread "main" java.lang.ArithmeticException: integer overflow
	at java.lang.Math.addExact(Math.java:790)
	at baeldung.underoverflow.OverUnderflow.main(OverUnderflow.java:115)
```

除了`addExact()`之外，Java 8 中的`Math`包为所有的算术运算提供了相应的精确方法。参见 [Java 文档](https://web.archive.org/web/20221208143830/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Math.html)中所有这些方法的列表。

此外，还有精确的转换方法，如果在转换到另一种数据类型的过程中出现溢出，这些方法会引发异常。

对于从`long`到`int`的转换:

```java
public static int toIntExact(long a)
```

对于从`BigInteger`到`int`或`long`的转换:

```java
BigInteger largeValue = BigInteger.TEN;
long longValue = largeValue.longValueExact();
int intValue = largeValue.intValueExact();
```

### 4.3。Java 8 之前

Java 8 中加入了精确的算术方法。如果我们使用早期版本，我们可以简单地自己创建这些方法。这样做的一个选择是实现与 Java 8 中相同的方法:

```java
public static int addExact(int x, int y) {
    int r = x + y;
    if (((x ^ r) & (y ^ r)) < 0) {
        throw new ArithmeticException("int overflow");
    }
    return r;
}
```

## 5.非整数数据类型

当进行算术运算时，非整数类型`float`和`double`的行为方式与整数数据类型不同。

一个区别是对浮点数的算术运算会产生一个`NaN`。我们有一篇关于 Java 中的 [NaN 的专门文章，所以在这篇文章中我们不会深入探讨。此外，`Math`包中没有针对非整数类型的`addExact`或`multiplyExact`等确切的算术方法。](/web/20221208143830/https://www.baeldung.com/java-not-a-number)

Java 的`float`和`double`数据类型遵循 [IEEE 浮点运算标准(IEEE 754)](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/IEEE_754) 。这个标准是 Java 处理浮点数上溢和下溢的方式的基础。

**在下面的章节中，我们将关注`double`数据类型的上溢和下溢，以及我们可以做些什么来处理它们发生的情况。**

### 5.1.泛滥

至于整数数据类型，我们可能期望:

```java
assertTrue(Double.MAX_VALUE + 1 == Double.MIN_VALUE);
```

然而，浮点变量就不是这样了。以下是真实的:

```java
assertTrue(Double.MAX_VALUE + 1 == Double.MAX_VALUE);
```

**这是因为一个`double`值只有有限数量的[有效位](/web/20221208143830/https://www.baeldung.com/cs/most-significant-bit)。如果我们将一个大的`double`值只增加 1，我们不会改变任何有效位。因此，该值保持不变。**

如果我们增加变量的值，从而增加变量的一个有效位，变量将具有值`INFINITY`:

```java
assertTrue(Double.MAX_VALUE * 2 == Double.POSITIVE_INFINITY);
```

和负值的`NEGATIVE_INFINITY`:

```java
assertTrue(Double.MAX_VALUE * -2 == Double.NEGATIVE_INFINITY);
```

**我们可以看到，与整数不同，溢出没有绕回，而是有两种不同的可能结果:值保持不变，或者我们得到一个特殊值，`POSITIVE_INFINITY`或`NEGATIVE_INFINITY`。**

### 5.2.下溢

为一个`double`值的最小值定义了两个常数:`MIN_VALUE` (4.9e-324)和`MIN_NORMAL` (2.2250738585072014E-308)。

IEEE 浮点运算标准(IEEE 754) 更详细地解释了它们之间的区别。

让我们关注一下为什么我们需要浮点数的最小值。

**`double`值不能任意小，因为我们只有有限数量的位来表示该值。**

Java SE 语言规范中关于[类型、值和变量](https://web.archive.org/web/20221208143830/https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.2.3)的章节描述了浮点类型是如何表示的。`double`的二进制表示的最小指数被给定为`-1074`。这意味着 double 可以拥有的最小正值是`Math.pow(2, -1074)`，它等于`4.9e-324`。

因此，Java 中的`double`的精度不支持 0 和`4.9e-324,` 之间的值，也不支持 `-4.9e-324` 和 `0` 之间的负值。

那么，如果我们试图给类型为`double`的变量赋一个太小的值，会发生什么呢？让我们看一个例子:

```java
for(int i = 1073; i <= 1076; i++) {
    System.out.println("2^" + i + " = " + Math.pow(2, -i));
}
```

带输出:

```java
2^1073 = 1.0E-323
2^1074 = 4.9E-324
2^1075 = 0.0
2^1076 = 0.0 
```

我们看到，如果我们分配一个太小的值，我们会得到一个下溢，结果值是`0.0`(正零)。
类似地，对于负值，下溢将导致值`-0.0`(负零)。

## 6.检测浮点数据类型的下溢和上溢

由于上溢将导致正或负无穷大，下溢将导致正或负零，我们不需要像整数数据类型那样的精确算术方法。相反，我们可以检查这些特殊的常数来检测上溢和下溢。

如果我们想在这种情况下抛出一个异常，我们可以实现一个 helper 方法。让我们看看如何寻找指数运算:

```java
public static double powExact(double base, double exponent) {
    if(base == 0.0) {
        return 0.0;
    }

    double result = Math.pow(base, exponent);

    if(result == Double.POSITIVE_INFINITY ) {
        throw new ArithmeticException("Double overflow resulting in POSITIVE_INFINITY");
    } else if(result == Double.NEGATIVE_INFINITY) {
        throw new ArithmeticException("Double overflow resulting in NEGATIVE_INFINITY");
    } else if(Double.compare(-0.0f, result) == 0) {
        throw new ArithmeticException("Double overflow resulting in negative zero");
    } else if(Double.compare(+0.0f, result) == 0) {
        throw new ArithmeticException("Double overflow resulting in positive zero");
    }

    return result;
}
```

**在这个方法中，我们需要用到方法`Double.compare()`。正常的比较运算符(`<`和`>`)不区分正负零。**

## 7.正负`Zero`

最后，让我们看一个例子，它说明了为什么我们在处理正负零和无穷大时需要小心。

让我们定义几个变量来演示:

```java
double a = +0f;
double b = -0f;
```

**因为正负`0`被认为相等:**

```java
assertTrue(a == b);
```

**而正负无穷大被认为是不同的:**

```java
assertTrue(1/a == Double.POSITIVE_INFINITY);
assertTrue(1/b == Double.NEGATIVE_INFINITY);
```

然而，下面的断言是正确的:

```java
assertTrue(1/a != 1/b);
```

这似乎与我们的第一个主张相矛盾。

## 8.结论

在本文中，我们看到了什么是溢出和下溢，它在 Java 中是如何发生的，以及整数和浮点数据类型之间的区别。

我们还看到了如何在程序执行期间检测溢出和下溢。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)