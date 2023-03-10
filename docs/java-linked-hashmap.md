# Java 中的 LinkedHashMap 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-linked-hashmap>

## 1。概述

在本文中，我们将探讨`LinkedHashMap`类的内部实现。 `LinkedHashMap`是`Map`接口的通用实现。

这个特殊的实现是`HashMap`的子类，因此共享了 [`HashMap`实现](/web/20221129002012/https://www.baeldung.com/java-hashmap)的核心构件。因此，强烈建议在继续阅读本文之前先温习一下。

## 2。`LinkedHashMap`vs`HashMap`

`LinkedHashMap`级在大多数方面与`HashMap`非常相似。然而，链接散列映射基于散列表和链表来增强散列映射的功能。

除了默认大小为 16 的底层数组之外，它还维护一个贯穿其所有条目的双向链表。

为了保持元素的顺序，链接散列表通过添加指向下一个和上一个条目的指针来修改`HashMap`的`Map.Entry`类:

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

注意，`Entry`类只是添加了两个指针；`before`和`after`，这使得它能够将自己挂钩到链表。除此之外，它使用 HashMap 的`Entry`类实现。

最后，记住这个链表定义了迭代的顺序，默认情况下是元素的插入顺序(insertion-order)。

## 3。插入顺序`LinkedHashMap`

让我们来看一个链接散列映射实例，它根据条目插入映射的方式对条目进行排序。它还保证在地图的整个生命周期中保持这种顺序:

```java
@Test
public void givenLinkedHashMap_whenGetsOrderedKeyset_thenCorrect() {
    LinkedHashMap<Integer, String> map = new LinkedHashMap<>();
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);

    Set<Integer> keys = map.keySet();
    Integer[] arr = keys.toArray(new Integer[0]);

    for (int i = 0; i < arr.length; i++) {
        assertEquals(new Integer(i + 1), arr[i]);
    }
}
```

这里，我们只是对链接散列图中条目的顺序进行了一个基本的、非结论性的测试。

我们可以保证这个测试将总是通过，因为插入顺序将总是保持不变。我们不能对散列表做同样的保证。

在接收任何地图、制作副本以进行操作并将其返回给调用代码的 API 中，该属性非常有用。如果客户机需要在调用 API 之前以同样的方式对返回的映射进行排序，那么链接的 hashmap 是一个不错的选择。

如果将键重新插入到地图中，插入顺序不受影响。

## 4。`LinkedHashMap`访问顺序

`LinkedHashMap`提供了一个特殊的构造函数，使我们能够在自定义负载系数(LF)和初始容量中指定一个不同的排序机制/策略，称为访问顺序:

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, .75f, true);
```

第一个参数是初始容量，随后是负载系数，**最后一个参数是订购模式**。因此，通过传入`true`，我们打开了访问顺序，而默认的是插入顺序。

这种机制确保元素的迭代顺序是元素最后被访问的顺序，从最近最少访问到最近访问。

因此，使用这种地图构建最近最少使用的(LRU)缓存非常简单实用。成功的`put`或`get`操作导致对条目的访问:

```java
@Test
public void givenLinkedHashMap_whenAccessOrderWorks_thenCorrect() {
    LinkedHashMap<Integer, String> map 
      = new LinkedHashMap<>(16, .75f, true);
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);

    Set<Integer> keys = map.keySet();
    assertEquals("[1, 2, 3, 4, 5]", keys.toString());

    map.get(4);
    assertEquals("[1, 2, 3, 5, 4]", keys.toString());

    map.get(1);
    assertEquals("[2, 3, 5, 4, 1]", keys.toString());

    map.get(3);
    assertEquals("[2, 5, 4, 1, 3]", keys.toString());
}
```

注意当我们在地图上执行访问操作时，键集中元素的顺序是如何转换的。

简单地说，对 map 的任何访问操作都会产生一个顺序，如果要立即执行迭代，那么被访问的元素将最后出现。

在上面的例子之后，很明显,`putAll`操作为指定映射中的每个映射生成一个条目访问。

自然地，在地图视图上的迭代不会影响支持地图的迭代顺序；**只有在地图上的显式访问操作才会影响顺序**。

`LinkedHashMap`还提供了一种机制，用于维护固定数量的映射，并在需要添加新条目的情况下保持删除最早的条目。

可以覆盖`removeEldestEntry`方法来强制执行自动删除过时映射的策略。

为了在实践中看到这一点，让我们创建我们自己的链接散列映射类，唯一的目的是通过扩展`LinkedHashMap`:

```java
public class MyLinkedHashMap<K, V> extends LinkedHashMap<K, V> {

