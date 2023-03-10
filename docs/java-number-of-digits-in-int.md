# Java 中整数的位数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-number-of-digits-in-int>

## 1。简介

在这个快速教程中，我们将探索在 Java 中获取`Integer` 位数的**种不同方法。**

我们还将分析不同的方法，找出哪种算法最适合每种情况。

## 延伸阅读:

## [如何在 Java 中将一个数字四舍五入到 N 位小数](/web/20220925224937/https://www.baeldung.com/java-round-decimal-number)

Overview of several ways of handling the common problem of rounding a decimal number in Java[Read more](/web/20220925224937/https://www.baeldung.com/java-round-decimal-number) →

## [在 Java 中检查字符串是否为数字](/web/20220925224937/https://www.baeldung.com/java-check-string-number)

Explore different ways to determine whether a String is numeric or not.[Read more](/web/20220925224937/https://www.baeldung.com/java-check-string-number) →

## 十进制格式实用指南

Explore the Java's DecimalFormat class along with its practical usages.[Read more](/web/20220925224937/https://www.baeldung.com/java-decimalformat) →

## 2。`Integer` 中的位数

对于这里讨论的方法，我们只考虑正整数。如果我们期望任何负输入，那么在使用这些方法之前，我们可以首先使用`Math.abs(number)`。

### 2.1。`String`基于解决方案

也许获得一个`Integer`的位数最简单的方法是将其转换成`String`，并调用`length()`方法。这将返回我们的数字的`String`表示的长度:

```java
int length = String.valueOf(number).length();
```

**然而，这可能是一种次优的方法，因为该语句涉及每次评估的`String` 的内存分配。**JVM 必须解析我们的数字，并将其复制到一个单独的`String,`中，还要执行许多其他不同的操作(比如保存临时副本、处理 Unicode 转换等)。

如果我们只有几个数字要评估，那么我们可以使用这种解决方案，因为这种方法和任何其他方法之间的差异都可以忽略不计，即使对于大量数字也是如此。

### 2.2。对数方法

对于以十进制形式表示的数字，如果我们以 10 为基数取它们的对数并向上取整，我们将得到该数字的位数:

```java
int length = (int) (Math.log10(number) + 1);
```

注意，任何数字的`log[10]0`都没有定义，所以如果我们期望任何值为`0`的输入，那么我们也可以对其进行检查。

**对数方法比基于`String`的方法快得多，因为它不需要经过任何数据转换过程。**它只是涉及一个简单直接的计算，没有任何额外的对象初始化或循环。

### 2.3。重复乘法

在这个方法中，我们将获取一个临时变量(初始化为 1 ),并不断将它乘以 10，直到它大于我们的数字。在这个过程中，我们还将使用一个`length`变量，它将跟踪数字的长度:

```java
int length = 0;
long temp = 1;
while (temp <= number) {
    length++;
    temp *= 10;
}
return length;
```

在这段代码中，`temp *= 10`和写`temp = (temp << 3) + (temp << 1)`是一样的。由于在某些处理器上，乘法运算通常比移位运算符代价更高，所以后者可能更高效一些。

### 2.4。除以 2 的幂

如果我们知道我们的数字范围，那么我们可以使用一个变量，这将进一步减少我们的比较。这种方法将数字除以 2 的幂(例如 1、2、4、8 等)。):

```java
int length = 1;
if (number >= 100000000) {
    length += 8;
    number /= 100000000;
}
if (number >= 10000) {
    length += 4;
    number /= 10000;
}
if (number >= 100) {
    length += 2;
    number /= 100;
}
if (number >= 10) {
    length += 1;
}
return length;
```

它利用了这样一个事实，即任何数都可以用 2 的幂相加来表示。比如 15 可以表示为 8+4+2+1，都是 2 的幂。

对于一个 15 位数的数字，我们在以前的方法中将进行 15 次比较，而在这种方法中只有 4 次。

### 2.5。各个击破

与这里描述的所有其他方法相比，这可能是最庞大的方法；然而，它也是**最快的**，因为我们不执行任何类型的转换、乘法、加法或对象初始化。

我们可以从三四个简单的陈述中找到答案:

```java
if (number < 100000) {
    if (number < 100) {
        if (number < 10) {
            return 1;
        } else {
            return 2;
        }
    } else {
        if (number < 1000) {
            return 3;
        } else {
            if (number < 10000) {
                return 4;
            } else {
                return 5;
            }
        }
    }
} else {
    if (number < 10000000) {
        if (number < 1000000) {
            return 6;
        } else {
            return 7;
        }
    } else {
        if (number < 100000000) {
            return 8;
        } else {
            if (number < 1000000000) {
                return 9;
            } else {
                return 10;
            }
        }
    }
}
```

类似于前面的方法，我们只有在知道我们的数的范围时才能使用这种方法。

## 3。基准测试

现在我们已经很好地理解了潜在的解决方案，让我们使用 [Java 微基准测试工具(JMH)](/web/20220925224937/https://www.baeldung.com/java-microbenchmark-harness) 对我们的方法进行一些简单的基准测试。

下表显示了每个操作的平均处理时间(以纳秒为单位):

```java
Benchmark                            Mode  Cnt   Score   Error  Units
Benchmarking.stringBasedSolution     avgt  200  32.736 ± 0.589  ns/op
Benchmarking.logarithmicApproach     avgt  200  26.123 ± 0.064  ns/op
Benchmarking.repeatedMultiplication  avgt  200   7.494 ± 0.207  ns/op
Benchmarking.dividingWithPowersOf2   avgt  200   1.264 ± 0.030  ns/op
Benchmarking.divideAndConquer        avgt  200   0.956 ± 0.011  ns/op
```

基于`String`的解决方案是最简单的，也是成本最高的操作，因为它是唯一需要数据转换和新对象初始化的解决方案。

对数方法明显比前一种解决方案更有效，因为它不涉及任何数据转换。此外，作为一个单线解决方案，它可能是基于`String-`的方法的一个很好的替代方案。

重复乘法涉及与数长度成比例的简单乘法；例如，如果一个数是 15 位数，那么这个方法将涉及 15 次乘法。

然而，下一种方法利用了每个数字都可以用 2 的幂来表示的事实(这种方法类似于 BCD)。它将同一个等式简化为四则除法运算，因此它甚至比前者更高效。

最后，正如我们可以推断的，**最有效的算法是冗长的分治实现，**只用三四个简单的`if`语句就给出了答案。如果我们有大量的数据需要分析，我们可以使用它。

## 4。结论

在这篇简短的文章中，我们概述了一些寻找`Integer,`中的位数的方法，并比较了每种方法的效率。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220925224937/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)