# 如何在 Java 中找到第 k 个最大的元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kth-largest-element>

## 1。简介

在本文中，我们将给出寻找唯一数字序列中第`k`个最大元素的各种解决方案。在我们的例子中，我们将使用一个整数数组。

我们还将讨论每个算法的平均和最坏情况下的时间复杂度。

## 2。解决方案

现在让我们探索几个可能的解决方案——一个使用简单排序，两个使用从快速排序派生的快速选择算法。

### 2.1。分类

当我们思考问题的时候，也许想到的**最明显的解决方案就是** **对数组**进行排序。

让我们定义所需的步骤:

*   按升序对数组进行排序
*   由于数组的最后一个元素将是最大的元素，第`k`个最大的元素将位于`xth`索引处，其中`x = length(array) – k`

正如我们所看到的，解决方案很简单，但是需要对整个数组进行排序。因此，时间复杂度将为`O(n*logn)`:

```java
public int findKthLargestBySorting(Integer[] arr, int k) {
    Arrays.sort(arr);
    int targetIndex = arr.length - k;
    return arr[targetIndex];
}
```

另一种方法是按降序对数组进行排序，并简单地返回第`(k-1)`个索引上的元素:

```java
public int findKthLargestBySortingDesc(Integer[] arr, int k) {
    Arrays.sort(arr, Collections.reverseOrder());
    return arr[k-1];
}
```

### 2.2。快速选择

这可以被认为是对先前方法的优化。在这里，我们选择[快速排序](https://web.archive.org/web/20220626194127/https://www.geeksforgeeks.org/quick-sort/)进行排序。通过分析问题陈述，我们认识到**我们实际上不需要对整个数组进行排序——我们只需要重新排列它的内容，这样数组的第`k`个元素就是第`k`个最大或最小的元素。**

在快速排序中，我们选择一个枢纽元素，并将其移动到正确的位置。我们还对它周围的数组进行了分区。**在 QuickSelect 中，想法是在枢轴本身是第`k`个最大元素的地方停止。**

如果我们不在枢轴的左右两侧重复，我们可以进一步优化算法。我们只需要根据支点的位置对其中一个进行递归。

让我们看看快速选择算法的基本思想:

*   选择一个枢纽元素，并相应地划分数组
    *   选择最右边的元素作为枢轴
    *   重新排列数组，以便将轴心元素放在正确的位置——小于轴心的所有元素将位于较低的索引处，而大于轴心的元素将位于比轴心更高的索引处
*   如果 pivot 被放在数组中的第`k`个元素，那么退出这个过程，因为 pivot 是第`k`个最大的元素
*   如果枢轴位置大于`k,`，则继续左侧子阵列的过程，否则，重复右侧子阵列的过程

我们可以编写通用逻辑，也可以用来寻找第`k`个最小的元素。我们将定义一个方法`findKthElementByQuickSelect()` ，它将返回排序数组中的第`k`个元素。

如果我们对数组进行升序排序，数组的第`k`个元素将是第`k`个最小的元素。为了找到第`k`个最大的元素，我们可以通过`k= length(Array) – k.`

让我们实现这个解决方案:

```java
public int 
  findKthElementByQuickSelect(Integer[] arr, int left, int right, int k) {
    if (k >= 0 && k <= right - left + 1) {
        int pos = partition(arr, left, right);
        if (pos - left == k) {
            return arr[pos];
        }
        if (pos - left > k) {
            return findKthElementByQuickSelect(arr, left, pos - 1, k);
        }
        return findKthElementByQuickSelect(arr, pos + 1,
          right, k - pos + left - 1);
    }
    return 0;
}
```

现在让我们实现`partition` 方法，该方法选取最右边的元素作为枢纽，将其放在适当的索引处，并以较低索引处的元素应该少于枢纽元素的方式对数组进行分区。

同样，索引较高的元素将大于 pivot 元素:

```java
public int partition(Integer[] arr, int left, int right) {
    int pivot = arr[right];
    Integer[] leftArr;
    Integer[] rightArr;

    leftArr = IntStream.range(left, right)
      .filter(i -> arr[i] < pivot)
      .map(i -> arr[i])
      .boxed()
      .toArray(Integer[]::new);

    rightArr = IntStream.range(left, right)
      .filter(i -> arr[i] > pivot)
      .map(i -> arr[i])
      .boxed()
      .toArray(Integer[]::new);

    int leftArraySize = leftArr.length;
    System.arraycopy(leftArr, 0, arr, left, leftArraySize);
    arr[leftArraySize+left] = pivot;
    System.arraycopy(rightArr, 0, arr, left + leftArraySize + 1,
      rightArr.length);

    return left + leftArraySize;
}
```

有一种更简单的迭代方法来实现分区:

```java
public int partitionIterative(Integer[] arr, int left, int right) {
    int pivot = arr[right], i = left;
    for (int j = left; j <= right - 1; j++) {
        if (arr[j] <= pivot) {
            swap(arr, i, j);
            i++;
        }
    }
    swap(arr, i, right);
    return i;
}

public void swap(Integer[] arr, int n1, int n2) {
    int temp = arr[n2];
    arr[n2] = arr[n1];
    arr[n1] = temp;
}
```

这个解决方案平均在`O(n)`时间内有效。但是，在最坏的情况下，时间复杂度会是`O(n^2)`。

### 2.3。带有随机分区的快速选择

这种方法是对以前方法的一点修改。如果数组几乎/完全排序，并且如果我们选择最右边的元素作为支点，则左右子数组的划分将非常不均匀。

这个方法建议**以随机的方式选取初始的枢纽元素。**虽然我们不需要改变分区逻辑。

我们不调用`partition`，而是调用`randomPartition` 方法，它挑选一个随机元素，并在最终调用`partition` 方法之前与最右边的元素交换。

让我们实现`randomPartition`方法:

```java
public int randomPartition(Integer arr[], int left, int right) {
    int n = right - left + 1;
    int pivot = (int) (Math.random()) * n;
    swap(arr, left + pivot, right);
    return partition(arr, left, right);
}
```

在大多数情况下，这种解决方案比前一种情况更好。

随机化快速选择的预期时间复杂度为`O(n)`。

然而，最差的时间复杂度仍然是`O(n^2)`。

## 3。结论

在本文中，我们讨论了寻找唯一数字数组中第`k`个最大(或最小)元素的不同解决方案。最简单的解决方案是对数组进行排序并返回第`k`个元素。这个解决方案的时间复杂度为`O(n*logn)`。

我们还讨论了快速选择的两种变体。这个算法并不简单，但在一般情况下，它的时间复杂度为`O(n)`。

与往常一样，该算法的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626194127/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-1)