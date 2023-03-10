# 在 Java 中寻找最小公倍数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-least-common-multiple>

## 1.概观

**两个非零整数`(a, b)`的[最小公倍数](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Least_common_multiple) (LCM)是能被`a`和`b`整除的最小正整数。**

在本教程中，我们将学习不同的方法来寻找两个或多个数字的 LCM。我们必须注意到，**负整数和零不是 LCM** 的候选者。

## 2.使用简单算法计算两个数的 LCM

我们可以利用 **[乘法](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Multiplication)是重复加法**这个简单的事实，求出两个数的 LCM。

### 2.1。算法

寻找 LCM 的简单算法是一种迭代方法，它利用了两个数的 LCM 的一些基本性质。

首先，我们知道任何带零的数的 **LCM 本身就是零**。因此，只要给定的整数中有一个为 0，我们就可以提前退出这个过程。

其次，我们还可以利用两个非零整数的 LCM 的**下界是两个数**的绝对值中较大的一个。

此外，如前所述，LCM 不能是负整数。所以，我们将**只使用整数**的绝对值来寻找可能的倍数，直到我们找到一个公倍数。

让我们来看看确定 lcm(a，b)需要遵循的确切步骤:

1.  如果 a = 0 或 b = 0，则返回 lcm(a，b) = 0，否则转到步骤 2。
2.  计算这两个数字的绝对值。
3.  将 lcm 初始化为步骤 2 中计算的两个值中的较高值。
4.  如果 lcm 能被较低的绝对值整除，则返回。
5.  将 lcm 增加两者中较高的绝对值，并转到步骤 4。

在我们开始实现这个简单的方法之前，让我们先试着找出 lcm(12，18)。

因为 12 和 18 都是正的，所以让我们跳到第 3 步，初始化 lcm = max(12，18) = 18，然后继续。

在我们的第一次迭代中，lcm = 18，它不能被 12 整除。因此，我们将它递增 18 并继续。

在第二次迭代中，我们可以看到 lcm = 36，并且现在可以被 12 整除。因此，我们可以从算法返回并得出 lcm(12，18)是 36 的结论。

### 2.2。实施

让我们用 Java 实现算法。我们的`lcm()`方法需要接受两个整数参数，并给出它们的 LCM 作为返回值。

