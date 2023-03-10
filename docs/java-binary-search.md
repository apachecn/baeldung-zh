# Java 中的二分搜索法算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-binary-search>

## 1。概述

在本文中，我们将介绍[二分搜索法相对于简单线性搜索](/web/20221009211958/https://www.baeldung.com/cs/linear-search-vs-binary-search)的优势，并介绍其在 Java 中的实现。

## 2。高效搜索的需求

假设我们从事葡萄酒销售业务，每天都有数百万买家访问我们的应用程序。

通过我们的应用程序，顾客可以筛选出价格低于`n`美元的商品，从搜索结果中选择一瓶，并将其添加到购物车中。每秒钟都有数百万用户在寻找限价葡萄酒。结果需要很快。

在后端，我们的算法对整个葡萄酒列表进行线性搜索，将客户输入的价格限制与列表中每瓶葡萄酒的价格进行比较。

然后，它返回价格低于或等于价格限制的项目。这种线性搜索的时间复杂度为`O(n)`。

这意味着我们系统中的酒瓶数量越多，花费的时间就越多。搜索时间与引入的新项目数量成比例增加。

如果我们开始按照排序的顺序保存项目，并使用二分搜索法搜索项目，我们可以实现复杂度为`O(log n)`。

使用二分搜索法，搜索结果所花费的时间自然会随着数据集的大小而增加，但不是成比例的。

## 3。二分搜索法

简单来说，算法将`key`值与数组的中间元素进行比较；如果它们不相等，则删除该键不属于其中的那一半，并继续搜索剩余的一半，直到成功。

记住，这里的关键是数组已经排序了。

如果搜索以剩余的一半为空结束，则`key`不在数组中。

### 3.1。迭代实现

```java
public int runBinarySearchIteratively(
  int[] sortedArray, int key, int low, int high) {
    int index = Integer.MAX_VALUE;

    while (low <= high) {
        int mid = low  + ((high - low) / 2);
        if (sortedArray[mid] < key) {
            low = mid + 1;
        } else if (sortedArray[mid] > key) {
            high = mid - 1;
        } else if (sortedArray[mid] == key) {
            index = mid;
            break;
        }
    }
    return index;
}
```

`runBinarySearchIteratively`方法以`sortedArray`的一个`sortedArray`、`key`T10、`low`、&、`high`指标作为自变量。当该方法第一次运行`low`时，`sortedArray,`的第一个索引为 0，而`high`，`sortedArray,`的最后一个索引等于其长度–1。

`middle`是`sortedArray`的中间索引。现在，算法运行一个`while`循环，将`key`与`sortedArray`的中间索引的数组值进行比较。

**注意中间的索引是如何生成的`(int mid = low + ((high – low) / 2)`。这是为了适应非常大的阵列。**如果简单地通过获取中间索引`(int mid = (low + high) / 2)`来生成中间索引，则对于包含 2 个 ^(30 个)或更多元素的数组来说，可能会发生溢出，因为`low + high`的总和很容易超过最大正`int`值。

### 3.2。递归实现

现在，让我们看看一个简单的递归实现:

```java
public int runBinarySearchRecursively(
  int[] sortedArray, int key, int low, int high) {
    int middle = low  + ((high - low) / 2);

    if (high < low) {
        return -1;
    }

    if (key == sortedArray[middle]) {
        return middle;
    } else if (key < sortedArray[middle]) {
        return runBinarySearchRecursively(
          sortedArray, key, low, middle - 1);
    } else {
        return runBinarySearchRecursively(
          sortedArray, key, middle + 1, high);
    }
} 
```

`runBinarySearchRecursively`方法接受`sortedArray`的一个`sortedArray`、`key,`、`low`和`high`索引。

### 3.3。使用`Arrays.` `binarySearch()`

```java
int index = Arrays.binarySearch(sortedArray, key); 
```

将在整数数组中搜索的`A sortedArray`和`int` `key`作为参数传递给 Java `Arrays`类的`binarySearch`方法。

### 3.4。使用`Collections.` `binarySearch()`

```java
int index = Collections.binarySearch(sortedList, key); 
```

将在`Integer` 对象列表中搜索的`A sortedList` & an `Integer` `key`作为参数传递给 Java `Collections`类的`binarySearch`方法。

### 3.5。性能

使用递归还是迭代的方法来编写算法主要是个人喜好的问题。但仍有几点我们应该注意:

1.由于维护一个`stack`的开销，递归会更慢，并且通常会占用更多的内存
2。递归不是友好的。在处理大数据集
3 时可能会造成`StackOverflowException`。递归增加了代码的清晰度，因为它比迭代方法更短

理想情况下，对于较大的 n 值，与线性搜索相比，二分搜索法执行的比较次数较少。对于较小的 n 值，线性搜索可能比二分搜索法执行得更好。

我们应该知道，这种分析是理论性的，可能会因环境而异。

另外，**二分搜索法算法需要一个有序的数据集，这也是有代价的**。如果我们使用合并排序算法对数据进行排序，我们的代码会增加额外的复杂度`n log n`。

因此，首先我们需要很好地分析我们的需求，然后决定哪种搜索算法最适合我们的需求。

## 4。结论

本教程演示了一个二分搜索法算法的实现，以及一个用它来代替线性搜索的场景。

请在 GitHub 上找到教程[的代码。](https://web.archive.org/web/20221009211958/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-searching)