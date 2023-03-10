# 等同于 ConcurrentHashMap 的 Java 并发 HashSet

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrent-hashset-concurrenthashmap>

## 1.概观

在本教程中，我们将看到创建线程安全的`HashSet`实例的可能性，以及`ConcurrentHashMap`对于`HashSet`的等价意义。此外，我们将了解每种方法的优点和缺点。

## 2.使用`ConcurrentHashMap`工厂方法的线程安全`HashSet`

首先我们来看看公开了静态`newKeySet()`方法的 [`ConcurrentHashMap`](/web/20221220114603/https://www.baeldung.com/java-concurrent-map) 类。基本上，这个方法返回一个尊重`java.util.Set`接口的实例，并允许使用像`add(), contains(),`等标准方法。

这可以简单地创建为:

```java
Set<Integer> threadSafeUniqueNumbers = ConcurrentHashMap.newKeySet();
threadSafeUniqueNumbers.add(23);
threadSafeUniqueNumbers.add(45);
```

**此外，返回的`Set`的性能类似于 [`Has` `hSet`](/web/20221220114603/https://www.baeldung.com/java-hashset) ，因为两者都是使用基于哈希的算法实现的。**此外，同步逻辑增加的开销也很小，因为实现使用了`ConcurrentHashMap`。

最后，缺点是只有从 Java 8 开始，这个方法才存在。

## 3.使用`ConcurrentHashMap`实例方法的线程安全`HashSet`

到目前为止，我们已经看了`ConcurrentHashMap.` 的静态方法，接下来，我们将处理`ConcurrentHashMap`可用的实例方法来创建线程安全的`Set`实例。有两种方法可用，`newKeySet()`和`newKeySet(defaultValue)` ，两者略有不同。

**两种方法都创建了一个`Set`，与原地图**链接。换句话说，每次我们向起始的`ConcurrentHashMap,`添加一个新条目时，`Set`都会收到那个值。更进一步，我们来看看这两种方法的区别。

### 3.1. `newKeySet()`法

如上所述，`newKeySet()`公开了一个包含原始映射的所有键的`Set`。该方法与`newKeySet(defaultValue)`的关键区别在于，当前的方法不支持向`Set`添加新元素。**所以如果我们试图调用像`add()`或`addAll(),`这样的方法，我们会得到一个** `**UnsupportedOperationException**.`

虽然像`remove(object)`或`clear()`这样的操作正在按预期工作，但我们需要注意的是，`Set`上的任何变化都会反映在原始地图中:

```java
ConcurrentHashMap<Integer,String> numbersMap = new ConcurrentHashMap<>();
Set<Integer> numbersSet = numbersMap.keySet();

numbersMap.put(1, "One");
numbersMap.put(2, "Two");
numbersMap.put(3, "Three");

System.out.println("Map before remove: "+ numbersMap);
System.out.println("Set before remove: "+ numbersSet);

numbersSet.remove(2);

System.out.println("Set after remove: "+ numbersSet);
System.out.println("Map after remove: "+ numbersMap);
```

接下来是上面代码的输出:

```java
Map before remove: {1=One, 2=Two, 3=Three}
Set before remove: [1, 2, 3]

Set after remove: [1, 3]
Map after remove: {1=One, 3=Three}
```

### 3.2. `newKeySet(defaultValue)`法

让我们来看另一种从地图中的键创建一个`Set`的方法。**与上面提到的相比，`newKeySet(defaultValue)`返回一个`Set`实例，支持通过调用集合上的`add()`或`addAll()` 来添加新元素。**

进一步查看作为参数传递的默认值，它被用作通过`add()`或`addAll()`方法添加的 map 中每个新条目的值。下面的例子说明了这是如何工作的:

```java
ConcurrentHashMap<Integer,String> numbersMap = new ConcurrentHashMap<>();
Set<Integer> numbersSet = numbersMap.keySet("SET-ENTRY");

numbersMap.put(1, "One");
numbersMap.put(2, "Two");
numbersMap.put(3, "Three");

System.out.println("Map before add: "+ numbersMap);
System.out.println("Set before add: "+ numbersSet);

numbersSet.addAll(asList(4,5));

System.out.println("Map after add: "+ numbersMap);
System.out.println("Set after add: "+ numbersSet);
```

下面是上面代码的输出:

```java
Map before add: {1=One, 2=Two, 3=Three}
Set before add: [1, 2, 3]
Map after add: {1=One, 2=Two, 3=Three, 4=SET-ENTRY, 5=SET-ENTRY}
Set after add: [1, 2, 3, 4, 5]
```

## 4.使用`Collections`实用程序类的线程安全`HashSet`

让我们使用`java.util.Collections`中可用的`synchronizedSet()`方法来创建一个线程安全的`HashSet` 实例:

```java
Set<Integer> syncNumbers = Collections.synchronizedSet(new HashSet<>());
syncNumbers.add(1);
```

**在使用这种方法之前，我们需要意识到它的效率比上面讨论的方法低**。基本上，`synchronizedSet()`只是将`Set`实例包装到一个同步装饰器中，而`ConcurrentHashMap` 实现了一个底层并发机制。

## 5.使用`CopyOnWriteArraySet`的线程安全`Set`

创建线程安全的`Set`实现的最后一种方法是 [`CopyOnWriteArraySet`](/web/20221220114603/https://www.baeldung.com/java-copy-on-write-arraylist) 。创建这个`Set`的实例很简单:

```java
Set<Integer> copyOnArraySet = new CopyOnWriteArraySet<>();
copyOnArraySet.add(1);
```

尽管使用这个类看起来很吸引人，但我们需要考虑一些严重的性能缺陷。在后台，`CopyOnWriteArraySet`使用一个`Array,`而不是一个`HashMap,`来存储数据。**这意味着像`contains()`或`remove()`这样的操作具有 O(n)复杂度，而当使用由`ConcurrentHashMap,` 支持的集合时，复杂度是 O(1)。**

当`Set`的大小通常较小并且只读操作占大多数时，建议使用这种实现。

## 6.结论

在本文中，我们看到了创建线程安全`Set`实例的不同可能性，并强调了它们之间的差异。**首先我们已经看到了`ConcurrentHashMap.newKeySet() `静态方法。当需要线程安全的`HashSet`时，这应该是第一选择。后来我们看了看`ConcurrentHashMap`静态方法和`ConcurrentHashMap `实例的`newKeySet(), newKeySet(defaultValue)`有什么不同。**

最后我们还讨论了`Collections.` `synchronizedSet()`和`CopyOnWriteArraySet` 以及它们的性能缺陷。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221220114603/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections-2)