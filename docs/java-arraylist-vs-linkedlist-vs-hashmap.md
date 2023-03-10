# Java 中的 ArrayList 与 LinkedList 和 HashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraylist-vs-linkedlist-vs-hashmap>

## 1.概观

Java 中的集合基于几个核心接口和十几个实现类。不同实现的广泛选择有时会导致混乱。

决定为特定用例使用哪种集合类型不是一件简单的任务。这个决定对我们的代码可读性和性能有很大的影响。

我们不会在一篇文章中解释所有类型的集合，而是解释三种最常见的集合:`ArrayList, LinkedList,` 和`HashMap.`在本教程中，我们将了解它们如何存储数据、它们的性能，并推荐何时使用它们。

## 2.收集

集合只是一个将其他对象组合在一起的 Java 对象。 `[Java Collections Framework](/web/20220926193758/https://www.baeldung.com/java-collections)` 包含一组用于表示和操作集合的数据结构和算法。如果应用正确，所提供的数据结构有助于减少编程工作量并提高性能。

### 2.1.接口

Java 集合框架包含四个基本接口:`List`、`Set`、`Map, and` 、T3。在查看实现类之前，理解这些接口的预期用途是很重要的。

让我们快速浏览一下我们将在本文中使用的四个核心接口中的三个:

*   接口专用于存储有序的对象集合。它允许我们定位访问和插入新元素，以及保存重复的值
*   *Map* 接口支持数据的键值对映射。要访问某个值，我们需要知道它的唯一键
*   `Queue`接口支持按照先进先出的顺序存储数据。类似于现实世界中的排队

