# 从 Java 集合中移除元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collection-remove-elements>

## 1。概述

在这个快速教程中，**我们将讨论从 Java `Collections`中删除匹配特定谓词的条目的四种不同方法。**

我们自然也会看到一些警告。

## 2。定义我们的系列

首先，我们将说明改变原始数据结构的两种方法。然后我们将讨论另外两个选项，它们不是删除项目，而是创建没有项目的原始`Collection`的副本。

让我们在整个示例中使用以下集合来演示如何使用不同的方法获得相同的结果:

```
Collection<String> names = new ArrayList<>();
names.add("John");
names.add("Ana");
names.add("Mary");
names.add("Anthony");
names.add("Mark");
```

## 3。用`Iterator` 删除元素

**Java 的`[Iterator](/web/20220815140904/https://www.baeldung.com/java-iterator)`让我们能够在`[Collection](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)`中行走和移除每一个单独的元素。**

为此，我们首先需要使用`[iterator](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#iterator())`方法检索元素上的迭代器。之后，我们可以借助 [`next`](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html#next()) 访问每个元素，并使用`[remove](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html#remove())`删除它们:

```
Iterator<String> i = names.iterator();

while(i.hasNext()) {
    String e = i.next();
    if (e.startsWith("A")) {
        i.remove();
    }
}
```

尽管它很简单，但我们应该考虑一些警告:

*   **根据不同的集合，我们可能会遇到 [`ConcurrentModificationException`](/web/20220815140904/https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator) 异常**
*   在移除元素之前，我们需要迭代这些元素
*   **根据不同的收藏，`remove`的行为可能与预期不同。**例如:`ArrayList.Iterator`从集合中移除元素并将后续数据左移，而`LinkedList.Iterator`只是将指针调整到下一个元素。因此，`LinkedList.Iterator`在移除物品时比`ArrayList.Iterator`表现好得多

## 4。Java 8 和`Collection.removeIf()`

Java 8 引入了一个新方法给**`Collection`接口，提供了一个更简洁的方法来使用`[Predicate](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html)`** `:`移除元素

```
names.removeIf(e -> e.startsWith("A"));
```

值得注意的是，与`Iterator`方法相反， [`removeIf`](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#removeIf(java.util.function.Predicate)) 在`LinkedList`和`ArrayList`中表现相似。

在 Java 8 中，`ArrayList`覆盖了默认的实现——它依赖于`Iterator`——并实现了一个不同的策略:首先，它遍历元素并标记那些与我们的`Predicate;`匹配的元素。之后，它再次遍历以移除(并移动)在第一次遍历中标记的元素。

## 5。Java 8 与`Stream`简介

Java 8 新的主要特性之一是增加了 [`Stream`](/web/20220815140904/https://www.baeldung.com/java-8-streams-introduction) (以及 [`Collectors`](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collector.html) )。从源头上创造一个`Stream`有很多种方法。然而，大多数影响`Stream`实例的操作不会改变它的源，相反，API 专注于创建源的副本并执行我们可能需要的任何操作。

让我们看看如何使用`Stream`和`Collectors`来查找/过滤与我们的`Predicate`匹配和不匹配的元素。

### 5.1。用`Stream` 删除元素

使用`Stream`删除**过滤元素非常简单**，我们只需要使用`Collection`创建一个`Stream`的实例，使用`Predicate`调用`[filter](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#filter(java.util.function.Predicate))`，然后在`Collectors:`的帮助下使用 [`collect`](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#collect(java.util.stream.Collector)) 调用结果

```
Collection<String> filteredCollection = names
  .stream()
  .filter(e -> !e.startsWith("A"))
  .collect(Collectors.toList());
```

与以前的方法相比，它的侵入性更小，它促进了隔离，并能够从同一来源创建多个副本。但是，我们应该记住，它也增加了应用程序使用的内存。

### 5.2。`Collectors.partitioningBy`

组合`Stream.filter`和`Collectors`非常方便，尽管**我们可能会遇到既需要匹配元素又需要非匹配元素的情况。**在这种情况下我们可以利用`[Collectors.partitioningBy](https://web.archive.org/web/20220815140904/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#partitioningBy(java.util.function.Predicate,java.util.stream.Collector))`:

```
Map<Boolean, List<String>> classifiedElements = names
    .stream()
    .collect(Collectors.partitioningBy((String e) -> 
      !e.startsWith("A")));

String matching = String.join(",",
  classifiedElements.get(true));
String nonMatching = String.join(",",
  classifiedElements.get(false));
```

这个方法返回一个只包含两个键的`Map`，`true`和`false`，每个键分别指向一个包含匹配和不匹配元素的列表。

## 6。结论

在本文中，我们研究了一些从`Collections`中移除元素的方法以及它们的一些注意事项。

你可以在 GitHub 上找到本文[的完整源代码和所有代码片段。](https://web.archive.org/web/20220815140904/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)