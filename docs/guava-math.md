# 番石榴数学应用指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-math>

## 1。概述

在本文中，我们将看到番石榴库中一些有用的数学运算。

番石榴有四个数学工具类:

1.  `IntMath`–对整数值的操作
2.  `LongMath`–长值运算
3.  `BigIntegerMath`–对`BigIntegers`的操作
4.  `DoubleMath`–双值运算

## 2。`IntMath`公用事业

`IntMath`用于对整数值进行数学运算。我们将浏览可用的方法列表，解释它们的每一种行为。

### 2.1。`binomial(int n, int k)`

这个函数计算 n 和 k 的二项式系数。它确保结果在整数范围内。否则，它给出`Integer.MAX_VALUE`。答案可以通过公式 n/k(n-k)得出:

```
@Test
public void whenBinomialOnTwoInt_shouldReturnResultIfUnderInt() {
    int result = IntMath.binomial(6, 3);

    assertEquals(20, result);
}

@Test
public void whenBinomialOnTwoInt_shouldReturnIntMaxIfOVerflowInt() {
    int result = IntMath.binomial(Integer.MAX_VALUE, 3);

    assertEquals(Integer.MAX_VALUE, result);
}
```

### 2.2。`ceilingPowerOfTwo(int x)`

这将计算大于或等于 x 的 2 的最小幂的值。结果 n 是这样的:2^(n-1) < x < 2 ^n:

```
@Test
public void whenCeilPowOfTwoInt_shouldReturnResult() {
  int result = IntMath.ceilingPowerOfTwo(20);

  assertEquals(32, result);
}
```

### 2.3。`checkedAdd(int a, int b)` 等人

该函数计算两个参数的和。这个函数提供了一个额外的检查，如果结果溢出就抛出`ArithmeticException`:

```
@Test
public void whenAddTwoInt_shouldReturnTheSumIfNotOverflow() {
    int result = IntMath.checkedAdd(1, 2);

    assertEquals(3, result);
}

@Test(expected = ArithmeticException.class)
public void whenAddTwoInt_shouldThrowArithmeticExceptionIfOverflow() {
    IntMath.checkedAdd(Integer.MAX_VALUE, 100);
} 
```

Guava 检查了另外三个可以溢出的运算符的方法:`checkedMultiply`、`checkedPow,` 和`checkedSubtract`。

### 2.4。`divide(int p, int q, RoundingMode mode)`

这是一个简单的除法，但允许我们定义一个舍入模式:

```
@Test
public void whenDivideTwoInt_shouldReturnTheResultForCeilingRounding() {
    int result = IntMath.divide(10, 3, RoundingMode.CEILING);

    assertEquals(4, result);
}

@Test(expected = ArithmeticException.class)
public void whenDivideTwoInt_shouldThrowArithmeticExIfRoundNotDefinedButNeeded() {
    IntMath.divide(10, 3, RoundingMode.UNNECESSARY);
}
```

### 2.5。`factorial(int n)`

计算 n 的阶乘值，即前 n 个正整数的乘积。如果 n = 0，则返回 1，如果结果不适合 int 范围，则返回`Integer.MAX_VALUE`。结果可由 n×n-1×n-2×x 得到..x 2 x 1:

```
@Test
public void whenFactorialInt_shouldReturnTheResultIfInIntRange() {
    int result = IntMath.factorial(5);

    assertEquals(120, result);
}

@Test
public void whenFactorialInt_shouldReturnIntMaxIfNotInIntRange() {
    int result = IntMath.factorial(Integer.MAX_VALUE);

    assertEquals(Integer.MAX_VALUE, result);
}
```

### 2.6。`floorPowerOfTwo(int x)`

返回 2 的最大幂，其结果小于或等于 x。结果 n 是 2^n < x < 2 ^(n+1):

```
@Test
public void whenFloorPowerOfInt_shouldReturnValue() {
    int result = IntMath.floorPowerOfTwo(30);

    assertEquals(16, result);
}
```

### 2.7。`gcd(int a, int b)`

这个函数给出了 a 和 b 的最大公约数:

```
@Test
public void whenGcdOfTwoInt_shouldReturnValue() {
    int result = IntMath.gcd(30, 40);
    assertEquals(10, result);
}
```

### 2.8。`isPowerOfTwo(int x)`

返回 x 是否是 2 的幂。如果值是 2 的幂，则返回 true，否则返回 false:

```
@Test
public void givenIntOfPowerTwo_whenIsPowOfTwo_shouldReturnTrue() {
    boolean result = IntMath.isPowerOfTwo(16);

    assertTrue(result);
}

@Test
public void givenIntNotOfPowerTwo_whenIsPowOfTwo_shouldReturnFalse() {
    boolean result = IntMath.isPowerOfTwo(20);

    assertFalse(result);
}
```

### 2.9。`isPrime(int n)`

这个函数将告诉我们传递的数字是否是质数:

```
@Test
public void givenNonPrimeInt_whenIsPrime_shouldReturnFalse() {
    boolean result = IntMath.isPrime(20);

    assertFalse(result);
}
```

### 2.10。`log10(int x, RoundingMode mode)`