    private static final int MAX_ENTRIES = 5;

    public MyLinkedHashMap(
      int initialCapacity, float loadFactor, boolean accessOrder) {
        super(initialCapacity, loadFactor, accessOrder);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

}
```

我们上面的覆盖将允许地图增长到 5 个条目的最大大小。当大小超过该值时，将插入每个新条目，代价是丢失映射中最老的条目，即最后访问时间在所有其他条目之前的条目:

```java
@Test
public void givenLinkedHashMap_whenRemovesEldestEntry_thenCorrect() {
    LinkedHashMap<Integer, String> map
      = new MyLinkedHashMap<>(16, .75f, true);
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);
    Set<Integer> keys = map.keySet();
    assertEquals("[1, 2, 3, 4, 5]", keys.toString());

    map.put(6, null);
    assertEquals("[2, 3, 4, 5, 6]", keys.toString());

    map.put(7, null);
    assertEquals("[3, 4, 5, 6, 7]", keys.toString());

    map.put(8, null);
    assertEquals("[4, 5, 6, 7, 8]", keys.toString());
}
```

请注意，当我们向地图中添加新条目时，键集开头的最旧条目是如何不断减少的。

## 5。性能考虑因素

就像`HashMap`，`LinkedHashMap`在常量时间内执行基本的`Map`操作:添加、删除和包含，只要散列函数是维数合适的。它还接受空键和空值。

然而，`LinkedHashMap`的这个**常量时间性能可能会比`HashMap`** 的常量时间稍差，因为维护一个双向链表会增加额外的开销。

对`LinkedHashMap`集合视图的迭代也需要线性时间`O(n)`，与`HashMap`相似。另一方面， **`LinkedHashMap`在迭代过程中的线性时间性能优于`HashMap`的线性时间**。

这是因为，对于`LinkedHashMap`，`O(n)`中的`n`只是地图中的条目数，与容量无关。而对于`HashMap`，`n`是容量和大小的总和，`O(size+capacity).`

负载系数和初始容量的精确定义见`HashMap`。然而，请注意，**选择过高的初始容量值对`LinkedHashMap`的惩罚没有对`HashMap`** 的惩罚严重，因为这个类的迭代时间不受容量的影响。

## 6。并发性

就像`HashMap`，`LinkedHashMap`实现是不同步的。因此，如果你要从多个线程中访问它，并且这些线程中至少有一个可能从结构上改变它，那么它必须是外部同步的。

最好在创建时这样做:

```java
Map m = Collections.synchronizedMap(new LinkedHashMap());
```

与`HashMap`的区别在于需要什么样的结构修改。**在访问有序的链接散列图中，仅仅调用`get` API 就会导致结构修改**。除此之外，还有像`put`和`remove`这样的操作。

## 7。结论

在本文中，我们探讨了 Java `LinkedHashMap`类作为`Map`接口最重要的实现之一的用法。我们也已经探索了它的内部工作方式与它的超类`HashMap`的区别。

希望在阅读完这篇文章之后，您能够做出更加明智和有效的决定，在您的用例中使用哪种 Map 实现。

本文中使用的所有示例的完整源代码可以在 GitHub 项目中找到。