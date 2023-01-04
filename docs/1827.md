# Java 中的斐波那契数列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-fibonacci>

## 1.概观

在本教程中，我们将看看斐波那契数列。

具体来说，我们将实现三种方法来计算斐波纳契数列的`n^(th)`项，最后一种是常数时间解决方案。

## 2.斐波纳契级数

**斐波那契数列是一个数列，其中每一项都是前两项之和**。它的前两个术语是`0`和`1`。

例如，系列的前 11 个术语是`0, 1, 1, 2, 3, 5, 8, 13, 21, 34,`和 `55`。

在数学术语中，斐波纳契数列的序列`S[n]`由递归关系定义:

```
S(n) = S(n-1) + S(n-2), with S(0) = 0 and S(1) = 1
```

现在，我们来看看如何计算斐波那契数列的`n^(th)`项。我们将关注的三种方法是递归、迭代和使用比奈公式。

### 2.1.递归方法

对于我们的第一个解决方案，让我们简单地用 Java 直接表达递归关系:

```
public static int nthFibonacciTerm(int n) {
    if (n == 1 || n == 0) {
        return n;
    }
    return nthFibonacciTerm(n-1) + nthFibonacciTerm(n-2);
}
```

正如我们所看到的，我们检查`n`是否等于`0`或`1.`，如果为真，那么我们返回那个值。在任何其他情况下，我们递归调用函数来计算`(n-1)^(th)`项和`(n-2)^(th)`项，并返回它们的和。

虽然递归方法实现起来很简单，但是我们看到这种方法做了大量的重复计算。例如，为了计算`6th`项，我们调用来计算`5th`和`4th`项。此外，计算`5th`项的调用再次调用计算`4th`项。**因为这个事实，[递归方法](/web/20221208143832/https://www.baeldung.com/java-recursion)做了很多多余的工作。**

事实证明，这使得**的[时间复杂度](/web/20221208143832/https://www.baeldung.com/cs/fibonacci-computational-complexity)呈指数增长；确切地说。**

### 2.2.迭代法

在迭代法中，我们可以避免递归方法中的重复计算。相反，我们计算级数的项，[存储前两项来计算下一项](/web/20221208143832/https://www.baeldung.com/java-knapsack#dp)。

让我们来看看它的实现:

```
public static int nthFibonacciTerm(int n) {
    if(n == 0 || n == 1) {
        return n;
    }
    int n0 = 0, n1 = 1;
    int tempNthTerm;
    for (int i = 2; i <= n; i++) {
        tempNthTerm = n0 + n1;
        n0 = n1;
        n1 = tempNthTerm;
    }
    return n1;
}
```

首先，我们检查要计算的项是`0^(th)`项还是`1^(st)`项。如果是这种情况，我们返回初始值。否则，我们使用`n0`和`n1`计算`2^(nd)`项。然后，我们修改`n0`和`n1`变量的值来分别存储`1^(st)`项和`2^(nd)`项。我们不断迭代，直到计算出所需的项。

迭代法通过将最后两个斐波那契项存储在变量中来避免重复工作。**迭代法的时间复杂度和空间复杂度分别为`O(n)` 和`O(1)` 。**

### 2.3.比奈公式

我们只根据前面的两个数字定义了`n^(th)`斐波纳契数。现在，我们将看看比奈公式，计算常数时间内的`n^(th)`斐波纳契数。

**斐波纳契项保持一个比率，称为** `golden ratio` **，由希腊字母φ**`**,**`表示，读作‘phi’**。**

首先，让我们看看`golden ratio`是如何计算的:

```
Φ = ( 1 + √5 )/2 = 1.6180339887...
```

现在，我们来看看`Binet's formula`:

```
Sn = Φⁿ–(– Φ⁻ⁿ)/√5
```

实际上，这意味着**我们应该能够通过一些算术运算得到`n^(th)`斐波那契数。**

让我们用 Java 来表达这个:

```
public static int nthFibonacciTerm(int n) {
    double squareRootOf5 = Math.sqrt(5);
    double phi = (1 + squareRootOf5)/2;
    int nthTerm = (int) ((Math.pow(phi, n) - Math.pow(-phi, -n))/squareRootOf5);
    return nthTerm;
}
```

我们首先计算出`squareRootof5`和`phi`并存储在变量中。随后，我们应用比奈公式得到所需的项。

因为我们在这里处理的是无理数，所以我们只能得到一个近似值。因此，我们需要为更高的斐波那契数保留更多的小数位数，以解决舍入误差。

**我们看到上面的方法计算的是常数时间内的`n^(th)`斐波那契项，或者说`O(1).`**

## 3.结论

在这篇简短的文章中，我们研究了斐波那契数列。我们看了一个递归和迭代的解决方案。然后，我们应用比奈公式来创建一个常数时间解决方案。

和往常一样，GitHub 上的[提供了工作示例的完整源代码。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)