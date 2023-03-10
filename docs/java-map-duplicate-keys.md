# 如何在 Java 中存储一个 Map 中的重复键？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-duplicate-keys>

## 1。概述

在本教程中，我们将探索处理具有重复键的`Map`的可用选项，或者换句话说，处理允许为单个键存储多个值的`Map`。

## 2。标准地图

Java 有几个接口`Map`的实现，每个都有自己的特点。

然而，**现有的 Java core Map 实现都不允许`Map`处理单个键的多个值。**

正如我们所看到的，如果我们试图为同一个键插入两个值，第二个值将被存储，而第一个值将被丢弃。

它也将被返回(通过每一个正确的`[put(K key, V value)](https://web.archive.org/web/20221103195028/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html#put(K,V))`方法的实现):

```java
Map<String, String> map = new HashMap<>();
assertThat(map.put("key1", "value1")).isEqualTo(null);
assertThat(map.put("key1", "value2")).isEqualTo("value1");
assertThat(map.get("key1")).isEqualTo("value2"); 
```

那么，我们怎样才能达到预期的行为呢？

## 3。作为值收集

显然，对我们的`Map`的每个值使用一个`Collection`就可以完成这项工作:

```java
Map<String, List<String>> map = new HashMap<>();
List<String> list = new ArrayList<>();
map.put("key1", list);
map.get("key1").add("value1");
map.get("key1").add("value2");

assertThat(map.get("key1").get(0)).isEqualTo("value1");
assertThat(map.get("key1").get(1)).isEqualTo("value2"); 
```

然而，这种冗长的解决方案有许多缺点，并且容易出错。这意味着我们需要为每个值实例化一个`Collection`,在添加或删除值之前检查它的存在，当没有值时手动删除它，等等。

在 Java 8 中，我们可以利用`compute()`方法并对其进行改进:

```java
Map<String, List<String>> map = new HashMap<>();
map.computeIfAbsent("key1", k -> new ArrayList<>()).add("value1");
map.computeIfAbsent("key1", k -> new ArrayList<>()).add("value2");

assertThat(map.get("key1").get(0)).isEqualTo("value1");
assertThat(map.get("key1").get(1)).isEqualTo("value2"); 
```

但是，我们应该避免它，除非有非常好的理由不这样做，例如限制性的公司政策阻止我们使用第三方库。

否则，在编写我们自己的定制`Map`实现和重新发明轮子之前，我们应该在现成可用的几个选项中进行选择。

## 4。Apache Commons Collections

像往常一样，`Apache`对我们的问题有一个解决方案。

先来导入最新发布的`Common Collections` (CC 从现在开始):

```java
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-collections4</artifactId>
  <version>4.1</version>
</dependency>
```

### 4.1。`MultiMap`

`[org.apache.commons.collections4.**MultiMap**](https://web.archive.org/web/20221103195028/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/MultiMap.html)`接口定义了一个映射，它保存了每个键的值的集合。

它是由`[org.apache.commons.collections4.map.**MultiValueMap**](https://web.archive.org/web/20221103195028/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/map/MultiValueMap.html)`类实现的，该类自动处理大部分样板文件:

```java
MultiMap<String, String> map = new MultiValueMap<>();
map.put("key1", "value1");
map.put("key1", "value2");
assertThat((Collection<String>) map.get("key1"))
  .contains("value1", "value2"); 
```

虽然这个类从 CC 3.2 开始就有了，**它不是线程安全的**，并且**它在 CC 4.1 中已经被否决了。**我们应该只在无法升级到新版本时使用它。

### 4.2。`MultiValuedMap`

`MultiMap`的后继者是`[org.apache.commons.collections4.**MultiValuedMap**](https://web.archive.org/web/20221103195028/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/MultiValuedMap.html)`接口。它有多个可以使用的实现。

让我们看看如何将我们的多个值存储到一个`ArrayList`中，它保留了副本:

