# 用 Java 删除列表中的所有重复项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-duplicates-from-list>

## 1。简介

在这个快速教程中，我们将学习如何从列表中清除重复的元素。首先，我们将使用普通 Java，然后是 Guava，最后是一个基于 Java 8 Lambda 的解决方案。

本教程是 Baeldung 上的[**Java-回到基础**系列](/web/20220626202510/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 2。使用普通 Java 从列表中删除重复项

我们可以使用标准的 Java 集合框架**通过集合**轻松地从列表中删除重复的元素:

```java
public void 
  givenListContainsDuplicates_whenRemovingDuplicatesWithPlainJava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = new ArrayList<>(
      new HashSet<>(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInAnyOrder(5, 0, 3, 1, 2));
}
```

我们可以看到，原来的列表保持不变。

在上面的例子中，我们使用了`HashSet`实现，这是一个无序的集合。因此，**被清理的`listWithoutDuplicates`的顺序可能与原来的`listWithDuplicates`的顺序不同。**

如果我们需要保留顺序，我们可以使用`LinkedHashSet`来代替:

```java
public void 
  givenListContainsDuplicates_whenRemovingDuplicatesPreservingOrderWithPlainJava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = new ArrayList<>(
      new LinkedHashSet<>(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInRelativeOrder(5, 0, 3, 1, 2));
}
```

## 延伸阅读:

## [Java 集合面试问题](/web/20220626202510/https://www.baeldung.com/java-collections-interview-questions)

A set of practical Collections-related Java interview questions[Read more](/web/20220626202510/https://www.baeldung.com/java-collections-interview-questions) →

## [Java–合并多个集合](/web/20220626202510/https://www.baeldung.com/java-combine-multiple-collections)

A quick and practical guide to combining multiple collections in Java[Read more](/web/20220626202510/https://www.baeldung.com/java-combine-multiple-collections) →

## [如何用 Java 找到列表中的元素](/web/20220626202510/https://www.baeldung.com/find-list-element-java)

Have a look at some quick ways to find an element in a list in Java[Read more](/web/20220626202510/https://www.baeldung.com/find-list-element-java) →

## 3。使用番石榴从列表中删除重复项

我们也可以用番石榴做同样的事情:

```java
public void 
  givenListContainsDuplicates_whenRemovingDuplicatesWithGuava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates 
      = Lists.newArrayList(Sets.newHashSet(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInAnyOrder(5, 0, 3, 1, 2));
}
```

这里，原始列表也保持不变。

同样，元素在清理列表中的顺序可能是随机的。

如果我们使用`LinkedHashSet`实现，我们将保留初始顺序:

```java
public void 
  givenListContainsDuplicates_whenRemovingDuplicatesPreservingOrderWithGuava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates 
      = Lists.newArrayList(Sets.newLinkedHashSet(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInRelativeOrder(5, 0, 3, 1, 2));
}
```

## 4。使用 Java 8 Lambdas 从列表中删除重复项

最后，让我们看一个新的解决方案，在 Java 8 中使用 Lambdas。我们将**使用来自流 API 的`distinct()`方法，**根据`equals()`方法返回的结果返回一个由不同元素组成的流。

此外，**对于有序流，不同元素的选择是稳定的**。这意味着，对于重复的元素，将保留遇到顺序中第一个出现的元素:

```java
public void 
  givenListContainsDuplicates_whenRemovingDuplicatesWithJava8_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = listWithDuplicates.stream()
     .distinct()
     .collect(Collectors.toList());

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInAnyOrder(5, 0, 3, 1, 2));
}
```

现在我们有了三种快速清理 a `List.`中所有重复项目的方法

## 5.结论

在本文中，我们展示了使用普通 Java、Google Guava 和 Java 8 从列表中删除重复项是多么容易。

所有这些例子和片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220626202510/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。