# Java 中 HashMap 和 Hashtable 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hashmap-hashtable-differences>

## 1。概述

在这个简短的教程中，我们将重点关注`[Hashtable](/web/20221129004202/https://www.baeldung.com/java-hash-table)`和`[HashMap](/web/20221129004202/https://www.baeldung.com/java-hashmap)`之间的核心区别。

## 2。Java 中的`Hashtable`和`HashMap`

`Hashtable`和`HashMap`非常相似——都是实现了`Map`接口的集合。

另外，`put()`、`get()`、`remove()`和`containsKey()`方法提供了常数时间性能 O(1)。在内部，这些方法基于使用存储桶存储数据的散列的一般概念。

这两个类都不保持元素的插入顺序。换句话说，当我们迭代这些值时，添加的第一项可能不是第一项。

但是它们也有一些不同之处，在某些情况下使一个比另一个更好。让我们仔细看看这些差异。

## 3。`Hashtable`与`HashMap`和的区别

### 3.1。同步

首先， **`Hashtable`是线程安全的**，可以在应用程序的多个线程之间共享。

另一方面，`HashMap`是不同步的，如果没有额外的同步代码，就不能被多线程访问。我们可以使用`Collections.synchronizedMap()` 来制作一个`HashMap`的线程安全版本。我们也可以通过使用`synchronized`关键字来创建定制的锁代码或使代码线程安全。

`HashMap`不同步，因此它比`Hashtable`更快，使用的内存更少。通常，在单线程应用程序中，非同步对象比同步对象更快。

### 3.2。空值

另一个区别是`null `处理。`HashMap`允许添加一个以`null`为关键字的`Entry`，以及多个以`null`为值的条目。相比之下， **`Hashtable`根本不允许`null`**。我们来看一个`null`和`HashMap`的例子:

```java
HashMap<String, String> map = new HashMap<String, String>();
map.put(null, "value");
map.put("key1", null);
map.put("key2", null);
```

这将导致:

```java
assertEquals(3, map.size());
```

接下来，让我们看看 Hashtable 有什么不同:

```java
Hashtable<String, String> table = new Hashtable<String, String>();
table.put("key", null);
```

这导致了一个`NullPointerException`。添加一个以`null`为关键字的对象也会导致一个`NullPointerException`:

```java
table.put(null, "value");
```

### 3.3。元素迭代

`HashMap`使用 [`Iterator`](/web/20221129004202/https://www.baeldung.com/java-iterator) 来迭代值，而`Hashtable`使用`Enumerator`来迭代值。`Iterator`是`Enumerator`的继任者，消除了它的一些缺点。例如，`Iterator`有一个`remove()`方法从底层集合中移除元素。

`Iterator`是一个[快速失效迭代器](/web/20221129004202/https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator)。换句话说，当底层集合在迭代过程中被修改时，它抛出一个 [`ConcurrentModificationException`](/web/20221129004202/https://www.baeldung.com/java-concurrentmodificationexception) 。让我们看看快速失效的例子:

```java
HashMap<String, String> map = new HashMap<String, String>();
map.put("key1", "value1");
map.put("key2", "value2");

Iterator<String> iterator = map.keySet().iterator();
while(iterator.hasNext()){ 
    iterator.next();
    map.put("key4", "value4");
}
```

这抛出了一个`ConcurrentModificationException`异常，因为我们在迭代集合时调用了`put()`。

## 4。什么时候选择`HashMap`而不是`Hashtable`

对于非同步或单线程应用程序，我们应该使用`HashMap`。

值得一提的是，从 JDK 1.8 开始，`Hashtable`已经被弃用。不过， [`ConcurrentHashMap`](/web/20221129004202/https://www.baeldung.com/java-concurrent-map) 是`Hashtable`的绝佳替代品。我们应该考虑在多线程应用中使用`ConcurrentHashMap`。

## 5。结论

在本文中，我们举例说明了`HashMap`和`Hashtable`之间的区别，以及当我们需要选择一个时要记住什么。

像往常一样，所有这些例子和代码片段的实现都在 Github 上[完成。](https://web.archive.org/web/20221129004202/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)