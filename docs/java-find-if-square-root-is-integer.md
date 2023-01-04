# 确定整数的平方根在 Java 中是否是整数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-find-if-square-root-is-integer>

## 1.概观

完美的平方是一个可以表示为两个相等整数的乘积的数。

在本文中，我们将发现在 Java 中确定一个整数是否是完美平方的多种方法。此外，我们将讨论每种技术的优点和缺点，以确定其效率以及哪种是最快的。

## 2.检查一个整数是否是正方

众所周知，Java 给了我们两种定义整数的数据类型。第一个是`int`，用 32 位表示数，另一个是`long`，用 64 位表示数。在本文中，我们将使用`long`数据类型来处理最坏的情况(最大可能的整数)。

由于 Java 用 64 位表示长数，所以长数的范围是从`-9,223,372,036,854,775,808`到 9 `,223,372,036,854,775,807`。而且，因为我们处理的是完美的正方形，所以我们只关心处理正整数集合，因为任何整数乘以它自己总会产生一个正数。

另外，由于最大数大约是 2 ^(63) ，也就是说大约有 2 ^(31.5) 个整数的平方小于 2 ^(63) 。此外，我们可以假设拥有这些数字的查找表是低效的。

### 2.1.在 Java 中使用`sqrt`方法

**检查一个整数是否是正方的最简单直接的方法是使用`sqrt`函数**。我们知道，`sqrt`函数返回一个`double`值。所以，我们需要做的就是把结果投射到`int`上，然后自己相乘。然后，我们检查结果是否等于我们开始时的整数:

```java
public static boolean isPerfectSquareByUsingSqrt(long n) {
    if (n <= 0) {
        return false;
    }
    double squareRoot = Math.sqrt(n);
    long tst = (long)(squareRoot + 0.5);
    return tst*tst == n;
}
```

注意，由于在处理`double`值时可能会遇到精度误差，我们可能需要在结果上加 0.5。有时，当整数被赋给一个`double`变量时，可以用一个小数点来表示。

例如，如果我们将数字 3 赋给一个`double`变量，那么它的值可能是 3.00000001 或 2.99999999。因此，为了避免这种表示，我们在将它转换为`long`之前加上 0.5，以确保我们得到的是实际值。

此外，如果我们用一个数字测试`sqrt`函数，我们会注意到执行时间很快。另一方面，如果我们需要多次调用`sqrt`函数，并且我们试图减少由`sqrt`函数执行的操作数量，这种微优化实际上会有所不同。

### 2.2.使用二分搜索法

**我们可以用二分搜索法来求一个数的平方根，而不用使用`sqrt`函数**。

由于数的范围是从 1 到 2 ^(63) ，所以根在 1 和 2 ^(31.5) 之间。因此，二分搜索法算法需要大约 16 次迭代才能得到平方根:

```java
public boolean isPerfectSquareByUsingBinarySearch(long low, long high, long number) {
    long check = (low + high) / 2L;
    if (high < low) {
        return false;
    }
    if (number == check * check) {
        return true;
    }
    else if (number < check * check) {
        high = check - 1L;
        return isPerfectSquareByUsingBinarySearch(low, high, number);
    }
    else {
        low = check + 1L;
        return isPerfectSquareByUsingBinarySearch(low, high, number);
    }
}
```

### 2.3.二分搜索法的增强功能

为了提高二分搜索法，我们可以注意到，如果我们确定了基本数的位数，这就给出了根的范围。

例如，如果数字仅由一个数字组成，那么平方根的范围在 1 和 4 之间。原因是一位数的最大整数是 9，它的根是 3。另外，如果数字由两位数组成，则范围在 4 到 10 之间，以此类推。

因此，我们可以**建立一个查找表，根据从**开始的数字的位数来指定平方根的范围。这将缩小二分搜索法的范围。因此，得到平方根需要更少的迭代:

```java
public class BinarySearchRange {
    private long low;
    private long high;

    // standard constructor and getters
}
```

```java
private void initiateOptimizedBinarySearchLookupTable() {
    lookupTable.add(new BinarySearchRange());
    lookupTable.add(new BinarySearchRange(1L, 4L));
    lookupTable.add(new BinarySearchRange(3L, 10L));
    for (int i = 3; i < 20; i++) {
        lookupTable.add(
          new BinarySearchRange(
            lookupTable.get(i - 2).low * 10,
            lookupTable.get(i - 2).high * 10));
    }
}
```

```java
public boolean isPerfectSquareByUsingOptimizedBinarySearch(long number) {
    int numberOfDigits = Long.toString(number).length();
    return isPerfectSquareByUsingBinarySearch(
      lookupTable.get(numberOfDigits).low,
      lookupTable.get(numberOfDigits).high, number);
}
```

### 2.4.整数运算的牛顿法

**一般情况下，我们可以用牛顿法求任意数的平方根，即使是非整数。**牛顿法的基本思想是假设一个数`X`是一个数`N`的平方根。之后，我们可以开始一个循环，不断计算根，这必将走向正确的`N`的平方根。

然而，通过对牛顿方法的一些修改，我们可以用它来检查一个整数是否是一个完美的平方:

```java
public static boolean isPerfectSquareByUsingNewtonMethod(long n) {
    long x1 = n;
    long x2 = 1L;
    while (x1 > x2) {
        x1 = (x1 + x2) / 2L;
        x2 = n / x1;
    }
    return x1 == x2 && n % x1 == 0L;
}
```

## 3.优化整数平方根算法

正如我们所讨论的，有多种算法来检查一个整数的平方根。尽管如此，我们总是可以通过使用一些技巧来优化任何算法。

**技巧应该考虑避免执行决定平方根的主要操作。**比如我们可以直接排除负数。

我们可以利用的一个事实是**“完美的正方形只能以 16 为基数以 0、1、4 或 9 结尾”**。因此，在开始计算之前，我们可以将一个整数转换为基数为 16 的整数。之后，我们排除将数字视为非完美平方根的情况:

```java
public static boolean isPerfectSquareWithOptimization(long n) {
    if (n < 0) {
        return false;
    }
    switch((int)(n & 0xF)) {
        case 0: case 1: case 4: case 9:
            long tst = (long)Math.sqrt(n);
            return tst*tst == n;
        default:
            return false;
    }
}
```

## 4.结论

在本文中，我们讨论了确定一个整数是否是正方的多种方法。正如我们已经看到的，我们总是可以通过使用一些技巧来增强算法。

这些招数在开始算法的主运算之前会排除大量的情况。原因是很多整数很容易被判定为非完美平方。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-4)