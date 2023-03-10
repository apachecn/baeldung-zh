# 在 Java 中将映射转换为数组、列表或集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-map-values-to-array-list-set>

## 1。概述

这篇短文将展示如何使用普通 Java 和一个基于快速[番石榴](https://web.archive.org/web/20221126213349/https://code.google.com/p/guava-libraries/ "The Google Guava Library")的例子将`Map`的值**转换为`Array,``List`或`Set`** 。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 延伸阅读:

## [在 Java 中迭代地图](/web/20221126213349/https://www.baeldung.com/java-iterate-map)

Learn different ways of iterating through the entries of a Map in Java.[Read more](/web/20221126213349/https://www.baeldung.com/java-iterate-map) →

## [map()和 flatMap()的区别](/web/20221126213349/https://www.baeldung.com/java-difference-map-and-flatmap)

Learn about the differences between map() and flatMap() by analyzing some examples of Streams and Optionals.[Read more](/web/20221126213349/https://www.baeldung.com/java-difference-map-and-flatmap) →

## [如何在 Java 中存储一个 Map 中的重复键？](/web/20221126213349/https://www.baeldung.com/java-map-duplicate-keys)

A quick and practical guide to handling duplicate keys by using multimaps in Java.[Read more](/web/20221126213349/https://www.baeldung.com/java-map-duplicate-keys) →

## 2。将值映射到数组

首先，让我们看看如何使用普通 java 将 Map 的值转换成一个数组**:**

```java
@Test
public void givenUsingCoreJava_whenMapValuesConvertedToArray_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    Collection<String> values = sourceMap.values();
    String[] targetArray = values.toArray(new String[0]);
}
```

注意，`toArray(new T[0])`是比`toArray(new T[size])`更好的方法。正如阿列克谢·希皮列夫在他的[博客文章](https://web.archive.org/web/20221126213349/https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)中所证明的，它似乎更快、更安全、更干净。

## 3。将值映射到列表

接下来，让我们使用普通 Java 将映射的值转换为列表:

```java
@Test
public void givenUsingCoreJava_whenMapValuesConvertedToList_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    List<String> targetList = new ArrayList<>(sourceMap.values());
}
```

用番石榴:

```java
@Test
public void givenUsingGuava_whenMapValuesConvertedToList_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    List<String> targetList = Lists.newArrayList(sourceMap.values());
}
```

## 4。要设置的映射值

最后，让我们使用普通 java 将 Map 的值转换成一个集合:

```java
@Test
public void givenUsingCoreJava_whenMapValuesConvertedToS_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    Set<String> targetSet = new HashSet<>(sourceMap.values());
}
```

## 5。结论

如您所见，所有转换都可以通过一行代码完成，只需使用 Java 标准集合库。

所有这些例子和代码片段**的实现可以在 [GitHub 项目](https://web.archive.org/web/20221126213349/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions "Conversion examples over on github")** 中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。