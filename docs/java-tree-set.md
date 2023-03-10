# Java 树集指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-tree-set>

## 1。概述

在本文中，我们将了解 Java 集合框架不可或缺的一部分以及最流行的`Set`实现之一`TreeSet` 。

## 2。`TreeSet`简介

简单地说，`TreeSet`是一个排序的集合，它扩展了`AbstractSet`类并实现了`NavigableSet`接口。

下面是该实现最重要方面的快速总结:

*   它储存独特的元素
*   它不保留元素的插入顺序
*   它按升序对元素进行排序
*   它不是线程安全的

**在这个实现中，对象根据它们的自然顺序以升序排序和存储**。`TreeSet`采用了自平衡二叉查找树，更具体地说[采用了`Red-Black`树](/web/20221126221455/https://www.baeldung.com/cs/red-black-trees)。

简单地说，作为一个自平衡二叉查找树，二叉树的每个节点包括一个额外的位，用于识别节点的颜色是红色还是黑色。在随后的插入和删除过程中，这些“颜色”位有助于确保树保持或多或少的平衡。

因此，让我们创建一个`TreeSet`的实例:

```java
Set<String> treeSet = new TreeSet<>();
```

### 2.1。带有构造函数比较器参数的树集

可选地，我们可以用一个构造函数构造一个`TreeSet`,让我们通过使用`Comparable`或`Comparator:`来定义元素排序的顺序

```java
Set<String> treeSet = new TreeSet<>(Comparator.comparing(String::length));
```

**虽然 `TreeSet` 不是线程安全的，但是它可以使用`Collections.synchronizedSet()`包装器:**在外部同步

```java
Set<String> syncTreeSet = Collections.synchronizedSet(treeSet);
```

好了，现在我们对如何创建一个`TreeSet`实例有了一个清晰的概念，让我们看看我们可用的常见操作。

## 3。 **`TreeSet`** **`add()`**

正如所料，`add()`方法可用于向`TreeSet`添加元素。如果添加了一个元素，该方法返回`true,`，否则返回`false.`

**该方法的契约声明，只有当`Set`中不存在某个元素时，才会添加该元素。**

让我们给一个`TreeSet`添加一个元素:

```java
@Test
public void whenAddingElement_shouldAddElement() {
    Set<String> treeSet = new TreeSet<>();

    assertTrue(treeSet.add("String Added"));
 }
```

**`add` 方法极其重要，因为该方法的实现细节说明了`TreeSet`如何在**内部工作，它如何利用`TreeMap's` `put`方法来存储元素:

```java
public boolean add(E e) {
    return m.put(e, PRESENT) == null;
}
```

变量`m`指的是内部支持`TreeMap`(注意`TreeMap`实现`NavigateableMap`):

```java
private transient NavigableMap<E, Object> m;
```

因此，`TreeSet` 在内部依赖于一个后备`NavigableMap`，当`TreeSet`的一个实例被创建时，这个后备`NavigableMap`用`TreeMap`的一个实例初始化:

```java
public TreeSet() {
    this(new TreeMap<E,Object>());
}
```

关于这一点的更多内容可以在[这篇文章](/web/20221126221455/https://www.baeldung.com/java-treemap)中找到。

## 4。`TreeSet contains()`

**`contains()`方法用于检查给定的`TreeSet`中是否存在给定的元素。**如果找到该元素，则返回 true，否则返回`false.`

让我们来看看`contains()`的行动:

```java
@Test
public void whenCheckingForElement_shouldSearchForElement() {
    Set<String> treeSetContains = new TreeSet<>();
    treeSetContains.add("String Added");

    assertTrue(treeSetContains.contains("String Added"));
}
```

## 5。` TreeSet remove()`

**`remove()`方法用于从集合中移除指定的元素，如果它存在的话。**

如果一个集合包含指定的元素，这个方法返回`true.`

让我们来看看它的实际应用:

```java
@Test
public void whenRemovingElement_shouldRemoveElement() {
    Set<String> removeFromTreeSet = new TreeSet<>();
    removeFromTreeSet.add("String Added");

    assertTrue(removeFromTreeSet.remove("String Added"));
}
```

## 6。`TreeSet clear()`

如果我们想从集合中删除所有的项目，我们可以使用`clear()` 方法:

```java
@Test
public void whenClearingTreeSet_shouldClearTreeSet() {
    Set<String> clearTreeSet = new TreeSet<>();
    clearTreeSet.add("String Added");
    clearTreeSet.clear();

    assertTrue(clearTreeSet.isEmpty());
}
```

## 7。`TreeSet size()`

`size()` 方法用于识别`TreeSet`中出现的元素数量。这是 API 中的基本方法之一:

```java
@Test
public void whenCheckingTheSizeOfTreeSet_shouldReturnThesize() {
    Set<String> treeSetSize = new TreeSet<>();
    treeSetSize.add("String Added");

    assertEquals(1, treeSetSize.size());
}
```

## 8。 `**TreeSet isEmpty()**`

`isEmpty()`方法可用于判断给定的`TreeSet`实例是否为空:

```java
@Test
public void whenCheckingForEmptyTreeSet_shouldCheckForEmpty() {
    Set<String> emptyTreeSet = new TreeSet<>();

    assertTrue(emptyTreeSet.isEmpty());
}
```

## 9。`TreeSet iterator()`

`iterator()`方法返回一个迭代器，该迭代器按照升序对`Set.` T2 中的元素进行迭代。这些迭代器是快速失效的。

我们可以在这里观察到递增的迭代顺序:

```java
@Test
public void whenIteratingTreeSet_shouldIterateTreeSetInAscendingOrder() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        System.out.println(itr.next());
    }
}
```

此外，`TreeSet` 使我们能够以降序遍历`Set`。

让我们看看实际情况:

```java
@Test
public void whenIteratingTreeSet_shouldIterateTreeSetInDescendingOrder() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.descendingIterator();
    while (itr.hasNext()) {
        System.out.println(itr.next());
    }
}
```

除了通过迭代器的`remove()`方法之外，在迭代器以任何方式创建后，如果集合被修改，则`Iterator`抛出`ConcurrentModificationException i`。

让我们为此创建一个测试:

```java
@Test(expected = ConcurrentModificationException.class)
public void whenModifyingTreeSetWhileIterating_shouldThrowException() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        itr.next();
        treeSet.remove("Second");
    }
} 
```

或者，如果我们使用了迭代器的 remove 方法，那么我们就不会遇到异常:

```java
@Test
public void whenRemovingElementUsingIterator_shouldRemoveElement() {

    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        String element = itr.next();
        if (element.equals("Second"))
           itr.remove();
    }

    assertEquals(2, treeSet.size());
}
```

迭代器的快速失效行为没有保证，因为在非同步并发修改的情况下不可能做出任何硬性保证。

关于这方面的更多信息可以在这里找到[。](/web/20221126221455/https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator)

## 10。`TreeSet first()`

如果不为空，这个方法返回第一个元素。否则抛出一个`NoSuchElementException`。

让我们看一个例子:

```java
@Test
public void whenCheckingFirstElement_shouldReturnFirstElement() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");

    assertEquals("First", treeSet.first());
}
```

## 11。`TreeSet last()`

与上面的示例类似，如果集合不为空，此方法将返回最后一个元素:

```java
@Test
public void whenCheckingLastElement_shouldReturnLastElement() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Last");

    assertEquals("Last", treeSet.last());
}
```

## 12。`TreeSet subSet()`

该方法将返回从`fromElement`到`toElement.` 的元素，注意`fromElement`是包含的，`toElement`是排他的:

```java
@Test
public void whenUsingSubSet_shouldReturnSubSetElements() {
    SortedSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> expectedSet = new TreeSet<>();
    expectedSet.add(2);
    expectedSet.add(3);
    expectedSet.add(4);
    expectedSet.add(5);

    Set<Integer> subSet = treeSet.subSet(2, 6);

    assertEquals(expectedSet, subSet);
}
```

## 13。`TreeSet headSet()`

该方法将返回小于指定元素的`TreeSet`元素:

```java
@Test
public void whenUsingHeadSet_shouldReturnHeadSetElements() {
    SortedSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> subSet = treeSet.headSet(6);

    assertEquals(subSet, treeSet.subSet(1, 6));
}
```

## 14。`TreeSet tailSet()`

该方法将返回大于或等于指定元素的`TreeSet`的元素:

```java
@Test
public void whenUsingTailSet_shouldReturnTailSetElements() {
    NavigableSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> subSet = treeSet.tailSet(3);

    assertEquals(subSet, treeSet.subSet(3, true, 6, true));
}
```

## 15。存储`Null`元素

**在 Java 7 之前，可以给空的** `**TreeSet.**` 添加`null` 元素

然而，这被认为是一个错误。因此，**T0 不再支持添加`null.`T3**

当我们将元素添加到`TreeSet,` 中时，这些元素根据它们的自然顺序或按照`comparator.`的指定进行排序，因此当与现有元素进行比较时，添加一个`null,`,导致一个`NullPointerException` ,因为`null`不能与任何值进行比较:

```java
@Test(expected = NullPointerException.class)
public void whenAddingNullToNonEmptyTreeSet_shouldThrowException() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add(null);
}
```

插入到`TreeSet`中的元素必须要么实现`Comparable`接口，要么至少被指定的比较器接受。**所有这样的元素必须是相互可比的，** `**i.e.**` **`e1.compareTo(e2)`或者`comparator.compare(e1, e2)`** **千万不能丢一个`ClassCastException`。**

让我们看一个例子:

```java
class Element {
    private Integer id;

    // Other methods...
}

Comparator<Element> comparator = (ele1, ele2) -> {
    return ele1.getId().compareTo(ele2.getId());
};

@Test
public void whenUsingComparator_shouldSortAndInsertElements() {
    Set<Element> treeSet = new TreeSet<>(comparator);
    Element ele1 = new Element();
    ele1.setId(100);
    Element ele2 = new Element();
    ele2.setId(200);

    treeSet.add(ele1);
    treeSet.add(ele2);

    System.out.println(treeSet);
}
```

## 16。`TreeSet` 的表现

与`HashSet` 相比，`TreeSet`的性能较低。像`add`、`remove`、`search`这样的操作需要`O(log n)`时间，而像按排序顺序打印`n`元素这样的操作需要`O(n)`时间。

如果我们希望保持条目的排序，那么 A `TreeSet`应该是我们的首选，因为可以按照升序或降序来访问和遍历`TreeSet`，并且升序操作和视图的性能可能比降序操作和视图更快。

局部性原则–是一个术语，指的是根据内存访问模式，相同的值或相关的存储位置被频繁访问的现象。

当我们说地点时:

*   应用程序经常以相似的频率访问相似的数据
*   如果给定一个排序，两个条目在附近，那么一个`TreeSet`将它们放在数据结构中彼此靠近的位置，从而放在内存中

A `TreeSet` 是一种具有更大局部性的数据结构，因此，根据局部性原则，我们可以得出这样的结论:如果我们的内存不足，并且如果我们想要访问根据自然顺序彼此相对接近的元素，我们应该优先考虑 a `TreeSet` 。

如果需要从硬盘读取数据(比从高速缓存或内存读取数据的延迟更长),则首选`TreeSet`,因为它具有更高的局部性

## 17。结论

在本文中，我们着重于理解如何在 Java 中使用标准的`TreeSet` 实现。我们看到了它的用途，以及它在可用性方面的效率，因为它能够避免重复和排序元素。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221126221455/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set)