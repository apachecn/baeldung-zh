# 用 Java 初始化 HashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-initialize-hashmap>

## 1.概观

在本教程中，我们将学习在 Java 中初始化`HashMap `的各种方法。

我们将使用 Java 8 和 Java 9。

## 延伸阅读:

## [比较 Java 中的两个 HashMaps】](/web/20220828131433/https://www.baeldung.com/java-compare-hashmaps)

Learn how to compare two HashMaps in Java as well as find the differences between them[Read more](/web/20220828131433/https://www.baeldung.com/java-compare-hashmaps) →

## [使用流处理地图](/web/20220828131433/https://www.baeldung.com/java-maps-streams)

Learn how to combine Java Maps and Streams[Read more](/web/20220828131433/https://www.baeldung.com/java-maps-streams) →

## 2.静态的静态初始化器`HashMap`

我们可以使用一个`static`代码块初始化一个`HashMap `:

```java
public static Map<String, String> articleMapOne;
static {
    articleMapOne = new HashMap<>();
    articleMapOne.put("ar01", "Intro to Map");
    articleMapOne.put("ar02", "Some article");
}
```

这种初始化的优点是映射是可变的，但它只对静态有效。因此，可以根据需要添加和删除条目。

让我们继续测试它:

```java
@Test
public void givenStaticMap_whenUpdated_thenCorrect() {

    MapInitializer.articleMapOne.put(
      "NewArticle1", "Convert array to List");

    assertEquals(
      MapInitializer.articleMapOne.get("NewArticle1"), 
      "Convert array to List");  
}
```

我们还可以使用双括号语法初始化映射:

```java
Map<String, String> doubleBraceMap  = new HashMap<String, String>() {{
    put("key1", "value1");
    put("key2", "value2");
}};
```

请注意，**我们必须尽量避免这种初始化技术，因为它在每次使用时都会创建一个匿名的额外类，保存对封闭对象**的隐藏引用，并可能导致内存泄漏问题。

## 3.使用 Java 集合

如果我们需要创建一个只有一个条目的单一不可变映射，`Collections.singletonMap() `变得非常有用:

```java
public static Map<String, String> createSingletonMap() {
    return Collections.singletonMap("username1", "password1");
}
```

注意，这里的映射是不可变的，如果我们试图添加更多的条目，它会抛出 `java.lang.UnsupportedOperationException.`

我们也可以使用`Collections.emptyMap():`创建一个不可变的空地图

```java
Map<String, String> emptyMap = Collections.emptyMap();
```

## 4.Java 8 之路

在这一节中，让我们看看使用 Java 8 `Stream API.`初始化 map 的方法

### 4.1。使用`Collectors.toMap()`

让我们使用一个二维`String`数组的`Stream`并将它们收集到一个地图中:

```java
Map<String, String> map = Stream.of(new String[][] {
  { "Hello", "World" }, 
  { "John", "Doe" }, 
}).collect(Collectors.toMap(data -> data[0], data -> data[1]));
```

注意这里键的数据类型和`Map`的值是相同的。

为了使它更通用，让我们取`Objects `的数组并执行相同的操作:

```java
 Map<String, Integer> map = Stream.of(new Object[][] { 
     { "data1", 1 }, 
     { "data2", 2 }, 
 }).collect(Collectors.toMap(data -> (String) data[0], data -> (Integer) data[1]));
```

因此，我们创建了一个键映射作为一个`String`，值映射作为一个`Integer`。

### 4.2.使用`Map.Entry`流

这里我们将使用`Map.Entry. `的实例，这是另一种方法，我们有不同的键和值类型。

首先，让我们使用`Entry `接口的`SimpleEntry `实现:

```java
Map<String, Integer> map = Stream.of(
  new AbstractMap.SimpleEntry<>("idea", 1), 
  new AbstractMap.SimpleEntry<>("mobile", 2))
  .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

现在让我们使用`SimpleImmutableEntry `实现来创建地图:

```java
Map<String, Integer> map = Stream.of(
  new AbstractMap.SimpleImmutableEntry<>("idea", 1),    
  new AbstractMap.SimpleImmutableEntry<>("mobile", 2))
  .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

### 4.3.初始化不可变映射

在某些用例中，我们需要初始化一个不可变的映射。这可以通过将`Collectors.toMap()`包装在`Collectors.collectingAndThen()`中来实现:

```java
Map<String, String> map = Stream.of(new String[][] { 
    { "Hello", "World" }, 
    { "John", "Doe" },
}).collect(Collectors.collectingAndThen(
    Collectors.toMap(data -> data[0], data -> data[1]), 
    Collections::<String, String> unmodifiableMap));
```

**注意，我们应该避免使用`Streams, `进行初始化，因为这会导致巨大的性能开销，并且会创建大量的垃圾对象来初始化映射。**

## 5.Java 9 之路

Java 9 在`Map`接口中提供了各种工厂方法，简化了不可变映射的创建和初始化。

让我们继续研究这些工厂方法。

### 5.1。`Map.of()`

此工厂方法不带任何参数、单个参数和可变参数:

```java
Map<String, String> emptyMap = Map.of();
Map<String, String> singletonMap = Map.of("key1", "value");
Map<String, String> map = Map.of("key1","value1", "key2", "value2");
```

注意，这个方法最多只支持 10 个键值对。

### `**5.2\. Map.ofEntries()**`

它与`Map.of() `类似，但是对键值对的数量没有限制:

```java
Map<String, String> map = Map.ofEntries(
  new AbstractMap.SimpleEntry<String, String>("name", "John"),
  new AbstractMap.SimpleEntry<String, String>("city", "budapest"),
  new AbstractMap.SimpleEntry<String, String>("zip", "000000"),
  new AbstractMap.SimpleEntry<String, String>("home", "1231231231")
);
```

注意，工厂方法产生不可变的映射，因此任何变异都会导致`UnsupportedOperationException.`

此外，它们不允许空键或重复键。

现在，如果我们在初始化后需要一个可变的或增长的映射，我们可以创建任何一个`Map`接口的实现，并在构造函数中传递这些不可变的映射:

```java
Map<String, String> map = new HashMap<String, String> (
  Map.of("key1","value1", "key2", "value2"));
Map<String, String> map2 = new HashMap<String, String> (
  Map.ofEntries(
    new AbstractMap.SimpleEntry<String, String>("name", "John"),    
    new AbstractMap.SimpleEntry<String, String>("city", "budapest")));
```

## 6.用番石榴

我们已经研究了使用核心 Java 的方法，让我们继续使用 Guava 库初始化一个 map:

```java
Map<String, String> articles 
  = ImmutableMap.of("Title", "My New Article", "Title2", "Second Article");
```

这将创建一个不可变的映射，并创建一个可变的映射:

```java
Map<String, String> articles 
  = Maps.newHashMap(ImmutableMap.of("Title", "My New Article", "Title2", "Second Article"));
```

方法`[ImmutableMap.of()](https://web.archive.org/web/20220828131433/https://guava.dev/releases/23.0/api/docs/com/google/common/collect/ImmutableMap.html#of--) `也有重载版本，可以接受多达 5 对键值参数。下面是一个有 2 对参数的示例:

```java
ImmutableMap.of("key1", "value1", "key2", "value2");
```

## 7.结论

在本文中，我们探索了初始化`Map`的各种方法，特别是创建空的、单例的、不可变的和可变的地图。**正如我们所见，自 Java 9 以来，这一领域有了巨大的进步。**

与往常一样，示例源代码位于 [Github 项目](https://web.archive.org/web/20220828131433/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)中。Java 9 示例位于[这里](https://web.archive.org/web/20220828131433/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)，而番石榴示例位于[这里](https://web.archive.org/web/20220828131433/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-map)。