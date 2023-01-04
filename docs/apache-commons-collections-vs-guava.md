# 阿帕奇公共收藏 vs 谷歌番石榴

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-collections-vs-guava>

## 1。概述

在本教程中，我们将比较**两个基于 Java 的开源库: [Apache Commons](https://web.archive.org/web/20220813160141/https://commons.apache.org/) 和[Google Guava](https://web.archive.org/web/20220813160141/https://github.com/google/guava)。这两个库都有丰富的特性集，主要在集合和 I/O 区域有许多实用 API。**

为了简洁起见，这里我们将只描述集合框架中少数最常用的方法以及代码示例。我们还将看到它们之间差异的总结。

此外，**我们收集了一系列文章，深入探究各种[公共资源](/web/20220813160141/https://www.baeldung.com/?s=apache+commons)和[番石榴](/web/20220813160141/https://www.baeldung.com/guava-guide)公用设施**。

## 2.两个图书馆的简史

谷歌番石榴是谷歌的一个项目，主要由该组织的工程师开发，尽管它现在已经开源了。启动它的主要动机是将 JDK 1.5 中引入的泛型纳入 Java 集合框架(T1)，或 T2 JCF(T3)，并增强其功能。

从一开始，这个库就扩展了它的功能，现在包括了图形、函数式编程、范围对象、缓存和`String`操作。

Apache Commons 最初是一个 Jakarta 项目，用来补充核心 Java 集合 API，最终成为 Apache Software Foundation 的一个项目。这些年来，它已经扩展到各种其他领域的大量可重用 Java 组件，包括(但不限于)图像、I/O、加密、缓存、网络、验证和对象池。

由于这是一个开源项目，来自 Apache 社区的开发人员不断向这个库添加内容以扩展其功能。然而，**他们非常注意保持向后兼容性**。

## 3。Maven 依赖关系

为了包含 Guava，我们需要将它的依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

它的最新版本信息可以在 [Maven](https://web.archive.org/web/20220813160141/https://search.maven.org/search?q=g:com.google.guava%20%20AND%20a:guava) 上找到。

对于 Apache Commons 来说，有点不同。根据我们想要使用的实用程序，我们必须添加特定的一个。例如，对于集合，我们需要添加:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

在我们的代码示例中，我们将使用 [`commons-collections4`](https://web.archive.org/web/20220813160141/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-collections4) 。

现在让我们开始有趣的部分吧！

## 4.双向地图

可以通过键和值访问的映射称为双向映射。JCF 没有这个功能。

让我们看看我们的两种技术如何提供它们。在这两种情况下，我们将以一周中的几天为例，根据给定的数字来获取日期的名称，反之亦然。

### 4.1.番石榴`BiMap`

Guava 提供了一个界面——[`BiMap`](/web/20220813160141/https://www.baeldung.com/guava-bimap)，作为双向图。**它可以用它的一个实现`EnumBiMap`、`EnumHashBiMap`、`HashBiMap`或`ImmutableBiMap`、**来实例化。

这里我们用的是`HashBiMap`:

```java
BiMap<Integer, String> daysOfWeek = HashBiMap.create();
```

填充它类似于 Java 中的任何映射:

```java
daysOfWeek.put(1, "Monday");
daysOfWeek.put(2, "Tuesday");
daysOfWeek.put(3, "Wednesday");
daysOfWeek.put(4, "Thursday");
daysOfWeek.put(5, "Friday");
daysOfWeek.put(6, "Saturday");
daysOfWeek.put(7, "Sunday");
```

这里有一些 JUnit 测试来证明这个概念:

```java
@Test
public void givenBiMap_whenValue_thenKeyReturned() {
    assertEquals(Integer.valueOf(7), daysOfWeek.inverse().get("Sunday"));
}

@Test
public void givenBiMap_whenKey_thenValueReturned() {
    assertEquals("Tuesday", daysOfWeek.get(2));
}
```

### 4.2.阿帕奇的`BidiMap`

同样，Apache 为我们提供了它的 [`BidiMap`](/web/20220813160141/https://www.baeldung.com/commons-collections-bidi-map) 接口:

```java
BidiMap<Integer, String> daysOfWeek = new TreeBidiMap<Integer, String>();
```

**这里我们用的是`TreeBidiMap`。然而，还有其他的实现，比如`DualHashBidiMap`和`DualTreeBidiMap`以及**。

为了填充它，我们可以像对上面的`BiMap`一样输入值。

它的用法也很相似:

```java
@Test
public void givenBidiMap_whenValue_thenKeyReturned() {
    assertEquals(Integer.valueOf(7), daysOfWeek.inverseBidiMap().get("Sunday"));
}

@Test
public void givenBidiMap_whenKey_thenValueReturned() {
    assertEquals("Tuesday", daysOfWeek.get(2));
}
```

在一些简单的性能测试中，[这种双向地图](https://web.archive.org/web/20220813160141/https://github.com/eugenp/tutorials/blob/master/libraries-6/src/test/java/com/baeldung/apache/commons/CollectionsUnitTest.java) **仅在插入方面落后于其对应的[番石榴](https://web.archive.org/web/20220813160141/https://github.com/eugenp/tutorials/blob/master/libraries-6/src/test/java/com/baeldung/guava/GuavaUnitTest.java)。它在获取键和值时要快得多**。

## 5.将键映射到多个值

对于我们希望将多个键映射到不同值的用例，比如水果和蔬菜的购物车集合，这两个库为我们提供了独特的解决方案。

### 5.1.番石榴`MultiMap`

首先，我们来看看如何实例化和初始化 [`MultiMap`](/web/20220813160141/https://www.baeldung.com/guava-multimap) :

```java
Multimap<String, String> groceryCart = ArrayListMultimap.create();

groceryCart.put("Fruits", "Apple");
groceryCart.put("Fruits", "Grapes");
groceryCart.put("Fruits", "Strawberries");
groceryCart.put("Vegetables", "Spinach");
groceryCart.put("Vegetables", "Cabbage");
```

然后，我们将使用几个 JUnit 测试来查看它的运行情况:

```java
@Test
public void givenMultiValuedMap_whenFruitsFetched_thenFruitsReturned() {
    List<String> fruits = Arrays.asList("Apple", "Grapes", "Strawberries");
    assertEquals(fruits, groceryCart.get("Fruits"));
}

@Test
public void givenMultiValuedMap_whenVeggiesFetched_thenVeggiesReturned() {
    List<String> veggies = Arrays.asList("Spinach", "Cabbage");
    assertEquals(veggies, groceryCart.get("Vegetables"));
} 
```

此外， **`MultiMap`让我们能够从映射**中删除给定条目或整组值:

```java
@Test
public void givenMultiValuedMap_whenFuitsRemoved_thenVeggiesPreserved() {

    assertEquals(5, groceryCart.size());

    groceryCart.remove("Fruits", "Apple");
    assertEquals(4, groceryCart.size());

    groceryCart.removeAll("Fruits");
    assertEquals(2, groceryCart.size());
}
```

正如我们所看到的，这里我们首先从`Fruits`集合中移除了`Apple`，然后移除了整个`Fruits`集合。

### 5.2.阿帕奇的`MultiValuedMap`

再次，让我们从实例化一个 [`MultiValuedMap`](/web/20220813160141/https://www.baeldung.com/apache-commons-multi-valued-map) 开始:

```java
MultiValuedMap<String, String> groceryCart = new ArrayListValuedHashMap<>();
```

由于填充它与我们在上一节中看到的相同，所以让我们快速查看一下用法:

```java
@Test
public void givenMultiValuedMap_whenFruitsFetched_thenFruitsReturned() {
    List<String> fruits = Arrays.asList("Apple", "Grapes", "Strawberries");
    assertEquals(fruits, groceryCart.get("Fruits"));
}

@Test
public void givenMultiValuedMap_whenVeggiesFetched_thenVeggiesReturned() {
    List<String> veggies = Arrays.asList("Spinach", "Cabbage");
    assertEquals(veggies, groceryCart.get("Vegetables"));
}
```

我们可以看到，它的用法也是一样的！

但是，在这种情况下，我们没有从`Fruits.` **中删除单个条目的灵活性，例如`Apple`，我们只能删除整个** `**Fruits**:`

```java
@Test
public void givenMultiValuedMap_whenFuitsRemoved_thenVeggiesPreserved() {
    assertEquals(5, groceryCart.size());

    groceryCart.remove("Fruits");
    assertEquals(2, groceryCart.size());
}
```

## 6.将多个键映射到一个值

这里，我们将举一个将纬度和经度映射到各个城市的例子:

```java
cityCoordinates.put("40.7128° N", "74.0060° W", "New York");
cityCoordinates.put("48.8566° N", "2.3522° E", "Paris");
cityCoordinates.put("19.0760° N", "72.8777° E", "Mumbai");
```

现在，我们来看看如何实现这一点。

### 6.1.番石榴`Table`

番石榴提供了满足上述用例的 [`Table`](/web/20220813160141/https://www.baeldung.com/guava-table) :

```java
Table<String, String, String> cityCoordinates = HashBasedTable.create();
```

下面是我们可以从中得出的一些用法:

```java
@Test
public void givenCoordinatesTable_whenFetched_thenOK() {

    List expectedLongitudes = Arrays.asList("74.0060° W", "2.3522° E", "72.8777° E");
    assertArrayEquals(expectedLongitudes.toArray(), cityCoordinates.columnKeySet().toArray());

    List expectedCities = Arrays.asList("New York", "Paris", "Mumbai");
    assertArrayEquals(expectedCities.toArray(), cityCoordinates.values().toArray());
    assertTrue(cityCoordinates.rowKeySet().contains("48.8566° N"));
}
```

如我们所见，我们可以获得行、列和值的`Set`视图。

**`Table`还为我们提供了查询其行或列的能力**。

让我们考虑一个电影表来演示这一点:

```java
Table<String, String, String> movies = HashBasedTable.create();

movies.put("Tom Hanks", "Meg Ryan", "You've Got Mail");
movies.put("Tom Hanks", "Catherine Zeta-Jones", "The Terminal");
movies.put("Bradley Cooper", "Lady Gaga", "A Star is Born");
movies.put("Keenu Reaves", "Sandra Bullock", "Speed");
movies.put("Tom Hanks", "Sandra Bullock", "Extremely Loud & Incredibly Close");
```

这里有一些简单明了的搜索示例，我们可以在`movies` `Table`上进行搜索:

```java
@Test
public void givenMoviesTable_whenFetched_thenOK() {
    assertEquals(3, movies.row("Tom Hanks").size());
    assertEquals(2, movies.column("Sandra Bullock").size());
    assertEquals("A Star is Born", movies.get("Bradley Cooper", "Lady Gaga"));
    assertTrue(movies.containsValue("Speed"));
}
```

然而， **`Table`限制我们只能将两个键映射到一个值**。在 Guava 中，我们还没有将两个以上的键映射到一个值的替代方法。

### 6.2.阿帕奇的`MultiKeyMap`

回到我们的`cityCoordinates`示例，下面是我们如何使用`MultiKeyMap`来操纵它:

```java
@Test
public void givenCoordinatesMultiKeyMap_whenQueried_thenOK() {
    MultiKeyMap<String, String> cityCoordinates = new MultiKeyMap<String, String>();

    // populate with keys and values as shown previously

    List expectedLongitudes = Arrays.asList("72.8777° E", "2.3522° E", "74.0060° W");
    List longitudes = new ArrayList<>();

    cityCoordinates.forEach((key, value) -> {
      longitudes.add(key.getKey(1));
    });
    assertArrayEquals(expectedLongitudes.toArray(), longitudes.toArray());

    List expectedCities = Arrays.asList("Mumbai", "Paris", "New York");
    List cities = new ArrayList<>();

    cityCoordinates.forEach((key, value) -> {
      cities.add(value);
    });
    assertArrayEquals(expectedCities.toArray(), cities.toArray());
}
```

从上面的代码片段中我们可以看到，为了得到与 Guava 的`Table`相同的断言，我们必须迭代`MultiKeyMap`。

然而， **`MultiKeyMap`也提供了将两个以上的键映射到一个值**的可能性。例如，它使我们能够将一周中的每一天映射为工作日或周末:

```java
@Test
public void givenDaysMultiKeyMap_whenFetched_thenOK() {
    days = new MultiKeyMap<String, String>();
    days.put("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Weekday");
    days.put("Saturday", "Sunday", "Weekend");

    assertFalse(days.get("Saturday", "Sunday").equals("Weekday"));
}
```

## 7.阿帕奇公共收藏 vs .谷歌番石榴

[据其工程师](https://web.archive.org/web/20220813160141/https://code.google.com/archive/p/google-collections/wikis/Faq.wiki) , **谷歌番石榴诞生于在库中使用泛型的需要，而 Apache Commons 没有提供这种功能**。它还遵循集合 API 对 tee 的要求。另一个主要优势是它正在积极开发中，新的版本会频繁出现。

然而，在从集合中获取值时，Apache 在性能方面具有优势。尽管如此，就插入时间而言，番石榴仍然独占鳌头。

虽然我们只比较了代码样本中的集合 API，但是与 Guava 相比， **Apache Commons 作为一个整体提供了更多的特性。**

## 8。结论

在本教程中，我们比较了 Apache Commons 和 Google Guava 提供的一些功能，特别是在集合框架方面。

在这里，我们仅仅触及了这两个库所能提供的皮毛。

而且，这不是一个非此即彼的比较。正如我们的代码示例所展示的，**两者都有各自独特的特性，而且可能会有两者共存的情况**。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220813160141/https://github.com/eugenp/tutorials/tree/master/libraries-6)