我们可以注意到，上面的算法涉及到对数字执行一些数学运算，比如寻找绝对值、最小值和最大值。为此，我们可以分别使用 [`Math`](https://web.archive.org/web/20221126225111/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Math.html) 类对应的静态方法，如`abs()`、`min(),`、`max()`。

让我们实现我们的`lcm()`方法:

```java
public static int lcm(int number1, int number2) {
    if (number1 == 0 || number2 == 0) {
        return 0;
    }
    int absNumber1 = Math.abs(number1);
    int absNumber2 = Math.abs(number2);
    int absHigherNumber = Math.max(absNumber1, absNumber2);
    int absLowerNumber = Math.min(absNumber1, absNumber2);
    int lcm = absHigherNumber;
    while (lcm % absLowerNumber != 0) {
        lcm += absHigherNumber;
    }
    return lcm;
}
```

接下来，让我们也验证一下这个方法:

```java
@Test
public void testLCM() {
    Assert.assertEquals(36, lcm(12, 18));
}
```

上面的测试用例通过断言 lcm(12，18)是 36 来验证`lcm()`方法的正确性。

## 3.使用质因数分解方法

**[算术基本定理](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Fundamental_theorem_of_arithmetic)指出，有可能将每个大于 1 的整数唯一地表示为素数幂的乘积。**

所以，对于任意整数 N > 1，我们有 N =(2^(k1))*(3^(k2))*(5^(k3))*…

使用这个定理的结果，我们现在将理解寻找两个数的 LCM 的质因数分解方法。

### 3.1。算法

质因数分解方法从两个数的质因数分解中计算出 [LCM。我们可以使用质因数分解中的质因数和指数来计算两个数的 LCM:](https://web.archive.org/web/20221126225111/https://proofwiki.org/wiki/LCM_from_Prime_Decomposition)

当| a | =(2^(P1))*(3^(p2))*(5^(P3))*…
和| b | =(2^(Q1))*(3^(Q2))*(5^(Q3))*…
那么， **lcm(a，b) = (2 ^(max(p[1] ，q)**

让我们看看如何使用这种方法计算 12 和 18 的 LCM:

首先我们需要将两个数的绝对值表示为质因数的乘积:
12 = 2 * 2 * 3 = 2 * 3
18 = 2 * 3 * 3 = 2 * 3

我们可以注意到，上面表示中的质因数是 2 和 3。

接下来，让我们确定 LCM 的每个质因数的指数。我们通过从两个表象中获取它的更高力量来做到这一点。

使用这种策略，LCM 中 2 的幂将是 max(2，1) = 2，LCM 中 3 的幂将是 max(1，2) = 2。

最后，我们可以通过将质因数乘以上一步中获得的相应幂来计算 LCM。因此，我们得到 lcm(12，18) = 2 * 3 = 36。

### 3.2。实施

我们的 Java 实现使用这两个数字的质因数分解表示来寻找 LCM。

为此，我们的`getPrimeFactors()`方法需要接受一个整数参数，并给出它的质因数分解表示。在 Java 中，**我们可以用一个`HashMap`** 来表示一个数的质因数分解，其中每个键表示质因数，与键相关联的值表示相应因数的指数。

让我们看看`getPrimeFactors()`方法的迭代实现:

```java
public static Map<Integer, Integer> getPrimeFactors(int number) {
    int absNumber = Math.abs(number);

    Map<Integer, Integer> primeFactorsMap = new HashMap<Integer, Integer>();

    for (int factor = 2; factor <= absNumber; factor++) {
        while (absNumber % factor == 0) {
            Integer power = primeFactorsMap.get(factor);
            if (power == null) {
                power = 0;
            }
            primeFactorsMap.put(factor, power + 1);
            absNumber /= factor;
        }
    }

    return primeFactorsMap;
}
```

我们知道 12 和 18 的素因子分解图分别是{2 → 2，3 → 1}和{2 → 1，3 → 2}。让我们用这个来测试上面的方法:

```java
@Test
public void testGetPrimeFactors() {
    Map<Integer, Integer> expectedPrimeFactorsMapForTwelve = new HashMap<>();
    expectedPrimeFactorsMapForTwelve.put(2, 2);
    expectedPrimeFactorsMapForTwelve.put(3, 1);

    Assert.assertEquals(expectedPrimeFactorsMapForTwelve, 
      PrimeFactorizationAlgorithm.getPrimeFactors(12));

    Map<Integer, Integer> expectedPrimeFactorsMapForEighteen = new HashMap<>();
    expectedPrimeFactorsMapForEighteen.put(2, 1);
    expectedPrimeFactorsMapForEighteen.put(3, 2);

    Assert.assertEquals(expectedPrimeFactorsMapForEighteen, 
      PrimeFactorizationAlgorithm.getPrimeFactors(18));
}
```

我们的`lcm()`方法首先使用`getPrimeFactors()`方法为每个数寻找质因数分解图。接下来，它使用这两个数字的质因数分解图来寻找它们的 LCM。让我们看看这个方法的迭代实现:

```java
public static int lcm(int number1, int number2) {
    if(number1 == 0 || number2 == 0) {
        return 0;
    }

    Map<Integer, Integer> primeFactorsForNum1 = getPrimeFactors(number1);
    Map<Integer, Integer> primeFactorsForNum2 = getPrimeFactors(number2);

    Set<Integer> primeFactorsUnionSet = new HashSet<>(primeFactorsForNum1.keySet());
    primeFactorsUnionSet.addAll(primeFactorsForNum2.keySet());

    int lcm = 1;

    for (Integer primeFactor : primeFactorsUnionSet) {
        lcm *= Math.pow(primeFactor, 
          Math.max(primeFactorsForNum1.getOrDefault(primeFactor, 0),
            primeFactorsForNum2.getOrDefault(primeFactor, 0)));
    }

    return lcm;
}
```

作为一个好的实践，我们现在将验证`lcm()`方法的逻辑正确性:

```java
@Test
public void testLCM() {
    Assert.assertEquals(36, PrimeFactorizationAlgorithm.lcm(12, 18));
}
```

## 4.使用欧几里德算法

两个数的 [LCM](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Least_common_multiple) 和 [GCD](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Greatest_common_divisor) (最大公约数)之间有一个有趣的关系，即两个数乘积的 [**绝对值等于它们 GCD 和 LCM**](https://web.archive.org/web/20221126225111/https://proofwiki.org/wiki/Product_of_GCD_and_LCM) 的乘积。

如前所述，gcd(a，b) * lcm(a，b) = |a * b|。

因此， **lcm(a，b) = |a * b|/gcd(a，b)** 。

使用这个公式，我们原来求 lcm(a，b)的问题现在已经简化为只求 gcd(a，b)。

诚然，**有多种策略可以找到两个数的 GCD** 。然而， **[欧几里德算法](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Euclidean_algorithm)被认为是所有算法中最有效的**之一。

为此，我们先简单了解一下这个算法的症结所在，可以归结为两个关系:

*   **gcd (a，b) = gcd(|a%b|，| a |)；其中|a| > = |b|**
*   **gcd(p，0) = gcd(0，p) = |p|**

让我们看看如何使用上述关系找到 lcm(12，18 ):

我们有 gcd(12，18) = gcd(18%12，12) = gcd(6，12) = gcd(12%6，6) = gcd(0，6) = 6

因此，lcm(12，18) = |12 x 18| / gcd(12，18) = (12 x 18) / 6 = 36

我们现在将看到欧几里德算法的**递归实现:**

```java
public static int gcd(int number1, int number2) {
    if (number1 == 0 || number2 == 0) {
        return number1 + number2;
    } else {
        int absNumber1 = Math.abs(number1);
        int absNumber2 = Math.abs(number2);
        int biggerValue = Math.max(absNumber1, absNumber2);
        int smallerValue = Math.min(absNumber1, absNumber2);
        return gcd(biggerValue % smallerValue, smallerValue);
    }
}
```

上面的实现使用了数字的绝对值——因为 GCD 是完美地将两个数字相除的最大正整数，所以我们对负除数不感兴趣。

我们现在准备验证上述实现是否如预期的那样工作:

```java
@Test
public void testGCD() {
    Assert.assertEquals(6, EuclideanAlgorithm.gcd(12, 18));
}
```

### 4.1。两位数的 LCM

使用早期的方法来寻找 GCD，我们现在可以很容易地计算 LCM。同样，我们的`lcm()`方法需要接受两个整数作为输入来返回它们的 LCM。让我们看看如何用 Java 实现这个方法:

```java
public static int lcm(int number1, int number2) {
    if (number1 == 0 || number2 == 0)
        return 0;
    else {
        int gcd = gcd(number1, number2);
        return Math.abs(number1 * number2) / gcd;
    }
}
```

我们现在可以验证上述方法的功能:

```java
@Test
public void testLCM() {
    Assert.assertEquals(36, EuclideanAlgorithm.lcm(12, 18));
}
```

### 4.2。使用`BigInteger`类的大数 LCM

为了计算大数的 LCM，我们可以利用`[BigInteger](https://web.archive.org/web/20221126225111/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigInteger.html)` 类。

在内部，`BigInteger`类的 **`gcd()`方法使用混合算法**来优化计算性能。此外，由于 **`BigInteger`对象是不可变的**，实现利用了 **`[MutableBigInteger](https://web.archive.org/web/20221126225111/https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/math/MutableBigInteger.java)`类的可变实例来避免频繁的内存重新分配**。

**首先，它使用传统的欧几里德算法**用较低整数的模数重复替换较高整数。

结果，这一对不仅越来越小，而且在连续分裂后彼此越来越近**。**最终，在它们各自的`int[]` 值数组中保存两个`MutableBigInteger`对象的大小所需的`int`数量的差异达到 1 或 0。

在这个阶段，策略切换到 **[二进制 GCD 算法](https://web.archive.org/web/20221126225111/https://en.wikipedia.org/wiki/Binary_GCD_algorithm)以获得更快的计算结果**。

在这种情况下，我们也将通过用数字乘积的绝对值除以它们的 GCD 来计算 LCM。与前面的例子类似，我们的`lcm()`方法将两个`BigInteger`值作为输入，并将这两个数字的 LCM 作为一个`BigInteger`返回。让我们来看看它的实际应用:

```java
public static BigInteger lcm(BigInteger number1, BigInteger number2) {
    BigInteger gcd = number1.gcd(number2);
    BigInteger absProduct = number1.multiply(number2).abs();
    return absProduct.divide(gcd);
}
```

最后，我们可以用一个测试案例来验证这一点:

```java
@Test
public void testLCM() {
    BigInteger number1 = new BigInteger("12");
    BigInteger number2 = new BigInteger("18");
    BigInteger expectedLCM = new BigInteger("36");
    Assert.assertEquals(expectedLCM, BigIntegerLCM.lcm(number1, number2));
}
```

## 5.结论

在本教程中，我们讨论了在 Java 中寻找两个数的最小公倍数的各种方法。

此外，我们还了解了数的乘积与它们的 LCM 和 GCD 之间的关系。给定可以有效计算两个数的 GCD 的算法，我们也将 LCM 计算的问题简化为 GCD 计算的问题。

和往常一样，本文中使用的 Java 实现的完整源代码可以在 [GitHub](https://web.archive.org/web/20221126225111/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-2) 上获得。