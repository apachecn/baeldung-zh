# Java 中的不可变数组列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-immutable-list>

## 1。概述

这个快速教程将展示如何用核心 JDK、番石榴和最后的 Apache Commons Collections 4 制作一个不可变的 T2。

这篇文章是 Baeldung 网站上的“**Java——回到基础的**”系列文章的一部分。

## 延伸阅读:

## [将 Java 流收集到一个不可变的集合中](/web/20221018104516/https://www.baeldung.com/java-stream-immutable-collection)

Learn how to collect Java Streams to immutable Collections.[Read more](/web/20221018104516/https://www.baeldung.com/java-stream-immutable-collection) →

## [不变量介绍](/web/20221018104516/https://www.baeldung.com/immutables)

A quick and practical intro to the Immutables library - used to generate immutable objects via the use of annotations.[Read more](/web/20221018104516/https://www.baeldung.com/immutables) →

## [Java——从列表中随机获取项目/元素](/web/20221018104516/https://www.baeldung.com/java-random-list-element)

A quick and practical guide to picking a random item/items from a List in Java.[Read more](/web/20221018104516/https://www.baeldung.com/java-random-list-element) →

## 2。与 JDK

首先，JDK 提供了一种从现有集合中获取不可修改集合的好方法:

```java
Collections.unmodifiableList(list);
```

此时，新集合应该不再是可修改的:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenUsingTheJdk_whenUnmodifiableListIsCreated_thenNotModifiable() {
    List<String> list = new ArrayList<>(Arrays.asList("one", "two", "three"));
    List<String> unmodifiableList = Collections.unmodifiableList(list);
    unmodifiableList.add("four");
}
```

### 2.1.使用 Java 9

从 Java 9 开始，我们可以使用一个`List<E>.of​(E… elements)`静态工厂方法来创建一个不可变列表:

```java
@Test(expected = UnsupportedOperationException.class)
public final void givenUsingTheJava9_whenUnmodifiableListIsCreated_thenNotModifiable() {
    final List<String> list = new ArrayList<>(Arrays.asList("one", "two", "three"));
    final List<String> unmodifiableList = List.of(list.toArray(new String[]{}));
    unmodifiableList.add("four");
}
```

注意我们必须如何将现有的`list`转换成一个数组。这是因为`List.of(elements)`接受 vararg 参数。

## 3。有番石榴

Guava 为创建自己的`ImmutableList`版本提供了类似的功能:

```java
ImmutableList.copyOf(list);
```

类似地，结果列表不可修改:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenUsingGuava_whenUnmodifiableListIsCreated_thenNotModifiable() {
    List<String> list = new ArrayList<>(Arrays.asList("one", "two", "three"));
    List<String> unmodifiableList = ImmutableList.copyOf(list);
    unmodifiableList.add("four");
}
```

注意，这个操作实际上将**创建一个原始列表**的副本，而不仅仅是一个视图。

Guava 还提供了一个构建器——它将返回强类型的`ImmutableList`,而不是简单的`List`:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenUsingGuavaBuilder_whenUnmodifiableListIsCreated_thenNoLongerModifiable() {
    List<String> list = new ArrayList<>(Arrays.asList("one", "two", "three"));
    ImmutableList<String> unmodifiableList = ImmutableList.<String>builder().addAll(list).build();
    unmodifiableList.add("four");
}
```

## 4。与 Apache Collections Commons

最后，Commons Collection 还提供了一个 API 来创建不可修改的列表:

```java
ListUtils.unmodifiableList(list);
```

同样，修改结果列表应该会导致一个`UnsupportedOperationException`:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenUsingCommonsCollections_whenUnmodifiableListIsCreated_thenNotModifiable() {
    List<String> list = new ArrayList<>(Arrays.asList("one", "two", "three"));
    List<String> unmodifiableList = ListUtils.unmodifiableList(list);
    unmodifiableList.add("four");
}
```

## 5。结论

本教程演示了如何使用核心的 JDK、谷歌番石榴或阿帕奇公共收藏，从现有的`ArrayList` 中轻松地**创建一个不可修改的列表。**

所有这些例子和代码片段**的实现可以在 [Github](https://web.archive.org/web/20221018104516/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9 "Github Project exemplifying how to create the immutable list")** 上找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。