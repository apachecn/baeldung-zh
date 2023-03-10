# 在 Java 中用 for 循环创建三角形

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-print-triangle>

## 1.介绍

在本教程中，我们将探索几种用 Java 打印三角形的方法。

自然，三角形有很多种。在这里，我们将只探索其中的几种:直角三角形和等腰三角形。

## 2.构建一个直角三角形

直角三角形是我们将要学习的最简单的三角形。让我们快速看一下我们想要获得的输出:

```java
*
**
***
****
*****
```

这里，我们注意到三角形由 5 行组成，每一行都有与当前行数相等的星。当然，这个观察可以推广:**对于从 1 到`N`的每一行，我们都要打印`r`颗星，其中`r`是当前行，`N`是总行数。**

因此，让我们使用两个`for`循环来构建三角形:

```java
public static String printARightTriangle(int N) {
    StringBuilder result = new StringBuilder();
    for (int r = 1; r <= N; r++) {
        for (int j = 1; j <= r; j++) {
            result.append("*");
        }
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

## 3.构建等腰三角形

现在，让我们来看看等腰三角形的形状:

```java
 *
   ***
  *****
 *******
*********
```

在这个案例中我们看到了什么？我们注意到，**除了星星，我们还需要为每一行打印一些空格。**因此，我们必须计算出每行要打印多少个空格和星号。当然，空格和星号的数量取决于当前行。

首先，我们看到我们需要为第一行打印 4 个空格，当我们沿着三角形向下时，我们需要 3 个空格，2 个空格，1 个空格，最后一行没有空格。**概括地说，我们需要为每一行**打印`N – r`个空格。

第二，与第一个例子相比，我们意识到这里我们需要奇数颗星星:1，3，5，7…

所以，**我们需要为每一行**打印`r x 2 – 1`颗星星。

### 3.1.使用嵌套的`for`循环

基于以上观察，让我们创建第二个示例:

```java
public static String printAnIsoscelesTriangle(int N) {
    StringBuilder result = new StringBuilder();
    for (int r = 1; r <= N; r++) {
        for (int sp = 1; sp <= N - r; sp++) {
            result.append(" ");
        }
        for (int c = 1; c <= (r * 2) - 1; c++) {
            result.append("*");
        }
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

### 3.2.使用单个`for`回路

实际上，我们还有另一种方法，即**只包含一个`for`循环——它使用了 [Apache Commons Lang 3 库](/web/20221205174257/https://www.baeldung.com/java-commons-lang-3)。**

我们将使用 for 循环来迭代三角形的行，就像我们在前面的例子中所做的那样。然后，我们将使用`StringUtils.repeat()`方法为每一行生成必要的字符:

```java
public static String printAnIsoscelesTriangleUsingStringUtils(int N) {
    StringBuilder result = new StringBuilder();

    for (int r = 1; r <= N; r++) {
        result.append(StringUtils.repeat(' ', N - r));
        result.append(StringUtils.repeat('*', 2 * r - 1));
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

或者，我们可以用**[`substring()`方法](/web/20221205174257/https://www.baeldung.com/java-substring)来做一个巧妙的把戏。**

我们可以提取上面的`StringUtils.repeat()`方法来构建一个助手字符串，然后对它应用`String.substring()`方法。**辅助字符串是我们打印三角形行所需的最大空格数和最大星号数的串联。**

查看前面的示例，我们注意到第一行需要最大数量的`N – 1`个空格，最后一行需要最大数量的`N x 2 – 1`个星号:

```java
String helperString = StringUtils.repeat(' ', N - 1) + StringUtils.repeat('*', N * 2 - 1);
// for N = 10, helperString = "    *********"
```

例如，当`N = 5`和`r = 3`时，我们需要打印“*****”，它包含在`helperString`变量中。我们需要做的就是为`substring()`方法找到正确的公式。

现在，让我们看看完整的示例:

```java
public static String printAnIsoscelesTriangleUsingSubstring(int N) {
    StringBuilder result = new StringBuilder();
    String helperString = StringUtils.repeat(' ', N - 1) + StringUtils.repeat('*', N * 2 - 1);

    for (int r = 0; r < N; r++) {
        result.append(helperString.substring(r, N + 2 * r));
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

同样，只要再多做一点工作，我们就可以把三角形打印出来。

## 4.复杂性

如果我们再看一下第一个例子，我们会注意到一个外循环和一个内循环，每个循环都有最大的`N`步。**因此，我们有`O(N^2)`时间复杂度，其中`N`是三角形的行数。**

第二个例子是相似的——唯一的区别是我们有两个内部循环，它们是连续的，不会增加时间复杂度。

然而，第三个例子只使用了一个带有`N`步骤的`for`循环。但是，在每一步，我们都在助手字符串上调用`StringUtils.repeat()`方法或`substring()`方法，每个方法都有`O(N)`复杂性。所以，总的时间复杂度保持不变。

最后，如果我们在谈论辅助空间，我们可以很快意识到，对于所有的例子，复杂性停留在 [`StringBuilder`](/web/20221205174257/https://www.baeldung.com/java-string-builder-string-buffer) 变量中。**通过将整个三角形添加到`result`变量中，我们的复杂度不能低于`O(N^2)`。**

当然，如果我们直接打印字符，前两个例子的空间复杂度是不变的。但是，第三个例子使用了辅助字符串，空间复杂度是`O(N)`。

## 5.结论

在本教程中，我们学习了如何用 Java 打印两种常见的三角形。

首先，**我们已经学习了直角三角形，这是我们可以用 Java 打印的最简单的三角形类型。**然后，**我们探索了两种构建等腰三角形的方法。第一个只使用了`for`循环，另一个利用了`StringUtils.repeat()`和`String.substring()`方法，帮助我们编写更少的代码。**

最后，我们分析了每个例子的时间和空间复杂度。

和往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221205174257/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)