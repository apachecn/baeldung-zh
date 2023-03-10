# Apache Commons 集合设置工具

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-setutils>

[This article is part of a series:](javascript:void(0);)[• Apache Commons Collections Bag](/web/20221208143854/https://www.baeldung.com/apache-commons-bag)
• Apache Commons Collections SetUtils (current article)[• Apache Commons Collections OrderedMap](/web/20221208143854/https://www.baeldung.com/apache-commons-ordered-map)
[• Apache Commons Collections BidiMap](/web/20221208143854/https://www.baeldung.com/commons-collections-bidi-map)
[• A Guide to Apache Commons Collections CollectionUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-collection-utils)
[• Apache Commons Collections MapUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-map-utils)
[• Guide to Apache Commons CircularFifoQueue](/web/20221208143854/https://www.baeldung.com/commons-circular-fifo-queue)

## 1。概述

在本文中，我们将探索 Apache Commons Collections 库的`SetUtils` API。简单地说，这些实用程序可以用来在 Java 中对`Set`数据结构执行某些操作。

## 2。依赖安装

为了在我们的项目中使用`SetUtils`库，我们需要向项目的`pom.xml`文件添加以下依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

或者，如果我们的项目是基于 Gradle 的，我们应该将依赖关系添加到项目的`build.gradle`文件中。此外，我们需要将`mavenCentral()`添加到`build.gradle`文件的存储库部分:

```java
compile 'org.apache.commons:commons-collections4:4.1'
```

## 3。谓词集

`SetUtils`库的`predicatedSet()`方法允许定义插入到集合中的所有元素应该满足的条件。它接受一个源`Set`对象和一个谓词。

我们可以用它来轻松地验证一个`Set`的所有元素都满足某个条件，这在开发第三方库/API 时会很方便。

如果任何元素的验证失败，将抛出一个`IllegalArgumentException`。**下面的代码片段防止将不是以‘L’**开头的 **字符串添加到`sourceSet`或返回的`validatingSet`中:**

```java
Set<String> validatingSet
  = SetUtils.predicatedSet(sourceSet, s -> s.startsWith("L"));
```

该库还有分别与`SortedSet`和`NavigableSet`一起工作的`predicatedSortedSet()`和`predicatedNavigableSet()`。

## 4。集合的并、差和交

该库提供了计算并集、差集和`Set`元素交集的方法。

`difference()` 方法接受两个 `Set`对象并返回一个不可变的`SetUtils.` `SetView`对象。返回的`SetUtils.`T5 包含集合`a`中的元素，但不包含集合`b`中的元素:

```java
Set<Integer> a = new HashSet<>(Arrays.asList(1, 2, 5));
Set<Integer> b = new HashSet<>(Arrays.asList(1, 2));
SetUtils.SetView<Integer> result = SetUtils.difference(a, b);

assertTrue(result.size() == 1 && result.contains(5));
```

注意，**试图对返回的`SetUtils.` `SetView` 执行写操作，如`add()`或`addAll()`，会抛出一个`UnsupportedOperationException`。**

要修改返回的结果，我们需要调用返回的`SetUtils.` `SetView` 的`toSet()`方法，获得一个可写的`Set`对象:

```java
Set<Integer> mutableSet = result.toSet();
```

`SetUtils`库的`union`方法确实如其名——它返回集合`a`和`b`的所有元素。`union`方法也返回一个不可变的`SetUtil.SetView`对象:

```java
Set<Integer> expected = new HashSet<>(Arrays.asList(1, 2, 5));
SetUtils.SetView<Integer> union = SetUtils.union(a, b);

assertTrue(SetUtils.isEqualSet(expected, union));
```

**注意 assert 语句中使用的`isEqualSet()`方法**。它是`SetUtils` 库的一个方便的静态方法，可以有效地检查两个集合是否相等。

为了得到一个集合的交集，即同时出现在集合`a`和集合`b`中的元素，我们将使用`SetUtils.` `intersection()`方法。这个方法也返回一个`SetUtil.SetView`对象:

```java
Set<Integer> expected = new HashSet<>(Arrays.asList(1, 2));
SetUtils.SetView<Integer> intersect = SetUtils.intersection(a, b);

assertTrue(SetUtils.isEqualSet(expected, intersect));
```

## 5。变换集合元素

再来看看另一个令人兴奋的方法——`SetUtils.``transformedSet()`。这个方法接受一个`Set`对象和一个`Transformer`接口。在源集合的支持下，它使用`Transformer`接口的`transform()` 方法来转换集合中的每个元素。

转换逻辑在`Transformer`接口的`transform()`方法中定义，应用于添加到集合中的每个元素。下面的代码片段将添加到集合中的每个元素乘以 2:

```java
Set<Integer> a = SetUtils.transformedSet(new HashSet<>(), e -> e * 2  );
a.add(2);

assertEquals(a.toArray()[0], 4);
```

`transformedSet()`方法非常方便——它们甚至可以用来对集合中的元素进行造型——比如从字符串到整数。只需确保输出的类型是输入的子类型。

假设我们正在使用`SortedSet` 或`NavigableSet`而不是`HashSet,`，我们可以分别使用`transformedSortedSet()`或`transformedNavigableSet()`。

注意，一个新的`HashSet`实例被传递给了`transformedSet()`方法。在将一个现有的非空的`Set`传递给方法的情况下，先前存在的元素将不会被转换。

如果我们想要转换预先存在的元素(以及后来添加的元素)，我们需要使用`org.apache.commons.collections4.set.TransformedSet`的`transformedSet()`方法:

```java
Set<Integer> source = new HashSet<>(Arrays.asList(1));
Set<Integer> newSet = TransformedSet.transformedSet(source, e -> e * 2);

assertEquals(newSet.toArray()[0], 2);
assertEquals(source.toArray()[0], 2);
```

注意，来自源集合的元素被转换，结果被复制到返回的`newSet.`

## 6.集合析取

`SetUtils`库提供了一个静态方法，可以用来寻找集合析取。集合`a`和集合`b`的析取都是集合 a 和集合 b 唯一的元素

让我们看看如何使用`SetUtils`库的`disjunction()`方法:

```java
Set<Integer> a = new HashSet<>(Arrays.asList(1, 2, 5));
Set<Integer> b = new HashSet<>(Arrays.asList(1, 2, 3));
SetUtils.SetView<Integer> result = SetUtils.disjunction(a, b);

assertTrue(
  result.toSet().contains(5) && result.toSet().contains(3));
```

## 7。`SetUtils`库中的其他方法

在`SetUtils`库中还有其他方法可以使集合数据的处理变得轻而易举:

*   我们可以使用`synchronizedSet()`或`synchronizedSortedSet()`来获得一个线程安全的`Set`。然而，如文档中所述，我们**必须手动同步**返回集合的迭代器，以避免不确定的行为
*   我们可以使用`SetUtils.unmodifiableSet()` 来获取一个只读集合。注意，向返回的`Set` 对象添加元素的尝试将抛出一个`UnsupportedOperationException`
*   还有一个返回类型安全、不可变空集的`SetUtils.emptySet()`方法
*   `SetUtils.emptyIfNull()` 方法接受一个可空的`Set`对象。如果提供的`Set` 为空，它返回一个空的、只读的 S`et`；否则，它返回提供的`Set`
*   *SetUtils.orderedSet()* 将返回一个保持元素添加顺序的`Set`对象
*   可以为一个集合生成一个 hashcode——通过这种方式，两组相同的元素将拥有相同的 hashcode
*   `SetUtils.newIdentityHashSet()` 将返回一个`HashSet` ，它使用`==`而不是`equals()`方法来匹配一个元素。请在这里阅读它的警告

## 8.结论

在本文中，我们探索了`SetUtils`库的本质。utility 类提供了静态方法，使得处理一组数据结构变得简单而令人兴奋。它还能提高生产率。

和往常一样，代码片段可以在 GitHub 网站[上找到。API 的官方文档可以在](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-collections)找到[。](https://web.archive.org/web/20221208143854/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/SetUtils.html)

Next **»**[Apache Commons Collections OrderedMap](/web/20221208143854/https://www.baeldung.com/apache-commons-ordered-map)**«** Previous[Apache Commons Collections Bag](/web/20221208143854/https://www.baeldung.com/apache-commons-bag)