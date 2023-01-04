# Java 树形图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-treemap>

## 1。概述

在本文中，我们将探索 Java 集合框架(JCF)的`Map`接口的`TreeMap`实现。

`TreeMap`是一个 map 实现，它根据键的自然顺序对条目进行排序，如果用户在构建时提供了比较器，那么使用比较器会更好。

之前，我们已经介绍了 [`HashMap`](/web/20221003101503/https://www.baeldung.com/java-hashmap) 和 [`LinkedHashMap`](/web/20221003101503/https://www.baeldung.com/java-linked-hashmap) 的实现，我们会意识到有相当多的关于这些类如何工作的信息是相似的。

在阅读本文之前，强烈推荐阅读上述文章。

## 2。 `TreeMap`中的默认排序

默认情况下，`TreeMap`根据条目的自然顺序对其进行排序。对于整数，这意味着升序，对于字符串，这意味着字母顺序。

让我们看看测试中的自然排序:

```java
@Test
public void givenTreeMap_whenOrdersEntriesNaturally_thenCorrect() {
    TreeMap<Integer, String> map = new TreeMap<>();
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");

    assertEquals("[1, 2, 3, 4, 5]", map.keySet().toString());
}
```

注意，我们以无序的方式放置了整数键，但是在检索键集时，我们确认它们确实是以升序维护的。这是整数的自然排序。

同样，当我们使用字符串时，它们将按自然顺序排序，即按字母顺序排序:

```java
@Test
public void givenTreeMap_whenOrdersEntriesNaturally_thenCorrect2() {
    TreeMap<String, String> map = new TreeMap<>();
    map.put("c", "val");
    map.put("b", "val");
    map.put("a", "val");
    map.put("e", "val");
    map.put("d", "val");

    assertEquals("[a, b, c, d, e]", map.keySet().toString());
}
```

`TreeMap`与哈希映射和链接哈希映射不同，它不使用哈希原理，因为它不使用数组来存储条目。

## 3。`TreeMap` 中的自定义排序

如果我们对`TreeMap`的自然排序不满意，我们还可以在构建树形图时通过比较器定义自己的排序规则。

在下面的例子中，我们希望整数键按降序排列:

```java
@Test
public void givenTreeMap_whenOrdersEntriesByComparator_thenCorrect() {
    TreeMap<Integer, String> map = 
      new TreeMap<>(Comparator.reverseOrder());
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");

    assertEquals("[5, 4, 3, 2, 1]", map.keySet().toString());
}
```

哈希映射不保证存储的键的顺序，特别是不保证这个顺序会随着时间的推移保持不变，但是树映射保证键总是按照指定的顺序排序。

## 4。`TreeMap`排序的重要性

我们现在知道`TreeMap`按照排序顺序存储所有条目。由于树映射的这个属性，我们可以执行如下查询:找到“最大的”，找到“最小的”，找到所有小于或大于某个值的键，等等。

下面的代码只涵盖了这些情况中的一小部分:

```java
@Test
public void givenTreeMap_whenPerformsQueries_thenCorrect() {
    TreeMap<Integer, String> map = new TreeMap<>();
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");

    Integer highestKey = map.lastKey();
    Integer lowestKey = map.firstKey();
    Set<Integer> keysLessThan3 = map.headMap(3).keySet();
    Set<Integer> keysGreaterThanEqTo3 = map.tailMap(3).keySet();

    assertEquals(new Integer(5), highestKey);
    assertEquals(new Integer(1), lowestKey);
    assertEquals("[1, 2]", keysLessThan3.toString());
    assertEquals("[3, 4, 5]", keysGreaterThanEqTo3.toString());
}
```

## 5。`TreeMap`内部实现

`TreeMap`实现`NavigableMap`接口，内部工作基于[红黑树](/web/20221003101503/https://www.baeldung.com/cs/red-black-trees)的原理:

```java
public class TreeMap<K,V> extends AbstractMap<K,V>
  implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

红黑树的原理超出了本文的范围，但是，为了理解它们如何适应`TreeMap`，需要记住一些关键的事情。

**首先**，红黑树是由节点组成的数据结构；想象一棵倒置的芒果树，它的根在天上，树枝向下生长。根将包含添加到树中的第一个元素。

规则是从根开始，任何节点的左分支中的任何元素总是小于节点本身中的元素。右边的总是更大。定义大于或小于的内容是由元素的自然顺序或我们前面看到的构造时定义的比较器决定的。

这个规则保证了一个树形图的条目总是按照排序和可预测的顺序排列。

**其次**，一棵红黑树是一个自我平衡的二叉查找树。这个属性和上面的内容保证了像搜索、获取、放置和删除这样的基本操作需要对数时间`O(log n)`。

自我平衡是这里的关键。随着我们不断地插入和删除条目，想象一下树的一边变长了，另一边变短了。

这意味着操作在较短的分支上花费的时间较短，而在离根最远的分支上花费的时间较长，这是我们不希望发生的情况。

所以在红黑树的设计上就照顾到了这一点。对于每一次插入和删除，树在任何边上的最大高度保持在`O(log n)` 处，即树不断地自我平衡。

就像哈希映射和链接哈希映射一样，树映射是不同步的，因此在多线程环境中使用它的规则类似于其他两种映射实现中的规则。

## 6。选择正确的地图

看了以前的 [`HashMap`](/web/20221003101503/https://www.baeldung.com/java-hashmap) 和现在的`TreeMap`的 [`LinkedHashMap`](/web/20221003101503/https://www.baeldung.com/java-linked-hashmap) 实现之后，对这三者做一个简单的比较来指导我们哪一个适合哪里是很重要的。

**哈希映射**是一个很好的通用映射实现，可以提供快速的存储和检索操作。然而，它的缺点是条目排列混乱无序。

这导致它在有大量迭代的情况下性能很差，因为底层数组的整个容量影响遍历，而不仅仅是条目的数量。

**一个链接的散列图**拥有散列图的良好属性，并增加了条目的顺序。在有大量迭代的情况下，它表现得更好，因为不管容量如何，只考虑条目的数量。

**树形图**通过提供对键排序方式的完全控制，将排序提升到了一个新的层次。另一方面，它提供了比其他两种选择更差的总体性能。

我们可以说,**链接的散列图减少了散列图排序的混乱，而不会导致树图**的性能损失。

## 7。结论

在本文中，我们探索了 Java `TreeMap`类及其内部实现。因为它是一系列通用 Map 接口实现中的最后一个，所以我们也简单讨论了它与其他两个接口最适合的地方。

本文中使用的所有示例的完整源代码可以在 GitHub 项目中找到。