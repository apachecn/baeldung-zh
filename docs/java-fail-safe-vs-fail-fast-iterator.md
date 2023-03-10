# 故障安全迭代器 vs 故障快速迭代器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator>

## 1。简介

在本文中，我们将介绍故障快速和故障安全的概念`Iterators`。

故障快速系统尽可能快地中止操作，立即暴露故障并停止整个操作。

然而，在出现故障的情况下，自动防故障系统不会中止操作。这种系统试图尽可能避免引发故障。

## 2。不倒的`Iterators`

当底层集合被修改时，Java 中的 Fail-fast 迭代器并不配合工作。

`Collections`维护一个名为`modCount`的内部计数器。每次从`Collection`中添加或删除一个项目时，该计数器就会增加。

当迭代时，在每个`next()`调用中，`modCount`的当前值与初始值进行比较。如果有不匹配，它抛出`ConcurrentModificationException` ，中止整个操作。

**从`java.util package`到`Collections`的默认迭代器，如`ArrayList`、`HashMap`等。不会出故障。**

```java
ArrayList<Integer> numbers = // ...

Iterator<Integer> iterator = numbers.iterator();
while (iterator.hasNext()) {
    Integer number = iterator.next();
    numbers.add(50);
}
```

在上面的代码片段中，`ConcurrentModificationException`在执行修改后的下一个迭代周期开始时抛出。

故障快速行为不能保证在所有场景中都发生，因为在并发修改的情况下不可能预测行为。这些迭代器尽最大努力抛出`ConcurrentModificationException`。

如果在对`Collection`、**的迭代过程中，使用`Iterator`的`remove()`方法移除了一个项目，那是完全安全的，不会抛出异常**。

但是，如果使用`Collection`的`remove()`方法移除一个元素，它会抛出一个异常:

```java
ArrayList<Integer> numbers = // ...

Iterator<Integer> iterator = numbers.iterator();
while (iterator.hasNext()) {
    if (iterator.next() == 30) {
        iterator.remove(); // ok!
    }
}

iterator = numbers.iterator();
while (iterator.hasNext()) {
    if (iterator.next() == 40) {
        numbers.remove(2); // exception
    }
}
```

## 3。故障安全迭代器

相对于异常处理的不便，故障安全迭代器更喜欢没有故障。

这些迭代器创建实际`Collection`的克隆，并对其进行迭代。如果迭代器创建后发生任何修改，副本仍然保持不变。因此，这些`Iterators`继续在`Collection`上循环，即使它被修改了。

然而，重要的是要记住没有真正的故障安全迭代器。正确的说法是弱一致。

也就是说，**如果一个** **`Collection`在被迭代时被修改，`Iterator`看到的是弱保证的**。对于不同的`Collections`，这种行为可能会有所不同，并在每个这样的`Collection`的 Javadocs 中进行了记录。

不过，自动防故障系统也有一些缺点。一个缺点是 **`Iterator`不能保证从`Collection`** 返回更新的数据，因为它是在克隆上工作，而不是在实际的 `Collection`上。

另一个缺点是创建`Collection`副本的开销，包括时间和内存。

**、`Iterators`、`Collections`上的`java.util.concurrent`包如`ConcurrentHashMap`、`CopyOnWriteArrayList`等。本质上是自动防故障的。**

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

map.put("First", 10);
map.put("Second", 20);
map.put("Third", 30);
map.put("Fourth", 40);

Iterator<String> iterator = map.keySet().iterator();

while (iterator.hasNext()) {
    String key = iterator.next();
    map.put("Fifth", 50);
}
```

在上面的代码片段中，我们使用了故障安全`Iterator`。因此，即使在迭代过程中向`Collection`添加了新元素，它也不会抛出异常。

`ConcurrentHashMap`的默认迭代器是弱一致的。这意味着这个`Iterator`可以容忍并发修改，遍历在`Iterator`建造时存在的元素，并且可能(但不保证)在`Iterator`建造后反映对`Collection`的修改。

因此，在上面的代码片段中，迭代循环了五次，这意味着**确实检测到了新添加到`Collection`中的元素。**

## 4。结论

在本教程中，我们已经了解了故障安全和故障快速的含义以及它们是如何在 Java 中实现的。

本文给出的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220625231750/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)