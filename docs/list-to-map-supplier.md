# 将列表转换为自定义供应商的映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/list-to-map-supplier>

## 1.概观

在本教程中，我们将把一个`List<E>`转换成一个`Map<K, List<E>>`。**我们将通过 [Java 的流 API](/web/20220626195940/https://www.baeldung.com/java-8-streams) 和[T2 的函数接口](/web/20220626195940/https://www.baeldung.com/java-8-functional-interfaces)** 来实现。

## 2.`Supplier`在 JDK 8

`Supplier`常用作工厂。一个方法可以接受一个`Supplier`作为输入，并使用一个有界通配符类型约束该类型，然后客户端可以传入一个创建给定类型的任何子类型的工厂。

除此之外，`Supplier`可以执行一个[懒代值](/web/20220626195940/https://www.baeldung.com/guava-memoizer)。

## 3.将`List`转换为`Map`

Stream API 提供了对`List`操作的支持。**一个这样的例子是`Stream#collect`方法**。然而，在流 API 方法中没有直接给下游参数提供`Suppliers`的方法。

在本教程中，我们将通过示例代码片段来看看 [`Collectors.groupingBy`](/web/20220626195940/https://www.baeldung.com/java-groupingby-collector) 、 [`Collectors.toMap`](/web/20220626195940/https://www.baeldung.com/java-collectors-tomap) 和 [`Stream.collect`](/web/20220626195940/https://www.baeldung.com/java-8-collectors) 方法。**我们将关注允许我们使用自定义`Supplier`** 的方法。

在本教程中，我们将在以下示例中处理一个`String List`集合:

```java
List source = Arrays.asList("List", "Map", "Set", "Tree");
```

我们将把上面的列表聚集成一个映射，它的关键字是字符串的长度。当我们完成后，我们将有一个看起来像这样的地图:

```java
{
    3: ["Map", "Set"],
    4: ["List", "Tree"]
}
```

### 3.1.`Collectors.groupingBy()`

使用`Collectors.groupingBy`，我们可以用特定的分类器将一个`Collection`转换成一个`Map`。分类器是一个元素的属性，我们将使用这个属性将元素合并到不同的组中:

```java
public Map<Integer, List> groupingByStringLength(List source, 
    Supplier<Map<Integer, List>> mapSupplier, 
    Supplier<List> listSupplier) {
    return source.stream()
        .collect(Collectors.groupingBy(String::length, mapSupplier, Collectors.toCollection(listSupplier)));
}
```

我们可以验证它是否适用于:

```java
Map<Integer, List> convertedMap = converter.groupingByStringLength(source, HashMap::new, ArrayList::new);
assertTrue(convertedMap.get(3).contains("Map"));
```

### 3.2.`Collectors.toMap()`

`Collectors.toMap`方法将流中的元素减少到一个`Map.`

我们首先用源字符串和`List`和`Map`供应商定义方法:

```java
public Map<Integer, List> collectorToMapByStringLength(List source, 
        Supplier<Map<Integer, List>> mapSupplier, 
        Supplier<List> listSupplier)
```

然后，我们定义如何从元素中获取键和值。为此，我们使用了两个新函数:

```java
Function<String, Integer> keyMapper = String::length;

Function<String, List> valueMapper = (element) -> {
    List collection = listSupplier.get();
    collection.add(element);
    return collection;
};
```

最后，我们定义一个在键冲突时调用的函数。在这种情况下，我们希望将两者的内容结合起来:

```java
BinaryOperator<List> mergeFunction = (existing, replacement) -> {
    existing.addAll(replacement);
    return existing;
};
```

综上所述，我们得到:

```java
source.stream().collect(Collectors.toMap(keyMapper, valueMapper, mergeFunction, mapSupplier))
```

注意，大多数时候我们定义的函数是方法的参数列表中的匿名内联函数。

让我们来测试一下:

```java
Map<Integer, List> convertedMap = converter.collectorToMapByStringLength(source, HashMap::new, ArrayList::new);
assertTrue(convertedMap.get(3).contains("Map"));
```

### 3.3.`Stream.collect()`

`Stream.collect`方法可以用来将流中的元素减少到任何类型的`Collection`中。

为此，我们还需要为`List`和`Map`供应商定义一个方法，一旦需要新的集合，就会调用该方法:

```java
public Map<Integer, List> streamCollectByStringLength(List source, 
        Supplier<Map<Integer, List>> mapSupplier, 
        Supplier<List> listSupplier)
```

然后我们定义一个`accumulator`,给定元素的键，获取一个现有的列表，或者创建一个新的列表，并将元素添加到响应中:

```java
BiConsumer<Map<Integer, List>, String> accumulator = (response, element) -> {
    Integer key = element.length();
    List values = response.getOrDefault(key, listSupplier.get());
    values.add(element);
    response.put(key, values);
};
```

最后，我们将合并累加器函数生成的值:

```java
BiConsumer<Map<Integer, List>, Map<Integer, List>> combiner = (res1, res2) -> {
    res1.putAll(res2);
};
```

把所有东西放在一起，然后我们只需要在元素流上调用`collect`方法:

```java
source.stream().collect(mapSupplier, accumulator, combiner);
```

注意，大多数时候我们定义的函数是方法的参数列表中的匿名内联函数。

测试结果将与其他两种方法相同:

```java
Map<Integer, List> convertedMap = converter.streamCollectByStringLength(source, HashMap::new, ArrayList::new);
assertTrue(convertedMap.get(3).contains("Map")); 
```

## 4.结论

在本教程中，我们展示了如何使用 Java 8 流 API 和自定义的`Supplier`将`List<E>`转换成`Map<K, List<E>>`

在 GitHub 上可以找到本教程中示例的完整源代码[。](https://web.archive.org/web/20220626195940/https://github.com/eugenp/tutorials/tree/master/core-java-modules/java-collections-conversions-2)