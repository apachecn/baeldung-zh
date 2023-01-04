# 如何在 Java 中反转地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-invert-map>

## 1.概观

在这篇简短的文章中，我们将看看**如何在 Java** 中反转`Map` 。这个想法是为给定的类型为`Map<K, V>`的地图创建一个新的`Map<V, K>`实例。此外，我们还将看到如何处理源映射中存在重复值的情况。

请参考我们的另一篇文章来了解更多关于 [*散列表*类](/web/20220813160102/https://www.baeldung.com/java-hashmap)本身的信息。

## 2.定义问题

让我们假设有一个`Map`和几个`Key-Value`对:

```java
Map<String, Integer> map = new HashMap<>();
map.put("first", 1);
map.put("second", 2); 
```

最初的`Map`将存储如下项目:

```java
{first=1, second=2}
```

相反，我们希望**将键转化为值，反之亦然**转化为新的`Map`对象。结果将是:

```java
{1=first, 2=second} 
```

## 3.使用传统的 for 循环

首先，让我们看看如何使用一个`for`循环来**反转一个`Map`:**

```java
public static <V, K> Map<V, K> invertMapUsingForLoop(Map<K, V> map) {
    Map<V, K> inversedMap = new HashMap<V, K>();
    for (Entry<K, V> entry : map.entrySet()) {
        inversedMap.put(entry.getValue(), entry.getKey());
    }
    return inversedMap;
}
```

这里，我们正在遍历`Map`对象的`entrySet()` 。之后，我们将原来的`Value` 作为新的`Key`并将原来的`Key`作为新的`Value` 添加到`inversedMap`对象`.` 中，换句话说，**我们通过用值替换键，用键替换值**来复制地图的内容。此外，这适用于 8 之前的 Java 版本，尽管我们应该注意，这种方法**只有在源映射的值是惟一的**时才有效。

## 4。使用流 API 反转地图

Java 8 从`Stream` API 提供了方便的方法，以一种更加函数化的风格来反转`Map`。让我们来看看其中的几个。

### 4.1.`**Collectors.toMap()**`

如果在源映射中没有任何重复的值，我们可以使用`Collectors.toMap()` **:**

```java
public static <V, K> Map<V, K> invertMapUsingStreams(Map<K, V> map) {
    Map<V, K> inversedMap = map.entrySet()
        .stream()
        .collect(Collectors.toMap(Entry::getValue, Entry::getKey));
    return inversedMap;
}
```

首先，`entrySet()`被转换成一个对象流。随后，我们使用`Collectors.toMap()`将`Key`和`Value` 收集到`inversedMap`对象`.`中

让我们考虑源映射包含重复的值。在这种情况下，**我们可以使用映射函数将自定义规则应用于输入元素**:

```java
public static <K, V> Map<V, K> invertMapUsingMapper(Map<K, V> sourceMap) {
    return sourceMap.entrySet()
        .stream().collect(
            Collectors.toMap(Entry::getValue, Entry::getKey, (oldValue, newValue) -> oldValue) 
        );
}
```

在这个方法中，`Collectors.toMap()`的最后一个参数是一个映射函数。**使用这个，我们可以定制在有重复**的情况下应该添加哪个键。在上面的例子中，如果源映射包含重复的值，我们保留第一个值作为键。但是，如果值重复，我们只能保留一个键。

### 4.2.`Collectors.groupingBy()`

有时，我们可能需要所有的键，即使源映射包含重复的值。或者， **`Collectors.groupingBy()`为处理重复值**提供了更好的控制。

例如，假设我们有下面的`Key`–`Value`对:

```java
{first=1, second=2, two=2}
```

这里，值“2”对于不同的键重复两次。在这些情况下，我们可以使用`groupingBy() `方法在`Value`对象上实现级联的“group by”操作:

```java
private static <V, K> Map<V, List<K>> invertMapUsingGroupingBy(Map<K, V> map) {
    Map<V, List<K>> inversedMap = map.entrySet()
        .stream()
        .collect(Collectors.groupingBy(Map.Entry::getValue, Collectors.mapping(Map.Entry::getKey, Collectors.toList())));
    return inversedMap;
}
```

简单解释一下，`Collectors.mapping()`函数使用指定的收集器对与给定键相关联的值执行归约操作。`groupingBy()`收集器将重复值收集到一个`List`、**中，产生一个`MultiMap`、**。现在的输出将是:

```java
{1=[first], 2=[two, second]}
```

## 5.结论

在本文中，我们通过例子快速回顾了几种内建的反转`HashMap` 的方法。此外，我们还看到了如何在反转一个`Map`对象时处理重复值。

同时，一些外部库在`Map`接口之上提供了额外的特性。我们之前已经演示了如何使用[谷歌番石榴`BiMap`和](/web/20220813160102/https://www.baeldung.com/apache-commons-collections-vs-guava#bimap)[阿帕奇`BidiMap`](https://web.archive.org/web/20220813160102/https://baeldung.com/apache-commons-collections-vs-guava#bidimap) 反转一个`Map`。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220813160102/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)