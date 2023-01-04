# 在 Java 中检查一个数是奇数还是偶数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-number-parity>

## 1.概观

我们知道，一个数的奇偶性是由其除以 2 的余数决定的**。**偶数产生余数 0，奇数产生余数 1。

在本教程中，**我们将看到在 Java 中检查一个数是偶数还是奇数的多种方法。**

## 2.除法方法

返回除法余数的算术运算符是[模数](/web/20220811081223/https://www.baeldung.com/modulo-java)运算符`%`。

我们可以验证一个数是偶数还是奇数的最简单的方法是通过将该数除以 2 并检查余数的数学运算:

```
boolean isEven(int x) {
    return x % 2 == 0;
}

boolean isOdd(int x) {
    return x % 2 != 0;
}
```

让我们编写几个测试来确认我们的方法的行为:

```
assertEquals(true, isEven(2));
assertEquals(true, isOdd(3));
```

## 3.位运算方法

我们可以对一个数字进行多种[位](/web/20220811081223/https://www.baeldung.com/java-bitwise-operators)运算，以确定它是偶数还是奇数。

**位运算** **比其他确定数的奇偶性的方法更有效** **。**

### 3.1.按位`OR` (|)

**一个偶数`OR` 1 将总是把数字增加 1** 。

**奇数`OR` 1 总会导致** **同数**:

```
boolean isOrEven(int x) {
    return (x | 1) > x;
}

boolean isOrOdd(int x) {
    return (x | 1) == x;
}
```

让我们通过一些测试来展示我们代码的行为:

```
assertEquals(true, isOrEven(4));
assertEquals(true, isOrOdd(5));
```

### 3.2.按位`AND` ( `&`)

**偶数`AND` 1 总是导致 0** 。另一方面，**奇数`AND`1**导致 1 :

```
boolean isAndEven(int x) {
    return (x & 1) == 0;
}

boolean isAndOdd(int x) {
    return (x & 1) == 1;
}
```

我们将通过一个小测试来确认这一行为:

```
assertEquals(true, isAndEven(6));
assertEquals(true, isAndOdd(7));
```

### 3.3.按位`XOR` ( `^`)

**按位`XOR`是**的最优解**来检查一个数的奇偶性。**

 ****偶数`XOR` 1 总** **加 1，n 奇数`XOR` 1 总** **减 1** :

```
boolean isXorEven(int x) {
    return (x ^ 1) > x;
}

boolean isXorOdd(int x) {
    return (x ^ 1) < x;
}
```

让我们编写一些小测试来检查我们的代码:

```
assertEquals(true, isXorEven(8));
assertEquals(true, isXorOdd(9));
```

## 4.最低有效位

我们介绍的最后一种方法是读取数字的最低有效位[位](/web/20220811081223/https://www.baeldung.com/java-get-bit-at-position)。

**偶数的最低有效位** **始终为 0，奇数的最低有效位始终为 1:**

```
boolean isLsbEven(int x) {
    return Integer.toBinaryString(x).endsWith("0");
}

boolean isLsbOdd(int x) {
    return Integer.toBinaryString(x).endsWith("1");
}
```

我们将用几行代码来演示这种行为:

```
assertEquals(true, isLsbEven(10));
assertEquals(true, isLsbOdd(11));
```

## 5.结论

在这篇文章中，我们学习了多种方法来检查一个数的奇偶性，即它是偶数还是奇数。我们看到**检查奇偶性的最优解是逐位`XOR`操作**。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220811081223/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-5)**