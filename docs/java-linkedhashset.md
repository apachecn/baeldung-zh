# Java 中的 LinkedHashSet 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-linkedhashset>

## 1。概述

在本文中，**我们将探讨 Java `Collections` API** 的 [`LinkedHashSet`](https://web.archive.org/web/20221208143859/https://docs.oracle.com/javase/7/docs/api/java/util/LinkedHashSet.html) 类。我们将深入研究这种数据结构的特性，并演示其功能。

## 2。`LinkedHashSet`简介

`LinkedHashSet`是属于`Java.util`库的通用数据结构。它是 [`HashSet`](/web/20221208143859/https://www.baeldung.com/java-hashset) 数据结构的直接后代，因此，在每个给定的时间都包含不重复的元素。

**除了有一个贯穿所有条目的双向链表之外，它的实现与`HashSet`的不同之处在于它维护了一个可预测的迭代顺序**。迭代顺序由元素在集合中的插入顺序定义。

**`LinkedHashSet`将客户端从`HashSet`提供的不可预测的排序中拯救出来，而不会招致 [`TreeSet`](/web/20221208143859/https://www.baeldung.com/java-tree-set)** 带来的复杂性。

虽然**的性能可能略低于`HashSet`** ，但由于维护链表的额外开销，**对于`add(), contains()` 和`remove()`操作**具有常数时间(O1)性能。

## 3.创建一个`LinkedHashSet`

有几个构造函数可以用来创建一个`LinkedHashSet`。让我们逐一看看:

### 3.1.默认无参数构造函数

```java
Set<String> linkedHashSet = new LinkedHashSet<>();
assertTrue(linkedHashSet.isEmpty());
```

### 3.2.用初始容量创建

初始容量代表`LinkedHashSet`的初始长度。**提供初始容量可以防止`Set`随着**的增长而不必要地调整大小。默认初始容量为 16:

```java
LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>(20);
```

### 3.3.从`Collection`创建

我们还可以在创建时使用`Collection`的内容来填充`LinkedHashSet`对象:

```java
@Test
 void whenCreatingLinkedHashSetWithExistingCollection_shouldContainAllElementOfCollection(){
      Collection<String> data = Arrays.asList("first", "second", "third", "fourth", "fifth");
      LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>(data);

      assertFalse(linkedHashSet.isEmpty());
      assertEquals(data.size(), linkedHashSet.size());
      assertTrue(linkedHashSet.containsAll(data) && data.containsAll(linkedHashSet));
 }
```

### 3.4.创建初始容量和装载系数

当`LinkedHashSet`的大小超过初始容量值时，新的容量是负载系数和先前容量的乘积。在下面的代码片段中，初始容量设置为 20，负载系数为 3。

```java
LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>(20, 3);
```

默认负载系数为 0.75。

## 4.向`LinkedHashSet`添加一个元素

我们可以通过分别使用`add()`和`addAll()`方法向`LinkedHashSet`添加单个元素或一个`Collection`元素。**如果`Set`** 中不存在某个元素，则会添加该元素。当一个元素被添加到集合中时，这些方法返回`true`，否则，它们返回`false` 。

### 4.1.添加单个元素

下面是一个向`LinkedHashSet`添加元素的实现:

```java
@Test
void whenAddingElement_shouldAddElement(){
    Set<Integer> linkedHashSet = new LinkedHashSet<>();
    assertTrue(linkedHashSet.add(0));
    assertFalse(linkedHashSet.add(0));
    assertTrue(linkedHashSet.contains(0));

}
```

### 4.2.添加一个`Collection`元素

如前所述，我们还可以向一个`LinkedHashSet`添加一个`Collection`元素:

```java
@Test
void whenAddingCollection_shouldAddAllContentOfCollection(){
    Collection<Integer> data = Arrays.asList(1,2,3);
    LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();

    assertTrue(linkedHashSet.addAll(data));
    assertTrue(data.containsAll(linkedHashSet) && linkedHashSet.containsAll(data));
 }
```

**不添加重复项的规则也适用于`addAll()`方法**，如下所示:

```java
@Test
void whenAddingCollectionWithDuplicateElements_shouldMaintainUniqueValuesInSet(){
    LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();
    linkedHashSet.add(2);
    Collection<Integer> data = Arrays.asList(1, 1, 2, 3);

    assertTrue(linkedHashSet.addAll(data));
    assertEquals(3, linkedHashSet.size());
    assertTrue(data.containsAll(linkedHashSet) && linkedHashSet.containsAll(data));
}
```

请注意，在调用`addAll()`方法之前，`data`变量包含重复值 1，而`LinkedHashSet`已经包含了`Integer`值 2。

## 5.遍历一个`LinkedHashSet`

像所有其他的`Collection`库的后代一样，我们可以遍历一个`LinkedHashSet`。**一个`LinkedHashSet`中有两种类型的迭代器: [`Iterator`](/web/20221208143859/https://www.baeldung.com/java-iterator) 和 [`Spliterator`](/web/20221208143859/https://www.baeldung.com/java-spliterator)** 。

**前者只能遍历一个`Collection`并执行任何基本操作，而后者将`Collection`分割成子集，并对每个子集并行执行不同的操作，从而使其成为线程安全的**。

### 5.1.使用`Iterator`进行迭代

```java
@Test
void whenIteratingWithIterator_assertThatElementIsPresent(){
    LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();
    linkedHashSet.add(0);
    linkedHashSet.add(1);
    linkedHashSet.add(2);

    Iterator<Integer> iterator = linkedHashSet.iterator();
    for (int i = 0; i < linkedHashSet.size(); i++) {
        int nextData = iterator.next();
        assertEquals(i, nextData);
    }
}
```

### 5.2.用一个`Spliterator`迭代

```java
@Test
void whenIteratingWithSpliterator_assertThatElementIsPresent(){
    LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();
    linkedHashSet.add(0);
    linkedHashSet.add(1);
    linkedHashSet.add(2);

    Spliterator<Integer> spliterator = linkedHashSet.spliterator();
    AtomicInteger counter = new AtomicInteger();
    spliterator.forEachRemaining(data -> {
       assertEquals(counter.get(), (int)data);
       counter.getAndIncrement();
    });
}
```

## 6.从 `LinkedHashSet`中删除元素

以下是从`LinkedHashSet`中删除元素的不同方法:

### 6.1.`remove()`

这个方法从`Set`中移除一个元素，假设我们知道我们想要移除的确切元素。它接受一个参数，即我们想要移除的实际元素，如果成功移除，则返回`true`，否则返回`false`:

```java
@Test
void whenRemovingAnElement_shouldRemoveElement(){
    Collection<String> data = Arrays.asList("first", "second", "third", "fourth", "fifth");
    LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>(data);

    assertTrue(linkedHashSet.remove("second"));
    assertFalse(linkedHashSet.contains("second"));
}
```

### 6.2.`removeIf()`

**`removeIf()`方法删除满足指定谓词条件的元素。**以下示例删除了`LinkedHashSet`中大于 2 的所有元素:

```java
@Test
void whenRemovingAnElementGreaterThanTwo_shouldRemoveElement(){
    LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();
    linkedHashSet.add(0);
    linkedHashSet.add(1);
    linkedHashSet.add(2);
    linkedHashSet.add(3);
    linkedHashSet.add(4);

    linkedHashSet.removeIf(data -> data > 2);
    assertFalse(linkedHashSet.contains(3));
    assertFalse(linkedHashSet.contains(4));
}
```

### 6.3.用`Iterator`移除

`iterator`也是我们可以用来从`LinkedHashSet`中移除元素的另一个选项。`Iterator`的`remove()`方法移除了`Iterator`当前所在的元素:

```java
@Test
void whenRemovingAnElementWithIterator_shouldRemoveElement(){
    LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();
    linkedHashSet.add(0);
    linkedHashSet.add(1);
    linkedHashSet.add(2);

    Iterator<Integer> iterator = linkedHashSet.iterator();
    int elementToRemove = 1;
    assertTrue(linkedHashSet.contains(elementToRemove));
    while(iterator.hasNext()){
        if(elementToRemove == iterator.next()){
           iterator.remove();
       }
    }
    assertFalse(linkedHashSet.contains(elementToRemove));
}
```

## 7.结论

在本文中，我们研究了 Java `Collections`库中的`LinkedHashSet`数据结构。我们演示了如何通过不同的构造函数创建一个`LinkedHashSet`，添加和删除元素，以及遍历它。我们还了解了这种数据结构的底层、它相对于`HashSet`的优势以及它的常见操作的时间复杂度。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set-2)