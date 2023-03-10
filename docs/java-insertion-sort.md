# Java 中的插入排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-insertion-sort>

## 1.概观

在本教程中，我们将讨论**插入排序算法，并看看它的 Java 实现**。

插入排序是对少量项目进行排序的有效算法。这种方法是基于打牌人对一手牌进行分类的方式。

我们从空着的左手和放在桌子上的牌开始。然后我们从桌子上一次取出一张卡片，把它插入左手的正确位置。为了找到新卡的正确位置，我们从右到左将它与手中已经排序的一组卡进行比较。

让我们从理解伪代码形式的算法步骤开始。

## 2.伪代码

我们将把插入排序的伪代码呈现为一个名为`INSERTION-SORT`的过程，将一个包含 n 个要排序的条目的数组`A[1 .. n]`作为参数。**该算法就地对输入数组进行排序**(通过重新排列数组 A 中的项目)。

该过程完成后，输入数组 A 包含输入序列的排列，但按排序顺序排列:

```java
INSERTION-SORT(A)

for i=2 to A.length
    key = A[i]
    j = i - 1 
    while j > 0 and A[j] > key
        A[j+1] = A[j]
        j = j - 1
    A[j + 1] = key
```

让我们简单回顾一下上面的算法。

索引`i`表示要处理的数组中当前项目的位置。

我们从第二项开始，根据定义，只有一项的数组被认为是已排序的。索引`i`处的项目称为`key`。一旦有了`key,`，算法的第二部分处理寻找它的正确索引。**如果`key`小于索引`j`项的值，则按键向左移动一个位置。**这个过程一直持续到我们到达一个比键更小的元素。

值得注意的是，在开始迭代寻找索引`i`处`key`的正确位置之前，数组`A[1 .. j – 1]`已经是`sorted`。

## 3.强制性实施

对于命令式的情况，我们将编写一个名为`insertionSortImperative`的函数，将一个整数数组作为参数。该函数从第二项开始迭代数组。

在迭代期间的任何给定时间，**我们可以认为这个数组在逻辑上被分成两部分；**左侧为已排序的项目，右侧包含尚未排序的项目。

这里需要注意的一点是，在找到插入新项目的正确位置后，**我们将项目向右移动(而不是交换)**以释放空间。

```java
public static void insertionSortImperative(int[] input) {
    for (int i = 1; i < input.length; i++) { 
        int key = input[i]; 
        int j = i - 1;
        while (j >= 0 && input[j] > key) {
            input[j + 1] = input[j];
            j = j - 1;
        }
        input[j + 1] = key; 
    }
}
```

接下来，让我们为上面的方法创建一个测试:

```java
@Test
public void givenUnsortedArray_whenInsertionSortImperative_thenSortedAsc() {
    int[] input = {6, 2, 3, 4, 5, 1};
    InsertionSort.insertionSortImperative(input);
    int[] expected = {1, 2, 3, 4, 5, 6};
    assertArrayEquals("the two arrays are not equal", expected, input);
}
```

上面的测试证明了该算法对输入数组`<6, 2, 3, 4, 5, 1>`的升序排序是正确的。

## 4.递归实现

递归情况下的函数称为`insertionSortR` `ecursive `，接受整数数组作为输入(与命令性情况相同)。

这里与命令式的区别(尽管它是递归的)是它调用一个重载函数，第二个参数等于要排序的项目数。

因为我们想对整个数组进行排序，所以我们将传递一些与其长度相等的项:

```java
public static void insertionSortRecursive(int[] input) {
    insertionSortRecursive(input, input.length);
}
```

递归的情况更有挑战性。当我们试图对只有一个项目的数组进行排序时，就会出现基本情况。在这种情况下，我们什么都不做。

所有后续的递归调用都对输入数组的预定义部分进行排序——从第二项开始，直到到达数组的末尾:

```java
private static void insertionSortRecursive(int[] input, int i) {
    if (i <= 1) {
        return;
    }
    insertionSortRecursive(input, i - 1);
    int key = input[i - 1];
    int j = i - 2;
    while (j >= 0 && input[j] > key) {
        input[j + 1] = input[j];
        j = j - 1;
    }
    input[j + 1] = key;
}
```

这是一个包含 6 项的输入数组的调用堆栈的样子:

```java
insertionSortRecursive(input, 6)
insertionSortRecursive(input, 5) and insert the 6th item into the sorted array
insertionSortRecursive(input, 4) and insert the 5th item into the sorted array
insertionSortRecursive(input, 3) and insert the 4th item into the sorted array
insertionSortRecursive(input, 2) and insert the 3rd item into the sorted array
insertionSortRecursive(input, 1) and insert the 2nd item into the sorted array
```

让我们也来看看对它的测试:

```java
@Test
public void givenUnsortedArray_whenInsertionSortRecursively_thenSortedAsc() {
    int[] input = {6, 4, 5, 2, 3, 1};
    InsertionSort.insertionSortRecursive(input);
    int[] expected = {1, 2, 3, 4, 5, 6};
    assertArrayEquals("the two arrays are not equal", expected, input);
}
```

上面的测试证明了该算法对输入数组`<6, 2, 3, 4, 5, 1>`的升序排序是正确的。

## 5.时间和空间复杂性

**`INSERTION-SORT`程序运行的时间为`O(n^2)`** 。对于每个新项目，我们从右到左遍历数组中已经排序的部分，以找到它的正确位置。然后，我们通过将项目向右移动一个位置来插入它。

该算法就地排序，因此其**空间复杂度对于命令式实现为`O(1)` ，对于递归实现为`O(n)`。**

## 6.结论

在本教程中，我们看到了如何实现插入排序。

该算法对于排序少量项目很有用。对超过 100 个条目的输入序列进行排序会变得低效。

请记住，尽管它是二次复杂度，但它在不需要辅助空间的情况下进行排序，就像`merge sort`一样。

完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129001002/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting)