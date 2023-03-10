# 在 Java 中生成素数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generate-prime-numbers>

## 1。简介

在本教程中，我们将展示使用 Java 生成素数的各种方法。

如果你想检查一个数字是否是质数，这里有一个如何检查的快速指南。

## 2。质数

先说核心定义。质数是一个大于一的自然数，它除了一和它自己之外没有其他的正除数。

例如，7 是质数，因为 1 和 7 是它仅有的正整数因子，而 12 不是，因为除了 1、4 和 6 之外，它还有约数 3 和 2。

## 3。生成素数

在这一节中，我们将看到如何有效地生成低于给定值的质数。

### 3.1。Java 7 及之前版本——暴力破解

```java
public static List<Integer> primeNumbersBruteForce(int n) {
    List<Integer> primeNumbers = new LinkedList<>();
    for (int i = 2; i <= n; i++) {
        if (isPrimeBruteForce(i)) {
            primeNumbers.add(i);
        }
    }
    return primeNumbers;
}
public static boolean isPrimeBruteForce(int number) {
    for (int i = 2; i < number; i++) {
        if (number % i == 0) {
            return false;
        }
    }
    return true;
} 
```

如您所见，`primeNumbersBruteForce` 正在从 2 到`n`迭代数字，并简单地调用`isPrimeBruteForce()`方法来检查一个数字是否是质数。

该方法通过从 2 到`number-1`范围内的数来检查每个数的整除性。

如果我们在任何一点遇到一个可整除的数，我们返回 false。最后，当我们发现一个数不能被它之前的任何数整除时，我们返回 true，表明它是一个质数。

### 3.2。效率和优化

先前的算法不是线性的，并且具有 O(n^2).的时间复杂度这个算法也不是很有效，显然还有改进的空间。

让我们看看`isPrimeBruteForce()`方法中的条件。

当一个数不是质数时，这个数可以分解成两个因子，即`a`和`b`，即`number` = a * b. **如果`a`和`b`都大于`n`的平方根，`a*b`就会大于`n`。**

因此，这些因子中至少有一个必须小于或等于一个数的平方根，为了检查一个数是否是质数，我们只需要测试小于或等于被检查数的平方根的因子。

此外，质数永远不可能是偶数，因为偶数都可以被 2 整除。

记住上面的想法，让我们改进算法:

```java
public static List<Integer> primeNumbersBruteForce(int n) {
    List<Integer> primeNumbers = new LinkedList<>();
    if (n >= 2) {
        primeNumbers.add(2);
    }
    for (int i = 3; i <= n; i += 2) {
        if (isPrimeBruteForce(i)) {
            primeNumbers.add(i);
        }
    }
    return primeNumbers;
}
private static boolean isPrimeBruteForce(int number) {
    for (int i = 2; i*i <= number; i++) {
        if (number % i == 0) {
            return false;
        }
    }
    return true;
} 
```

### 3.3。使用 Java 8

让我们看看如何使用 Java 8 的习惯用法重写前面的解决方案:

```java
public static List<Integer> primeNumbersTill(int n) {
    return IntStream.rangeClosed(2, n)
      .filter(x -> isPrime(x)).boxed()
      .collect(Collectors.toList());
}
private static boolean isPrime(int number) {
    return IntStream.rangeClosed(2, (int) (Math.sqrt(number)))
      .allMatch(n -> x % n != 0);
} 
```

### 3.4。使用厄拉多塞的筛子

还有另一种有效的方法可以帮助我们有效地生成素数，它被称为厄拉多塞筛。其时间效率为 O(n logn)。

让我们来看看这个算法的步骤:

1.  创建一个从 2 到`n` : (2，3，4，…，n)的连续整数列表
2.  最初，让`p`等于 2，第一个质数
3.  从`p`开始，以`p`为增量向上计数，并在列表中标记每个大于`p`本身的数字。这些数字将是 2p、3p、4p 等。；请注意，其中一些可能已经被标记
4.  在列表中找到第一个没有标记的大于`p`的数字。如果没有这个号码，请停止。否则，现在让`p`等于这个数(它是下一个质数)，并从步骤 3 开始重复

当算法结束时，列表中所有没有标记的数字都是素数。

代码如下所示:

```java
public static List<Integer> sieveOfEratosthenes(int n) {
    boolean prime[] = new boolean[n + 1];
    Arrays.fill(prime, true);
    for (int p = 2; p * p <= n; p++) {
        if (prime[p]) {
            for (int i = p * 2; i <= n; i += p) {
                prime[i] = false;
            }
        }
    }
    List<Integer> primeNumbers = new LinkedList<>();
    for (int i = 2; i <= n; i++) {
        if (prime[i]) {
            primeNumbers.add(i);
        }
    }
    return primeNumbers;
} 
```

### 3.5。厄拉多塞筛网工作示例

让我们来看看对于 n=30 它是如何工作的。

[![](img/2ed1c3c5cbed4ef687c3b8d7c517c96c.png)](/web/20220523230624/https://www.baeldung.com/wp-content/uploads/2017/11/Primes.jpg)

考虑上面的图像，下面是算法执行的过程:

1.  循环从 2 开始，所以我们不标记 2，而标记 2 的所有除数。它在图像中用红色标出
2.  循环移动到 3，所以我们不标记 3，标记 3 的所有尚未标记的除数。它在图像中用绿色标出
3.  循环移动到 4，它已经被标记了，所以我们继续
4.  循环移动到 5，所以我们不标记 5，标记 5 的所有尚未标记的除数。它在图像中用紫色标记
5.  我们继续上述步骤，直到循环达到等于`n`的平方根

## 4。结论

在这个快速教程中，我们举例说明了生成质数直到“N”值的方法。

这些例子的实现可以在 GitHub 的[中找到。](https://web.archive.org/web/20220523230624/https://github.com/eugenp/tutorials/tree/master/java-numbers-2 "Spring XML injection examples on Github")