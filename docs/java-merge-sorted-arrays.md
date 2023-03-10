# 如何在 Java 中合并两个排序后的数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-merge-sorted-arrays>

## 1.介绍

在本教程中，我们将学习如何将两个有序数组合并成一个有序数组。

## 2.问题

让我们来理解这个问题。我们有两个排序数组，我们想把它们合并成一个。

![Merge Sorted Arrays](img/572d96f46afb02ba35b00f23abc6ce18.png)

## 3.算法

当我们分析这个问题时，**很容易观察到我们可以通过使用[合并排序](/web/20221208143956/https://www.baeldung.com/java-merge-sort)的合并操作来解决这个问题。**

假设我们有两个长度分别为`fooLength`和`barLength`的排序数组`foo`和`bar`。接下来，我们可以声明另一个大小为`fooLength + barLength`的数组`merged`。

然后，我们应该在同一个循环中遍历这两个数组。我们将为`fooPosition`和`barPosition`维护一个当前的索引值。在循环的给定迭代中，**我们取在索引处具有较小值元素的数组，并推进该索引。**这个元素将占据`merged`数组中的下一个位置。

最后，一旦我们从一个数组中转移了所有元素，我们将把另一个数组中剩余的元素复制到`merged`数组中。

现在让我们用图片来看看这个过程，以便更好地理解这个算法。

**第一步:**

我们首先比较两个数组中的元素，然后选择较小的一个。

![Merge Arrays First Step](img/3bf2017951d0ebe1bc8eb6c98030194f.png)

然后我们增加在`first`数组中的位置。

**第二步:**

![Merge Arrays Second Step](img/8013f474964e2ba049d759142008e38d.png)

这里我们增加了在`second`数组中的位置，并移动到下一个元素 8。

**第三步:**
![Merge Arrays Third Step](img/923bfd96f71b192da60208f2b814847f.png)

在这个迭代的最后，我们已经遍历了`first`数组的所有元素。

**第四步:**

在这一步，我们只是将所有剩余的元素从`second`数组复制到`result`。

![Merge Arrays Fourth Step](img/15f5c0666319094f6c564a593718e701.png)

## 4.履行

现在让我们看看如何实现它:

```java
public static int[] merge(int[] foo, int[] bar) {

    int fooLength = foo.length;
    int barLength = bar.length;

    int[] merged = new int[fooLength + barLength];

    int fooPosition, barPosition, mergedPosition;
    fooPosition = barPosition = mergedPosition = 0;

    while(fooPosition < fooLength && barPosition < barLength) {
        if (foo[fooPosition] < bar[barPosition]) {
            merged[mergedPosition++] = foo[fooPosition++];
        } else {
            merged[mergedPosition++] = bar[barPosition++];
        }
    }

    while (fooPosition < fooLength) {
        merged[mergedPosition++] = foo[fooPosition++];
    }

    while (barPosition < barLength) {
        merged[mergedPosition++] = bar[barPosition++];
    }

    return merged;
}
```

让我们进行一个简单的测试:

```java
@Test
public void givenTwoSortedArrays_whenMerged_thenReturnMergedSortedArray() {

    int[] foo = { 3, 7 };
    int[] bar = { 4, 8, 11 };
    int[] merged = { 3, 4, 7, 8, 11 };

    assertArrayEquals(merged, SortedArrays.merge(foo, bar));
}
```

## 5.复杂性

我们遍历两个数组，选择较小的元素。最后，我们从`foo`或`bar`数组中复制剩余的元素。**所以时间复杂度变成了`O(fooLength + barLength)`** 。我们使用了一个辅助数组来获得结果。**所以空间复杂度也是`O(fooLength + barLength)`** 。

## 6.结论

在本教程中，我们学习了如何将两个排序后的数组合并为一个。

像往常一样，本教程的源代码可以在 GitHub 上的[中找到。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-5)