# 改变求和运算的顺序会产生不同的结果？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-floating-point-sum-order>

## 1。概述

在这篇简短的文章中，我们将看看为什么改变求和顺序会得到不同的结果。

## 2。问题

当我们看下面这段代码的时候，我们很容易预测出正确答案(13.22 + 4.88 + 21.45 = 39.55)。对我们来说容易的事情，Java 编译器可能会有不同的解释:

```java
double a = 13.22;
double b = 4.88;
double c = 21.45;

double abc = a + b + c;
System.out.println("a + b + c = " + abc); // Outputs: a + b + c = 39.55

double acb = a + c + b;
System.out.println("a + c + b = " + acb); // Outputs: a + c + b = 39.550000000000004 
```

从数学的角度来看，改变求和的顺序总是会得到相同的结果:

(A + B) + C = (A + C) + B

这是真的，并且在 Java(和其他计算机编程语言)中对整数很有效。然而，几乎所有的 CPU 都使用非整数数字 [IEEE 754 二进制浮点标准](https://web.archive.org/web/20221126215908/https://en.wikipedia.org/wiki/IEEE_floating_point)，这在十进制数字存储为二进制值时会引入不准确性。计算机不能精确地表示所有的实数。

当我们改变顺序时，我们也改变了存储在存储器中的中间值，因此结果可能不同。在下一个例子中，我们简单地从 A+B 或 A+C 的和开始:

```java
double ab = 18.1; // = 13.22 + 4.88
double ac = 34.67; // = 13.22 + 21.45
double sum_ab_c = ab + c;
double sum_ac_b = ac + b;
System.out.println("ab + c = " + sum_ab_c); // Outputs: 39.55
System.out.println("ac + b = " + sum_ac_b); // Outputs: 39.550000000000004 
```

## 3。解决方案

由于众所周知的浮点数的不精确性，double 不应该用于精确值。这包括货币。对于精确的值，我们可以使用`BigDecimal`类:

```java
BigDecimal d = new BigDecimal(String.valueOf(a));
BigDecimal e = new BigDecimal(String.valueOf(b));
BigDecimal f = new BigDecimal(String.valueOf(c));

BigDecimal def = d.add(e).add(f);
BigDecimal dfe = d.add(f).add(e);

System.out.println("d + e + f = " + def); // Outputs: 39.55
System.out.println("d + f + e = " + dfe); // Outputs: 39.55 
```

现在我们可以看到在这两种情况下结果是相同的。

## 4。结论

当处理十进制值时，我们需要记住浮点数的表示是不正确的，这可能会导致意想不到的结果。当要求精度时，必须使用`BigDecimal`类。

和往常一样，整篇文章中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126215908/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)