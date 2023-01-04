# 在 Java 中迭代一个集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterate-set>

## 1.介绍

迭代元素是我们可以对集合执行的最基本的操作之一。

在本教程中，我们将看看如何迭代`Set`的元素，以及它与`List`或数组上的类似任务有何不同。

## 2.访问集合中的元素

与`List`和许多其他集合不同，`Set,`不是连续的。它们的元素没有索引，根据实现的不同，它们可能不保持顺序。

这意味着我们不能通过数字来询问集合中的特定元素。因此，我们不能使用典型的`for`循环或任何其他基于索引的方法。

### 2.1.`Iterator`

迭代集合的最基本和最接近金属的方法是调用由每个`Set`公开的`iterator`方法:

```
Set<String> names = Sets.newHashSet("Tom", "Jane", "Karen");
Iterator<String> namesIterator = names.iterator();
```

**然后我们可以用得到的`iterator`来得到那个`Set`的元素，一个一个的**。最具标志性的方法是检查`iterator`在`while`循环中是否有下一个元素:

```
while(namesIterator.hasNext()) {
   System.out.println(namesIterator.next());
}
```

我们也可以使用 Java 8 中添加的`forEachRemaining`方法:

```
namesIterator.forEachRemaining(System.out::println);
```

我们还可以混合这些解决方案:

```
String firstName = namesIterator.next(); // save first name to variable
namesIterator.forEachRemaining(System.out::println); // print rest of the names 
```

所有其他方法都会在某种程度上使用一个`Iterator`。

## 3.`Stream`年代

每个`Set`都公开了`spliterator()`方法。正因为如此，一个 **`Set`很容易被改造成一个`Stream`** :

```
names.stream().forEach(System.out::println);
```

我们还可以利用丰富的 [`Streams` API](/web/20221128114851/https://www.baeldung.com/java-8-streams) 来创建更复杂的管道。例如，让我们映射、记录，然后将集合中的元素缩减为单个字符串:

```
String namesJoined = names.stream()
    .map(String::toUpperCase)
    .peek(System.out::println)
    .collect(Collectors.joining());
```

## 4.增强型环路

虽然我们不能使用简单的索引`for`循环来迭代`Set`，但是我们可以使用 Java 5:

```
for (String name : names) {
    System.out.println(name);
}
```

## 5.使用索引迭代

### 5.1.转换为数组

没有索引，但是我们可以人工添加一个索引。一个可能的解决方案是简单地通过**将`Set`转换成某种更容易理解的数据结构，比如数组**:

```
Object[] namesArray = names.toArray();
for (int i = 0; i < namesArray.length; i++) {
    System.out.println(i + ": " + namesArray[i]);
}
```

请注意，仅转换为数组就将在`Set`上迭代一次。因此，就复杂性而言，我们将对`Set`迭代两次。如果性能至关重要，这可能是一个问题。

### 5.2.带索引的拉链

另一种方法是创建一个索引并用我们的`Set`压缩它。虽然我们可以用普通的 Java 来实现，但是有一些库专门为此提供了工具。

例如，我们可以使用 Vavr 的流:

```
Stream.ofAll(names)
  .zipWithIndex()
  .forEach(t -> System.out.println(t._2() + ": " + t._1()));
```

## 6.摘要

在本教程中，我们研究了迭代`Set`实例元素的各种方法。我们探讨了迭代器、流和循环的用法，以及它们之间的区别。

和往常一样，GitHub 上的例子[是可用的。](https://web.archive.org/web/20221128114851/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions-2)