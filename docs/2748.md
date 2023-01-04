# Collection.clear()和 Collection.removeAll()之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collection-clear-vs-removeall>

## 1。概述

在这个快速教程中，我们将学习两个`Collection`方法，它们看起来可能做同样的事情，但事实并非如此:`clear()`和`removeAll()`。

我们将首先看到方法定义，然后在简短的例子中使用它们。

## T2`2\. Collection.clear()`

我们将首先深入研究`Collection.clear()`方法。让我们检查一下方法的 Javadoc。根据它的说法，***clear()*的目的是从列表中删除每一个单独的元素。**

所以，基本上，在任何列表上调用`clear()`都会导致列表变空。

## T2`3\. Collection.removeAll()`

我们现在来看看`Collection.removeAll()`的 [Javadoc](https://web.archive.org/web/20220626195156/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#removeAll(java.util.Collection)) 。我们可以看到该方法将一个`Collection`作为参数。**及其目的是** **删除列表和集合之间所有的公共元素。**

因此，当在集合上调用它时，它将从传递的参数中移除所有元素，这些元素也在我们调用`removeAll()`的集合中。

## 4。示例

现在让我们来看一些代码，看看这些方法是如何工作的。我们将首先创建一个名为`ClearVsRemoveAllUnitTest`的测试类。

之后，我们将为`Collection.clear()`创建第一个测试。

我们将用几个数字初始化一个`Integers`集合，并对其调用`clear()`,这样列表中就没有元素了:

```
@Test
void whenClear_thenListBecomesEmpty() {
    Collection<Integer> collection = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

    collection.clear();

    assertTrue(collection.isEmpty());
}
```

正如我们可以看到的，在调用`clear()`之后，集合是空的。

让我们用两个集合创建第二个测试，一个包含从 1 到 5 的数字，另一个包含从 3 到 7 的数字。之后，我们将调用第一个集合上的`removeAll()`,并将第二个集合作为参数。

我们希望第一个集合中只保留数字 1 和 2(而第二个集合保持不变):

```
@Test
void whenRemoveAll_thenFirstListMissElementsFromSecondList() {
    Collection<Integer> firstCollection = new ArrayList<>(
      Arrays.asList(1, 2, 3, 4, 5));
    Collection<Integer> secondCollection = new ArrayList<>(
      Arrays.asList(3, 4, 5, 6, 7));

    firstCollection.removeAll(secondCollection);

    assertEquals(
      Arrays.asList(1, 2), 
      firstCollection);
    assertEquals(
      Arrays.asList(3, 4, 5, 6, 7), 
      secondCollection);
}
```

我们的期望达到了。第一个集合中只剩下数字 1 和 2，第二个集合没有改变。

## 5。结论

在本文中，我们已经看到了`Collection.clear()`和`Collection.removeAll()`的用途。

不管我们一开始怎么想，他们做的并不是同样的事情。`clear()`删除集合中的所有元素，而`removeAll()`只删除与另一个`Collection`中的元素相匹配的元素。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626195156/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)