```java
MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
map.put("key1", "value1");
map.put("key1", "value2");
map.put("key1", "value2");
assertThat((Collection<String>) map.get("key1"))
  .containsExactly("value1", "value2", "value2"); 
```

或者，我们可以使用一个`HashSet`，它会删除重复项:

```java
MultiValuedMap<String, String> map = new HashSetValuedHashMap<>();
map.put("key1", "value1");
map.put("key1", "value1");
assertThat((Collection<String>) map.get("key1"))
  .containsExactly("value1"); 
```

以上两个**实现都不是线程安全的。**

让我们看看如何使用`UnmodifiableMultiValuedMap`装饰器使它们不可变:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenUnmodifiableMultiValuedMap_whenInserting_thenThrowingException() {
    MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
    map.put("key1", "value1");
    map.put("key1", "value2");
    MultiValuedMap<String, String> immutableMap =
      MultiMapUtils.unmodifiableMultiValuedMap(map);
    immutableMap.put("key1", "value3");
} 
```

## 5。`Multimap`番石榴

Guava 是用于 Java API 的 Google 核心库。

让我们从在我们的项目中导入番石榴开始:

```java
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>31.0.1-jre</version>
</dependency>
```

番石榴从一开始就遵循了多重实现的路径。

最常见的是`[com.google.common.collect.**ArrayListMultimap**](https://web.archive.org/web/20221103195028/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/collect/ArrayListMultimap.html)`，它使用一个`HashMap`和一个`ArrayList`作为每个值的后盾:

```java
Multimap<String, String> map = ArrayListMultimap.create();
map.put("key1", "value2");
map.put("key1", "value1");
assertThat((Collection<String>) map.get("key1"))
  .containsExactly("value2", "value1"); 
```

和往常一样，我们应该更喜欢 Multimap 接口的不可变实现:`[com.google.common.collect.**ImmutableListMultimap**](https://web.archive.org/web/20221103195028/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/collect/ImmutableListMultimap.html)`和`[com.google.common.collect.**ImmutableSetMultimap**](https://web.archive.org/web/20221103195028/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/collect/ImmutableSetMultimap.html)`。

### 5.1。常见地图实现

当我们需要一个特定的`Map`实现时，首先要做的是检查它是否存在，因为 Guava 很可能已经实现了它。

例如，我们可以使用`[com.google.common.collect.**LinkedHashMultimap**](https://web.archive.org/web/20221103195028/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/collect/LinkedHashMultimap.html)`，它保留了键和值的插入顺序:

```java
Multimap<String, String> map = LinkedHashMultimap.create();
map.put("key1", "value3");
map.put("key1", "value1");
map.put("key1", "value2");
assertThat((Collection<String>) map.get("key1"))
  .containsExactly("value3", "value1", "value2"); 
```

或者，我们可以使用一个`[com.google.common.collect.**TreeMultimap**](https://web.archive.org/web/20221103195028/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/collect/TreeMultimap.html)`，它按照自然顺序迭代键和值:

```java
Multimap<String, String> map = TreeMultimap.create();
map.put("key1", "value3");
map.put("key1", "value1");
map.put("key1", "value2");
assertThat((Collection<String>) map.get("key1"))
  .containsExactly("value1", "value2", "value3"); 
```

### 5.2。`MultiMap`伪造我们的风俗

许多其他实现是可用的。

然而，我们可能想要修饰一个还没有实现的`Map`和/或`List`。

幸运的是，番石榴有一个工厂化的方法让我们可以这样做——`[Multimap.newMultimap()](https://web.archive.org/web/20221103195028/https://google.github.io/guava/releases/23.0/api/docs/com/google/common/collect/Multimaps.html#newMultimap-java.util.Map-com.google.common.base.Supplier-)`。

## 6。结论

我们已经看到了如何以所有现有的主要方式在一个`Map`中存储一个键的多个值。

我们已经探讨了 Apache Commons 集合和 Guava 的最流行的实现，如果可能的话，它们应该优先于定制解决方案。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221103195028/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps)