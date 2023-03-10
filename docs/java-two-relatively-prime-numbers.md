# 在 Java 中找出两个数是否互质

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-two-relatively-prime-numbers>

## 1.概观

给定两个整数，`a`和`b`，如果除这两个整数的唯一因子是 1，我们说它们是**互素的。互质或互质是相对质数的同义词。**

在这个快速教程中，我们将使用 Java 完成这个问题的解决方案。

## 2.最大公因数算法

事实证明，如果 2 个数`a`和`b`的最大公约数(`gcd`)为 1(即`gcd(a, b) = 1`，那么`a`和`b`互质。因此，确定两个数字是否互质只需简单地判断`gcd`是否为 1。

## 3.欧几里德算法实现

在这一节中，我们将使用[欧几里德算法](https://web.archive.org/web/20220926201423/https://en.wikipedia.org/wiki/Euclidean_algorithm)来计算两个数的`gcd`。

在展示我们的实现之前，为了便于理解，让我们总结一下这个算法，并看一个如何应用它的快速示例。

所以，假设我们有两个整数，`a`和`b`。在迭代方法中，我们首先将`a`除以`b`，得到余数。接下来，我们给`a`分配`b`的值，给`b`分配余数。我们重复这个过程，直到`b` = `0`。最后，到了这一点，我们把`a`的值作为`gcd`结果返回，如果`a` = `1`，就可以说`a`和`b`互质。

让我们在两个整数上试一下，`a = 81 `和`b = 35`。

在这种情况下，`81 `和`35 (81 % 35) `的余数就是`11`。因此，在第一个迭代步骤中，我们以`a = 35`和`b = 11`结束。因此，我们将进行另一次迭代。

`35`除以`11 `的余数是`2`。结果，我们现在有了`a = 11`(我们交换了值)和`b = 2`。我们继续吧。

再多一步就会产生`a = 2`和`b = 1`。现在，我们已经接近尾声了。

最后，在一次迭代之后，我们将到达`a = 1 `和`b = 0`。算法返回`1`，我们可以得出结论`81 `和`35`确实是互质的。

### 3.1.强制性实施

首先，让我们实现如上所述的欧几里德算法的强制 Java 版本:

```java
int iterativeGCD(int a, int b) {
    int tmp;
    while (b != 0) {
        if (a < b) {
            tmp = a;
            a = b;
            b = tmp;
        }
        tmp = b;
        b = a % b;
        a = tmp;
    }
    return a;
} 
```

正如我们可以注意到的，在`a `小于`b`的情况下，我们在继续之前交换值。当`b `为 0 时，算法停止。

### 3.2.递归实现

接下来，让我们看一个递归实现。这可能更干净，因为它避免了显式的变量值交换:

```java
int recursiveGCD(int a, int b) {
    if (b == 0) {
        return a;
    }
    if (a < b) {
        return recursiveGCD(b, a);
    }
    return recursiveGCD(b, a % b);
} 
```

## 4.使用`BigInteger`的实现

但是等等——`gcd`算法不是已经用 Java 实现了吗？是的，它是！`BigInteger`类提供了一个`gcd`方法，该方法实现了寻找最大公约数的欧几里德算法。

使用这种方法，我们可以更容易地将相对质数的算法写成:

```java
boolean bigIntegerRelativelyPrime(int a, int b) {
    return BigInteger.valueOf(a).gcd(BigInteger.valueOf(b)).equals(BigInteger.ONE);
} 
```

## 5.结论

在这个快速教程中，我们提出了一个解决方法，使用三个`gcd`算法的实现来发现两个数字是否互质。

和往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220926201423/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-5)