# 查找数组中最小的缺失整数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-smallest-missing-integer-in-array>

## 1.概观

在本教程中，我们将看到不同的算法，让我们找到一个数组中最小的正整数。

首先，我们来看一下对问题的解释。之后，我们会看到三种不同的算法满足我们的需求。最后，我们将讨论它们的复杂性。

## 2.问题解释

首先，我们来解释一下算法的目标是什么。我们想在一个正整数数组中寻找最小的丢失的正整数。**即在一个由`x`个元素组成的数组中，找出`0` 和`x – 1`之间不在数组中的最小元素。**如果数组包含全部，那么解就是`x`，数组大小。

例如，我们来考虑下面这个数组:`[0, 1, 3, 5, 6]`。它有`5`个元素。这意味着我们在寻找不在这个数组中的在`0`和`4`之间的最小整数。在这个具体的例子中，是`2`。

现在，让我们想象另一个数组:`[0, 1, 2, 3]`。因为它有`4`个元素，所以我们在寻找一个在`0` 和`3`之间的整数。没有遗漏，因此不在数组中的最小整数是`4`。

## 3.分类数组

现在，让我们看看如何在一个排序数组中找到最小的缺失数。在一个排序数组中，最小的缺失整数将是第一个不将自身作为值的索引。

我们来考虑下面这个排序后的数组:`[0, 1, 3, 4, 6, 7]`。现在，让我们看看哪个值匹配哪个索引:

```java
Index: 0 1 2 3 4 5
Value: 0 1 3 4 6 7
```

正如我们所见，值索引不保存整数`2`，因此`2`是数组中最小的缺失整数。

用 Java 实现这个算法怎么样？让我们首先用方法`searchInSortedArray()`创建一个类`SmallestMissingPositiveInteger`:

```java
public class SmallestMissingPositiveInteger {
    public static int searchInSortedArray(int[] input) {
        // ...
    }
}
```

现在，我们可以迭代数组，**搜索第一个不包含自身作为值**的索引，并将其作为结果返回:

```java
for (int i = 0; i < input.length; i++) {
    if (i != input[i]) {
        return i;
    }
}
```

最后，**如果我们完成了循环而没有找到丢失的元素，我们必须返回下一个整数，这是数组长度**，因为我们从索引`0`开始:

```java
return input.length;
```

让我们检查一下这一切是否如预期的那样工作。想象一个从`0`到`5`的整数数组，缺少数字`3`:

```java
int[] input = new int[] {0, 1, 2, 4, 5};
```

然后，如果我们搜索第一个丢失的整数，应该返回`3`:

```java
int result = SmallestMissingPositiveInteger.searchInSortedArray(input);

assertThat(result).isEqualTo(3);
```

但是，如果我们在一个没有丢失任何整数的数组中搜索一个丢失的数字:

```java
int[] input = new int[] {0, 1, 2, 3, 4, 5};
```

我们会发现第一个缺失的整数是`6`，它是数组的长度:

```java
int result = SmallestMissingPositiveInteger.searchInSortedArray(input);

assertThat(result).isEqualTo(input.length);
```

接下来，我们将看到如何处理未排序的数组。

## 4.未排序数组

那么，在一个未排序的数组中寻找最小的缺失整数呢？有多种解决方案。第一种是先简单地对数组进行排序，然后重用我们之前的算法。另一种方法是使用另一个数组来标记存在的整数，然后遍历该数组来查找第一个丢失的整数。

### 4.1.首先对数组进行排序

让我们从第一个解决方案开始，创建一个新的`searchInUnsortedArraySortingFirst()`方法。

**因此，我们将重用我们的算法，但首先，我们需要对输入数组进行排序。**为了做到这一点，我们将使用`Arrays.sort()`:

```java
Arrays.sort(input);
```

