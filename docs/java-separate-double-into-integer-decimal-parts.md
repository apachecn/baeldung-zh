# 如何将 Double 分成整数和小数部分

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-separate-double-into-integer-decimal-parts>

## 1.概观

在本教程中，我们将探索在 Java 中分离浮点类型的整数和小数部分的各种方法，即`float`和`double`。

## 2.浮点类型的问题

让我们先来看看一个简单的分数，以及一个通过造型来执行分离的简单方法:

```java
double doubleNumber = 24.04;
int intPart = (int) doubleNumber;
System.out.println("Double Number: " + doubleNumber);
System.out.println("Integer Part: " + intPart);
System.out.println("Decimal Part: " + (doubleNumber - intPart));
```

当我们尝试运行上面的代码时，我们会得到以下结果:

```java
Double Number: 24.04
Integer Part: 24
Decimal Part: 0.03999999999999915
```

与我们的预期相反，输出没有正确打印小数部分。因此，浮点数不适合于不能容忍舍入误差的计算。

## 3.第一种方法:拆分`String`

首先，我们把十进制数转换成一个`String`等效值。然后我们可以在小数点索引处拆分`String`。

让我们用一个例子来理解这一点:

```java
double doubleNumber = 24.04;
String doubleAsString = String.valueOf(doubleNumber);
int indexOfDecimal = doubleAsString.indexOf(".");
System.out.println("Double Number: " + doubleNumber);
System.out.println("Integer Part: " + doubleAsString.substring(0, indexOfDecimal));
System.out.println("Decimal Part: " + doubleAsString.substring(indexOfDecimal)); 
```

上面代码的输出是:

```java
Double Number: 24.04
Integer Part: 24
Decimal Part: .04
```

输出正是我们所期望的。但是，这里的问题是使用字符串的限制——这意味着我们现在不能对它执行任何其他算术运算。

## 4.第二种方法:使用`BigDecimal`

Java 中的 [`BigDecimal`](https://web.archive.org/web/20220626083717/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html) 类为用户提供了对舍入行为的完全控制。该类还提供算术、小数位数操作、舍入、比较、哈希和格式转换等操作。

让我们使用`BigDecimal`来获得浮点数的整数和小数部分:

```java
double doubleNumber = 24.04;
BigDecimal bigDecimal = new BigDecimal(String.valueOf(doubleNumber));
int intValue = bigDecimal.intValue();
System.out.println("Double Number: " + bigDecimal.toPlainString());
System.out.println("Integer Part: " + intValue);
System.out.println("Decimal Part: " + bigDecimal.subtract(
  new BigDecimal(intValue)).toPlainString());
```

输出将是:

```java
Double Number: 24.04
Integer Part: 24
Decimal Part: 0.04
```

正如我们在上面看到的，输出是预期的。我们还可以借助`BigDecimal`类提供的方法进行算术运算。

## 5.结论

在本文中，我们讨论了分离浮点类型的整数和小数部分的各种方法。我们还讨论了使用`BigDecimal`进行浮点计算的好处。

此外，我们关于 Java 中的 BigDecimal 和 BigInteger 的详细教程讨论了这两个类的更多特征和使用场景。

最后，本教程的完整源代码[可在 GitHub 上获得。](https://web.archive.org/web/20220626083717/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)