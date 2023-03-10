# Java 中的 HashSet 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashset>

## 1。概述

在本文中，我们将深入探讨`HashSet.`，它是最流行的`Set`实现之一，也是 Java 集合框架不可或缺的一部分。

## 2。`HashSet`简介

`HashSet`是 Java 集合 API `.`中的基本数据结构之一

让我们回忆一下该实现最重要的方面:

*   它存储唯一的元素并允许空值
*   它背后有一个`HashMap`
*   它不保持插入顺序
*   它不是线程安全的

注意，这个内部的`HashMap`在`HashSet`的实例被创建时被初始化:

```java
public HashSet() {
    map = new HashMap<>();
}
```

如果你想更深入地了解`HashMap`是如何工作的，你可以在这里阅读[关于它的文章。](/web/20220926185105/https://www.baeldung.com/java-hashmap)

## 3。**API**

在这一节中，我们将回顾最常用的方法，并看一些简单的例子。

### 3.1。`add()`

`add()`方法可用于向集合中添加元素。**方法契约声明只有当一个元素不存在于集合中时才会被添加。**如果添加了元素，该方法返回`true,`，否则返回`false.`

我们可以给`HashSet`添加一个元素，比如:

```java
@Test
public void whenAddingElement_shouldAddElement() {
    Set<String> hashset = new HashSet<>();

    assertTrue(hashset.add("String Added"));
}
```

从实现的角度来看，`add` 方法是一个非常重要的方法。实现细节说明了`HashSet`如何在内部工作并利用`HashMap's`和`put`方法:

```java
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

`map` 变量是对内部变量的引用，支持`HashMap:`

```java
private transient HashMap<E, Object> map;
```

最好先熟悉一下 [`hashcode`](/web/20220926185105/https://www.baeldung.com/java-hashcode) ，以便详细了解基于散列的数据结构中的元素是如何组织的。

总结:

*   一个`HashMap`是一个默认容量为 16 个元素的`buckets`数组——每个桶对应一个不同的 hashcode 值
*   如果不同的对象有相同的 hashcode 值，它们被存储在一个桶中
*   如果到达了`load factor` ,就会创建一个两倍于前一个数组大小的新数组，所有元素都会被重新散列并在新的对应存储桶中重新分配
*   为了检索一个值，我们散列一个键，修改它，然后转到一个相应的桶，在可能的链表中搜索，以防有不止一个对象

### 3.2。`contains()`

**`contains`方法的目的是检查一个元素是否出现在给定的`HashSet`中。**如果找到该元素，则返回`true` ，否则返回`false.`

我们可以在`HashSet`中检查一个元素:

```java
@Test
public void whenCheckingForElement_shouldSearchForElement() {
    Set<String> hashsetContains = new HashSet<>();
    hashsetContains.add("String Added");

    assertTrue(hashsetContains.contains("String Added"));
}
```

每当将对象传递给此方法时，都会计算哈希值。然后，相应的存储桶位置被解析和遍历。

### 3.3。 `remove()`

方法从集合中移除指定的元素(如果它存在)。如果一个集合包含指定的元素，这个方法返回`true` 。

让我们看一个工作示例:

```java
@Test
public void whenRemovingElement_shouldRemoveElement() {
    Set<String> removeFromHashSet = new HashSet<>();
    removeFromHashSet.add("String Added");

    assertTrue(removeFromHashSet.remove("String Added"));
}
```

### 3.4。`clear()`

当我们打算从一个集合中删除所有项目时，我们使用这种方法。底层实现简单地从底层`HashMap.`中清除所有元素

让我们看看实际情况:

```java
@Test
public void whenClearingHashSet_shouldClearHashSet() {
    Set<String> clearHashSet = new HashSet<>();
    clearHashSet.add("String Added");
    clearHashSet.clear();

    assertTrue(clearHashSet.isEmpty());
}
```

### 3.5。`size()`

这是 API 中的基本方法之一。它被大量使用，因为它有助于识别出现在`HashSet`中的元素数量。底层实现只是将计算委托给了`HashMap's size()`方法。

让我们看看实际情况:

```java
@Test
public void whenCheckingTheSizeOfHashSet_shouldReturnThesize() {
    Set<String> hashSetSize = new HashSet<>();
    hashSetSize.add("String Added");

    assertEquals(1, hashSetSize.size());
}
```

### 3.6。`isEmpty()`

我们可以用这个方法来判断一个给定的`HashSet`实例是否为空。如果集合不包含元素，该方法返回`true`:

```java
@Test
public void whenCheckingForEmptyHashSet_shouldCheckForEmpty() {
    Set<String> emptyHashSet = new HashSet<>();

    assertTrue(emptyHashSet.isEmpty());
}
```

### 3.7。`iterator()`

该方法返回一个遍历`Set`中元素的迭代器。**元素的访问没有特定的顺序，迭代器是无故障的**。

我们可以在这里观察随机迭代顺序:

```java
@Test
public void whenIteratingHashSet_shouldIterateHashSet() {
    Set<String> hashset = new HashSet<>();
    hashset.add("First");
    hashset.add("Second");
    hashset.add("Third");
    Iterator<String> itr = hashset.iterator();
    while(itr.hasNext()){
        System.out.println(itr.next());
    }
}
```

**如果在迭代器创建后的任何时候修改集合，除了通过迭代器自己的 remove 方法，`Iterator`抛出一个`ConcurrentModificationException`。**

让我们看看实际情况:

```java
@Test(expected = ConcurrentModificationException.class)
public void whenModifyingHashSetWhileIterating_shouldThrowException() {

    Set<String> hashset = new HashSet<>();
    hashset.add("First");
    hashset.add("Second");
    hashset.add("Third");
    Iterator<String> itr = hashset.iterator();
    while (itr.hasNext()) {
        itr.next();
        hashset.remove("Second");
    }
} 
```

或者，如果我们使用迭代器的 remove 方法，我们就不会遇到异常:

```java
@Test
public void whenRemovingElementUsingIterator_shouldRemoveElement() {

    Set<String> hashset = new HashSet<>();
    hashset.add("First");
    hashset.add("Second");
    hashset.add("Third");
    Iterator<String> itr = hashset.iterator();
    while (itr.hasNext()) {
        String element = itr.next();
        if (element.equals("Second"))
            itr.remove();
    }

    assertEquals(2, hashset.size());
}
```

迭代器的快速失效行为无法保证，因为在存在非同步并发修改的情况下不可能做出任何硬保证。

快速失败迭代器尽最大努力抛出`ConcurrentModificationException`。因此，编写依赖这个异常来保证正确性的程序是错误的。

## 4。`HashSet`如何保持唯一性？

当我们将一个对象放入`HashSet`时，它使用对象的`hashcode`值来确定一个元素是否已经不在集合中。

每个散列码值对应于可以包含各种元素的某个桶位置，对于这些元素，计算出的散列值是相同的。**但是具有相同`hashCode`的两个对象可能不等于**。

因此，相同存储桶中的对象将使用`equals()`方法进行比较。

## 5。`HashSet` 的表现

一个`HashSet`的性能主要受两个参数影响——它的`Initial Capacity`和`Load Factor`。

向集合中添加元素的预期时间复杂度是`O(1)`，在最坏的情况下(只有一个存储桶存在)，该时间复杂度可以下降到`O(n)`——因此，**保持正确的`HashSet's`容量是至关重要的。**

一个重要的提示:自从 JDK 8，[最坏情况的时间复杂度是`O(log*n)`。](https://web.archive.org/web/20220926185105/https://openjdk.java.net/jeps/180)

负载系数描述了最大填充水平，超过该水平，器械组将需要调整大小。

我们还可以为`initial capacity`和`load factor`创建一个带有自定义值的`HashSet`:

```java
Set<String> hashset = new HashSet<>();
Set<String> hashset = new HashSet<>(20);
Set<String> hashset = new HashSet<>(20, 0.5f); 
```

在第一种情况下，使用默认值-初始容量为 16，负载系数为 0.75。在第二个示例中，我们覆盖默认容量，在第三个示例中，我们覆盖两者。

低初始容量降低了空间复杂性，但增加了重新散列的频率，这是一个昂贵的过程。

另一方面，**高初始容量增加了迭代成本和初始内存消耗。** 

根据经验法则:

*   高初始容量有利于大量条目以及很少或没有迭代
*   低初始容量适合迭代次数多的少量条目

因此，在两者之间取得正确的平衡非常重要。通常，默认的实现是经过优化的，运行良好，如果我们觉得有必要调整这些参数来满足需求，我们需要明智地这样做。

## 6。结论

在这篇文章中，我们概述了一个`HashSet`的效用，它的目的以及它的底层工作。鉴于其恒定的时间性能和避免重复的能力，我们看到了它在可用性方面的效率。

我们研究了 API 中的一些重要方法，它们如何帮助我们作为开发人员利用`HashSet`的潜力。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220926185105/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set)