这个 API 计算给定数字的以 10 为底的对数。使用提供的舍入模式对结果进行舍入:

```
@Test
public void whenLog10Int_shouldReturnTheResultForCeilingRounding() {
    int result = IntMath.log10(30, RoundingMode.CEILING);

    assertEquals(2, result);
}

@Test(expected = ArithmeticException.class)
public void whenLog10Int_shouldThrowArithmeticExIfRoundNotDefinedButNeeded() {
    IntMath.log10(30, RoundingMode.UNNECESSARY);
}
```

### 2.11。`log2(int x, RoundingMode mode)`

返回给定数字的以 2 为底的对数。使用提供的舍入模式对结果进行舍入:

```
@Test
public void whenLog2Int_shouldReturnTheResultForCeilingRounding() {
    int result = IntMath.log2(30, RoundingMode.CEILING);

    assertEquals(5, result);
}

@Test(expected = ArithmeticException.class)
public void whenLog2Int_shouldThrowArithmeticExIfRoundNotDefinedButNeeded() {
    IntMath.log2(30, RoundingMode.UNNECESSARY);
}
```

### 2.12。`mean(int x, int y)`

使用此函数，我们可以计算两个值的平均值:

```
@Test
public void whenMeanTwoInt_shouldReturnTheResult() {
    int result = IntMath.mean(30, 20);

    assertEquals(25, result);
}
```

### 2.13。`mod(int x, int m)`

返回一个数被另一个数整除的余数:

```
@Test
public void whenModTwoInt_shouldReturnTheResult() {
    int result = IntMath.mod(30, 4);
    assertEquals(2, result);
}
```

### 2.14。`pow(int b, int k)`

返回 b 的 k 次方值:

```
@Test
public void whenPowTwoInt_shouldReturnTheResult() {
    int result = IntMath.pow(6, 4);

    assertEquals(1296, result);
}
```

### 2.15。`saturatedAdd(int a, int b)`等人

一个 sum 函数，通过在发生溢出或下溢时分别返回值`Integer.MAX_VALUE`或`Integer.MIN_VALUE`来控制溢出或下溢:

```
@Test:
public void whenSaturatedAddTwoInt_shouldReturnTheResult() {
    int result = IntMath.saturatedAdd(6, 4);

    assertEquals(10, result);
}

@Test
public void whenSaturatedAddTwoInt_shouldReturnIntMaxIfOverflow() {
    int result = IntMath.saturatedAdd(Integer.MAX_VALUE, 1000);

    assertEquals(Integer.MAX_VALUE, result);
} 
```

另外还有三个饱和 API:`saturatedMultiply`、`saturatedPow` 和`saturatedSubtract`。

### 2.16。`sqrt(int x, RoundingMode mode)`

返回给定数字的平方根。使用提供的舍入模式对结果进行舍入:

```
@Test
public void whenSqrtInt_shouldReturnTheResultForCeilingRounding() {
    int result = IntMath.sqrt(30, RoundingMode.CEILING);

    assertEquals(6, result);
}

@Test(expected = ArithmeticException.class)
public void whenSqrtInt_shouldThrowArithmeticExIfRoundNotDefinedButNeded() {
    IntMath.sqrt(30, RoundingMode.UNNECESSARY);
}
```

## 3。`LongMath`公用事业

`LongMath`具有用于`Long`值的工具。大多数操作与`IntMath`实用程序相似，这里只描述了少数例外。

### 3.1。`mod(long x, int m)`和`mod(long x, long m)`

返回 x mod m。x 除以 m 的整数的余数:

```
@Test
public void whenModLongAndInt_shouldModThemAndReturnTheResult() {
    int result = LongMath.mod(30L, 4);

    assertEquals(2, result);
}
```

```
@Test
public void whenModTwoLongValues_shouldModThemAndReturnTheResult() {
    long result = LongMath.mod(30L, 4L);

    assertEquals(2L, result);
}
```

## 4。`BigIntegerMath`公用事业

`BigIntegerMath`用于对类型`BigInteger`进行数学运算。

该实用程序有一些类似于`IntMath.`的方法

## 5。双重数学实用程序

`DoubleMath`实用程序用于对双精度值执行运算。

与`BigInteger`实用程序相似，可用操作的数量有限，并且与`IntMath`实用程序相似。我们将列出一些仅适用于该实用程序类的异常函数。

### 5.1。`isMathematicalInteger(double x)`

返回 x 是否是一个数学整数。它检查数字是否可以表示为整数而不丢失数据:

```
@Test
public void givenInt_whenMathematicalDouble_shouldReturnTrue() {
    boolean result = DoubleMath.isMathematicalInteger(5);

    assertTrue(result);
}

@Test
public void givenDouble_whenMathematicalInt_shouldReturnFalse() {
    boolean result = DoubleMath.isMathematicalInteger(5.2);

    assertFalse(result);
}
```

### 5.2。`log2(double x)`

计算 x 的以 2 为底的对数:

```
@Test
public void whenLog2Double_shouldReturnResult() {
    double result = DoubleMath.log2(4);

    assertEquals(2, result, 0);
}
```

## 6。结论

在这个快速教程中，我们探索了一些有用的番石榴数学实用函数。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)