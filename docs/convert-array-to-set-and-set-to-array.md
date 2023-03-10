# Java 中数组和集合之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-array-to-set-and-set-to-array>

## 1。概述

在这篇短文中，我们将看看**在`array`和`Set`** 之间的转换——首先使用普通 java，然后是 Guava 和 Apache 的 Commons Collections 库。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 2。将`Array`转换为`Set`

### 2.1。使用普通 Java

让我们先看看如何使用普通 Java 将数组**转换为`Set`:**

```java
@Test
public void givenUsingCoreJavaV1_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = new HashSet<Integer>(Arrays.asList(sourceArray));
}
```

或者，可以先创建`Set`,然后用数组元素填充:

```java
@Test
public void givenUsingCoreJavaV2_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = new HashSet<Integer>();
    Collections.addAll(targetSet, sourceArray);
}
```

### 2.2。使用谷歌番石榴

接下来，让我们看看**从数组到集合**的番石榴转换:

```java
@Test
public void givenUsingGuava_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = Sets.newHashSet(sourceArray);
}
```

### 2.3。使用 Apache Commons 集合

最后，让我们使用 Apache 的 Commons 集合库进行转换:

```java
@Test
public void givenUsingCommonsCollections_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = new HashSet<>(6);
    CollectionUtils.addAll(targetSet, sourceArray);
}
```

## 3。将集合转换为数组

### 3.1。使用普通 Java

现在让我们来看看相反的情况—**将现有的集合转换成数组**:

```java
@Test
public void givenUsingCoreJava_whenSetConvertedToArray_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    Integer[] targetArray = sourceSet.toArray(new Integer[0]);
}
```

注意，`toArray(new T[0])`是比`toArray(new T[size])`更好的方法。正如阿列克谢·希皮列夫在他的[博客文章](https://web.archive.org/web/20221208143956/https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)中所证明的，它似乎更快、更安全、更干净。

### 3.2。使用番石榴

接下来——番石榴溶液:

```java
@Test
public void givenUsingGuava_whenSetConvertedToArray_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    int[] targetArray = Ints.toArray(sourceSet);
}
```

注意，我们使用的是来自 Guava 的`Ints` API，所以这个解决方案特定于我们正在处理的数据类型。

## 4。结论

所有这些例子和代码片段**的实现可以在 [Github](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions "Conversion examples over on github")** 上找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。