该方法根据输入的自然顺序对其进行排序。对于整数，这意味着从最小到最大。在我们关于 Java 中的[排序数组的文章中有更多关于排序算法的细节。](/web/20221208143956/https://www.baeldung.com/java-sorting-arrays)

之后，我们可以用现在排序的输入调用我们的算法:

```java
return searchInSortedArray(input);
```

就这样，我们现在可以检查一切是否如预期的那样工作。让我们想象一下下面的数组，其中有未排序的整数和缺少的数字`1`和`3`:

```java
int[] input = new int[] {4, 2, 0, 5};
```

由于`1`是最小的缺失整数，我们希望它是调用我们的方法的结果:

```java
int result = SmallestMissingPositiveInteger.searchInUnsortedArraySortingFirst(input);

assertThat(result).isEqualTo(1);
```

现在，让我们在一个没有遗漏数字的数组上尝试一下:

```java
int[] input = new int[] {4, 5, 1, 3, 0, 2};

int result = SmallestMissingPositiveInteger.searchInUnsortedArraySortingFirst(input);

assertThat(result).isEqualTo(input.length);
```

就是这样，算法返回`6`，那就是数组长度。

### 4.2.使用布尔数组

另一种可能是使用另一个数组——与输入数组长度相同——它保存了`boolean`值，告诉我们是否在输入数组中找到了与索引匹配的整数。

首先，让我们创建第三个方法，`searchInUnsortedArrayBooleanArray()`。

之后，让我们创建布尔数组，`flags`和**对于输入数组中与`boolean`数组的索引相匹配的每个整数，我们将相应的值设置为`true`** :

```java
boolean[] flags = new boolean[input.length];
for (int number : input) {
    if (number < flags.length) {
        flags[number] = true;
    }
}
```

现在，我们的`flags`数组为输入数组中的每个整数保存`true`，否则保存`false`。然后，我们可以**遍历`flags`数组并返回第一个索引保存`false`** 。如果没有，我们返回数组长度:

```java
for (int i = 0; i < flags.length; i++) {
    if (!flags[i]) {
        return i;
    }
}

return flags.length;
```

同样，让我们用我们的例子来试试这个算法。我们将首先重用缺少`1`和`3`的数组:

```java
int[] input = new int[] {4, 2, 0, 5};
```

然后，当用我们的新算法搜索最小的缺失整数时，答案仍然是`1`:

```java
int result = SmallestMissingPositiveInteger.searchInUnsortedArrayBooleanArray(input);

assertThat(result).isEqualTo(1);
```

对于整个数组，答案也没有改变，仍然是`6`:

```java
int[] input = new int[] {4, 5, 1, 3, 0, 2};

int result = SmallestMissingPositiveInteger.searchInUnsortedArrayBooleanArray(input);

assertThat(result).isEqualTo(input.length);
```

## 5.复杂性

现在我们已经讨论了算法，让我们使用[大 O 符号](/web/20221208143956/https://www.baeldung.com/big-o-notation)来讨论它们的复杂性。

### 5.1.分类数组

让我们从第一个算法开始，它的输入已经排序。在这种情况下，最坏的情况是找不到丢失的整数，因此遍历整个数组。这意味着**我们有线性复杂度**，记为`O(n)`，考虑到`n `是我们输入的长度。

### 5.2.带有排序算法的未排序数组

现在，让我们考虑我们的第二个算法。在这种情况下，输入数组是不排序的，我们在应用第一个算法之前对它进行排序。这里，**的复杂度将是排序机制的复杂度和算法本身的复杂度之间最大的**。

从 Java 11 开始，`Arrays.sort()`方法使用双支点[快速排序算法](/web/20221208143956/https://www.baeldung.com/java-quicksort)对数组进行排序。一般来说，这种排序算法的复杂度是`O(n log(n))`，尽管它可以降低到`O(n²)`。这意味着**我们算法的复杂度通常是`O(n log(n))`，并且也可以降低到`O(n²)`** 的二次复杂度。

这是针对时间复杂性的，但我们不要忘记空间。虽然搜索算法不会占用额外的空间，但排序算法会。**快速排序算法需要占用`O(log(n))`空间来执行。**这是我们在为大型数组选择算法时可能要考虑的事情。

### 5.3.带有布尔数组的未排序数组

最后，让我们看看第三个也是最后一个算法的表现。对于这一个，我们不排序输入数组，这意味着**我们不会遭受排序的复杂性**。事实上，我们只遍历了两个大小相同的数组。那就是说**我们的时间复杂度应该是`O(2n)`，简化为`O(n)`** 。这比之前的算法要好。

但是，当涉及到空间复杂性时，我们将创建与输入大小相同的第二个数组。这意味着**我们有`O(n)`空间复杂度**，比之前的算法更差。

知道了所有这些，我们就可以根据使用的条件来选择最适合我们需求的算法。

## 6.结论

在本文中，我们研究了寻找数组中最小缺失正整数的算法。我们已经看到了如何在有序数组和无序数组中实现这一点。我们还讨论了不同算法的时间和空间复杂性，允许我们根据自己的需要明智地选择一种算法。

像往常一样，本文中显示的完整代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)