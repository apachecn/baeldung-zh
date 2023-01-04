# 在 Java 中寻找最大公约数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-greatest-common-divisor>

## 1.概观

在数学中，两个不为零的整数的 [GCD](https://web.archive.org/web/20221127165209/https://en.wikipedia.org/wiki/Greatest_common_divisor) 是**将每个整数均分的最大正整数。**

在本教程中，我们将研究三种方法来寻找两个整数的最大公约数(GCD)。此外，我们将看看它们在 Java 中的实现。

## 2.蛮力

对于我们的第一种方法，我们从 1 迭代到给定的最小数，并检查给定的整数是否能被索引整除。**分割给定数字**的最大指标是给定数字的 GCD:

```
int gcdByBruteForce(int n1, int n2) {
    int gcd = 1;
    for (int i = 1; i <= n1 && i <= n2; i++) {
        if (n1 % i == 0 && n2 % i == 0) {
            gcd = i;
        }
    }
    return gcd;
}
```

正如我们所看到的，上面实现的复杂度是`O(min(n1, n2))`，因为我们需要在循环上迭代`n`次(相当于较小的数)来找到 GCD。

## 3.欧几里德算法

第二，我们可以使用欧几里德的算法来寻找 GCD。欧几里德的算法不仅高效，而且易于理解，易于用 Java 递归实现。

欧几里德的方法依赖于两个重要的定理:

*   首先，如果我们从较大的数字中减去较小的数字，GCD 不会改变—**因此，如果我们继续减去这个数字，我们最终会得到他们的 GCD**
*   第二，当较小的数恰好除以较大的数时，较小的数就是两个给定数的 GCD。

请注意，在我们的实现中，我们将使用模而不是减法，因为它基本上是一次多次减法:

```
int gcdByEuclidsAlgorithm(int n1, int n2) {
    if (n2 == 0) {
        return n1;
    }
    return gcdByEuclidsAlgorithm(n2, n1 % n2);
}
```

另外，请注意我们如何在`.`算法的递归步骤中，在`n1`的位置使用`n2` 并在 n2 的位置使用余数

此外，**欧几里德算法的[复杂度是`O(Log min(n1, n2))`，这比我们之前看到的蛮力方法要好。](/web/20221127165209/https://www.baeldung.com/cs/euclid-time-complexity)**

## 4.Stein 算法或二进制 GCD 算法

最后，我们可以使用 Stein 的算法，也称为二进制 GCD 算法，来找到两个非负整数的 GCD。该算法使用简单的算术运算，如算术移位、比较和减法。

Stein 的算法反复应用以下与 GCD 相关的基本恒等式来求两个非负整数的 GCD:

1.  `gcd(0, 0) = 0, gcd(n1, 0) = n1, gcd(0, n2) = n2`
2.  当`n1`和`n2`都是偶数时，那么`gcd(n1, n2) = 2 * gcd(n1/2, n2/2)`，因为 2 是公约数
3.  如果`n1`是偶数而`n2`是奇数，那么`gcd(n1, n2) = gcd(n1/2, n2)`，因为 2 不是公约数，反之亦然
4.  如果`n1`和`n2`都是奇整数，并且`n1 >= n2`，那么`gcd(n1, n2) = gcd((n1-n2)/2, n2)`，反之亦然

我们重复步骤 2-4，直到`n1`等于`n2`，或`n1 = 0`。GCD 是`(2^n) * n2`。这里，`n`是在执行步骤 2 时在`n1`和`n2`中发现 2 的次数:

```
int gcdBySteinsAlgorithm(int n1, int n2) {
    if (n1 == 0) {
        return n2;
    }

    if (n2 == 0) {
        return n1;
    }

    int n;
    for (n = 0; ((n1 | n2) & 1) == 0; n++) {
        n1 >>= 1;
        n2 >>= 1;
    }

    while ((n1 & 1) == 0) {
        n1 >>= 1;
    }

    do {
        while ((n2 & 1) == 0) {
            n2 >>= 1;
        }

        if (n1 > n2) {
            int temp = n1;
            n1 = n2;
            n2 = temp;
        }
        n2 = (n2 - n1);
    } while (n2 != 0);
    return n1 << n;
}
```

我们可以看到，为了除以或乘以 2，我们使用了算术移位运算。此外，我们使用减法来减少给定的数字。

**当`n1 > n2`为`O((log[2]n1)²)` 时 Stein 算法的复杂度反之。当 `n1 < n2,` 是`O((log[2]n2)²).`**

## 5.结论

在本教程中，我们研究了计算两个数的 GCD 的各种方法。我们还用 Java 实现了这些，并快速了解了它们的复杂性。

和往常一样，我们这里的例子的完整源代码，和往常一样，在 GitHub 上的[。](https://web.archive.org/web/20221127165209/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)