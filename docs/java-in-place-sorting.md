# 使用 Java 实现的就地排序算法指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-in-place-sorting>

## 1。简介

在本教程中，我们将解释就地排序算法是如何工作的。

## 2.原地算法

**就地算法是那些不需要任何辅助数据结构来转换输入数据的算法。**基本上，这意味着算法不会使用额外的空间来进行输入操作。它实际上用输出覆盖了输入。

然而，在现实中，该算法实际上可能需要用于辅助变量的小且不恒定的额外空间。这个空间的复杂性在大多数情况下是`O(log n)`，尽管有时任何非线性的东西都是允许的。

## 3.伪代码

现在让我们来看一些伪代码，并将就地算法与非就地算法进行比较。

我们将假设我们想要反转一组`n`数字。

### 3.1.原地算法

如果我们考虑这个问题，我们会看到我们有一个输入数组和一个反向数组作为输出。最后，我们实际上并不需要我们的原始数组，只需要反转的数组。

那么，为什么我们不覆盖输入，而是将它的值移动到一个全新的数组中，因为这可能是最明显的方法。为此，**我们只需要一个额外的变量**来临时存储我们当前正在处理的值:

```java
reversInPlace(array A[n])
    for i from 0 to n/2
    temp = A[i]
    A[i] = A[n - 1 - i]
    A[n - 1 - i] = temp
```

值得一提的是，无论数组有多大，在这种情况下我们需要的额外空间总是`O(1)`。

该图显示我们需要的步骤比前一个案例少:

[![](img/61f4351cff77a8a3ad249fd1c521118d.png)](/web/20220627093541/https://www.baeldung.com/wp-content/uploads/2019/08/Screen-Shot-2019-08-07-at-3.40.33-PM.png)

### 3.2.错位算法

另一方面，我们也可以用一种非常简单、更明显的方式做到这一点。我们可以创建一个相同大小的新数组，按照相应的顺序复制原始数组中的值，然后删除原始数组:

```java
reverseOutOfPlace(array A[n])
    create new array B[n]
    for i from 0 to n - 1
        B[i] = A[i]
    delete A
    return B
```

虽然这将做我们想要它做的，但它的效率不够高。**我们需要`O(n)`额外的空间** **，因为我们有两个数组要用**来操作。除此之外，创建和删除一个新数组通常是一个缓慢的操作。

让我们来看看这个过程的图示:

[![](img/1dc789149127c2e666ec944707883f11.png)](/web/20220627093541/https://www.baeldung.com/wp-content/uploads/2019/08/Screen-Shot-2019-08-07-at-3.40.22-PM.png)

## 4.Java 实现

现在让我们看看如何用 Java 实现我们在上一节中学到的东西。

首先，我们将实现一个就地算法:

```java
public static int[] reverseInPlace(int A[]) {
    int n = A.length;
    for (int i = 0; i < n / 2; i++) {
        int temp = A[i];
        A[i] = A[n - 1 - i];
        A[n - 1 - i] = temp;
    }
    return A;
}
```

我们可以很容易地测试这是否如预期的那样工作:

```java
@Test
public void givenArray_whenInPlaceSort_thenReversed() {
    int[] input = {1, 2, 3, 4, 5, 6, 7};
    int[] expected = {7, 6, 5, 4, 3, 2, 1};
    assertArrayEquals("the two arrays are not equal", expected,
      InOutSort.reverseInPlace(input));
}
```

其次，让我们来看看错位算法的实现:

```java
public static int[] reverseOutOfPlace(int A[]) {
    int n = A.length;
    int[] B = new int[n];
    for (int i = 0; i < n; i++) {
        B[n - i - 1] = A[i];
    }
    return B;
}
```

测试非常简单:

```java
@Test
public void givenArray_whenOutOfPlaceSort_thenReversed() {
    int[] input = {1, 2, 3, 4, 5, 6, 7};
    int[] expected = {7, 6, 5, 4, 3, 2, 1};
    assertArrayEquals("the two arrays are not equal", expected,
      InOutSort.reverseOutOfPlace(input));
}
```

## 5.例子

有许多排序算法正在使用就地方法。其中一些是[插入排序](/web/20220627093541/https://www.baeldung.com/java-insertion-sort)、[冒泡排序](/web/20220627093541/https://www.baeldung.com/java-bubble-sort)、[堆排序](/web/20220627093541/https://www.baeldung.com/java-heap-sort)、[快速排序](/web/20220627093541/https://www.baeldung.com/java-quicksort)和[外壳排序](/web/20220627093541/https://www.baeldung.com/java-shell-sort)，你可以了解它们的更多信息，并查看它们的 Java 实现。

还有，我们需要提一下 comb sort 和 heapsort。这些都有空间复杂度`O(log n)`。

了解更多关于 Big-O 符号的[理论，以及查看一些关于算法](/web/20220627093541/https://www.baeldung.com/big-o-notation)复杂性的[实际 Java 示例也是有用的。](/web/20220627093541/https://www.baeldung.com/java-algorithm-complexity)

## 6.结论

在本文中，我们描述了所谓的就地算法，用伪代码和几个例子说明了它们是如何工作的，列出了几个基于这一原理的算法，最后用 Java 实现了基本的例子。

像往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627093541/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting-2)