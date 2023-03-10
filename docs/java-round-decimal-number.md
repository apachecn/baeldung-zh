# 如何在 Java 中将一个数字四舍五入到 N 位小数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-round-decimal-number>

## 1。概述

在这个简短的教程中，我们将学习如何用 Java 将一个数字四舍五入到小数点后`n`位。

## 延伸阅读:

## [Java 中整数的位数](/web/20221206185533/https://www.baeldung.com/java-number-of-digits-in-int)

Learn different ways of getting the number of digits in an Integer in Java.[Read more](/web/20221206185533/https://www.baeldung.com/java-number-of-digits-in-int) →

## [在 Java 中检查一个数是否是质数](/web/20221206185533/https://www.baeldung.com/java-prime-numbers)

Learn how to check the primality of the number using Java.[Read more](/web/20221206185533/https://www.baeldung.com/java-prime-numbers) →

## [在 Java 中检查字符串是否为数字](/web/20221206185533/https://www.baeldung.com/java-check-string-number)

Explore different ways to determine whether a String is numeric or not.[Read more](/web/20221206185533/https://www.baeldung.com/java-check-string-number) →

## 2。Java 中的十进制数字

Java 提供了两种我们可以用来存储十进制数的基本类型:`float`和`double`。`Double`是默认类型:

```java
double PI = 3.1415;
```

然而，我们**不应该对精确值**使用任何一种类型，比如货币。为此，也为了舍入，我们可以使用`BigDecimal`类。

## 3。格式化一个十进制数

如果我们只想打印小数点后有`n`位数的十进制数，我们可以简单地格式化输出字符串:

```java
System.out.printf("Value with 3 digits after decimal point %.3f %n", PI);
// OUTPUTS: Value with 3 digits after decimal point 3.142
```

或者，我们可以用`[DecimalFormat](https://web.archive.org/web/20221206185533/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html)` 类格式化这个值:

```java
DecimalFormat df = new DecimalFormat("###.###");
System.out.println(df.format(PI));
```

`DecimalFormat`允许我们显式设置舍入行为，比上面使用的`String.format()`提供更多的输出控制。

## 4。用`BigDecimal`和舍入`Double` s

为了将`double` s 四舍五入到`n`小数位，我们可以编写一个**助手方法**:

```java
private static double round(double value, int places) {
    if (places < 0) throw new IllegalArgumentException();

    BigDecimal bd = new BigDecimal(Double.toString(value));
    bd = bd.setScale(places, RoundingMode.HALF_UP);
    return bd.doubleValue();
}
```

在这个解决方案中有一件重要的事情需要注意；构造`BigDecimal`时，我们必须**总是使用*【BigDecimal(String)】*构造函数**。这防止了表示不精确值的问题。

我们可以通过使用 [Apache Commons Math](https://web.archive.org/web/20221206185533/https://commons.apache.org/proper/commons-math/) 库获得相同的结果:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.5</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20221206185533/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-math3%22)

一旦我们将库添加到项目中，我们就可以使用`Precision.round()`方法，它有两个参数——value 和 scale:

```java
Precision.round(PI, 3);
```

默认情况下，它使用与我们的 helper 方法相同的`HALF_UP`舍入方法；所以，结果应该是一样的。

请注意，我们可以通过将所需的舍入方法作为第三个参数来更改舍入行为。

## 5。`DoubleRounder`用舍入双打

`DoubleRounder`是 [decimal4j](https://web.archive.org/web/20221206185533/https://github.com/tools4j/decimal4j) 库中的一个实用程序。它提供了一种快速且无垃圾的方法来对 0 到 18 位小数点进行双精度舍入。

我们可以通过向`pom.xml`添加依赖关系来获得这个库(最新版本可以在[这里找到](https://web.archive.org/web/20221206185533/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.decimal4j%22%20AND%20a%3A%22decimal4j%22)):

```java
<dependency>
    <groupId>org.decimal4j</groupId>
    <artifactId>decimal4j</artifactId>
    <version>1.0.3</version>
</dependency>
```

现在我们可以简单地使用:

```java
DoubleRounder.round(PI, 3);
```

然而，`DoubleRounder`在一些情况下会失败:

```java
System.out.println(DoubleRounder.round(256.025d, 2));
// OUTPUTS: 256.02 instead of expected 256.03
```

## 6。Math.round()方法

另一种舍入数字的方法是使用 `Math.Round()`方法。

在这种情况下，我们可以通过乘以并除以`10^n`来控制`n`的小数位数:

```java
public static double roundAvoid(double value, int places) {
    double scale = Math.pow(10, places);
    return Math.round(value * scale) / scale;
}
```

不推荐使用这种方法，因为它会截断值。在许多情况下，值被不正确地舍入:

```java
System.out.println(roundAvoid(1000.0d, 17));
// OUTPUTS: 92.23372036854776 !!
System.out.println(roundAvoid(260.775d, 2));
// OUTPUTS: 260.77 instead of expected 260.78
```

因此，此处列出此方法仅供学习之用。

## 7。结论

在本文中，我们介绍了将数字四舍五入到小数位的不同技术。

我们可以简单地格式化输出而不改变值，或者我们可以通过使用一个助手方法来舍入变量。我们还讨论了几个处理这个问题的库。

本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206185533/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)