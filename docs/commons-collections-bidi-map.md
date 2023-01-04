# Apache Commons 集合 BidiMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/commons-collections-bidi-map>

[This article is part of a series:](javascript:void(0);)[• Apache Commons Collections Bag](/web/20220701022749/https://www.baeldung.com/apache-commons-bag)
[• Apache Commons Collections SetUtils](/web/20220701022749/https://www.baeldung.com/apache-commons-setutils)
[• Apache Commons Collections OrderedMap](/web/20220701022749/https://www.baeldung.com/apache-commons-ordered-map)
• Apache Commons Collections BidiMap (current article)[• A Guide to Apache Commons Collections CollectionUtils](/web/20220701022749/https://www.baeldung.com/apache-commons-collection-utils)
[• Apache Commons Collections MapUtils](/web/20220701022749/https://www.baeldung.com/apache-commons-map-utils)
[• Guide to Apache Commons CircularFifoQueue](/web/20220701022749/https://www.baeldung.com/commons-circular-fifo-queue)

## 1。概述

在这篇短文中，我们将看到 Apache Commons Collections 库中一个有趣的数据结构——`BidiMap`。

`BidiMap`增加了在标准`Map`接口上使用相应值查找密钥的可能性。

## 2。依赖性

我们需要在项目中包含以下依赖项，以便使用`BidiMap`及其实现。对于基于 Maven 的项目，我们必须向我们的`pom.xml`添加以下依赖项:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

对于基于 Gradle 的项目，我们必须将相同的工件添加到我们的`build.gradle`文件中:

```
compile 'org.apache.commons:commons-collections4:4.1'
```

这个依赖的最新版本可以在 Maven Central 上找到[。](https://web.archive.org/web/20220701022749/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22)

## 3。实施和实例化

它本身只是一个定义双向地图特有行为的接口——当然还有多种实现方式。

理解`BidiMap`的实现**不允许键和值重复**是很重要的。当一个`BidiMap`被反转时，任何重复的值都将被转换成重复的键，并将违反映射契约。一个映射必须总是有唯一的键。

让我们看看这个接口的不同具体实现:

*   `DualHashBidiMap`:这个实现使用两个`HashMap`实例在内部实现双向映射`.` 它使用条目的键或值提供快速的条目查找。然而，必须维护`HashMap`的两个实例
*   `DualLinkedHashBidiMap:`这个实现使用了两个`LinkedHashMap` 实例，因此保持了映射条目的插入顺序。如果我们不需要维护地图条目的插入顺序，我们可以使用更便宜的`DualHashBidiMap`
*   这个实现是高效的，并且是通过红黑树实现的。使用键和值的自然排序，保证了`TreeBidiMap` 的键和值按升序排序
*   还有一个`DualTreeBidiMap`使用了两个`TreeMap`的实例来实现和`TreeBidiMap`一样的东西。`DualTreeBidiMap` 明显比`TreeBidiMap`贵

`BidiMap`接口扩展了`java.util.Map` 接口，因此可以直接替代它。我们可以使用具体实现的无参数构造函数来实例化一个具体的对象实例`.`

## 4。独特的`BidiMap`方法

既然我们已经探索了不同的实现，那么让我们来看看接口特有的方法。

****`put()`****将新的键值条目插入映射**。请注意，如果新条目的值与任何现有条目的值匹配，则现有条目将被删除，以支持新条目。**

 **该方法返回删除的旧条目，如果没有旧条目，则返回`null`:

```
BidiMap<String, String> map = new DualHashBidiMap<>();
map.put("key1", "value1");
map.put("key2", "value2");
assertEquals(map.size(), 2);
```

**`inverseBidiMap()`反转一个**的键值对`**BidiMap.**`这个方法返回一个新的`BidiMap`，其中键变成了值，反之亦然。该操作在翻译和词典应用中非常有用:

```
BidiMap<String, String> rMap = map.inverseBidiMap();
assertTrue(rMap.containsKey("value1") && rMap.containsKey("value2"));
```

**`removeValue()`用于通过指定一个值来删除一个地图条目，而不是一个键**。这是对`java.util`包中的`Map`实现的补充:

```
map.removeValue("value2");
assertFalse(map.containsKey("key2"));
```

**我们可以使用`getKey().`** 将键映射到`BidiMap`中的特定值。如果没有键映射到指定值，该方法返回`null`:

```
assertEquals(map.getKey("value1"), "key1");
```

## 5。结论

这篇快速教程介绍了 Apache Commons 集合库——特别是在`BidiMap`,它的实现和特殊方法。

`BidiMap`最激动人心和与众不同的特性是它能够通过键和值来查找和操作条目。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220701022749/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-collections)

Next **»**[A Guide to Apache Commons Collections CollectionUtils](/web/20220701022749/https://www.baeldung.com/apache-commons-collection-utils)**«** Previous[Apache Commons Collections OrderedMap](/web/20220701022749/https://www.baeldung.com/apache-commons-ordered-map)**