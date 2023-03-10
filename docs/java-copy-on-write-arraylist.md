# CopyOnWriteArrayList 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-on-write-arraylist>

## 1。概述

在这篇简短的文章中，我们将关注来自`java.util.concurrent`包的`[CopyOnWriteArrayList](https://web.archive.org/web/20221209131326/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html)` 。

这在多线程程序中是一个非常有用的构造——当我们想要以线程安全的方式迭代一个列表而不需要显式同步时。

## 2。`CopyOnWriteArrayList` API

`CopyOnWriteArrayList` 的设计使用了一种有趣的技术，使得它在不需要同步的情况下是线程安全的。当我们使用任何修改方法——比如`add()`或`remove() –` —`CopyOnWriteArrayList`的全部内容都被复制到新的内部副本中。

由于这个简单的事实，**我们可以以一种安全的方式遍历列表，即使并发修改正在发生**。

当我们在`CopyOnWriteArrayList,` 上调用`iterator()` 方法时，我们得到一个由`CopyOnWriteArrayList`内容的不可变快照备份的`Iterator`。

它的内容是从创建`Iterator`时开始的`ArrayList`中的数据的精确副本。即使与此同时，其他线程在列表中添加或删除了某个元素，这种修改也是在制作数据的新副本，该副本将在该列表的任何进一步数据查找中使用。

这种数据结构的特性使得它在迭代次数多于修改次数的情况下特别有用。如果添加元素是我们场景中的常见操作，那么`CopyOnWriteArrayList` 将不是一个好的选择——因为额外的副本肯定会导致低于标准的性能。

## 3。插入时迭代`CopyOnWriteArrayList`

假设我们正在创建一个存储整数的`CopyOnWriteArrayList` 实例:

```java
CopyOnWriteArrayList<Integer> numbers 
  = new CopyOnWriteArrayList<>(new Integer[]{1, 3, 5, 8});
```

接下来，我们想要迭代这个数组，所以我们创建了一个`Iterator` 实例:

```java
Iterator<Integer> iterator = numbers.iterator();
```

在创建了`Iterator`之后，我们向`numbers` 列表中添加了一个新元素:

```java
numbers.add(10);
```

请记住，当我们为`CopyOnWriteArrayList,` 创建迭代器时，我们会在调用`iterator()` 时获得列表中数据的不可变快照。

因此，在对它进行迭代时，我们不会在迭代中看到数字`10`:

```java
List<Integer> result = new LinkedList<>();
iterator.forEachRemaining(result::add);

assertThat(result).containsOnly(1, 3, 5, 8);
```

使用新创建的`Iterator`的后续迭代也将返回添加的数字 10:

```java
Iterator<Integer> iterator2 = numbers.iterator();
List<Integer> result2 = new LinkedList<>();
iterator2.forEachRemaining(result2::add);

assertThat(result2).containsOnly(1, 3, 5, 8, 10);
```

## 4。不允许在迭代时删除

创建`CopyOnWriteArrayList` 是为了考虑到安全迭代元素的可能性，即使底层列表被修改。

由于复制机制，不允许对返回的`Iterator` 进行`remove()` 操作——导致`UnsupportedOperationException:`

```java
@Test(expected = UnsupportedOperationException.class)
public void whenIterateOverItAndTryToRemoveElement_thenShouldThrowException() {

    CopyOnWriteArrayList<Integer> numbers
      = new CopyOnWriteArrayList<>(new Integer[]{1, 3, 5, 8});

    Iterator<Integer> iterator = numbers.iterator();
    while (iterator.hasNext()) {
        iterator.remove();
    }
}
```

## 5。结论

在这个快速教程中，我们看了一下来自`java.util.concurrent`包的`CopyOnWriteArrayList` 实现。

我们看到了这个列表有趣的语义，以及如何以线程安全的方式迭代它，而其他线程可以继续从中插入或删除元素。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221209131326/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。