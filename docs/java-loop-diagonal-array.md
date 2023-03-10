# 对角遍历一个 2d Java 数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-loop-diagonal-array>

## 1.概观

在本教程中，我们将看到如何通过一个二维数组对角循环。我们提供的解决方案可以用于任何大小的正方形二维阵列。

## 2.二维数组

使用数组元素的关键是知道如何从数组中获取特定的元素。对于二维数组，我们使用行和列索引来获取数组的元素。对于这个问题，我们将使用下图来展示如何获取这些元素。

[![LoopingDiagonallyThrough2DArray 2](img/630159246685856bf16e453e6c4a7a20.png)](/web/20221206062814/https://www.baeldung.com/wp-content/uploads/2019/07/LoopingDiagonallyThrough2DArray-2.png)

接下来，我们需要了解数组中有多少条对角线，如图所示。我们首先获取数组一维的长度，然后用它来获取对角线的数量(`diagonalLines` ) `.`

然后，我们使用对角线的数量来获得中点，这将有助于搜索行和列索引。

在本例中，中点是三:

```java
int length = twoDArray.length
int diagonalLines = (length + length) - 1
int midPoint = (diagonalLines / 2) + 1
```

## 3.获取行和列索引

为了遍历整个数组，我们从 1 开始循环，直到循环变量小于或等于`diagonalLines` 变量。

```java
for (int i = 1; i <= diagonalLines; i++) {
    // some operations
}
```

我们也引入一条对角线上的项数的概念，称之为`itemsInDiagonal`。例如，上图中的第 3 行有 3 个项目(g、e、c)，第 4 行有 2 个项目(h、f)。当循环变量`i `小于或等于*中点*时，该变量在循环中增加 1。否则就递减 1。

在递增或递减`itemsInDiagonal,`之后，我们就有了一个新的循环变量`j`。变量`j `从 0 开始递增，直到小于`itemsInDiagonal.`

然后我们使用循环变量`i `和`j` 来获得行和列的索引。这个计算的逻辑取决于循环变量`i` 是否大于`midPoint`。当`i `大于`midPoint`时，我们也使用`length`变量来确定行和列索引:

```java
int rowIndex;
int columnIndex;

if (i <= midPoint) {
    itemsInDiagonal++;
    for (int j = 0; j < itemsInDiagonal; j++) {
        rowIndex = (i - j) - 1;
        columnIndex = j;
        items.append(twoDArray[rowIndex][columnIndex]);
    }
} else {
    itemsInDiagonal--;
    for (int j = 0; j < itemsInDiagonal; j++) {
        rowIndex = (length - 1) - j;
        columnIndex = (i - length) + j;
        items.append(twoDArray[rowIndex][columnIndex]);
    }
}
```

## 4.结论

在本教程中，我们展示了如何使用一种有助于获取行和列索引的方法对角遍历一个正方形二维数组。

和往常一样，这个例子的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221206062814/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-multidimensional)