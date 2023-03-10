# Java 双指针技术

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-two-pointer-technique>

## 1.概观

在本教程中，我们将讨论解决数组和列表问题的双指针方法。这项技术是提高我们算法性能的一种简单而有效的方法。

## 2.技术描述

在许多涉及数组或列表的问题中，我们必须分析数组中的每个元素与其他元素的比较。

为了解决这样的问题，我们通常从第一个索引开始，根据我们的实现，遍历数组一次或多次。有时，我们还必须根据问题的需求创建一个临时数组。

上面的方法可能会给我们正确的结果，但它可能不会给我们最节省空间和时间的解决方案。

因此，考虑我们的问题是否可以通过使用`two-pointers approach`来有效地解决通常是很好的。

在双指针方法中，指针指的是数组的索引。通过使用指针，我们可以在每个循环中处理两个元素，而不是一个。

双指针方法中的常见模式包括:

*   两个指针分别从起点和终点开始，直到它们相遇
*   一个指针以较慢的速度移动，而另一个指针以较快的速度移动

上述两种模式都可以帮助我们减少问题的[](/web/20221106024153/https://www.baeldung.com/java-algorithm-complexity)**时间和空间复杂度，因为我们可以在更少的迭代中获得预期的结果，并且不需要使用太多的额外空间。**

 **现在，让我们看几个例子，帮助我们更好地理解这项技术。

## 3.总和存在于一个数组中

问题:给定一个排序的整数数组，我们需要看看其中是否有两个数使得它们的和等于一个特定值。

例如，如果我们的输入数组是`[1, 1, 2, 3, 4, 6, 8, 9]`，目标值是`11`，那么我们的方法应该返回`true`。但是，如果目标值是`20`，它应该返回`false`。

让我们先来看一个天真的解决方案:

```java
public boolean twoSumSlow(int[] input, int targetValue) {

    for (int i = 0; i < input.length; i++) {
        for (int j = 1; j < input.length; j++) {
            if (input[i] + input[j] == targetValue) {
                return true;
            }
        }
    }
    return false;
}
```

在上面的解决方案中，我们对输入数组进行了两次循环，以获得所有可能的组合。我们对照目标值检查了组合和，如果匹配就返回`true`。**这个解的时间复杂度是`O(n^2)`** 。

现在让我们看看如何在这里应用两点技术:

```java
public boolean twoSum(int[] input, int targetValue) {

    int pointerOne = 0;
    int pointerTwo = input.length - 1;

    while (pointerOne < pointerTwo) {
        int sum = input[pointerOne] + input[pointerTwo];

        if (sum == targetValue) {
            return true;
        } else if (sum < targetValue) {
            pointerOne++;
        } else {
            pointerTwo--;
        }
    }

    return false;
}
```

由于数组已经排序，我们可以使用两个指针。一个指针从数组的开头开始，另一个指针从数组的结尾开始，然后我们把这些指针上的值相加。如果这些值的总和小于目标值，我们递增左指针，如果总和大于目标值，我们递减右指针。

我们一直移动这些指针，直到我们得到与目标值匹配的和，或者我们已经到达数组的中间，并且没有找到任何组合。**这个方案的时间复杂度是`O(n)` 和 空间复杂度是`O(1)`**`, `比我们第一次实现有了显著的改进。

## 4.旋转阵列`k`步

问题:给定一个数组，将数组向右旋转`k`步，其中`k`为非负。比如我们的输入数组是`[1, 2, 3, 4, 5, 6, 7]`，`k`是`4`，那么输出应该是`[4, 5, 6, 7, 1, 2, 3]`。

我们可以通过再次使用两个循环来解决这个问题，这将增加时间复杂度`O(n^2)`，或者使用一个额外的临时数组，但这将增加空间复杂度`O(n)`。

让我们使用双指针技术来解决这个问题:

```java
public void rotate(int[] input, int step) {
    step %= input.length;
    reverse(input, 0, input.length - 1);
    reverse(input, 0, step - 1);
    reverse(input, step, input.length - 1);
}

private void reverse(int[] input, int start, int end) {
    while (start < end) {
        int temp = input[start];
        input[start] = input[end];
        input[end] = temp;
        start++;
        end--;
    }
}
```

在上述方法中，我们多次就地反转输入数组的各个部分，以获得所需的结果。为了反转这些部分，我们使用了双指针方法，在数组部分的两端交换元素。

具体来说，我们首先反转数组的所有元素。然后，我们反转第一个`k`元素，接着反转其余的元素。**此解的时间复杂度为`O(n)`****空间复杂度为`O(1)`** 。

## 5.a 中的中间元素`LinkedList`

问题:给定一个单独的`LinkedList`，求它的中间元素。例如，如果我们的输入`LinkedList`是`1->2->3->4->5,`，那么输出应该是`3`。

我们还可以在类似于数组的其他数据结构中使用双指针技术，如`LinkedList`:

```java
public <T> T findMiddle(MyNode<T> head) {
    MyNode<T> slowPointer = head;
    MyNode<T> fastPointer = head;

    while (fastPointer.next != null && fastPointer.next.next != null) {
        fastPointer = fastPointer.next.next;
        slowPointer = slowPointer.next;
    }
    return slowPointer.data;
}
```

在这种方法中，我们使用两个指针遍历链表。一个指针增加 1，而另一个指针增加 2。当快速指针到达末尾时，慢速指针将位于链表的中间。**此解时间复杂度为`O(n)` ， 空间复杂度为`O(1)`。**

## 6.结论

在本文中，我们通过一些例子讨论了如何应用双指针技术，并研究了它如何提高我们算法的效率。

本文中的代码可以在 Github 上的[处获得。](https://web.archive.org/web/20221106024153/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)**