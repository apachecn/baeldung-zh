# 在数组列表中的特定位置插入一个对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-insert-object-arraylist-specific-position>

## 1.概观

在本教程中，我们将学习如何在 [`ArrayList`](/web/20221116222839/https://www.baeldung.com/java-arraylist) 的特定位置插入一个对象。

## 2.例子

如果我们想在一个 [`ArrayList`](/web/20221116222839/https://www.baeldung.com/java-arraylist) 的特定位置添加一个元素，我们可以使用 **`add(int index, E element) `方法，这个方法是通过`List<E>`** 接口的实现提供的。这个方法让我们在特定索引处添加一个元素。

它还可以**在索引超出范围的情况下抛出`IndexOutOfBoundsException`(索引< 0 或索引> size())** 。这意味着如果我们在一个`ArrayList`中只有 4 个项目，我们不能用它在位置 4 添加一个项目，因为我们从 0 开始计数。在这里，我们必须使用标准的`add(E e) `方法。

首先，我们将创建一个新的`ArrayList`并向其添加四个元素:

```java
List<Integer> integers = new ArrayList<>();
integers.add(5);
integers.add(6);
integers.add(7);
integers.add(8);
System.out.println(integers);
```

这将导致:

[![](img/ce09f4309a4494690616d5e20c0bc6a6.png)](/web/20221116222839/https://www.baeldung.com/wp-content/uploads/2022/11/img_637528671724b.svg)

现在，如果我们在索引 1 处添加另一个元素:

```java
integers.add(1,9);
System.out.println(integers);
```

ArrayList 将首先在内部从给定的索引开始移动对象:

[![](img/5ab467fdaec3119459d5e6c86b3ebe90.png)](/web/20221116222839/https://www.baeldung.com/wp-content/uploads/2022/11/img_637528683cc07.svg)

这很有效，因为`ArrayList`是一个可增长的数组，可以根据需要自动调整容量大小:

[![](img/3562eacd86320c53dd77bdd0ffa981c2.png)](/web/20221116222839/https://www.baeldung.com/wp-content/uploads/2022/11/img_63752869916a0.svg)

然后在给定的索引处添加新项目:

[![](img/09561adbdb19df4e74ee37d3eec7febf.png)](/web/20221116222839/https://www.baeldung.com/wp-content/uploads/2022/11/img_6375286ad6a38.svg)

**增加一个特定的指标将导致`ArrayList`的平均**为 O(n/2)的操作性能。例如，A `[LinkedList](/web/20221116222839/https://www.baeldung.com/java-linkedlist),`平均复杂度为 O(n/4 ),如果索引为 0，复杂度为 O(1)。因此，如果我们严重依赖在特定位置添加元素，我们需要仔细研究一下`[LinkedList](/web/20221116222839/https://www.baeldung.com/java-linkedlist)`。

我们还可以看到元素的顺序不再正确。当我们在特定位置手动添加项目时，这是我们经常想要实现的。否则，我们可以使用`integers.sort(Integer::compareTo) `再次对`ArrayList`进行排序，或者实现我们自己的`Comparator.`

## 3.结论

在本文中，我们讨论了`add(int index, E element) `方法，因此我们可以在特定位置向`ArrayList<E>`添加新元素。我们必须注意保持在`ArrayList`的索引范围内，并确保我们允许正确的对象。

文章中提到的所有代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20221116222839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)