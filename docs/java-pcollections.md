# p 系列简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pcollections>

## 1。概述

在本文中，我们将会看到 [PCollections](https://web.archive.org/web/20220630012233/https://pcollections.org/) ，这是一个 **Java 库，提供持久的、不可变的集合。**

[持久数据](https://web.archive.org/web/20220630012233/https://en.wikipedia.org/wiki/Persistent_data_structure)结构(集合)不能在更新操作期间直接修改，而是返回一个带有更新操作结果的新对象。它们不仅是不可变的，而且是持久的——这意味着在执行修改后，集合的先前版本保持不变。

PCollections 类似于 Java Collections 框架并与之兼容。

## 2。依赖性

让我们将以下依赖项添加到我们的`pom.xml`中，以便在我们的项目中使用 PCollections:

```java
<dependency>
    <groupId>org.pcollections</groupId>
    <artifactId>pcollections</artifactId>
    <version>2.1.2</version>
</dependency>
```

如果我们的项目是基于 Gradle 的，我们可以将相同的工件添加到我们的`build.gradle`文件中:

```java
compile 'org.pcollections:pcollections:2.1.2'
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220630012233/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22pcollections%22%20g%3A%22org.pcollections%22) 上找到。

## 3。地图结构(`HashPMap` )

`HashPMap`是一种持久的地图数据结构。它是`java.util.HashMap` 的模拟，用于存储非空的键值数据。

我们可以通过在`HashTreePMap.` 中使用方便的静态方法来实例化`HashPMap`，这些静态方法返回一个由`IntTreePMap.`支持的`HashPMap`实例

`HashTreePMap`类的静态`empty()`方法创建一个没有元素的空的`HashPMap`——就像使用默认的`java.util.HashMap`构造函数一样:

```java
HashPMap<String, String> pmap = HashTreePMap.empty();
```

我们可以使用另外两个静态方法来创建`HashPMap`。`singleton()`方法创建一个只有一个条目的`HashPMap`:

```java
HashPMap<String, String> pmap1 = HashTreePMap.singleton("key1", "value1");
assertEquals(pmap1.size(), 1);
```

`from()`方法从现有的`java.util.HashMap`实例(和其他`java.util.Map`实现)创建一个`HashPMap`:

```java
Map map = new HashMap();
map.put("mkey1", "mval1");
map.put("mkey2", "mval2");

HashPMap<String, String> pmap2 = HashTreePMap.from(map);
assertEquals(pmap2.size(), 2);
```

虽然`HashPMap`继承了`java.util.AbstractMap`和`java.util.Map`的一些方法，但是它有自己独特的方法。

`minus()`方法从映射中删除单个条目，而`minusAll()`方法删除多个条目。还有分别添加单个和多个条目的`plus()`和`plusAll()`方法:

```java
HashPMap<String, String> pmap = HashTreePMap.empty();
HashPMap<String, String> pmap0 = pmap.plus("key1", "value1");

Map map = new HashMap();
map.put("key2", "val2");
map.put("key3", "val3");
HashPMap<String, String> pmap1 = pmap0.plusAll(map);

HashPMap<String, String> pmap2 = pmap1.minus("key1");

HashPMap<String, String> pmap3 = pmap2.minusAll(map.keySet());

assertEquals(pmap0.size(), 1);
assertEquals(pmap1.size(), 3);
assertFalse(pmap2.containsKey("key1"));
assertEquals(pmap3.size(), 0);
```

需要注意的是，在`pmap`上调用`put()`会抛出一个`UnsupportedOperationException.`，因为 PCollections 对象是持久的和不可变的，所以每次修改操作都会返回一个对象的新实例(`HashPMap`)。

让我们继续看看其他的数据结构。

## 4。列表结构(`TreePVector and ConsPStack` )

`TreePVector`是`java.util.ArrayList`的持续模拟，而`ConsPStack`是`java.util.LinkedList`的模拟。`TreePVector`和`ConsPStack`有方便的静态方法来创建新实例——就像`HashPMap`一样。

`empty()`方法创建一个空的`TreePVector`，而`singleton()`方法创建一个只有一个元素的`TreePVector`。还有一个`from()`方法，可以用来从任何`java.util.Collection`创建一个`TreePVector`的实例。

`ConsPStack`具有相同名称的静态方法，可以实现相同的目标。

有操作它的方法。它有`minus()`和`minusAll()`两种去除元素的方法；用于添加元素的`plus()`和`plusAll()`。

`with()`用于替换指定索引处的元素，`subList()`从集合中获取一系列元素。

这些方法也可以在`ConsPStack` 中找到。

让我们考虑下面的代码片段，它举例说明了上面提到的方法:

```java
TreePVector pVector = TreePVector.empty();

TreePVector pV1 = pVector.plus("e1");
TreePVector pV2 = pV1.plusAll(Arrays.asList("e2", "e3", "e4"));
assertEquals(1, pV1.size());
assertEquals(4, pV2.size());

TreePVector pV3 = pV2.minus("e1");
TreePVector pV4 = pV3.minusAll(Arrays.asList("e2", "e3", "e4"));
assertEquals(pV3.size(), 3);
assertEquals(pV4.size(), 0);

TreePVector pSub = pV2.subList(0, 2);
assertTrue(pSub.contains("e1") && pSub.contains("e2"));

TreePVector pVW = (TreePVector) pV2.with(0, "e10");
assertEquals(pVW.get(0), "e10");
```

在上面的代码片段中，`pSub` 是另一个`TreePVector`对象，并且独立于`pV2`。可以观察到，`pV2`没有被`subList()`操作改变；相反，创建了一个新的`TreePVector`对象，并用从索引 0 到 2 的`pV2`元素填充。

这就是不变性的含义，也是集合的所有修改方法都会发生的情况。

## 5。设定结构(`MapPSet` )

`MapPSet`是`java.util.HashSet`的一个持久的、地图支持的模拟。可以通过`HashTreePSet –` 、`from()`、`singleton()`的静态方法方便地实例化。它们的功能与前面示例中解释的方式相同。

`MapPSet`有`plus()`、`plusAll()`、`minus()`和`minusAll()`四种操作设定数据的方法。此外，它继承了`java.util.Set`、`java.util.AbstractCollection`和`java.util.AbstractSet`的方法:

```java
MapPSet pSet = HashTreePSet.empty()     
  .plusAll(Arrays.asList("e1","e2","e3","e4"));
assertEquals(pSet.size(), 4);

MapPSet pSet1 = pSet.minus("e4");
assertFalse(pSet1.contains("e4"));
```

最后，还有`OrderedPSet`——它维护元素的插入顺序，就像`java.util.LinkedHashSet`一样。

## 6。结论

总之，在这个快速教程中，我们探索了 p collections——类似于 Java 中核心集合的持久数据结构。当然，PCollections [Javadoc](https://web.archive.org/web/20220630012233/https://www.javadoc.io/doc/org.pcollections/pcollections/2.1.2) 提供了对库的复杂性的更多洞察。

和往常一样，完整的代码可以在 Github 上找到[。](https://web.archive.org/web/20220630012233/https://github.com/eugenp/tutorials/tree/master/libraries-4)