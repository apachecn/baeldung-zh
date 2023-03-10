# Java 中如何计算 Levenshtein 距离？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-levenshtein-distance>

## 1。简介

在本文中，我们描述了 Levenshtein 距离，也称为编辑距离。这里解释的算法是由俄罗斯科学家 Vladimir Levenshtein 在 1965 年设计的。

我们将提供该算法的迭代和递归 Java 实现。

## 2。什么是 Levenshtein 距离？

[Levenshtein 距离](/web/20220626104153/https://www.baeldung.com/cs/levenshtein-distance-computation)是两个`Strings.`之间不相似性的度量，在数学上，给定两个`Strings` *x* 和 *y* ，该距离度量将`x`转换为`y`所需的最小字符编辑数。

通常允许三种类型编辑:

1.  插入一个字符`c`
2.  删除一个字符`c`
3.  用`c`替换字符`c`

例如:**如果`x = ‘shot'`和 `y = ‘spot'`，两者之间的编辑距离为 1，因为通过将`h`替换为`p`，可以将`‘shot'`转换为`‘spot'`。**

在问题的某些子类中，与每种类型的编辑相关联的成本可能是不同的。

例如，用位于键盘附近的字符替换成本较低，否则成本较高。为了简单起见，在本文中我们将认为所有的成本都是相等的。

编辑距离的一些应用有:

1.  拼写检查器-检测文本中的拼写错误，并在字典中找到最接近的正确拼写
2.  剽窃检测(参考–[IEEE 论文](https://web.archive.org/web/20220626104153/http://ieeexplore.ieee.org/document/4603758/)
3.  DNA 分析——寻找两个序列之间的相似性
4.  语音识别(参考–[微软研究院](https://web.archive.org/web/20220626104153/https://www.microsoft.com/en-us/research/publication/context-dependent-phonetic-string-edit-distance-for-automatic-speech-recognition/))

## 3。算法公式

让我们分别取两个长度为`m`和`n`的`Strings x`和`y`。我们可以将每个`String`表示为`x[1:m]`和 `y[1:n].`

我们知道，在转换结束时，两个`Strings`的长度相等，并且在每个位置都有匹配的字符。因此，如果我们考虑每个`String,`的第一个字符，我们有三个选项:

1.  替代:
    1.  确定用`y[1]`代替`x[1]`的成本( *D1* )。如果两个字符相同，这一步的成本将为零。如果不是，那么成本就是 1
    2.  在步骤 1.1 之后，我们知道两个`Strings`以相同的字符开始。因此，总成本现在将是步骤 1.1 的成本和将剩余的`String x[2:m]`转换为`y[2:n]`的成本之和
2.  插入:
    1.  在`x`中插入一个字符来匹配`y`中的第一个字符，这一步的成本是 1
    2.  在 2.1 之后，我们已经处理了来自`y`的一个字符。因此，总成本现在将是步骤 2.1 的成本(即 1)和将全部`x[1:m]`转换为剩余`y (y[2:n])`的成本之和
3.  删除:
    1.  删除`x`中的第一个字符，这一步的成本是 1
    2.  在 3.1 之后，我们已经处理了从`x`开始的一个字符，但是完整的`y`还有待处理。总成本将是 3.1 的成本(即 1)和将剩余的`x`转换为全部`y`的成本之和

解决方案的下一部分是找出从这三个选项中选择哪个选项。因为我们不知道哪一个选择最终会导致最小的成本，所以我们必须尝试所有的选择并选择最好的一个。

## 4。简单递归实现

我们可以看到，第 3 节中每个选项的第二步主要是相同的编辑距离问题，但在原始`Strings.`的子字符串上，这意味着每次迭代后，我们都以相同的问题结束，但具有更小的`Strings.`

这种观察是形成递归算法的关键。递归关系可以定义为:

`D(x[1:m], y[1:n])` `= min {`

`D(x[2:m], y[2:n]) + Cost of Replacing x[1] to y[1],`

`D(x[1:m], y[2:n]) + 1,`

`D(x[2:m], y[1:n]) + 1`

`}`

我们还必须为我们的递归算法定义基本情况，在我们的例子中是当一个或两个`Strings`为空时:

1.  当两个`Strings`都为空时，它们之间的距离为零
2.  当其中一个`Strings`为空时，它们之间的编辑距离是另一个`String,`的长度，因为我们需要许多插入/删除来将一个转换成另一个:
    *   例如:如果一个`String`是`“dog”`，另一个`String`是`“”`(空的)，我们需要在空的`String`中插入三个来使其成为`“dog”`，或者我们需要在`“dog”`中删除三个来使其成为空的。因此，它们之间的编辑距离是 3

该算法的简单递归实现:

```java
public class EditDistanceRecursive {

   static int calculate(String x, String y) {
        if (x.isEmpty()) {
            return y.length();
        }

        if (y.isEmpty()) {
            return x.length();
        } 

        int substitution = calculate(x.substring(1), y.substring(1)) 
         + costOfSubstitution(x.charAt(0), y.charAt(0));
        int insertion = calculate(x, y.substring(1)) + 1;
        int deletion = calculate(x.substring(1), y) + 1;

        return min(substitution, insertion, deletion);
    }

    public static int costOfSubstitution(char a, char b) {
        return a == b ? 0 : 1;
    }

    public static int min(int... numbers) {
        return Arrays.stream(numbers)
          .min().orElse(Integer.MAX_VALUE);
    }
}
```

**该算法具有指数复杂度。**在每一步，我们分支成三个递归调用，构建一个`O(3^n)`复杂度。

在下一节中，我们将看到如何对此进行改进。

## 5。动态规划方法

在分析递归调用时，我们观察到子问题的自变量是原始`Strings.`的后缀，这意味着只能有`m*n`个唯一的递归调用(其中`m`和`n`是多个`x`和`y`的后缀)。因此最优解的复杂度应该是二次的，`O(m*n)`。

让我们看看一些子问题(根据第 4 节中定义的递归关系):

1.  `D(x[1:m], y[1:n])` 的子问题是`D(x[2:m], y[2:n]), D(x[1:m], y[2:n])` 和`D(x[2:m], y[1:n])`
2.  `D(x[1:m], y[2:n])` 的子问题是`D(x[2:m], y[3:n]), D(x[1:m], y[3:n])` 和`D(x[2:m], y[2:n])`
3.  `D(x[2:m], y[1:n])` 的子问题是`D(x[3:m], y[2:n]), D(x[2:m], y[2:n])` 和`D(x[3:m], y[1:n])`

在所有三种情况下，其中一个子问题是`D(x[2:m], y[2:n]).` ,而不是像我们在简单实现中那样计算三次，我们可以计算一次，并在需要时再次使用结果。

这个问题有很多重叠的子问题，但是如果知道子问题的解法，就很容易找到原问题的答案。因此，我们拥有了制定动态规划解所需的两个性质，即[重叠子问题](https://web.archive.org/web/20220626104153/https://en.wikipedia.org/wiki/Overlapping_subproblems)和[最优子结构](https://web.archive.org/web/20220626104153/https://en.wikipedia.org/wiki/Optimal_substructure)。

我们可以通过引入[记忆化](https://web.archive.org/web/20220626104153/https://en.wikipedia.org/wiki/Memoization)来优化朴素实现，即，将子问题的结果存储在一个数组中，并重用缓存的结果。

或者，我们也可以通过使用基于表格的方法迭代地实现这一点:

```java
static int calculate(String x, String y) {
    int[][] dp = new int[x.length() + 1][y.length() + 1];

    for (int i = 0; i <= x.length(); i++) {
        for (int j = 0; j <= y.length(); j++) {
            if (i == 0) {
                dp[i][j] = j;
            }
            else if (j == 0) {
                dp[i][j] = i;
            }
            else {
                dp[i][j] = min(dp[i - 1][j - 1] 
                 + costOfSubstitution(x.charAt(i - 1), y.charAt(j - 1)), 
                  dp[i - 1][j] + 1, 
                  dp[i][j - 1] + 1);
            }
        }
    }

    return dp[x.length()][y.length()];
} 
```

**该算法的性能明显优于递归实现。但是，这涉及到大量的内存消耗。**

这可以通过观察我们只需要表格中三个相邻单元格的值来找到当前单元格的值来进一步优化。

## 6。结论

在本文中，我们描述了什么是 Levenshtein 距离，以及如何使用基于递归和动态编程的方法来计算它。

Levenshtein 距离只是字符串相似性的度量之一，其他一些度量是[余弦相似性](https://web.archive.org/web/20220626104153/https://en.wikipedia.org/wiki/Cosine_similarity)(它使用基于令牌的方法，并将字符串视为向量)、[骰子系数](https://web.archive.org/web/20220626104153/https://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient)等。

像往常一样，例子的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220626104153/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-1)