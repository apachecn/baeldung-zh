# Java 中的选择排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-selection-sort>

## 1.介绍

在本教程中，我们将**学习选择排序**，看看它在 Java 中的实现，并分析它的性能。

## 2.算法概述

选择排序**从未排序数组**的第一个位置的元素开始，扫描后续元素到**找到最小的元素**。一旦找到，最小的元素与 1 ^(st) 位置的元素交换。

然后，算法移动到第 2 个^(和第 2 个)位置的元素，并扫描后续元素以找到第 2 个^(和第 3 个)最小元素的索引。一旦找到，第二小的元素与第 2 ^(和第)位置的元素交换。

这个过程一直持续到我们到达数组的第 n-1 个元素，这将第 n-1 个最小的元素放在第 n-1 个位置。在第 n-1 次迭代中，最后一个元素自动就位，从而对数组进行排序。

我们找到了最大的元素而不是最小的元素来对数组进行降序排序。

我们来看一个未排序数组的例子，按升序排序，直观理解算法。

### 2.1.一个例子

考虑以下未排序的数组:

`int[] arr = {5, 4, 1, 6, 2}`

**迭代 1**

考虑到算法的上述工作方式，我们从第 1 个位置的元素-5 开始，扫描所有后续元素，找到最小的元素-1。然后，我们将最小的元素与第一个位置的元素交换。

修改后的数组现在看起来像这样:

`{1, 4, 5, 6, 2}`

比较总数:4

**迭代 2**

在第二次迭代中，我们移动到第 2 个^(和第 1 个)元素–4–并扫描后续元素以找到第二小的元素–2。然后，我们将第二小的元素与第二个^(和第三个)位置的元素交换。

修改后的数组现在看起来像这样:

`{1, 2, 5, 6, 4}`

比较总数:3

继续类似地，我们有以下迭代:

**迭代 3**

`{1, 2, 4, 6, 5}`

比较总数:2

**迭代 4**

`{1, 2, 4, 5, 6}`

比较总数:1

## 3。 **实现**

让我们使用几个`for`循环来实现选择排序:

```java
public static void sortAscending(final int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int minElementIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[minElementIndex] > arr[j]) {
                minElementIndex = j;
            }
        }

        if (minElementIndex != i) {
            int temp = arr[i];
            arr[i] = arr[minElementIndex];
            arr[minElementIndex] = temp;
        }
    }
}
```

当然，要逆转它，我们可以做一些非常类似的事情:

```java
public static void sortDescending(final int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int maxElementIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[maxElementIndex] < arr[j]) {
                maxElementIndex = j;
            }
        }

        if (maxElementIndex != i) {
            int temp = arr[i];
            arr[i] = arr[maxElementIndex];
            arr[maxElementIndex] = temp;
        }
    }
}
```

再努力一点，我们可以用 [`Comparator` s](/web/20221205150215/https://www.baeldung.com/java-comparator-comparable) 把它们结合起来。

## 4.性能概述

### 4.1.时间

在我们之前看到的例子中，**选择最小的元素需要总共`(n-1)`次比较**，然后将它交换到第一个位置。类似地，**选择下一个最小的元素需要总的`(n-2)`** 比较，然后交换到第 2 个^(和第)位置，依此类推。

因此，从索引 0 开始，我们执行`n-1, n-2, n-3, n-4 …. 1`比较。由于之前的迭代和交换，最后一个元素自动就位。

从数学上讲，第一个`n-1`自然数的**和将告诉我们需要多少次比较，才能使用选择排序对一个大小为`n`的数组进行排序。**

**`n`自然数总和的公式为 `n(n+1)/2`。**

在我们的例子中，我们需要第一个`n-1`自然数的和。因此，我们将上面公式中的`n`替换为`n-1`，得到:

`(n-1)(n-1+1)/2 = (n-1)n/2 = **(n^2-n)/2**`

随着`n^2`随着`n`的增长而显著增长，我们考虑将`n`的更高幂作为性能基准，使得该算法具有`O(n^2).` 的**时间复杂度**

### 4.2.空间

就辅助空间复杂度而言，选择排序需要一个额外的变量来临时保存值以便交换。因此，选择排序的**空间复杂度为`O(1)`。**

## 5.结论

选择排序是一种非常容易理解和实现的排序算法。不幸的是，**的二次时间复杂度使其成为一种昂贵的排序技术**。此外，由于算法必须扫描每个元素，**最佳情况、一般情况和最坏情况的时间复杂度是相同的**。

其他排序技术如[插入排序](/web/20221205150215/https://www.baeldung.com/java-insertion-sort)和[外壳排序](/web/20221205150215/https://www.baeldung.com/java-shell-sort)也有二次最坏情况时间复杂度，但它们在最好和一般情况下表现更好。

在 GitHub 上查看选择排序[的完整代码。](https://web.archive.org/web/20221205150215/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting)