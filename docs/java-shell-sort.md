# Java 中的 Shell 排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-shell-sort>

## 1。简介

在本教程中，我们将描述 Java 中的 Shell 排序算法。

## 2.外壳排序概述

### 2.1.描述

让我们首先描述一下 Shell 排序算法，这样我们就知道我们要实现什么了。

Shell sort 基于[插入排序算法](/web/20221208143830/https://www.baeldung.com/java-insertion-sort)，属于非常高效的算法群。一般来说，**算法将一个原始集合分成更小的子集，然后使用插入排序**对每个子集进行排序。

但是，它如何产生子集并不简单。它不会像我们预期的那样选择相邻元素来形成子集。相反，shell sort 使用所谓的`interval`或`gap`来创建子集。例如，如果我们有间隙`I`，这意味着一个子集将包含相距`I`位置的 元素 。

首先，算法对相距较远的元素进行排序。然后，差距变小，比较更接近的元素。这样，一些位置不正确的元素可以比我们从相邻元素中提取子集更快地定位。

### 2.2.一个例子

让我们在具有 3 和 1 的间隙以及 9 个元素的未排序列表的示例中看到这一点:

[![unsorted list of 9 elements](img/5e83251bb37f4fd234cf1100844d5c93.png)](/web/20221208143830/https://www.baeldung.com/wp-content/uploads/2019/07/Screen-Shot-2019-07-22-at-11.10.24-PM.png)

如果我们遵循上面的描述，在第一次迭代中，我们将有三个包含 3 个元素的子集(用相同的颜色突出显示):

[![three subsets with 3 elements](img/9a94649f28562d35344a24e66fe18272.png)](/web/20221208143830/https://www.baeldung.com/wp-content/uploads/2019/07/Screen-Shot-2019-07-22-at-11.12.28-PM.png)

在第一次迭代中对每个子集进行排序后，列表将看起来像:

[![sorting each of the subsets in the first iteration](img/e2a1a42ea5092568f825aa2ad9dfd8a1.png)](/web/20221208143830/https://www.baeldung.com/wp-content/uploads/2019/07/Screen-Shot-2019-07-22-at-11.13.29-PM.png)

我们可以注意到，虽然我们还没有一个排序的列表，但是元素现在更接近期望的位置。

最后，我们需要以 1 为增量再做一次排序，这实际上是一个基本的插入排序。我们需要执行的对列表排序的移位操作的数量现在比我们不做第一次迭代的情况要少:

[![one more sort](img/d07a7e5449e8cfa66a96ae94500502bf.png)](/web/20221208143830/https://www.baeldung.com/wp-content/uploads/2019/07/Screen-Shot-2019-07-22-at-11.43.27-PM.png)

### 2.3.选择间隙序列

正如我们提到的，外壳排序有一种选择空位序列的独特方式。这是一项艰巨的任务，我们应该小心，不要选择太少或太多的差距。更多细节可以在[最建议的 gap 序列列表](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/Shellsort#Gap_sequences)中找到。

## 3.履行

现在让我们看一下实现。对于间隔增量，我们将使用 Shell 的原始序列:

```java
N/2, N/4, …, 1 (continuously dividing by 2)
```

实现本身并不复杂:

```java
public void sort(int arrayToSort[]) {
    int n = arrayToSort.length;

    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i++) {
            int key = arrayToSort[i];
            int j = i;
            while (j >= gap && arrayToSort[j - gap] > key) {
                arrayToSort[j] = arrayToSort[j - gap];
                j -= gap;
            }
            arrayToSort[j] = key;
        }
    }
}
```

我们首先用 for 循环创建一个空位序列，然后对每个空位大小进行插入排序。

现在，我们可以很容易地测试我们的方法:

```java
@Test
public void givenUnsortedArray_whenShellSort_thenSortedAsc() {
    int[] input = {41, 15, 82, 5, 65, 19, 32, 43, 8};
    ShellSort.sort(input);
    int[] expected = {5, 8, 15, 19, 32, 41, 43, 65, 82};
    assertArrayEquals("the two arrays are not equal", expected, input);
}
```

## 4.复杂性

一般来说，**Shell 排序算法对于中等大小的列表非常有效**。复杂度很难确定，因为它很大程度上取决于间隙序列，但是时间复杂度在`O(N)`和`O(N^2)`之间变化。

最坏情况的空间复杂度是`O(N)`带`O(1)`辅助空间。

## 5.结论

在本教程中，我们描述了 Shell 排序，并举例说明了如何用 Java 实现它。

像往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting)