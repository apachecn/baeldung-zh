# 哈希集和树集比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashset-vs-treeset>

## 1。简介

在本文中，我们将比较两种最流行的 Java 接口实现——`java.util.Set` 和`TreeSet`。

## 2。差异

`HashSet`和`TreeSet`是同一个分支的叶子，但它们在一些重要的事情上有所不同。

### 2.1。订购

**`HashSet`以随机顺序存储对象，而`TreeSet`应用元素的自然顺序。**让我们看看下面的例子:

```
@Test
public void givenTreeSet_whenRetrievesObjects_thenNaturalOrder() {
    Set<String> set = new TreeSet<>();
    set.add("Baeldung");
    set.add("is");
    set.add("Awesome");

    assertEquals(3, set.size());
    assertTrue(set.iterator().next().equals("Awesome"));
}
```

将`String`对象添加到`TreeSet`后，我们看到第一个对象是“棒极了”，尽管它是在最后添加的。用`HashSet`完成的类似操作不能保证元素的顺序会随着时间保持不变。

### 2.2。`Null`对象

另一个区别是 **`HashSet`可以存储`null`对象，而`TreeSet`不允许**:

```
@Test(expected = NullPointerException.class)
public void givenTreeSet_whenAddNullObject_thenNullPointer() {
    Set<String> set = new TreeSet<>();
    set.add("Baeldung");
    set.add("is");
    set.add(null);
}

@Test
public void givenHashSet_whenAddNullObject_thenOK() {
    Set<String> set = new HashSet<>();
    set.add("Baeldung");
    set.add("is");
    set.add(null);

    assertEquals(3, set.size());
}
```

如果我们试图将`null`对象存储在`TreeSet`中，该操作将导致抛出`NullPointerException`。唯一的例外是在 Java 7 中，在`TreeSet`中只允许有一个`null`元素。

### 2.3。性能

**简单来说， `HashSet`比`TreeSet`快。**

`HashSet` 为大多数操作提供恒定时间性能，如`add()`、`remove()`和`contains()`，而`TreeSet.` 提供`log` ( `n`)时间

通常，我们可以看到**向`TreeSet`添加元素的执行时间要比`HashSet`** 多得多。

请记住，JVM 可能没有预热，所以执行时间可能不同。关于如何使用各种`Set`实现来设计和执行微测试的讨论可以在[这里](https://web.archive.org/web/20220627091052/https://stackoverflow.com/questions/23168490/hashset-and-treeset-performance-test)找到。

### 2.4。实施方法

**`TreeSet`功能丰富**，实现了额外的方法，如:

*   **`pollFirst()`**–返回第一个元素，如果`Set`为空，则返回`null`
*   **`pollLast()`**–检索并删除最后一个元素，如果`Set`为空，则返回`null`
*   **`first()`**–返回第一项
*   **`last()`–**返回最后一项
*   **`ceiling()`**–返回大于或等于给定元素的最小元素，如果没有这样的元素，则返回`null`
*   **`lower()`**–返回严格小于给定元素的最大元素，如果没有这样的元素，则返回`null`

上面提到的方法让`TreeSet`比`HashSet`好用得多，功能也更强大。

## 3。相似之处

### 3.1。独特元素

`TreeSet`和`HashSet`都保证了**元素的无重复集合，因为**是通用`Set`接口的一部分:

```
@Test
public void givenHashSetAndTreeSet_whenAddDuplicates_thenOnlyUnique() {
    Set<String> set = new HashSet<>();
    set.add("Baeldung");
    set.add("Baeldung");

    assertTrue(set.size() == 1);

    Set<String> set2 = new TreeSet<>();
    set2.add("Baeldung");
    set2.add("Baeldung");

    assertTrue(set2.size() == 1);
}
```

### 3.2。不是`synchronized`

**描述的`Set`实现都不是`synchronized`。**这意味着如果多个线程同时访问一个`Set`，并且至少有一个线程修改了它，那么它必须在外部同步。

### 3.3。快速失败迭代器

**由`TreeSet`和`HashSet`返回的`Iterator`是快速失效的。**

这意味着在`Iterator`创建后的任何时候对`Set`的任何修改都会抛出一个`ConcurrentModificationException:`

```
@Test(expected = ConcurrentModificationException.class)
public void givenHashSet_whenModifyWhenIterator_thenFailFast() {
    Set<String> set = new HashSet<>();
    set.add("Baeldung");
    Iterator<String> it = set.iterator();

    while (it.hasNext()) {
        set.add("Awesome");
        it.next();
    }
}
```

## 4。使用哪个实现？

这两个实现都实现了集合概念的契约，所以这取决于我们可能使用哪个实现的上下文。

以下是一些需要快速记住的要点:

*   如果我们想让我们的条目保持有序，我们需要使用`TreeSet`
*   如果我们更看重性能而不是内存消耗，我们应该选择`HashSet`
*   如果我们的内存不足，我们应该选择`TreeSet`
*   如果我们想要访问根据自然顺序彼此相对接近的元素，我们可能想要考虑`TreeSet`,因为它具有更大的局部性
*   使用`initialCapacity`和`loadFactor`可以调整`HashSet`的性能，这对于`TreeSet`是不可能的
*   如果我们想保持插入顺序并从常量时间访问中获益，我们可以使用`LinkedHashSet`

## 5。结论

在本文中，我们讨论了`TreeSet`和`HashSet`之间的区别和相似之处。

和往常一样，本文的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627091052/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-set)