[`HashMap`](/web/20220926193758/https://www.baeldung.com/java-hashmap) 实现了*贴图*接口。`List`接口由 [`ArrayList`](/web/20220926193758/https://www.baeldung.com/java-arraylist) 和 [`LinkedList`](/web/20220926193758/https://www.baeldung.com/java-linkedlist) 共同实现。`LinkedList`额外实现了`Queue` 接口。

### 2.2.`List`对`Map`

我们有时会遇到的一个常见反模式是试图使用地图来维护顺序。因此，不使用更适合该作业的其他集合类型。

仅仅因为我们可以用一个集合类型解决许多问题，并不意味着我们应该这样做。

让我们来看一个糟糕的例子，我们使用一个地图来保存基于位置键的数据:

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "Daniel");
map.put(2, "Marko");
for (String name : map.values()) {
    assertThat(name).isIn(map.values());
}
assertThat(map.values()).containsExactlyInAnyOrder("Daniel", "Marko");
```

当我们遍历地图值时，我们不能保证按照我们放入它们的顺序来检索它们。这只是因为地图不是为维护元素的顺序而设计的。

我们可以用一个列表以一种可读性更强的方式重写这个例子。`Lists`根据定义是有序的，所以我们可以按照插入的顺序遍历这些项目:

```java
List<String> list = new ArrayList<>();
list.add("Daniel");
list.add("Marko");
for (String name : list) {
    assertThat(name).isIn(list);
}
assertThat(list).containsExactly("Daniel", "Marko");
```

`Maps`专为基于唯一键的快速访问和搜索而设计。当我们想要维护顺序或者使用基于位置的索引时，列表是一个自然的选择。

## 3.`ArrayList`

`ArrayList` 是 Java 中最常用的`List`接口的实现。它基于内置的[数组](/web/20220926193758/https://www.baeldung.com/java-arrays-guide)，但是可以随着我们添加或删除元素而动态地增长和收缩。

我们使用从零开始的索引来访问列表元素。我们可以在列表的末尾或特定位置插入一个新元素:

```java
List<String> list = new ArrayList<>();
list.add("Daniel");
list.add(0, "Marko");
assertThat(list).hasSize(2);
assertThat(list.get(0)).isEqualTo("Marko");
```

要从列表中删除元素，我们需要提供对象引用或其索引:

```java
List<String> list = new ArrayList<>(Arrays.asList("Daniel", "Marko"));
list.remove(1);
assertThat(list).hasSize(1);
assertThat(list).doesNotContain("Marko");
```

### 3.1.表演

`ArrayList`为我们提供了 Java 中的动态数组。虽然比内置数组慢，但是`ArrayList` 帮助我们节省了一些编程工作，并提高了代码的可读性。

当我们谈论[时间复杂度](/web/20220926193758/https://www.baeldung.com/java-collections-complexity)时，我们使用 Big-O 符号。符号描述了执行算法的时间如何随着输入的大小而增长。

**`ArrayList`允许随机访问，因为数组是基于索引的。这意味着访问任何元素总是需要一个恒定的时间`O(1)`。**

添加新元素也需要`O(1)`时间，除了在特定位置/索引添加元素时，则需要`O(n)`。检查特定元素是否存在于给定列表中以线性`O(n)`时间运行。

对于元素的移除也是如此。我们需要迭代整个数组，以找到选择移除的元素。

### 3.2.使用

每当我们不确定使用哪种集合类型时，从 `ArrayList.`开始可能是个好主意。记住，基于索引访问项目将会非常快。然而，基于它们的值搜索项目或者在特定位置添加/移除项目将是昂贵的。

**当保持项目的相同顺序很重要时，使用`ArrayList `是有意义的，并且基于位置/索引的快速访问时间是一个重要的标准。**

当项目的顺序不重要时，避免使用`ArrayList` 。还有，经常需要将物品添加到特定位置时，尽量避免。同样，记住当搜索特定的项目值是一个重要需求时，`ArrayList` 可能不是最佳选择，尤其是当列表很大时。

## 4.`LinkedList`

`LinkedList`是一个双向链表实现。实现了`List`和*dequee*(*`Queue)`*接口的扩展)。与`ArrayList`不同，当我们在`LinkedList`中存储数据时，每个元素都保持着与前一个元素的链接。

除了标准的`List`插入方法之外，`LinkedList`还支持额外的方法，可以在列表的开头或结尾添加元素:

```java
LinkedList<String> list = new LinkedList<>();
list.addLast("Daniel");
list.addFirst("Marko");
assertThat(list).hasSize(2);
assertThat(list.getLast()).isEqualTo("Daniel");
```

这个列表实现还提供了从列表的开头或结尾移除元素的方法:

```java
LinkedList<String> list = new LinkedList<>(Arrays.asList("Daniel", "Marko", "David"));
list.removeFirst();
list.removeLast();
assertThat(list).hasSize(1);
assertThat(list).containsExactly("Marko");
```

实现的`Deque`接口提供了类似队列的方法来检索、添加和删除元素:

```java
LinkedList<String> list = new LinkedList<>();
list.push("Daniel");
list.push("Marko");
assertThat(list.poll()).isEqualTo("Marko");
assertThat(list).hasSize(1);
```

### 4.1.表演

一个`LinkedList`比一个`ArrayList`消耗更多的内存，因为每个节点存储两个对前一个和下一个元素的引用。

在`LinkedList`中插入、添加和移除操作更快，因为没有在后台调整数组的大小。当一个新项目被添加到列表中间的某个地方时，只有周围元素中的引用需要改变。

**`LinkedList`支持`O(1)`在集合中任意位置的恒定时间插入。然而，它在访问特定位置的项目时效率较低，需要花费`O(n) `时间。**

移除一个元素也需要花费`O(1)`恒定的时间，因为我们只需要修改几个指针。检查特定元素是否存在于给定列表中需要`O(n)` 线性时间，与`ArrayList.`一样

### 4.2.使用

大多数时候我们可以使用`ArrayList `作为默认的`List`实现。然而，在某些用例中，我们应该利用`LinkedList.`,包括当我们更喜欢恒定的插入和删除时间，而不是恒定的访问时间和有效的内存使用时。

**当保持项目的相同顺序和快速插入时间(在任何位置添加和移除项目)是一个重要标准时，使用`LinkedList` 是有意义的。**

像`ArrayList`一样，当项目的顺序不重要时，我们应该避免使用`LinkedList` 。当快速访问时间或搜索项目是一项重要要求时，`LinkedList`不是最佳选择。

## 5.`HashMap`

与`ArrayList`和`LinkedList`不同， *HashMap* 实现了`Map`接口。这意味着每个键都映射到一个值。我们总是需要知道从集合中检索相应值的键:

```java
Map<String, String> map = new HashMap<>();
map.put("123456", "Daniel");
map.put("654321", "Marko");
assertThat(map.get("654321")).isEqualTo("Marko");
```

同样，我们只能使用值的键从集合中删除值:

```java
Map<String, String> map = new HashMap<>();
map.put("123456", "Daniel");
map.put("654321", "Marko");
map.remove("654321");
assertThat(map).hasSize(1);
```

### 5.1.表演

有人可能会问，为什么不简单地用一个 `List` 把所有的键都去掉呢？尤其是因为`HashMap`为保存密钥消耗了更多的内存，而且它的条目没有排序。答案在于搜索元素的性能优势。

**`HashMap`在检查一个键是否存在或根据一个键检索一个值时非常有效。这些操作平均需要`O(1)`。**

基于一个键从一个 `HashMap` 中添加和删除元素需要`O(1)`恒定的时间。在不知道密钥的情况下检查一个元素需要线性时间`O(n),`,因为需要遍历所有元素。

### 5.2.使用

和`ArrayList`，`HashMap`是 Java 中最常用的数据结构之一。与不同的列表实现不同，`HashMap`利用索引来执行到特定值的跳转，使得搜索时间保持不变，即使对于大型集合也是如此。

**只有当我们想要存储的数据有唯一的键时，使用`HashMap` 才有意义。我们应该在基于关键字搜索项目时使用它，快速访问时间是一个重要的要求。**

当保持集合中项目的相同顺序很重要时，我们应该避免使用`HashMap` 。

## 6.结论

在本文中，我们探讨了`Java` : `ArrayList, LinkedList,` 和`HashMap`中三种常见的集合类型。**我们观察了他们在添加、删除和搜索商品方面的表现。基于此，我们提供了在 Java 应用程序中何时应用它们的建议。**

在示例中，我们只讨论了添加和删除项目的基本方法。要更详细地了解每个实现 API，请访问我们专门的`[ArrayList,](/web/20220926193758/https://www.baeldung.com/java-arraylist)` `[ArrayList](/web/20220926193758/https://www.baeldung.com/java-arraylist)`和 [*HashMap*](/web/20220926193758/https://www.baeldung.com/java-hashmap) 文章。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926193758/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)