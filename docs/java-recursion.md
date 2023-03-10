# Java 中的递归

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-recursion>

## 1。简介

在本文中，我们将关注任何编程语言中的一个核心概念——递归。

我们将解释`recursive function` 的特征，并展示如何使用递归来解决 Java 中的各种问题。

## 2。理解递归

### 2.1。定义

在 Java 中，函数调用机制支持**方法调用本身的可能性**。这种功能被称为`recursion` `.`

例如，假设我们想对从 0 到某个值`n`的整数求和:

```java
public int sum(int n) {
    if (n >= 1) {
        return sum(n - 1) + n;
    }
    return n;
}
```

递归函数有两个主要要求:

*   **停止条件**–函数在满足某个条件时返回值，无需进一步递归调用
*   **递归调用**–该函数用一个 `input`调用自己，这是一个更接近停止条件的步骤

每次递归调用都会向 JVM 的堆栈内存中添加一个新的帧。因此，如果我们不注意递归调用的深度，可能会出现内存不足的异常。

这个潜在的问题可以通过利用尾部递归优化来避免。

### 2.2。尾部递归对头部递归

当递归调用是函数执行的最后一件事时，我们称递归函数为`tail-recursion` **。**否则称为`head-recursion`。

我们上面的`sum()`函数的实现是头递归的一个例子，可以改成尾递归:

```java
public int tailSum(int currentSum, int n) {
    if (n <= 1) {
        return currentSum + n;
    }
    return tailSum(currentSum + n, n - 1);
}
```

使用尾部递归，递归调用是方法做的最后一件事，所以在当前函数中没有什么要执行的。

因此，逻辑上不需要存储当前函数的堆栈帧。

尽管编译器`can`利用这一点来优化内存，但应该注意的是 **Java 编译器目前还没有针对尾递归进行优化**。

### 2.3。递归与迭代

递归可以使代码更清晰、可读性更强，从而有助于简化一些复杂问题的实现。

但是正如我们已经看到的，递归方法通常需要更多的内存，因为每次递归调用所需的堆栈内存都会增加。

作为替代，如果我们可以用递归来解决问题，我们也可以用迭代来解决。

例如，我们的`sum`方法可以使用迭代来实现:

```java
public int iterativeSum(int n) {
    int sum = 0;
    if(n < 0) {
        return -1;
    }
    for(int i=0; i<=n; i++) {
        sum += i;
    }
    return sum;
}
```

与递归相比，迭代方法可能提供更好的性能。也就是说，与递归相比，迭代将更加复杂和难以理解，例如:遍历二叉树。

在头递归、尾递归和迭代方法之间做出正确的选择都取决于具体的问题和情况。

## 3。示例

现在，让我们试着用递归的方式解决一些问题。

### 3.1。求十的 N 次方

假设我们需要计算 10 的`n`次方。这里我们的输入是`n.`用递归的方式思考，我们可以先计算`(n-1)`的 10 次方，再把结果乘以 10。

然后，计算 10 的`(n-1)`次方将是 10 的`(n-2)`次方，并将结果乘以 10，以此类推。我们将继续这样，直到我们需要计算 10 的(n-n)次方，也就是 1。

如果我们想用 Java 实现它，我们应该写:

```java
public int powerOf10(int n) {
    if (n == 0) {
        return 1;
    }
    return powerOf10(n-1) * 10;
}
```

### 3.2。寻找斐波那契数列的第 N 个元素

从`0`和`1`、**、`Fibonacci Sequence`开始，是一个数字序列，其中每个数字被定义为它前面的两个数字之和** : `0 1 1 2 3 5 8 13 21 34 55` …

所以，给定一个数字`n`，我们的问题是找到`Fibonacci Sequence`的第`n`个元素。**为了实现递归解决方案，我们需要找出`Stop Condition` 和** `**Recursive Call**.`

幸运的是，这真的很简单。

让我们称`f(n)`为序列的第`n`个值。然后我们会有`f(n) = f(n-1) + f(n-2)`(T3)`.`

同时，`f(0) = 0` 和`f(1) = 1` ( `Stop Condition)`)。

然后，很明显我们需要定义一个递归方法来解决这个问题:

```java
public int fibonacci(int n) {
    if (n <= 1) {
        return n;
    }
    return fibonacci(n-1) + fibonacci(n-2);
}
```

### 3.3。从十进制转换为二进制

现在，让我们考虑将十进制数转换成二进制数的问题。要求是实现一个接收正整数值`n`并返回二进制表示的方法。

将十进制数转换为二进制数的一种方法是将该值除以 2，记录余数，然后继续将商除以 2。

我们继续这样除法，直到得到一个`0`的商。然后，通过按保留顺序写出所有的余数，我们得到二进制字符串。

因此，我们的问题是编写一个方法，以保留顺序返回这些余数:

```java
public String toBinary(int n) {
    if (n <= 1 ) {
        return String.valueOf(n);
    }
    return toBinary(n / 2) + String.valueOf(n % 2);
}
```

### 3.4。二叉树的高度

二叉树的高度被定义为从根到最深叶子的边数。我们现在的问题是计算给定二叉树的这个值。

一个简单的方法是找到最深的叶子，然后计算根和叶子之间的边数。

但是尝试考虑一个递归的解决方案，我们可以重新定义二叉树的高度为根的左分支和根的右分支的最大高度，加上`1`。

如果根没有左分支和右分支，则其高度为`zero`。

下面是我们的实现:

```java
public int calculateTreeHeight(BinaryNode root){
    if (root!= null) {
        if (root.getLeft() != null || root.getRight() != null) {
            return 1 + 
              max(calculateTreeHeight(root.left), 
                calculateTreeHeight(root.right));
        }
    }
    return 0;
}
```

因此，我们看到有些问题可以用递归以非常简单的方式解决。

## 4。结论

在本教程中，我们介绍了 Java 中递归的概念，并用几个简单的例子进行了演示。

这篇文章的实现可以在 Github 上找到[。](https://web.archive.org/web/20220523134007/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)