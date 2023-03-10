# 替换 Java 数组列表中特定索引处的元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraylist-replace-at-index>

## 1。概述

通过本教程，我们将了解如何在 Java `ArrayList`中替换特定索引处的元素。

## 2。惯例

要替换一个现有的元素，首先，我们需要[找到那个元素](/web/20220923180544/https://www.baeldung.com/find-list-element-java)在`ArrayList`中的准确位置。这个位置就是我们所说的指数。然后，我们可以用新元素替换旧元素。

**在 Java `ArrayList`中替换元素最常见的方法是使用`set (int index, Object element)`方法**。`set()`方法有两个参数:现有项目和新项目的索引。

一个`ArrayList`的索引是从零开始的。因此，要替换第一个元素，0 必须是作为参数传递的索引。

**如果提供的索引超出范围**，将发生`IndexOutOfBoundsException`。

## 3。 **实现**

让我们通过一个例子来看看如何在 Java `ArrayList`中的特定索引处替换一个元素。

```java
List<Integer> EXPECTED = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

List<Integer> aList = new ArrayList<>(Arrays.asList(1, 2, 7, 4, 5));
aList.set(2, 3);

assertThat(aList).isEqualTo(EXPECTED);
```

首先，我们创建一个包含五个元素的`ArrayList`。然后，我们用值 7 替换第三个元素，索引 2 为 3。最后，我们可以看到值为 7 的索引 2 从列表中删除，并用新值 3 更新。另外，请注意，列表大小不受影响。

## 4。结论

在这篇简短的文章中，我们学习了如何在 Java `ArrayList`中替换特定索引处的元素。此外，您可以将此方法用于任何其他的`List`类型，如`LinkedList`。只要确保您使用的`List`不是不可变的。

和往常一样，本文的完整源代码可以在 GitHub 上找到。