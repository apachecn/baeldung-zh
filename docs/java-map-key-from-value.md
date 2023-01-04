# 从 Java 映射中获取值的键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-key-from-value>

## 1.介绍

在这个快速教程中，我们将演示三种不同的方法来从映射中检索给定值的键。我们还将讨论各种解决方案的优缺点。

要了解更多关于`Map`界面的信息，你可以查看[这篇文章](/web/20220914035401/https://www.baeldung.com/java-hashmap)。

## 2.迭代方法

`Java Collections`的`Map`接口提供了一个叫做`entrySet()`的方法。它在一个`Set`中返回映射的所有条目或键值对。

**这个想法是迭代这个条目集并返回其值与所提供的值相匹配的键:**

```java
public <K, V> K getKey(Map<K, V> map, V value) {
    for (Entry<K, V> entry : map.entrySet()) {
        if (entry.getValue().equals(value)) {
            return entry.getKey();
        }
    }
    return null;
}
```

但是，可能有多个键指向同一个值。

在这种情况下，如果找到了匹配的值，我们将这个键添加到一个`Set`中，并继续循环。最后，我们返回包含所有想要的键的`Set`:

```java
public <K, V> Set<K> getKeys(Map<K, V> map, V value) {
    Set<K> keys = new HashSet<>();
    for (Entry<K, V> entry : map.entrySet()) {
        if (entry.getValue().equals(value)) {
            keys.add(entry.getKey());
        }
    }
    return keys;
}
```

虽然这是一个非常直接的实现，但是即使在几次迭代之后找到了所有的匹配，它也会比较所有的条目。

## 3.功能方法

**随着 Java 8 中 Lambda 表达式的引入，我们可以用一种更灵活、可读性更强的方式来做这件事。**我们将条目集转换成一个`Stream`，并提供一个 lambda 来过滤那些具有给定值的条目。

然后，我们使用 map 方法从过滤后的条目中返回一个`Stream`键:

```java
public <K, V> Stream<K> keys(Map<K, V> map, V value) {
    return map
      .entrySet()
      .stream()
      .filter(entry -> value.equals(entry.getValue()))
      .map(Map.Entry::getKey);
}
```

返回流的好处是它可以满足广泛的客户需求。调用代码可能只需要一个键或指向所提供值的所有键。由于流的评估是惰性的，客户端可以根据它的需求控制迭代的次数。

此外，客户端可以使用适当的收集器将流转换为任何集合:

```java
Stream<String> keyStream1 = keys(capitalCountryMap, "South Africa");
String capital = keyStream1.findFirst().get();

Stream<String> keyStream2 = keys(capitalCountryMap, "South Africa");
Set<String> capitals = keyStream2.collect(Collectors.toSet());
```

## 4.使用 Apache Commons 集合

如果我们需要非常频繁地调用特定地图的函数，上面的想法不会很有帮助**。它将不必要地一次又一次迭代它的密钥集。**

在这种情况下，**维护另一个值到键的映射会更有意义，因为检索值的键需要持续的时间。**

由`Apache`提供的`Commons Collections`库提供了这样一个双向`Map`称为 [`BidiMap`](https://web.archive.org/web/20220914035401/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/BidiMap.html) 。它有一个名为`getKey()`的方法，用于检索给定值的键:

```java
BidiMap<String, String> capitalCountryMap = new DualHashBidiMap<>();
capitalCountryMap.put("Berlin", "Germany");
capitalCountryMap.put("Cape Town", "South Africa");
String capitalOfGermany = capitalCountryMap.getKey("Germany");
```

然而， **`BidiMap`在其键和值**之间强加了 1:1 的关系。如果我们试图放置一个键值对，它的值已经存在于`Map,`中，那么它会删除旧的条目。换句话说，它根据值更新键。

此外，它需要大量的内存来保存反向映射。

关于如何使用`BidiMap`的更多细节在[本教程](/web/20220914035401/https://www.baeldung.com/commons-collections-bidi-map)中。

## 5.使用谷歌番石榴

**我们可能会使用谷歌开发的另一种双向`Map`称为`[BiMap](https://web.archive.org/web/20220914035401/https://google.github.io/guava/releases/19.0/api/docs/com/google/common/collect/BiMap.html)`。**这个类提供了一个名为`inverse()`的方法来获取值键`Map`或相反的`Map`来获取基于给定值的键:

```java
HashBiMap<String, String> capitalCountryMap = HashBiMap.create();
capitalCountryMap.put("Berlin", "Germany");
capitalCountryMap.put("Cape Town", "South Africa");
String capitalOfGermany = capitalCountryMap.inverse().get("Germany");
```

与`BidiMap`、**、`BiMap`一样，也不允许多个键引用同一个值**。如果我们试图做出这样的尝试，它会抛出一个`java.lang.IllegalArgumentException` 。

不用说，`BiMap`也使用了大量的内存，因为它必须存储内部的逆映射。如果你有兴趣了解更多关于`BiMap`的信息，可以查看[这篇教程](/web/20220914035401/https://www.baeldung.com/guava-bimap)。

## 6.结论

在这篇简短的文章中，我们讨论了在给定值的情况下检索一个`Map's`键的一些方法。每种方法都有自己的优点和缺点。我们应该始终考虑用例，并根据情况选择最合适的用例。

上述教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220914035401/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps)