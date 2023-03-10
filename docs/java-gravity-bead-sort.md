# Java 中的重力/珠子排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-gravity-bead-sort>

## 1.介绍

在本教程中，我们将讨论[重力排序](/web/20221208143837/https://www.baeldung.com/cs/gravity-sort)算法及其在 Java 中的单线程实现。

## 2.该算法

重力排序是一种受自然事件启发的自然排序算法——在这种情况下，是重力。也被称为珠排序，**这种算法模拟重力来排序正整数列表**。

**该算法的思想是用竖条和横条中的珠子来表示正整数**——类似于算盘，除了每一层表示输入列表中的一个数字。下一步是将珠子放在尽可能低的位置，这样算盘的数字就会按升序排列:

例如，下面是对[4，2]的输入列表进行排序的过程:

[![](img/1915d7c28d7fe10b72b2cc09d3238f88.png)](/web/20221208143837/https://www.baeldung.com/wp-content/uploads/2022/10/1_Gravity-Sort-in-Java-Diagram-2.png)

我们对算盘施加重力。随后 **，所有珠子在落下**后将处于其可能的最低位置。**现在算盘的结果状态是从上到下正整数的升序排列。**

事实上，重力会同时落下所有的珠子。但是，在软件中，我们必须模拟珠子在不同的迭代中下落。接下来，我们将看看如何将算盘表示为二维数组，以及如何模拟下落的珠子来对列表中的数字进行排序。

## 3.履行

为了在软件中实现重力排序，我们将按照本文中的伪代码用 Java 编写代码。

**首先，我们需要将输入列表转换成算盘。**我们将使用一个二维数组来表示棒(列)和水平(行)作为矩阵的维度。此外，我们将使用`true`或`false` 分别表示一个珠子或一个空单元格。

**在设置我们的算盘之前，让我们先找到矩阵的维数。**列数`m` 等于列表中最大的元素。因此，让我们创建一个方法来查找这个数字:

```java
static int findMax(int[] A) {
    int max = A[0];
    for (int i = 1; i < A.length; i++) {
        if (A[i] > max) {
            max = A[i];
        }
    }
    return max;
}
```

现在，我们可以将最大的数字分配给`m`:

```java
int[] A = {1, 3, 4, 2};
int m = findMax(A);
```

**使用`m,` 我们现在能够创建一个算盘的表示。**我们将使用`setupAbacus()`方法来完成这项工作:

```java
static boolean[][] setupAbacus(int[] A, int m) {
    boolean[][] abacus = new boolean[A.length][m];
    for (int i = 0; i < abacus.length; i++) {
        int number = A[i];
        for (int j = 0; j < abacus[0].length && j < number; j++) {
            abacus[A.length - 1 - i][j] = true;
        }
    }
    return abacus;
}
```

**`setupAbacus()`方法返回算盘的初始状态。** **该方法遍历矩阵中的每个单元格，指定一个珠子或一个空单元格。**

在矩阵的每一层，我们将使用列表`A`中的第`i`个`number`来确定一行中珠子的数量。此外，我们遍历每一列`j`，如果`number` 大于第`j`列的索引，我们标记这个单元格`true`来表示一个珠子。否则，循环提前终止，剩下的单元格为空或第`i`行中的`false`。

让我们来创造算盘:

```java
boolean[][] abacus = setupAbacus(A, m);
```

**我们现在准备好让重力将珠子降到最低位置进行分类**:

```java
static void dropBeads(boolean[][] abacus, int[] A, int m) {
    for (int i = 1; i < A.length; i++) {
        for (int j = m - 1; j >= 0; j--) {
            if (abacus[i][j] == true) {
                int x = i;
                while (x > 0 && abacus[x - 1][j] == false) {
                    boolean temp = abacus[x - 1][j];
                    abacus[x - 1][j] = abacus[x][j];
                    abacus[x][j] = temp;
                    x--;
                }
            }
        }
    }
}
```

最初，`dropBeads()` 方法遍历矩阵中的每个单元格。从 1 开始，`i `是开始的行，因为不会有任何珠子从最底层的 0 级掉落。关于列，我们从`j = m – 1` 开始从右到左放下珠子。

**在每次迭代中，我们检查当前单元格`abacus[i][j]`是否包含一个珠子。如果是这样，我们使用一个变量`x`来存储下落珠子的当前高度。** **我们通过减少`x `来掉珠，只要不是最底层，下面的单元格是空的。**

**最后，我们需要将算盘的最终状态转换成一个排序数组。**`toSortedList()` 方法接受算盘作为参数，以及原始输入列表，并相应地修改数组:

```java
static void toSortedList(boolean[][] abacus, int[] A) {
    int index = 0;
    for (int i = abacus.length - 1; i >=0; i--) {
        int beads = 0;
        for (int j = 0; j < abacus[0].length && abacus[i][j] == true; j++) {
            beads++;
        }
        A[index++] = beads;
    }
}
```

我们可以回忆一下，每行中珠子的数量代表列表中的一个数字。因此，该方法遍历算盘中的每一层，对珠子进行计数，并将值赋给列表。**该方法从最高行值开始按升序设置值。然而，从`i = 0, it` 开始，数字以降序排列。**

**让我们把算法的所有部分放在一个单独的`gravitySort()` 方法中:**

```java
static void gravitySort(int[] A) {
    int m = findMax(A);
    boolean[][] abacus = setupAbacus(A, m);
    dropBeads(abacus, A, m);
    transformToList(abacus, A);
}
```

我们可以通过创建一个单元测试来确认算法是否有效:

```java
@Test
public void givenIntegerArray_whenSortedWithGravitySort_thenGetSortedArray() {
    int[] actual = {9, 9, 100, 3, 57, 12, 3, 78, 0, 2, 2, 40, 21, 9};
    int[] expected = {0, 2, 2, 3, 3, 9, 9, 9, 12, 21, 40, 57, 78, 100};
    GravitySort.sort(actual);
    Assert.assertArrayEquals(expected, actual);
}
```

## 4.复杂性分析

我们看到重力排序算法需要大量的处理。所以，我们把它分解成时间和空间复杂度。

### 4.1.时间复杂度

重力排序算法的实现从找到输入列表中的最大数`m` 开始。这个过程是一个`O(n)`运行时操作，因为我们遍历了一次数组。一旦我们获得了`m`，我们就建立了算盘的初始状态。因为算盘实际上是一个矩阵，访问每一行和每一列中的每个单元格都会导致一个`O(m * n)`操作，其中`n` 是行数。

**一旦我们的设置准备就绪，我们必须把珠子放到矩阵中尽可能低的位置，就好像重力正在影响我们的算盘一样。**同样，我们沿着一个内部循环访问矩阵中的每一个单元，这个内部循环在每一列的最多`n` 层放下珠子。**这个流程有一个`O(n * n * m)`运行时。**

此外，我们必须执行`O(n)`附加步骤，根据算盘中的排序表示重新创建我们的列表。

总的来说，重力排序是一种`O(n * n * m)`算法，考虑到它模拟珠子下落的努力。

### 4.2.空间复杂性

重力排序的空间复杂度与输入列表的大小及其最大数成正比。例如，**一个有`n` 个元素和最大数量`m`的列表需要一个`n x m` 维度的矩阵表示。因此**，**空间复杂度为`O(n * m)`为矩阵分配额外的内存空间。**

尽管如此，我们试图通过用单个位或数字表示珠子和空单元格来最小化空间复杂度。即，`1`或`true`表示珠子，类似地，`0`或`false`值是空单元格。

## 5.结论

在本文中，我们学习了如何实现重力排序算法的单线程方法。也称为珠排序，这种算法的灵感来自重力自然排序正整数，我们在算盘中表示为珠。然而，在软件中，我们使用二维矩阵和单位值来再现这种环境。

虽然单线程实现具有昂贵的时间和空间复杂度，但是该算法在硬件和多线程应用中表现良好。尽管如此，重力排序算法还是展示了自然事件如何激发软件实现的解决方案。

和往常一样，实现这个算法的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-sorting-2)