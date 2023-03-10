# 从列表中移除第一个元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-first-element-from-list>

## 1。概述

在这个快速教程中，我们将展示如何从`List`中移除第一个元素。

我们将为`List`接口的两个常见实现——`ArrayList`和`LinkedList`执行该操作。

## 2。创造一个`List`

首先，让我们填充我们的`List` s:

```java
@Before
public void init() {
    list.add("cat");
    list.add("dog");
    list.add("pig");
    list.add("cow");
    list.add("goat");

    linkedList.add("cat");
    linkedList.add("dog");
    linkedList.add("pig");
    linkedList.add("cow");
    linkedList.add("goat");
}
```

## 3。`ArrayList`

其次，让我们从`ArrayList,`中移除第一个元素，并确保我们的列表不再包含它:

```java
@Test
public void givenList_whenRemoveFirst_thenRemoved() {
    list.remove(0);

    assertThat(list, hasSize(4));
    assertThat(list, not(contains("cat")));
}
```

如上所示，我们使用`remove(index)`方法移除第一个元素——**,这也适用于`List`接口的任何实现。**

## 4。`LinkedList`

`LinkedList`也实现了`remove(index)`方法(以它自己的方式),但是它也有`removeFirst()`方法。

让我们确保它按预期工作:

```java
@Test
public void givenLinkedList_whenRemoveFirst_thenRemoved() {
    linkedList.removeFirst();

    assertThat(linkedList, hasSize(4));
    assertThat(linkedList, not(contains("cat")));
}
```

## 5。时间复杂度

尽管这些方法看起来相似，但它们的效率不同。`ArrayList`的`remove()`方法需要 O(n)时间，而`LinkedList`的`removeFirst()`方法需要 O(1)时间。

这是因为`ArrayList`使用了一个隐藏的数组，而`remove()`操作需要将数组的其余部分复制到开头。数组越大，需要移动的元素就越多。

与此不同，`LinkedList`使用指针，这意味着每个元素指向下一个和前一个元素。

因此，删除第一个元素意味着只改变指向第一个元素的指针。该操作总是需要相同的时间，而不依赖于列表的大小。

## 6。结论

在本文中，我们已经介绍了如何从一个`List,`中移除第一个元素，并且比较了这个操作对于`ArrayList`和`LinkedList `实现的效率。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126222929/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)