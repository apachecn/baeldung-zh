# Java 中小于给定数字的 2 的最大幂

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-largest-power-of-2-less-than-number>

## 1.概观

在这篇文章中，我们将看到如何找到小于给定数字的 2 的最大幂。

对于我们的例子，我们将采用样本输入 9。2⁰ 是 1，我们能找到的比给定输入小 2 的幂的最小有效输入是 2。因此，我们只将大于 1 的输入视为有效。

## 2.天真的方法

让我们从 2⁰ 开始，也就是 1，我们将**继续将这个数字乘以 2，直到我们找到一个小于输入**的数字:

```java
public long findLargestPowerOf2LessThanTheGivenNumber(long input) {
    Assert.isTrue(input > 1, "Invalid input");
    long firstPowerOf2 = 1;
    long nextPowerOf2 = 2;
    while (nextPowerOf2 < input) {
        firstPowerOf2 = nextPowerOf2;
        nextPowerOf2 = nextPowerOf2 * 2;
    }
    return firstPowerOf2;
}
```

让我们来理解一下样本`input` = 9 的代码。

`firstPowerOf2`的初始值为 1，而`nextPowerOf2`的初始值为 2。正如我们所见，2 < 9 为真，我们进入了 while 循环。

第一次迭代，`firstPowerOf2`为 2，`nextPowerOf2`为 2 * 2 = 4。再次 4 < 9 让我们继续 while 循环。

对于第二次迭代，`firstPowerOf2`是 4，`nextPowerOf2`是 4 * 2 = 8。现在 8 < 9、让我们继续前进。

第三次迭代，`firstPowerOf2`为 8，`nextPowerOf2`为 8 * 2 = 16。while 条件 16 < 9 为假，所以它跳出了 while 循环。8 是小于 9 的 2 的最大幂。

让我们运行一些测试来验证我们的代码:

```java
assertEquals(8, findPowerOf2LessThanTheGivenNumber(9));
assertEquals(16, findPowerOf2LessThanTheGivenNumber(32)); 
```

我们解决方案的时间复杂度是 O(log[2] (N)) 。在我们的例子中，我们迭代了 log[2] (9) = 3 次。

## 3.使用`Math.log`

以 2 为底的对数将给出我们可以递归地将一个数除以 2 的次数换句话说，一个数的对数给出了 2 的幂。让我们看一些例子来理解这一点。

log(2, 8) = 3，log(2, 16) = 4。一般情况下，我们可以看到 y = log(2, x) 其中 x = 2^y 。

因此，如果我们找到一个能被 2 整除的数，我们就从中减去 1，这样就避免了这个数是 2 的完美幂的情况。

