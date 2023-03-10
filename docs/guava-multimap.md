# 番石榴多地图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-multimap>

## 1。概述

在本文中，我们将看看 Google Guava 库中的`Map`实现之一—`Multimap`。它是将键映射到值的集合，类似于`java.util.Map`，但是其中每个键可能与多个值相关联。

## 2。Maven 依赖关系

首先，让我们添加一个依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220629003946/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)

## 3。 **`Multimap` 实现**

在 Guava `Multimap,` 的情况下，如果我们为同一个键添加两个值，第二个值不会覆盖第一个值。相反，我们将在结果`map`中有两个值。让我们看一个测试案例:

```java
String key = "a-key";
Multimap<String, String> map = ArrayListMultimap.create();

map.put(key, "firstValue");
map.put(key, "secondValue");

assertEquals(2, map.size()); 
```

打印`map`的内容将输出:

```java
{a-key=[firstValue, secondValue]}
```

当我们通过键“a-key”获取值时，我们将得到包含“firstValue”和“secondValue”的`Collection<String>` ,结果是:

```java
Collection<String> values = map.get(key);
```

打印值将输出:

```java
[firstValue, secondValue]
```

## 4。`Map`对比标准

来自`java.util`包的标准映射没有给我们为同一个键分配多个值的能力。让我们考虑一个简单的例子，当我们使用同一个键将两个值`put()` 转换成一个`Map` 时:

```java
String key = "a-key";
Map<String, String> map = new LinkedHashMap<>();

map.put(key, "firstValue");
map.put(key, "secondValue");

assertEquals(1, map.size()); 
```

结果`map`只有一个元素(`“secondValue”),`，因为第二个`put()` 操作覆盖了第一个值。如果我们想要实现与 Guava 的`Multimap` `,` 相同的行为，我们需要创建一个`Map`，它有一个`List<String>` 作为值类型:

```java
String key = "a-key";
Map<String, List<String>> map = new LinkedHashMap<>();

List<String> values = map.get(key);
if(values == null) {
    values = new LinkedList<>();
    values.add("firstValue");
    values.add("secondValue");
 }

map.put(key, values);

assertEquals(1, map.size());
```

显然用起来不是很方便。如果我们的代码中有这样的需求，那么 Guava 的`Multimap` 可能是比`java.util.Map.`更好的选择

这里要注意的一件事是，尽管我们有一个包含两个元素的列表，`size()`方法返回 1。In `Multimap, size()`返回存储在`Map,`中的值的实际数量，而`keySet().size()`返回不同键的数量。

## 5。`Multimap`赞成

多地图通常用在原本会出现`Map<K, Collection<V>>`的地方。不同之处包括:

*   在添加带有`put()`的条目之前，无需填充空集合
*   `The get() method`从不返回`null`，只返回一个空集合(我们不需要像在`Map<String, Collection<V>>`测试用例中那样检查`null`)
*   当且仅当一个键映射到至少一个值时，它才包含在`Multimap`中。任何导致一个键没有关联值的操作，都会从`Multimap`中删除那个键(在`Map<String, Collection<V>>,` 中，即使我们从集合中删除所有值，我们仍然保留一个空的`Collection`作为值，这是不必要的内存开销)
*   总输入值计数可用`size()`

## 6。结论

这篇文章展示了如何以及何时使用番石榴`Multimap.`它与标准的`java.util.Map`进行了比较，并展示了番石榴`Multimap.`的优点

所有这些例子和代码片段都可以在 [GitHub 项目](https://web.archive.org/web/20220629003946/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-map)中找到——这是一个 Maven 项目，所以它应该很容易导入和运行。