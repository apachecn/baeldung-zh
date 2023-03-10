# arrays . sort vs arrays . parallel sort

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arrays-sort-vs-parallelsort>

## 1.概观

我们都使用过`Arrays.sort()`对一组对象或原语进行排序。在 JDK 8 中，创建者增强了 API 以提供一个新方法:`Arrays.parallelSort()`。

在本教程中，我们将对`sort()`和`parallelSort()`方法进行比较。

## 2.`Arrays.sort()`

方法对对象或原语的数组进行排序。**该方法使用的排序算法是双枢纽[快速排序](/web/20221205211628/https://www.baeldung.com/algorithm-quicksort)。**换句话说，它是快速排序算法的自定义实现，以实现更好的性能。

**该方法是单线程的**，有两种变体:

*   `sort(array)`–将整个数组按升序排序
*   `sort(array, fromIndex, toIndex)`–仅对从`fromIndex`到`toIndex` 的元素进行排序

让我们看一个两种变体的例子:

```java
@Test
public void givenArrayOfIntegers_whenUsingArraysSortMethod_thenSortFullArrayInAscendingOrder() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    Arrays.sort(array);

    assertArrayEquals(expected, array);

}

@Test
public void givenArrayOfIntegers_whenUsingArraysSortWithRange_thenSortRangeOfArrayAsc() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 10, 4, 1, 2, 6, 7, 8, 9, 3, 5 };

    Arrays.sort(array, 2, 8);

    assertArrayEquals(expected, array);
}
```

让我们总结一下这种方法的优缺点:

| 赞成的意见 | 面向连接的网络服务(Connection Oriented Network Service) |
| 快速处理较小的数据集 | 大型数据集的性能会下降 |
|  | 系统的多个内核没有得到利用 |

## 3.`Arrays.parallelSort()`

此方法还对对象或基元的数组进行排序。与`sort()`相似，它也有两种变体来排序全数组和部分数组:

```java
@Test
public void givenArrayOfIntegers_whenUsingArraysParallelSortMethod_thenSortFullArrayInAscendingOrder() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    Arrays.parallelSort(array);

    assertArrayEquals(expected, array);
}

@Test
public void givenArrayOfIntegers_whenUsingArraysParallelSortWithRange_thenSortRangeOfArrayAsc() {
    int[] array = { 10, 4, 6, 2, 1, 9, 7, 8, 3, 5 };
    int[] expected = { 10, 4, 1, 2, 6, 7, 8, 9, 3, 5 };

    Arrays.parallelSort(array, 2, 8);

    assertArrayEquals(expected, array);
}
```

`parallelSort()`功能不同。与使用单线程顺序排序数据的`sort()`不同，**使用并行排序-合并排序算法**。它将数组分解成子数组，子数组本身经过排序，然后进行合并。

为了执行并行任务，它使用了`ForkJoin`池。

但是我们要知道，它只有在满足一定条件的情况下才会使用并行。如果数组大小小于或等于 8192，或者处理器只有一个内核，则它使用顺序双枢轴快速排序算法。否则，它使用并行排序。

我们来总结一下使用它的优缺点:

| 赞成的意见 | 面向连接的网络服务(Connection Oriented Network Service) |
| 为大型数据集提供更好的性能 | 较小的阵列速度较慢 |
| 利用系统的多个内核 |  |

## 4.比较

现在让我们看看这两种方法在不同大小的数据集上的表现。下面的数字是使用 [JMH 基准得出的。](/web/20221205211628/https://www.baeldung.com/java-microbenchmark-harness)测试环境使用 AMD A10 PRO 2.1Ghz 四核处理器和 JDK 1.8.0_221:

| 数组大小 | `Arrays.sort()` | `Arrays.parallelSort()` |
| One thousand | o.048 | Zero point zero five four |
| ten thousand | Zero point eight four seven | Zero point four two five |
| One hundred thousand | Seven point five seven | Four point three nine five |
| One million | Sixty-five point three zero one | Thirty-seven point nine nine eight |

## 5.结论

在这篇简短的文章中，我们看到了`sort()`和`parallelSort()`的区别。

基于性能结果，我们可以得出结论，当我们有一个大数据集要排序时，`parallelSort()`可能是一个更好的选择。然而，对于较小的阵列，最好使用`sort()`,因为它提供了更好的性能。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205211628/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-sorting)