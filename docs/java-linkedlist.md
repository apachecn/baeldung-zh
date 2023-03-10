# Java 链接列表指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-linkedlist>

## 1.介绍

`[LinkedList](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/LinkedList.html)`是`[List](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html)`和`[Deque](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Deque.html)`接口的双向链表实现。它实现了所有可选的列表操作，并允许所有的元素(包括`null`)。

## 2.特征

下面你可以找到`LinkedList`最重要的属性:

*   索引到列表中的操作将从列表的开头或结尾遍历列表，无论哪一个更接近指定的索引
*   它不是[同步的](https://web.archive.org/web/20220706104859/https://stackoverflow.com/a/1085745/2486904)
*   它的`[Iterator](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html)`和`[ListIterator](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ListIterator.html)`迭代器是[快速失效](https://web.archive.org/web/20220706104859/https://stackoverflow.com/questions/17377407/what-is-fail-safe-fail-fast-iterators-in-java-how-they-are-implemented)(这意味着在迭代器创建之后，如果列表被修改，将抛出一个`[ConcurrentModificationException](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ConcurrentModificationException.html)`)
*   每个元素都是一个节点，它保存了对下一个和上一个元素的引用
*   它保持插入顺序

虽然`LinkedList`不是同步的，但是我们可以通过调用`[Collections.synchronizedList](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#synchronizedList(java.util.List))`方法来检索它的同步版本，比如:

```java
List list = Collections.synchronizedList(new LinkedList(...));
```

## 3.与`ArrayList`的比较

虽然两者都实现了`List`接口，但是它们有不同的语义——这肯定会影响使用哪一个的决定。

### 3.1。结构

`ArrayList`是由`Array`支持的基于索引的数据结构。它提供对其元素的随机访问，性能等于 O(1)。

另一方面，`LinkedList`将其数据存储为元素列表，每个元素都链接到它的前一个和下一个元素。在这种情况下，项目的搜索操作的执行时间等于 O(n)。

### 3.2。操作

在`LinkedList`中，项目的插入、添加和移除操作更快，因为当一个元素被添加到集合中的任意位置时，不需要调整数组的大小或更新索引，只有周围元素中的引用会改变。

### 3.3。内存使用量

一个`LinkedList`比一个`ArrayList`消耗更多的内存，因为一个`LinkedList`中的每个节点存储两个引用，一个用于它的前一个元素，一个用于它的下一个元素，而`ArrayList`只保存数据和它的索引。

## 4.使用

下面是一些展示如何使用`LinkedList`的代码示例:

### 4.1。创作

```java
LinkedList<Object> linkedList = new LinkedList<>();
```

### 4.2。添加元素

`LinkedList`实现了`List`和`Deque`接口，除了标准的`add()`和`addAll()` 方法之外，你还可以找到`addFirst()` 和`addLast()`，它们分别在开头或结尾添加了一个元素。

### 4.3。移除元件

类似于元素添加，这个列表实现提供了`removeFirst()`和`removeLast().` 

此外，有一个方便的方法`removeFirstOccurence()`和`removeLastOccurence()`返回布尔值(如果集合包含指定的元素，则为真)。

### 4.4。队列操作

`Deque`接口提供了类似队列的行为(实际上`Deque`扩展了`Queue`接口):

```java
linkedList.poll();
linkedList.pop();
```

这些方法检索第一个元素并将其从列表中移除。

`poll()` 和`pop()` 的区别在于`pop`会将`NoSuchElementException()` 抛出到空列表中，而`poll`返回 null。API`pollFirst()` 和`pollLast()`也是可用的。

下面是`push` API 如何工作的例子:

```java
linkedList.push(Object o);
```

它将元素作为集合的头部插入。

`LinkedList`有许多其他方法，其中大部分应该为已经使用过`Lists`的用户所熟悉。由`Deque`提供的其他方法可能是“标准”方法的一种方便的替代方法。

完整的文档可以在[这里](https://web.archive.org/web/20220706104859/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/LinkedList.html)找到。

## 5.结论

`ArrayList` 通常是默认的`List`实现。

然而，在某些使用案例中，使用`LinkedList`会更合适，例如偏好恒定的插入/删除时间(例如，频繁的插入/删除/更新)，而不是恒定的访问时间和有效的内存使用。

代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220706104859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)