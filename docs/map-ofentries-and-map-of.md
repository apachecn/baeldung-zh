# Map.ofEntries()和 Map.of()之间的差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/map-ofentries-and-map-of>

## 1。简介

Java 8 引入了`Map.of()`方法，该方法使得创建不可变地图变得更加容易。Java 9 获得了的`Map.ofEntries()`方法，其功能略有不同。

在本教程中，我们将仔细研究这两个用于不可变映射的静态工厂方法，并解释哪一个适合哪一种用途。

## 2。`Map.of()`

`Map.of()` 方法**将指定数量的键值对作为参数**，并返回包含每个键值对的不可变映射。参数中配对的顺序对应于它们被添加到地图中的顺序。如果我们试图添加一个具有重复键的键值对，它将抛出一个`IllegalArgumentException`。如果我们试图添加一个`null`键或值，它将抛出一个`NullPointerException`。

作为重载的`static`工厂方法实现，第一个方法让我们创建一个空地图:

```java
static <K, V> Map<K, V> of() {
    return (Map<K,V>) ImmutableCollections.EMPTY_MAP;
}
```

让我们来看看用法:

```java
Map<Long, String> map = Map.of();
```

在`Map<K, V>`的接口中还定义了一个方法，它接受一个键和值:

```java
static <K, V> Map<K, V> of(K k1, V v1) {
    return new ImmutableCollections.Map1<>(k1, v1);
}
```

姑且称之为:

```java
Map<Long, String> map = Map.of(1L, "value1"); 
```

那些工厂**方法被重载 9 次以上，接受多达 10 个键和 10 个值**，我们可以在 [OpenJDK 17](https://web.archive.org/web/20221208143859/https://openjdk.org/projects/jdk/17/) 中找到:

```java
static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7, K k8, V v8, K k9, V v9, K k10, V v10) {
    return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10);
}
```

尽管这些方法非常有用，但是创建更多的方法会很麻烦。此外，我们不能使用`Map.of()`方法从现有的键和值创建映射，因为该方法只接受未定义的键值对作为参数。这就是`Map.ofEntries()` 方法的用武之地。

## 3。`Map.ofEntries()`

`Map.ofEntries()` 方法**将未指定数量的`Map.Entry<K, V>`对象作为参数**，并返回一个不可变的映射。同样，参数中配对的顺序与它们被添加到映射中的顺序相同。如果我们试图添加一个有重复键的键值对，它会抛出一个`IllegalArgumentException`。

我们来看看根据 [OpenJDK 17](https://web.archive.org/web/20221208143859/https://openjdk.org/projects/jdk/17/) 的静态工厂方法实现:

```java
static <K, V> Map<K, V> ofEntries(Entry<? extends K, ? extends V>... entries) {
    if (entries.length == 0) { // implicit null check of entries array
        var map = (Map<K,V>) ImmutableCollections.EMPTY_MAP;
        return map;
    } else if (entries.length == 1) {
        // implicit null check of the array slot
        return new ImmutableCollections.Map1<>(entries[0].getKey(), entries[0].getValue());
    } else {
        Object[] kva = new Object[entries.length << 1];
        int a = 0;
        for (Entry<? extends K, ? extends V> entry : entries) {
            // implicit null checks of each array slot
            kva[a++] = entry.getKey();
            kva[a++] = entry.getValue();
        }
        return new ImmutableCollections.MapN<>(kva);
     }
}
```

[可变参数](/web/20221208143859/https://www.baeldung.com/java-varargs)实现允许我们传递可变数量的条目。

例如，我们可以创建一个空地图:

```java
Map<Long, String> map = Map.ofEntries();
```

或者我们可以创建并填充一个地图:

```java
Map<Long, String> longUserMap = Map.ofEntries(Map.entry(1L, "User A"), Map.entry(2L, "User B"));
```

`Map.ofEntries()` 方法的一个很大的优点是我们也可以用它**从现有的键和值**中创建一个映射。这对于`Map.of()` 方法是不可能的，因为它只接受未定义的键值对作为参数。

## 4。结论

**`Map.of()` 方法仅适用于最多 10 个元素**的地图。这是因为它被实现为 11 个不同的重载方法，这些方法接受 0 到 10 个名称-值对作为参数。**另一方面，`Map.ofEntries()` 方法可以用于任何大小的地图**，因为它利用了特性[变量参数](/web/20221208143859/https://www.baeldung.com/java-varargs)。

GitHub 上的[提供了完整的示例。](https://web.archive.org/web/20221208143859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)