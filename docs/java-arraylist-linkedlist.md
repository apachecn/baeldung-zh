# Java ArrayList 与 linkedlist

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraylist-linkedlist>

## 1.概观

谈到[集合](/web/20221031224531/https://www.baeldung.com/java-collections)，Java 标准库提供了大量选项供选择。在这些选项中，有两个著名的`List `实现，称为`ArrayList `和`LinkedList, `，每个都有自己的属性和用例。

在本教程中，我们将看到这两个是如何实现的。然后，我们将评估每种产品的不同应用。

## 2.`ArrayList`

在内部， **`ArrayList `正在用数组实现`List `接口**。由于 Java 中数组的大小是固定的，`ArrayList `创建一个有初始容量的数组。在此过程中，如果我们需要存储比默认容量更多的项目，它会用一个新的更大的数组来替换该数组。

为了更好地理解它的属性，让我们从它的三个主要操作来评估这个数据结构:[添加条目](https://web.archive.org/web/20221031224531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#add(E))、[按索引获取一个条目](https://web.archive.org/web/20221031224531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#get(int))和[按索引移除条目](https://web.archive.org/web/20221031224531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ArrayList.html#remove(int))。

### 2.1.增加

当我们创建一个空的`ArrayList, `时，它用一个[默认容量](https://web.archive.org/web/20221031224531/https://github.com/openjdk/jdk/blob/827e5e32264666639d36990edd5e7d0b7e7c78a9/src/java.base/share/classes/java/util/ArrayList.java#L118)(当前为 10)初始化它的后备数组:

[![](img/c95df41513c44c2b3a5dc4be837af598.png)](/web/20221031224531/https://www.baeldung.com/wp-content/uploads/2020/03/init-cap.png)

在数组未满的情况下添加一个新项，就像将该项分配给一个特定的数组索引一样简单。这个数组索引是由当前数组大小决定的，因为我们实际上是追加到列表中:

```
backingArray[size] = newItem;
size++;
```

所以，**在最好和一般的情况下，加法运算的[时间复杂度](/web/20221031224531/https://www.baeldung.com/java-collections-complexity)为`O(1)`** `, `，相当快。然而，随着后备阵列变满，add 实现变得效率更低:

[![](img/ad9ee74cacc923afa6daba049d71c1cb.png)](/web/20221031224531/https://www.baeldung.com/wp-content/uploads/2020/03/full.png)

**要添加一个新项目，我们应该首先初始化一个有[更多容量](https://web.archive.org/web/20221031224531/https://github.com/openjdk/jdk/blob/827e5e32264666639d36990edd5e7d0b7e7c78a9/src/java.base/share/classes/java/util/ArrayList.java#L230)的全新数组，并将所有现有项目复制到新数组中。**只有在复制当前元素后，我们才能添加新项目。因此，在最坏的情况下，时间复杂度是`O(n) `，因为我们必须首先复制`n`个元素。

**从理论上讲，增加一个新元素需要经过[摊销](https://web.archive.org/web/20221031224531/https://en.wikipedia.org/wiki/Amortized_analysis)常数时间。即添加`n `个元素需要`O(n) `个时间。但是，由于复制开销，一些单个加法可能会执行得很差。**

### 2.2.通过索引访问

通过索引访问项目是`ArrayList `真正的亮点。要检索索引`i, `处的项目，我们只需从后备数组中返回位于`i^(th)`索引处的项目。因此，**索引操作访问的时间复杂度总是`O(1). `**

### 2.3.按索引删除

假设我们要从我们的`ArrayList, `中移除索引 6，它对应于我们的后备数组中的元素 15:

[![](img/bca70dec263530e4687c1c3e91aa9ba5.png)](/web/20221031224531/https://www.baeldung.com/wp-content/uploads/2020/03/remove.png)

在将所需的元素标记为已删除后，我们应该将它后面的所有元素向后移动一个索引。**显然，越靠近数组开始的元素，我们应该移动的元素就越多。**所以时间复杂度在最好的情况下是`O(1) `，在平均和最坏的情况下是`O(n) `。

### 2.4.应用和限制

通常，`ArrayList `是许多开发者需要`List `实现时的默认选择。事实上，**当读取的次数远远大于写入的次数时，这实际上是一个明智的选择**。

有时我们需要同样频繁的读写。**如果我们对可能的项目的最大数量有一个估计，那么使用`ArrayList`** 仍然是有意义的。如果是这种情况，我们可以用初始容量初始化`ArrayList `:

```
int possibleUpperBound = 10_000;
List<String> items = new ArrayList<>(possibleUpperBound);
```

这种估计可以防止大量不必要的复制和数组分配。

而且，**数组在 Java 中是由 [`int `值](https://web.archive.org/web/20221031224531/https://docs.oracle.com/javase/specs/jls/se13/html/jls-10.html#jls-10.4)索引的。因此，在一个 Java 数组中不可能存储超过`2^(32)`个元素，因此在`ArrayList`T6 中也不可能。**

## 3.`LinkedList`

`LinkedList`，顾名思义，**使用链接节点的集合来存储和检索元素**。例如，下面是添加四个元素后 Java 实现的样子:

[![](img/f419b23804658c4c541fce2afc8e47bc.png)](/web/20221031224531/https://www.baeldung.com/wp-content/uploads/2020/03/four-nodes.png)

每个节点维护两个指针:一个指向下一个元素，另一个指向前一个元素。在此基础上扩展，[双向链表](https://web.archive.org/web/20221031224531/https://en.wikipedia.org/wiki/Doubly_linked_list)有两个指针指向第一个和最后一个条目。

同样，让我们根据相同的基本操作来评估这个实现。

### 3.1.增加

为了添加新节点，首先，我们应该将当前的最后一个节点链接到新节点:

[![](img/553b76adcb6682862f374bc347380157.png)](/web/20221031224531/https://www.baeldung.com/wp-content/uploads/2020/03/link-last.png)

然后更新最后一个指针:

[![](img/318cc1a45474390465960627d69e42a9.png)](/web/20221031224531/https://www.baeldung.com/wp-content/uploads/2020/03/update-last.png)

**由于这两个操作都是琐碎的，所以加法操作的时间复杂度总是`O(1)`。**

### 3.2.通过索引访问

**`LinkedList, `与`ArrayList, `相反不支持快速随机存取。因此，为了通过索引找到一个元素，我们应该手动遍历列表的某个部分****。**

 **在最好的情况下，当请求的项目接近列表的开始或结束时，时间复杂度将与`O(1). `一样快。然而，在一般和最坏的情况下，我们可能会以`O(n) `的访问时间结束，因为我们必须一个接一个地检查许多节点。

### 3.3.按索引删除

**为了[移除](https://web.archive.org/web/20221031224531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/LinkedList.html#remove(int))一个物品，我们应该首先找到所请求的物品，然后将其从列表**中解除链接。因此，访问时间决定了时间复杂度——也就是说，在最好的情况下是`O(1) `,在平均情况下和最差情况下是`O(n)`。

### 3.4.应用程序

当添加速率远高于读取速率时 **`LinkedLists `更合适。**

此外，当大多数情况下我们需要第一个或最后一个元素时，它可以用在大量读取的场景中。值得一提的是，`LinkedList `还实现了`[Deque](https://web.archive.org/web/20221031224531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Deque.html) `接口——支持对集合两端的高效访问。

一般来说，如果我们知道它们的实现差异，那么我们可以很容易地为特定的用例选择一个。

例如，假设我们将在一个类似列表的数据结构中存储大量时间序列事件。我们知道每秒钟都会收到突发事件。

此外，我们需要定期一个接一个地检查所有事件，并提供一些统计数据。对于这个用例，`LinkedList `是一个更好的选择，因为添加速率远高于读取速率。

此外，我们会读取所有的项目，所以我们不能击败`O(n) `上限。

## 4.结论

在本教程中，首先，我们深入了解了如何在 Java 中实现`ArrayList `和`LinkLists `。

我们还评估了每种产品的不同使用案例。**