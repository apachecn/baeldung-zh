# 大 O 符号的实用 Java 示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-algorithm-complexity>

## 1。概述

在本教程中，我们将讨论**大 O 符号的含义。我们将通过几个例子来研究它对代码运行时间的影响。**

## 2。大 O 符号的直觉

我们经常听到用[大 O 符号](/web/20220628094334/https://www.baeldung.com/cs/big-o-notation)描述算法的**性能。**

对算法性能(或算法复杂性)的研究属于[算法分析](https://web.archive.org/web/20220628094334/https://en.wikipedia.org/wiki/Analysis_of_algorithms)的领域。算法分析回答了算法消耗多少资源(如磁盘空间或时间)的问题。

我们将把时间视为一种资源。通常，算法完成所需的时间越少越好。

## 3。常数时间算法-`O(1)`

算法的这个输入大小如何影响它的运行时间？理解大 O 的关键是理解事物增长的速率。这里讨论的速率是每个输入尺寸花费的**时间。**

考虑这段简单的代码:

```java
int n = 1000;
System.out.println("Hey - your input is: " + n);
```

显然，上面的`n`是什么并不重要。这段代码需要持续运行一段时间。它不依赖于 n 的大小。

类似地:

```java
int n = 1000;
System.out.println("Hey - your input is: " + n);
System.out.println("Hmm.. I'm doing more stuff with: " + n);
System.out.println("And more: " + n);
```

上面的例子也是常数时间。即使它需要 3 倍的时间来运行，它`doesn't depend on the size of the input, n.`我们将常数时间算法表示如下:`O(1)`。请注意，`O(2)`、`O(3)`甚至`O(1000)`都是同样的意思。

我们不关心运行的确切时间，只关心它需要恒定的时间。

## 4。对数时间算法—`O(log n)`

常数时间算法是(渐近地)最快的。对数时间是第二快的时间。不幸的是，它们有点难以想象。

对数时间算法的一个常见例子是[二分搜索法](https://web.archive.org/web/20220628094334/https://en.wikipedia.org/wiki/Binary_search_algorithm)算法。要查看如何用 Java 实现二分搜索法，[点击这里。](/web/20220628094334/https://www.baeldung.com/java-binary-search)

这里重要的是，**运行时间与输入的对数成比例增长(在这种情况下，对数以 2 为底):**

```java
for (int i = 1; i < n; i = i * 2){
    System.out.println("Hey - I'm busy looking at: " + i);
}
```

如果`n`为 8，则输出如下:

```java
Hey - I'm busy looking at: 1
Hey - I'm busy looking at: 2
Hey - I'm busy looking at: 4
```

我们的简单算法运行 log(8) = 3 次。

## 5。线性时间算法—`O(n)`

在对数时间算法之后，我们得到下一个最快的类:**线性时间算法。**

如果我们说某样东西线性增长，我们的意思是它的增长与其投入的大小成正比。

想象一个简单的 for 循环:

```java
for (int i = 0; i < n; i++) {
    System.out.println("Hey - I'm busy looking at: " + i);
}
```

这个 for 循环运行了多少次？ `n`倍，当然！我们不知道这到底需要多长时间，我们也不担心。

我们所知道的是，上面介绍的简单算法将随着其输入的大小而线性增长。

我们更喜欢运行时间为`0.1n`而不是`(1000n + 1000)`，但两者仍然是线性算法；它们都与投入的规模成正比增长。

同样，如果算法更改为以下形式:

```java
for (int i = 0; i < n; i++) {
    System.out.println("Hey - I'm busy looking at: " + i);
    System.out.println("Hmm.. Let's have another look at: " + i);
    System.out.println("And another: " + i);
}
```

运行时的输入大小`n`仍然是线性的。我们将线性算法表示如下: `O(n)`。

和常数时间算法一样，我们不关心运行时的细节。 **`O(2n+1)`与 `O(n)`** 相同，因为大 O 符号与输入大小的增长有关。

## 6。N Log N 时间算法-`O(n log n)`

**`n log n`是下一类算法。**运行时间与输入的`n log n`成比例增长:

```java
for (int i = 1; i <= n; i++){
    for(int j = 1; j < n; j = j * 2) {
        System.out.println("Hey - I'm busy looking at: " + i + " and " + j);
    }
} 
```

例如，如果`n`是 8，那么这个算法将运行`8 * log(8) = 8 * 3 = 24`次。在 for 循环中，我们是否有严格的不等式，这与大 O 符号无关。

## 7。多项式时间算法—`O(n^p)`

接下来我们有多项式时间算法。这些算法甚至比`n log n`算法还慢。

多项式这个术语是一个通称，包含二次 `(n²)`、三次`(n³)`、四次 `(n⁴)`等。功能。**重要的是要知道 `O(n²)`比`O(n³)`快，而【】又比`O(n⁴)`等快。**

让我们来看一个二次时间算法的简单例子:

```java
for (int i = 1; i <= n; i++) {
    for(int j = 1; j <= n; j++) {
        System.out.println("Hey - I'm busy looking at: " + i + " and " + j);
    }
} 
```

该算法将运行`8² = 64`次。注意，如果我们要嵌套另一个 for 循环，这将成为一个`O(n³)`算法。

## 8。指数时间算法—`O(`k`^n)`

现在我们正进入一个危险的领域；这些算法与输入大小的某个指数因子成比例增长。

例如， **`O(2^n)`算法每增加一个输入就翻倍。**所以，如果`n = 2`，这些算法会运行四次；如果`n = 3`，它们将运行八次(有点像对数时间算法的对立面)。

**`O(3^n)`每增加一个输入，算法就增加三倍，`O(k^n)`每增加一个输入，算法就增加 k 倍。**

让我们来看一个简单的`O(2^n)`时间算法的例子:

```java
for (int i = 1; i <= Math.pow(2, n); i++){
    System.out.println("Hey - I'm busy looking at: " + i);
}
```

该算法将运行`2⁸ = 256`次。

## 9。阶乘时间算法—`O(n!)`

在大多数情况下，这是最糟糕的情况。这类算法的运行时间与输入大小的[阶乘](https://web.archive.org/web/20220628094334/https://en.wikipedia.org/wiki/Factorial)成比例。

这方面的一个经典例子是使用蛮力方法解决旅行推销员问题。

对旅行推销员问题的解决方案的解释超出了本文的范围。

相反，让我们来看一个简单的`O(n!)`算法，如前几节所述:

```java
for (int i = 1; i <= factorial(n); i++){
    System.out.println("Hey - I'm busy looking at: " + i);
}
```

其中`factorial(n)`简单计算 n！。如果 n 是 8，这个算法将运行`8! = 40320`次。

## 10。渐近函数

**大 O 就是所谓的`asymptotic function`。**所有这一切意味着，它关心的是算法`at the limit`的性能，也就是说，当大量的输入被抛向它时。

大 O 不关心你的算法对小尺寸的输入有多好。它涉及到大量的输入(想想排序一个有一百万个数字的列表和排序一个有五个数字的列表)。

另外要注意的是**还有其他渐近函数。**大θ(theta)和大ω(omega)也都描述了极限下的算法(记住，`the limit`这只是指巨大的输入)。

为了理解这 3 个重要函数之间的差异，我们首先需要知道大 O、大θ和大ω中的每一个都描述了一个`set`(即元素的集合)。

在这里，我们集合的成员就是算法本身:

*   大 O 描述了以一定速度运行的所有算法的集合(这是一个上限)
*   相反，大ω描述了运行速度超过某一速度的所有算法的集合(这是一个下限)
*   最后，大θ描述了以一定速度运行的所有算法的集合(就像等式一样)

我们上面给出的定义在数学上并不精确，但它们有助于我们的理解。

**通常情况下，你会听到用大 O** 描述的东西，但了解一下大θ和大ω也无妨。

## 11。结论

在本文中，我们讨论了大 O 符号，以及**理解算法的复杂性如何影响代码的运行时间。**

在这里可以找到不同复杂性类别[的可视化效果。](https://web.archive.org/web/20220628094334/http://bigocheatsheet.com/)

像往常一样，本教程的代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220628094334/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)