[`Math.log`](/web/20221208143856/https://www.baeldung.com/java-lang-math) 是日志[10] 。为了计算 log[2] (x)，我们可以使用公式 log[2](x)= log[10](x)/log[10](2)

让我们把它写成代码:

```java
public long findLargestPowerOf2LessThanTheGivenNumberUsingLogBase2(long input) {
    Assert.isTrue(input > 1, "Invalid input");
    long temp = input;
    if (input % 2 == 0) {
        temp = input - 1;
    }
    long power = (long) (Math.log(temp) / Math.log(2));
    long result = (long) Math.pow(2, power);
    return result;
}
```

假设我们的样本输入为 9，`temp`的初始值为 9。

9 % 2 是 1，所以我们的`temp`变量是 9。这里我们用的是模除法，它会给出 9/2 的余数。

为了找到 log[2] (9)，我们做 log[10](9)/log[10](2)= 0.95424/0.30103 ~ = 3。

现在，`result`是 2 ³ 也就是 8。

让我们验证我们的代码:

```java
assertEquals(8, findLargestPowerOf2LessThanTheGivenNumberUsingLogBase2(9));
assertEquals(16, findLargestPowerOf2LessThanTheGivenNumberUsingLogBase2(32));
```

实际上，`Math.pow`将会进行我们在方法 1 中所做的同样的迭代。因此我们可以说，对于这个解决方案，**的时间复杂度是 O(Log[2] (N))** 。

## 4.逐位技术

对于这种方法，我们将使用按位移位技术。首先，让我们看看 2 的幂的二进制表示，考虑到我们有 4 位来表示这个数

| 2⁰ | one | `0001` |
| 2¹ | Two | `0010` |
| 2² | four | `0100` |
| 2³ | eight | `1000` |

仔细观察，我们可以发现我们可以通过左移 1 的字节来计算 2 的幂。即。2 ² 是将 1 的字节左移 2 位，以此类推。

让我们用位移法编码:

```java
public long findLargestPowerOf2LessThanTheGivenNumberUsingBitShiftApproach(long input) {
    Assert.isTrue(input > 1, "Invalid input");
    long result = 1;
    long powerOf2;
    for (long i = 0; i < Long.BYTES * 8; i++) {
        powerOf2 = 1 << i;
        if (powerOf2 >= input) {
            break;
        }
        result = powerOf2;
    }
    return result;
}
```

在上面的代码中，我们使用`long` 作为我们的数据类型，它使用 8 个字节或 64 位。所以我们将计算 2 的 2 次方到 2 的 64 次方。我们使用移位运算符`<<` 来计算 2 的幂。对于我们的样本输入 9，在第 4 次^(迭代)之后，`powerOf2` = 16 和`result` = 8 的值，在这里我们跳出循环作为 16 > 9 的`input`。

让我们检查一下我们的代码是否如预期的那样工作:

```java
assertEquals(8, findLargestPowerOf2LessThanTheGivenNumberUsingBitShiftApproach(9));
assertEquals(16, findLargestPowerOf2LessThanTheGivenNumberUsingBitShiftApproach(32));
```

这种方法的**最坏情况时间复杂度也是 O(log[2] (N))** ，类似于我们看到的第一种方法。然而，这种方法更好，因为与乘法运算相比，**位移操作更有效。**

## 5.按位`AND`

对于我们的下一个方法，我们将使用这个公式 **2 ^n 和 2 ^n -1 = 0** 。

让我们看一些例子来理解它是如何工作的。

4 的二进制表示是`0100,`，3 是`0011`。

让我们对这两个数做一个[按位 AND 运算](/web/20221208143856/https://www.baeldung.com/java-bitwise-operators)。`0100`和`0011`是`0000`。对于 2 的任何次方以及比它小的数，我们也可以这样说。我们取 16 (2 ⁴ )和 15 分别表示为`1000`、`0111`。同样，我们看到这两个位的 AND 结果为 0。我们也可以说，除了这两个数字之外，对任何其他数字的 AND 运算都不会得到 0。

让我们看看使用按位 AND 解决这个问题的代码:

```java
public long findLargestPowerOf2LessThanTheGivenNumberUsingBitwiseAnd(long input) { 
    Assert.isTrue(input > 1, "Invalid input");
    long result = 1;
    for (long i = input - 1; i > 1; i--) {
        if ((i & (i - 1)) == 0) {
            result = i;
            break;
        }
    }
    return result;
}
```

在上面的代码中，我们对小于输入的数字进行循环。每当我们发现一个数的按位“与”且数-1 为零时，我们就跳出了这个循环，因为我们知道这个数将是 2 的幂。在这种情况下，对于我们的样本`input` 9，当`i` = 8 且`i`–1 = 7 时，我们脱离循环。

现在，让我们验证几个场景:

```java
assertEquals(8, findLargestPowerOf2LessThanTheGivenNumberUsingBitwiseAnd(9));
assertEquals(16, findLargestPowerOf2LessThanTheGivenNumberUsingBitwiseAnd(32));
```

当输入是 2 的精确幂时，该方法的最坏情况**时间复杂度是 O(N/2)** 。正如我们所看到的，这不是最有效的解决方案，但是了解这种技术是有好处的，因为在处理类似的问题时它会派上用场。

## 6.结论

我们已经看到了寻找小于给定数字的 2 的最大幂的不同方法。我们还注意到在某些情况下，位运算可以简化计算。

本文单元测试的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-2)