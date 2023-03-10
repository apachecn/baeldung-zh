# Java 树形图 vs HashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-treemap-vs-hashmap>

## 1。简介

在本文中，我们将比较两个`Map`实现:`TreeMap`和`HashMap`。

这两种实现都构成了 Java `Collections`框架不可分割的一部分，并将数据存储为 `key-value`对。

## 2。差异

### 2.1。实施

我们将首先讨论`HashMap` ，它是一个基于哈希表的实现。它扩展了`AbstractMap` 类并实现了`Map`接口。一个`HashMap`按照`[hashing](/web/20220630011606/https://www.baeldung.com/java-hashcode)`的原理工作。

这个`Map`实现通常充当一个桶`hash table`，但是**当桶变得太大时，它们被转换成`TreeNodes`的节点**，每个节点的结构与`java.util.TreeMap.`中的相似

你可以在[的文章](/web/20220630011606/https://www.baeldung.com/java-hashmap)中找到更多关于`HashMap's`内部的内容。

另一方面，`TreeMap`扩展了`AbstractMap`类，实现了`NavigableMap`接口。一个`TreeMap`将地图元素存储在一个`Red-Black`树中，这是一个**自平衡`Binary Search Tree`** 。

此外，你还可以在这里找到更多关于[中`TreeMap's`内部的内容。](/web/20220630011606/https://www.baeldung.com/java-treemap)

### 2.2。订单

**`HashMap`对`Map`** 中元素的排列方式不提供任何保证。

意思是，**我们在迭代`HashMap` :** 的`keys`和`values`时不能假定任何顺序

```java
@Test
public void whenInsertObjectsHashMap_thenRandomOrder() {
    Map<Integer, String> hashmap = new HashMap<>();
    hashmap.put(3, "TreeMap");
    hashmap.put(2, "vs");
    hashmap.put(1, "HashMap");

    assertThat(hashmap.keySet(), containsInAnyOrder(1, 2, 3));
}
```

然而，`TreeMap`中的项目按照它们的自然顺序被**排序。**

如果`TreeMap`对象不能按照自然顺序排序，那么我们可以利用`Comparator` 或`Comparable` 来定义元素在`Map:`中的排列顺序

```java
@Test
public void whenInsertObjectsTreeMap_thenNaturalOrder() {
    Map<Integer, String> treemap = new TreeMap<>();
    treemap.put(3, "TreeMap");
    treemap.put(2, "vs");
    treemap.put(1, "HashMap");

    assertThat(treemap.keySet(), contains(1, 2, 3));
}
```

### 2.3。`Null` 价值观

`HashMap`允许存储最多一个`null` `key`和多个`null`值。

让我们看一个例子:

```java
@Test
public void whenInsertNullInHashMap_thenInsertsNull() {
    Map<Integer, String> hashmap = new HashMap<>();
    hashmap.put(null, null);

    assertNull(hashmap.get(null));
}
```

但是，`TreeMap`不允许有一个`null` `key`，但是可以包含多个`null`值。

不允许使用`null` 键，因为`compareTo()`或`compare()` 方法抛出了`NullPointerException:`

```java
@Test(expected = NullPointerException.class)
public void whenInsertNullInTreeMap_thenException() {
    Map<Integer, String> treemap = new TreeMap<>();
    treemap.put(null, "NullPointerException");
}
```

**如果我们将`TreeMap`与用户定义的`Comparator`一起使用，那么这取决于 compare `()`方法的实现如何处理`null`值。**

## 3。性能分析

性能是帮助我们理解给定用例的数据结构的适用性的最关键的度量。

在本节中，我们将对`HashMap` 和`TreeMap.`的性能进行综合分析

### 3.1。`HashMap`

**`HashMap,` 是一个基于哈希表的实现，内部使用一个基于数组的数据结构根据`hash function`来组织它的元素。**

`HashMap` 为`add()`、`remove()`和`contains().` 等大多数操作提供预期的恒定时间性能`O(1)`，因此，它比`TreeMap`快得多。

在合理的假设下，在散列表中搜索元素的平均时间是`O(1).` ，但是，`hash function`的不正确实现可能导致桶中值的不良分布，从而导致:

*   内存开销–许多存储桶仍未使用
*   性能下降**–**碰撞次数越多，性能越低

**在 Java 8 之前，`Separate Chaining`是处理冲突的唯一首选方式。**通常使用链表来实现，`i.e.`，如果有任何冲突或者两个不同的元素具有相同的哈希值，那么将两个项目存储在同一个链表中。

因此，在最坏的情况下，搜索一个`HashMap,`中的元素可能会花费与搜索一个链表`i.e.` `O(n)`中的元素一样长的时间。

**然而，随着 [JEP 180](https://web.archive.org/web/20220630011606/https://openjdk.java.net/jeps/180) 的出现，元素在**中的排列方式发生了微妙的变化

根据规范，当桶变得太大并且包含足够多的节点时，它们被转换成`TreeNodes`的模式，每种模式的结构都类似于`TreeMap`中的模式。

**因此，在高哈希冲突的情况下，最坏情况下的性能将从`O(n)`提高到`O(log n).`**

执行此转换的代码如下所示:

```java
if(binCount >= TREEIFY_THRESHOLD - 1) {
    treeifyBin(tab, hash);
}
```

`TREEIFY_THRESHOLD`的值是 8，这有效地表示使用树而不是桶的链表的阈值计数。

很明显:

*   一个 `HashMap`需要的内存比保存它的数据所需要的要多得多
*   A `HashMap`不应该超过 70%–75%满。如果它接近了，它会调整大小并重新散列条目
*   重新散列需要`n`操作，这是高成本的，其中我们的常数时间插入变得无序`O(n)`
*   哈希算法决定了在`HashMap`中插入对象的顺序

**在`HashMap`对象创建本身的时候，可以通过设置自定义`initial capacity`和`load factor`T5 来调整`HashMap`的性能。**

然而，我们应该选择一个*散列表*，如果:

*   我们知道我们的收藏中大约有多少项目需要维护
*   我们不想按自然顺序提取项目

在上述情况下，`HashMap`是我们的最佳选择，因为它提供了固定时间的插入、搜索和删除。

### 3.2。`TreeMap`

一个 `TreeMap`将它的数据存储在一个分层树中，该树能够在一个自定义`Comparator.`的帮助下对元素进行排序

对其性能的总结:

*   `TreeMap` 为`add()`、`remove()`和`contains()` 等大多数操作提供了`O(log(n))`的性能
*   `Treemap`可以节省内存(与`HashMap)`相比，因为它只使用保存其项目所需的内存量，不像`HashMap` 使用连续的内存区域
*   树应该保持其平衡，以保持其预期的性能，这需要相当大的努力，因此使实现变得复杂

无论何时，我们都应该去旅行:

*   必须考虑内存限制
*   我们不知道需要在内存中存储多少项
*   我们想以自然的顺序提取物体
*   如果项目将被一致地添加和删除
*   我们愿意接受搜索时间

## 4。相似之处

### 4.1。独特元素

`TreeMap`和`HashMap`都不支持重复键。如果添加，它将覆盖前面的元素(没有错误或异常):

```java
@Test
public void givenHashMapAndTreeMap_whenputDuplicates_thenOnlyUnique() {
    Map<Integer, String> treeMap = new HashMap<>();
    treeMap.put(1, "Baeldung");
    treeMap.put(1, "Baeldung");

    assertTrue(treeMap.size() == 1);

    Map<Integer, String> treeMap2 = new TreeMap<>();
    treeMap2.put(1, "Baeldung");
    treeMap2.put(1, "Baeldung");

    assertTrue(treeMap2.size() == 1);
}
```

### 4.2。并发访问

**两个`Map` 实现都不是`synchronized`** ，我们需要自己管理并发访问。

只要有多个线程并发访问它们，并且至少有一个线程修改它们，它们都必须在外部同步。

我们必须显式地使用`Collections.synchronizedMap(mapName)`来获得所提供地图的同步视图。

### 4.3。快速失败迭代器

一旦迭代器被创建，如果`Map`在任何时候以任何方式被修改，那么`Iterator`将抛出一个`ConcurrentModificationException` 。

此外，我们可以使用迭代器的 remove 方法在迭代过程中改变`Map` 。

让我们看一个例子:

```java
@Test
public void whenModifyMapDuringIteration_thenThrowExecption() {
    Map<Integer, String> hashmap = new HashMap<>();
    hashmap.put(1, "One");
    hashmap.put(2, "Two");

    Executable executable = () -> hashmap
      .forEach((key,value) -> hashmap.remove(1));

    assertThrows(ConcurrentModificationException.class, executable);
}
```

## 5。使用哪个实现？

总的来说，这两种实现都有各自的优缺点，然而，**这是关于理解潜在的期望和需求，这些期望和需求必须支配我们的选择。**

总结:

*   如果我们想对条目进行排序，我们应该使用一个`TreeMap`
*   如果我们优先考虑性能而不是内存消耗，我们应该使用一个`HashMap`
*   因为一个`TreeMap`有一个更重要的局部性，如果我们想要访问根据它们的自然顺序彼此相对接近的对象，我们可以考虑它
*   `HashMap` 可以使用`initialCapacity`和`loadFactor`进行调谐，这对于`TreeMap`是不可能的
*   如果我们想保留插入顺序，同时受益于恒定时间访问，我们可以使用`LinkedHashMap`

## 6。结论

在本文中，我们展示了`TreeMap`和`HashMap`之间的区别和相似之处。

和往常一样，本文的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630011606/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-3)