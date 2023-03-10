# Java 中的冒泡排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bubble-sort>

## 1。简介

在这篇简短的文章中，我们将详细探讨冒泡排序算法，重点是 Java 实现。

这是最简单的排序算法之一；核心思想是 **如果数组中相邻元素的顺序不正确，就一直交换它们，直到集合被排序。**

当我们迭代数据结构时，小项目“冒泡”到列表的顶部。因此，这种技术被称为冒泡排序。

由于排序是通过交换执行的，我们可以说它执行就地排序。

此外，**如果两个元素有相同的值，结果数据将保留它们的顺序**——这使得[成为稳定排序](/web/20220529020543/https://www.baeldung.com/cs/stable-sorting-algorithms)。

## 2。方法学

如前所述，为了对数组进行排序，我们在比较相邻元素的同时遍历数组，并在必要时交换它们。对于大小为`n`的数组，我们执行`n-1`次这样的迭代。

让我们举一个例子来理解这个方法。我们希望按升序对数组进行排序:

4 2 1 6 3 5

我们通过比较 4 和 2 开始第一次迭代；它们的顺序肯定不对。交换将导致:

**【2 4】**1 6 3 5

现在，对 4 和 1 重复同样的操作:

2**【1****4】**6 3 5

我们一直坚持到最后:

2 1 [ **4 6]** 3 5

2 1 4**【3****6】**5

2 1 4 3**【5 6】**

正如我们所看到的，在第一次迭代结束时，我们将最后一个元素放在了正确的位置。现在，我们需要做的就是在进一步的迭代中重复相同的过程。除此之外，我们排除了已经排序的元素。

在第二次迭代中，我们将遍历整个数组，除了最后一个元素。类似地，对于第三次迭代，我们省略最后两个元素。一般来说，对于第 k 次迭代，我们迭代到索引`n-k`(排除)。在`n-1`迭代结束时，我们将得到排序后的数组。

既然您已经理解了这种技术，那么让我们深入到实现中。

## 3。实施

让我们使用 Java 8 方法实现我们讨论的示例数组的排序:

```java
void bubbleSort(Integer[] arr) {
    int n = arr.length;
    IntStream.range(0, n - 1)
    .flatMap(i -> IntStream.range(1, n - i))
    .forEach(j -> {
        if (arr[j - 1] > arr[j]) {
            int temp = arr[j];
            arr[j] = arr[j - 1];
            arr[j - 1] = temp;
            }
     });
}
```

以及对算法的快速 JUnit 测试:

```java
@Test
public void whenSortedWithBubbleSort_thenGetSortedArray() {
    Integer[] array = { 2, 1, 4, 6, 3, 5 };
    Integer[] sortedArray = { 1, 2, 3, 4, 5, 6 };
    BubbleSort bubbleSort = new BubbleSort();
    bubbleSort.bubbleSort(array);

    assertArrayEquals(array, sortedArray);
}
```

## 4。复杂性和优化

我们可以看到，**为平均和最差情况**，**的时间复杂度为**，**，**。

此外，**的空间复杂度**，即使在最坏的情况下，**也是 O(1)，因为冒泡排序算法不需要任何额外的内存**，并且排序发生在原始数组中。

通过仔细分析解决方案，我们可以看到**如果在一次迭代中没有发现交换，我们就不需要进一步迭代**。

在前面讨论的例子中，在第二次迭代之后，我们得到:

1 2 3 4 5 6

在第三次迭代中，我们不需要交换任何一对相邻的元素。所以我们可以跳过所有剩余的迭代。

在排序数组的情况下，在第一次迭代中不需要交换，这意味着我们可以停止执行。这是最好的情况，算法的时间复杂度为 O(n) 。

现在，让我们实现优化的解决方案。

```java
public void optimizedBubbleSort(Integer[] arr) {
    int i = 0, n = arr.length;
    boolean swapNeeded = true;
    while (i < n - 1 && swapNeeded) {
        swapNeeded = false;
        for (int j = 1; j < n - i; j++) {
            if (arr[j - 1] > arr[j]) {
                int temp = arr[j - 1];
                arr[j - 1] = arr[j];
                arr[j] = temp;
                swapNeeded = true;
            }
        }
        if(!swapNeeded) {
            break;
        }
        i++;
    }
}
```

让我们检查优化算法的输出:

```java
@Test
public void 
  givenIntegerArray_whenSortedWithOptimizedBubbleSort_thenGetSortedArray() {
      Integer[] array = { 2, 1, 4, 6, 3, 5 };
      Integer[] sortedArray = { 1, 2, 3, 4, 5, 6 };
      BubbleSort bubbleSort = new BubbleSort();
      bubbleSort.optimizedBubbleSort(array);

      assertArrayEquals(array, sortedArray);
}
```

## 5。结论

在本教程中，我们看到了冒泡排序是如何工作的，以及它在 Java 中的实现。我们还看到了如何对其进行优化。总而言之，这是一个就地稳定的算法，具有时间复杂性:

*   最坏和一般情况:O(n*n)，当数组顺序相反时
*   最佳情况:O(n)，当数组已经排序时

该算法在计算机图形学中很流行，因为它能够检测排序中的一些小错误。例如，在一个几乎排序的数组中，只需要交换两个元素，就可以得到一个完全排序的数组。冒泡排序可以修复这样的错误(即对这个数组进行排序)。

和往常一样，实现这个算法的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220529020543/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting)