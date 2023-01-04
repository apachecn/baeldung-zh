# 在 Java 中比较双精度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-comparing-doubles>

## 1.概观

在本教程中，我们将讨论在 Java 中比较双精度值的不同方法。特别是，这不像比较其他原始类型那样容易。事实上，它在许多其他语言中都有问题，不仅仅是 Java。

首先，我们将解释为什么使用 simple ==操作符是不准确的，并且可能导致难以跟踪运行时的错误。然后，我们将展示如何正确地比较普通 Java 和常见第三方库中的双精度值。

## 2.使用==运算符

使用==运算符进行比较的不准确性是由双精度值在计算机内存中的存储方式造成的。我们需要记住，有限的内存空间(通常是 64 位)必须容纳无限多的值。因此，**我们无法在我们的计算机**中得到大多数双精度值的精确表示。**他们必须变圆才能得救**。

由于舍入不准确，可能会出现有趣的错误:

```
double d1 = 0;
for (int i = 1; i <= 8; i++) {
    d1 += 0.1;
 }

double d2 = 0.1 * 8;

System.out.println(d1);
System.out.println(d2);
```

两个变量 `d1` 和`d2, `都应该等于 0.8。但是，当我们运行上面的代码时，我们会看到以下结果:

```
0.7999999999999999
0.8
```

在这种情况下，用==运算符比较这两个值会产生错误的结果。为此，我们必须使用更复杂的比较算法。

如果我们想要对舍入机制有最好的精度和控制，我们可以使用 [`java.math.BigDecimal`](/web/20220819091322/https://www.baeldung.com/java-bigdecimal-biginteger) 类。

## 3.在普通 Java 中比较双精度值

在普通 Java 中比较双精度值的推荐算法是一种**阈值比较方法**。在这种情况下，我们需要检查两个数之差**是否在规定的公差范围内，通常称为** `**epsilon**:`

```
double epsilon = 0.000001d;

assertThat(Math.abs(d1 - d2) < epsilon).isTrue();
```

ε值越小，比较准确度越高。然而，如果我们指定的公差值太小，我们将得到与简单==比较相同的错误结果。一般来说， **epsilon 的 5 位和 6 位小数的值通常是开始**的好地方。

不幸的是，标准 JDK 中没有实用程序可以用来以推荐的精确方式比较双精度值。幸运的是，我们不需要自己写。我们可以使用免费且广为人知的第三方库提供的各种专用方法。

## 4.使用 Apache Commons 数学

[Apache Commons Math](/web/20220819091322/https://www.baeldung.com/apache-commons-math) 是最大的致力于数学和统计组件的开源库之一。从各种不同的类和方法中，**我们将特别关注`org.apache.commons.math3.util.Precision`类。它包含两个有用的`equals()`方法来正确比较双精度值**:

```
double epsilon = 0.000001d;

assertThat(Precision.equals(d1, d2, epsilon)).isTrue();
assertThat(Precision.equals(d1, d2)).isTrue();
```

这里使用的`epsilon`变量与前面例子中的含义相同。这是一个允许的绝对误差量。然而，这并不是与阈值算法的唯一相似之处。特别是，两种`equals`方法都使用相同的方法。

双参数函数版本只是` [equals(d1, d2, 1)](https://web.archive.org/web/20220819091322/https://commons.apache.org/proper/commons-math/javadocs/api-3.6/org/apache/commons/math3/util/Precision.html#equals(double,%20double,%20int)) `方法调用的快捷方式。在这种情况下，`d1`和`d2`被认为是相等的，如果它们之间没有浮点数。

## 5.用番石榴

谷歌的 [Guava](/web/20220819091322/https://www.baeldung.com/guava-guide) 是一个大型的核心 Java 库集，它扩展了标准的 JDK 功能。它在`com.google.common.math`包中包含了大量有用的数学工具。**为了在 Guava 中正确地比较 double 值，让我们从`DoubleMath` 类`:`中实现`fuzzyEquals()`** 方法

```
double epsilon = 0.000001d;

assertThat(DoubleMath.fuzzyEquals(d1, d2, epsilon)).isTrue();
```

方法名不同于 Apache Commons Math 中的方法名，但是实际上它的工作方式是一样的。唯一的区别是，epsilon 的默认值没有重载方法。

## 6.使用 JUnit

JUnit 是最广泛使用的 Java 单元测试框架之一。一般来说，每个单元测试通常以分析期望值和实际值之间的差异而结束。因此，测试框架必须有正确和精确的比较算法。事实上，JUnit 为公共对象、集合和基本类型提供了一组比较方法，包括检查双值相等的专用方法:

```
double epsilon = 0.000001d;
assertEquals(d1, d2, epsilon);
```

事实上，它的工作原理与前面描述的 Guava 和 Apache Commons 的方法相同。

需要指出的是，还有一个不推荐使用的不带 epsilon 参数的双参数版本。然而，如果我们想确保我们的结果总是正确的，我们应该坚持三参数版本。

## 7.结论

在本文中，我们探索了在 Java 中比较双精度值的不同方法。

我们已经解释了为什么简单的比较可能会导致难以跟踪运行时的错误。然后，我们展示了如何正确地比较普通 Java 和公共库中的值。

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220819091322/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)