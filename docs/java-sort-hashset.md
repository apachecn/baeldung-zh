# 在 Java 中对哈希表排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sort-hashset>

## 1.概观

一个 [`HashSet`](/web/20221218223415/https://www.baeldung.com/java-hashset) 是来自`java.util`包的集合类。这个类继承自`AbstractSet`类，并实现了 [`Set`](/web/20221218223415/https://www.baeldung.com/java-set-operations) 接口。此外，`HashSet`不能保持元素的顺序，因此需要找到排序这些元素的方法。

在这个快速教程中，**我们将学习多种技术来排序`HashSet`** 的元素。

## 2.使用`Collections.sort()`方法

`Collections.sort()`方法对实现`java.util.List`接口的对象集合进行排序。因此，我们可以将我们的`HashSet`转换成`List`，然后使用`Collections.sort()`对其进行排序:

```java
HashSet<Integer> numberHashSet = new HashSet<>();
numberHashSet.add(2);
numberHashSet.add(1);
numberHashSet.add(4);
numberHashSet.add(3);

// converting HashSet to arraylist
ArrayList arrayList = new ArrayList(numberHashSet);

// sorting the list
Collections.sort(arrayList);

assertThat(arrayList).containsExactly(1, 2, 3, 4);
```

在上面的例子中，我们首先将`HashSet`的元素复制到一个`[ArrayList](/web/20221218223415/https://www.baeldung.com/java-arraylist)`中。然后，我们使用我们的`ArrayList`作为`Collections.sort()`方法的一个参数。除了`ArrayList`，我们还可以使用 [`LinkedList`](/web/20221218223415/https://www.baeldung.com/java-linkedlist) 或 [`Vector`](/web/20221218223415/https://www.baeldung.com/java-arraylist-vs-vector#vector) 。

## 3.使用`TreeSet`

使用这种方法，我们将`HashSet`转换为`[TreeSet](/web/20221218223415/https://www.baeldung.com/java-tree-set)`，这与`HashSet`类似，只是它以升序存储元素。因此，当`HashSet`转换为`TreeSet` 时，**`HashSet`元素按顺序排列:**

```java
HashSet<Integer> numberHashSet = new HashSet<>();
numberHashSet.add(2);
numberHashSet.add(1);
numberHashSet.add(4);
numberHashSet.add(3);

TreeSet<Integer> treeSet = new TreeSet<>();
treeSet.addAll(numberHashSet);

assertThat(treeSet).containsExactly(1, 2, 3, 4);
```

我们可以看到，用一个`TreeSet` 对一个`HashSet`进行排序是非常简单的。我们只需要创建一个带有作为参数的`HashSet`列表的`TreeSet`实例。

## 4.使用`stream().sorted()`方法

有一种简洁的方法可以使用[流 API](/web/20221218223415/https://www.baeldung.com/java-8-streams) 的 `stream().sorted()` 方法对`HashSet`进行排序。Java 8 中引入的这个 API 允许我们对一组元素执行函数操作。此外，它可以从不同的集合中获取对象，并根据我们使用的管道方法以所需的方式显示它们。

在我们的示例**中，我们将使用** **`stream().sorted()`方法，该方法返回一个`Stream`，其元素按照一定的顺序**排序。应该注意的是，由于原始的`HashSet`保持不变，我们需要将排序的结果保存在一个新的`Collection`中。我们将使用`collect()`方法将数据存储回一个新的`HashSet`:

```java
HashSet<Integer> numberHashSet = new HashSet<>();
numberHashSet.add(200);
numberHashSet.add(100);
numberHashSet.add(400);
numberHashSet.add(300);

HashSet<Integer> sortedHashSet = numberHashSet.stream()
  .sorted()
  .collect(Collectors.toCollection(LinkedHashSet::new));

assertThat(sortedHashSet).containsExactly(100, 200, 300, 400);
```

我们应该注意，当我们使用`stream()`时。`sorted()` 方法没有参数，它按照自然顺序对`HashSet`进行排序`.` 我们也可以用一个比较器重载它来定义一个定制的排序顺序。

## 5.结论

在本文中，我们讨论了如何使用三种方法在 Java 中对`HashSet`进行排序:使用`Collections.sort()` 方法、使用`TreeSet`方法和使用`stream().sorted()`方法。

与往常一样，这些代码片段可以在 GitHub 网站[上找到。](https://web.archive.org/web/20221218223415/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)