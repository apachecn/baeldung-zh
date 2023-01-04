# 用 Java 对列表进行分区

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-split>

## 1。概述

在本文中，我们将展示如何将一个列表分割成几个给定大小的子列表。

对于一个相对简单的操作，标准 Java 集合 API 中没有支持，这令人惊讶。幸运的是，[番石榴](https://web.archive.org/web/20220929201248/https://github.com/google/guava "The Guava Library")和[阿帕奇公共收藏](https://web.archive.org/web/20220929201248/https://commons.apache.org/proper/commons-collections/ "The Apache Commons Collections library")都以类似的方式实现了这个操作。

这篇文章是 Baeldung 网站上的“**Java——回到基础的**”系列文章的一部分。

## 延伸阅读:

## [在 Java 中将列表转换成字符串](/web/20220929201248/https://www.baeldung.com/java-list-to-string)

Learn how to convert a List to a String using different techniques.[Read more](/web/20220929201248/https://www.baeldung.com/java-list-to-string) →

## [在 Java 中混洗收藏](/web/20220929201248/https://www.baeldung.com/java-shuffle-collection)

Learn how to shuffle various collections in Java.[Read more](/web/20220929201248/https://www.baeldung.com/java-shuffle-collection) →

## [Java 中的 Spliterator 简介](/web/20220929201248/https://www.baeldung.com/java-spliterator)

Learn about the Spliterator interface that can be used for traversing and partitioning sequences.[Read more](/web/20220929201248/https://www.baeldung.com/java-spliterator) →

## 2。用番石榴分割列表

Guava 通过**`Lists.partition`操作**将列表划分为指定大小的子列表:

```java
@Test
public void givenList_whenParitioningIntoNSublists_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = Lists.partition(intList, 3);

    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

## 3。用番石榴分割收藏

**番石榴也可用于分割系列**:

```java
@Test
public void givenCollection_whenParitioningIntoNSublists_thenCorrect() {
    Collection<Integer> intCollection = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);

    Iterable<List<Integer>> subSets = Iterables.partition(intCollection, 3);

    List<Integer> firstPartition = subSets.iterator().next();
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(1, 2, 3);
    assertThat(firstPartition, equalTo(expectedLastPartition));
}
```

请记住，分区是原始集合的**子列表视图，**这意味着原始集合中的更改将反映在分区中:

```java
@Test
public void givenListPartitioned_whenOriginalListIsModified_thenPartitionsChangeAsWell() {
    // Given
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = Lists.partition(intList, 3);

    // When
    intList.add(9);

    // Then
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8, 9);
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

## 4。使用 Apache Commons 集合对列表进行分区

Apache Commons Collections 的最新版本最近也增加了对列表分区的支持:

```java
@Test
public void givenList_whenParitioningIntoNSublists_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = ListUtils.partition(intList, 3);

    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

与 Guava Iterables.partition 类似，Commons Collections 没有相应的选项来对原始集合进行分区。

最后，同样的警告也适用于这里:产生的分区是原始列表的视图。

## 5。使用 Java8 对列表进行分区

现在让我们看看如何使用 Java8 对我们的列表进行分区。

### 5.1。`partitioningBy`吏

我们可以使用`Collectors.partitioningBy()`将列表分成两个子列表:

```java
@Test
public void givenList_whenParitioningIntoSublistsUsingPartitionBy_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);

    Map<Boolean, List<Integer>> groups = 
      intList.stream().collect(Collectors.partitioningBy(s -> s > 6));
    List<List<Integer>> subSets = new ArrayList<List<Integer>>(groups.values());

    List<Integer> lastPartition = subSets.get(1);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(2));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

注意:产生的分区不是主列表的视图，所以主列表发生的任何变化都不会影响分区。

### 5.2。`groupingBy`吏

我们也可以使用 `Collectors.groupingBy()`将我们的列表分成多个分区:

```java
@Test
public final void givenList_whenParitioningIntoNSublistsUsingGroupingBy_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);

    Map<Integer, List<Integer>> groups = 
      intList.stream().collect(Collectors.groupingBy(s -> (s - 1) / 3));
    List<List<Integer>> subSets = new ArrayList<List<Integer>>(groups.values());

    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

注意:就像`Collectors.partitioningBy(),`一样，主列表中的变化不会影响最终的分区。

### 5.3。用分隔符分割列表

我们还可以使用 Java8 通过分隔符来分割我们的列表:

```java
@Test
public void givenList_whenSplittingBySeparator_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 0, 4, 5, 6, 0, 7, 8);

    int[] indexes = 
      Stream.of(IntStream.of(-1), IntStream.range(0, intList.size())
      .filter(i -> intList.get(i) == 0), IntStream.of(intList.size()))
      .flatMapToInt(s -> s).toArray();
    List<List<Integer>> subSets = 
      IntStream.range(0, indexes.length - 1)
               .mapToObj(i -> intList.subList(indexes[i] + 1, indexes[i + 1]))
               .collect(Collectors.toList());

    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
```

注意:我们使用“0”作为分隔符。我们首先获得列表中所有“0”元素的索引，然后在这些索引上拆分`List`。

## 6。结论

这里介绍的解决方案利用了额外的库，即 Guava 和 Apache Commons 集合。总的来说，这两者都是非常轻量级和非常有用的，所以在类路径中包含其中一个非常有意义。然而，如果这不是一个选项，这里显示的是一个只有 Java 的解决方案。

所有这些例子和代码片段的实现可以在 GitHub 上找到。 这是一个基于 Maven 的项目，所以应该很容易导入和运行。