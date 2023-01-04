# 用 Java 列出一个数的所有因子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-factors-integer>

## 1.概观

在本教程中，我们将编写一个 Java 程序来寻找一个给定整数的所有因子。

## 2.问题简介

在开始编写 Java 代码之前，让我们先了解一下整数的因子是什么。

**给定一个整数`n`，如果这个整数`i`能整除这个数`i`，那么这个整数`i`就是`n`的因子。**在这里，完全可除意味着当我们用 *i 和*除`n`时，余数为零。

几个例子可以很快解释这一点:

*   `n = 10`，其因素: `1, 2, 5,` 和`10`
*   `n = 13`，其因素:`1` 和 `13`
*   `n = 1`、`n`只有一个因子:`1`
*   `n = 0`，零没有因子

如例所示，通常情况下，一个整数`n`的因子总是包含 *1* 和`n`，即使`n`是质数，例如`13`。但是，**零是一个特殊的整数。它没有因素。**

现在我们已经理解了因子的概念，让我们创建一个 Java 程序来寻找给定整数的所有因子。

为了简单起见，我们将使用单元测试断言来验证我们的解决方案是否如预期的那样工作。

## 3.创建一个方法来寻找一个整数的所有因子

寻找一个整数`n`的所有因子的最直接的方法是通过**从 1 到`n`循环，并测试哪个数可以完全除`n`** 。我们可以把那些能把`n`整除的数存储在一个`Set`里。当循环结束时，该`Set`将保持`n`的所有因子。

用 Java 实现这个想法对我们来说并不困难:

```java
static Set<Integer> getAllFactorsVer1(int n) {
    Set<Integer> factors = new HashSet<>();
    for (int i = 1; i <= n; i++) {
        if (n % i == 0) {
            factors.add(i);
        }
    }
    return factors;
} 
```

接下来，让我们编写一些测试来检查我们的方法是否如预期的那样工作。首先，让我们创建一个`Map`来准备一些要测试的数字及其预期因素:

```java
final static Map<Integer, Set<Integer>> FACTOR_MAP = ImmutableMap.of(
    0, ImmutableSet.of(),
    1, ImmutableSet.of(1),
    20, ImmutableSet.of(1, 2, 4, 5, 10, 20),
    24, ImmutableSet.of(1, 2, 3, 4, 6, 8, 12, 24),
    97, ImmutableSet.of(1, 97),
    99, ImmutableSet.of(1, 3, 9, 11, 33, 99),
    100, ImmutableSet.of(1, 2, 4, 5, 10, 20, 25, 50, 100)
); 
```

现在，对于上面的`FACTOR_MAP`中的每个数字，我们调用已经实现的`getAllFactorsVer1()`方法，看看它是否能找到想要的因子:

```java
FACTOR_MAP.forEach((number, expected) -> assertEquals(expected, FactorsOfInteger.getAllFactorsVer1(number)));
```

如果我们运行它，测试就通过了。所以，方法解决了问题，太好了！

明眼人可能会发现我们用`Ver1. `来命名这个方法。通常，这意味着我们将在教程中引入不同的版本。换句话说，该解决方案仍有改进的空间。

接下来，让我们看看如何优化版本 1 的实现。

## 4.优化–版本 2

让我们回顾一下该方法中的主要逻辑:

```java
for (int i = 1; i <= n; i++) {
   if (n % i == 0) {
       factors.add(i);
   }
}
```

如上面的代码所示，我们将执行`n % i`次计算`n`。现在，如果我们检查一个整数的因子，我们会看到**因子总是成对出现**。让我们以`n =100`为例来了解这一因素特征:

```java
 1    2    4    5    10    20    25    50    100
   │    │    │    │    |      │     │     │     │
   │    │    │    │  [10,10]  │     │     │     │
   │    │    │    │           │     │     │     │
   │    │    │    └──[5, 20] ─┘     │     │     │
   │    │    │                      │     │     │
   │    │    └───────[4, 25]────────┘     │     │
   │    │                                 │     │
   │    └────────────[2, 50]──────────────┘     │
   │                                            │
   └─────────────────[1, 100]───────────────────┘ 
```

正如我们所见，`100`的所有因素都是成对的。因此，**如果我们找到了`n`的一个因子`i` ，我们就可以得到配对的一个`i'= n/i`** 。也就是说，我们不需要循环`n`次。相反，**我们从 1 开始检查数字`n`的平方根，并找到所有的`i`和`i'`对。**这样，给定`n=100`，我们只循环十次。

接下来，让我们优化我们的版本 1 方法:

```java
static Set<Integer> getAllFactorsVer2(int n) {
    Set<Integer> factors = new HashSet<>();
    for (int i = 1; i <= Math.sqrt(n); i++) {
        if (n % i == 0) {
            factors.add(i);
            factors.add(n / i);
        }
    }
    return factors;
} 
```

如上面的代码所示，我们使用了 Java 标准库中的`Math.sqrt()`方法来让[计算`n`](/web/20221021151454/https://www.baeldung.com/java-find-if-square-root-is-integer) 的平方根。

现在，让我们用相同的测试数据测试第二个版本的实现:

```java
FACTOR_MAP.forEach((number, expected) -> assertEquals(expected, FactorsOfInteger.getAllFactorsVer2(number)));
```

如果我们进行测试，就会通过。所以优化后的版本 2 如预期的那样工作。

我们已经成功地将因子确定时间从`n`减少到`n`的平方根。这是一个显著的进步。但是，仍有进一步优化的空间。所以，接下来，我们来进一步分析一下。

## 5.进一步优化–版本 3

首先，我们来做一些简单的数学分析。

我们知道，给定的整数`n`可以是偶数，也可以是奇数。**如果`n`是一个偶数，我们不能断定它的因子是偶数还是奇数。**比如 20 的因子是 1，2，4，5，10，20。所以有偶数也有奇数。

但是，**如果`n`是奇数，那么它的所有因子也一定是奇数**。例如，99 的因子是 1、3、9、11、33 和 99。所以，都是奇数。

因此，我们可以根据`n`是否为奇数来调整循环的增量步长。**当我们的循环从 `i = 1`开始时，如果给我们一个奇数，我们可以设置增量`step = 2`来跳过对所有偶数的检查。**

接下来，让我们在版本 3 中实现这个想法:

```java
static Set<Integer> getAllFactorsVer3(int n) {
    Set<Integer> factors = new HashSet<>();
    int step = n % 2 == 0 ? 1 : 2;
    for (int i = 1; i <= Math.sqrt(n); i += step) {
        if (n % i == 0) {
            factors.add(i);
            factors.add(n / i);
        }
    }
    return factors;
} 
```

通过这种优化，如果`n`是一个偶数，循环将执行`sqrt(n)`次，与版本 2 相同。

然而，**如果`n `是奇数，循环总共执行`sqrt(n)/2`次。**

最后，让我们测试我们的版本 3 解决方案:

```java
FACTOR_MAP.forEach((number, expected) -> assertEquals(expected, FactorsOfInteger.getAllFactorsVer3(number)));
```

如果我们试一试，测试就会通过。因此，它正确地完成了工作。

## 6.结论

在本文中，我们创建了一个 Java 方法来查找一个整数的所有因子。此外，我们已经讨论了对初始解决方案的两种优化。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。