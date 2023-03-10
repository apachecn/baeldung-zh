# Java 中数组和列表之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-array-to-list-and-list-to-array>

## 1。概述

在这个快速教程中，我们将学习如何使用核心 Java 库、Guava 和 Apache Commons 集合在数组和列表之间进行转换。

本文是 Baeldung 网站上的[“Java—回到基础”系列文章](/web/20220819110153/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [将一个基元数组转换成一个列表](/web/20220819110153/https://www.baeldung.com/java-primitive-array-to-list)

Learn how to convert an array of primitives to a List of objects of the corresponding type.[Read more](/web/20220819110153/https://www.baeldung.com/java-primitive-array-to-list) →

## [在 Java 中将集合转换为数组列表](/web/20220819110153/https://www.baeldung.com/java-convert-collection-arraylist)

A brief tutorial to building ArrayLists given a collection in Java.[Read more](/web/20220819110153/https://www.baeldung.com/java-convert-collection-arraylist) →

## [如何在 Java 中将列表转换成地图](/web/20220819110153/https://www.baeldung.com/java-list-to-map)

Learn about different ways of converting a List to a Map in Java, using core functionalities and some popular libraries[Read more](/web/20220819110153/https://www.baeldung.com/java-list-to-map) →

## 2。将`List`转换为数组

### 2.1。使用普通 Java

让我们从使用普通 Java 从`List`到数组**的转换开始:**

```java
@Test
public void givenUsingCoreJava_whenListConvertedToArray_thenCorrect() {
    List<Integer> sourceList = Arrays.asList(0, 1, 2, 3, 4, 5);
    Integer[] targetArray = sourceList.toArray(new Integer[0]);
}
```

请注意，我们使用该方法的首选方式是`toArray(new T[0])`对`toArray(new T[size])`。正如阿列克谢·希皮列夫在他的[博客文章](https://web.archive.org/web/20220819110153/https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)中所证明的，它似乎更快、更安全、更干净。

### 2.2。使用番石榴

现在让我们使用**番石榴 API** 进行相同的转换:

```java
@Test
public void givenUsingGuava_whenListConvertedToArray_thenCorrect() {
    List<Integer> sourceList = Lists.newArrayList(0, 1, 2, 3, 4, 5);
    int[] targetArray = Ints.toArray(sourceList);
}
```

## 3。将数组转换为`List`

### 3.1。使用普通 Java

让我们从将数组转换成`List`的普通 Java 解决方案开始:

```java
@Test
public void givenUsingCoreJava_whenArrayConvertedToList_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    List<Integer> targetList = Arrays.asList(sourceArray);
}
```

请注意，这是一个固定大小的列表，仍将由数组支持。如果我们想要一个标准的`ArrayList,`,我们可以简单地实例化一个:

```java
List<Integer> targetList = new ArrayList<Integer>(Arrays.asList(sourceArray));
```

### 3.2。使用番石榴

现在让我们使用**番石榴 API** 进行相同的转换:

```java
@Test
public void givenUsingGuava_whenArrayConvertedToList_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    List<Integer> targetList = Lists.newArrayList(sourceArray);
} 
```

### 3.3。使用公共集合

最后，让我们使用[Apache Commons Collections](https://web.archive.org/web/20220819110153/https://commons.apache.org/proper/commons-collections/javadocs/)`CollectionUtils.addAll` API 来填充空列表中的数组元素:

```java
@Test 
public void givenUsingCommonsCollections_whenArrayConvertedToList_thenCorrect() { 
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 }; 
    List<Integer> targetList = new ArrayList<>(6); 
    CollectionUtils.addAll(targetList, sourceArray); 
}
```

## 4。结论

所有这些例子和代码片段的实现可以在 GitHub 上找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。