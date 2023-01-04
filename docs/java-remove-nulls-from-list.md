# 用 Java 删除列表中的所有空值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-nulls-from-list>

这个快速教程将展示如何使用普通 Java、Guava、Apache Commons 集合和更新的 Java 8 lambda 支持从一个`List` `,`中移除所有的`null`元素。

这篇文章是 Baeldung 网站上的“**Java——回到基础的**”系列文章的一部分。

## 1。使用普通 Java 从`List`中移除空值

Java Collections 框架提供了一个简单的解决方案来删除`List`中的所有空元素，这是一个基本的`while` 循环:

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithPlainJava_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    while (list.remove(null));

    assertThat(list, hasSize(1));
}
```

或者，我们也可以使用下面的简单方法:

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithPlainJavaAlternative_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    list.removeAll(Collections.singleton(null));

    assertThat(list, hasSize(1));
}
```

请注意，这两种解决方案都会修改源列表。

## 2。使用谷歌番石榴移除空值

我们还可以使用 Guava 和一种更具功能性的方法，通过谓词来移除空值:

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithGuavaV1_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    Iterables.removeIf(list, Predicates.isNull());

    assertThat(list, hasSize(1));
}
```

或者，**如果我们不想修改源列表**，Guava 将允许我们创建一个新的过滤列表:

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithGuavaV2_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null, 2, 3);
    List<Integer> listWithoutNulls = Lists.newArrayList(
      Iterables.filter(list, Predicates.notNull()));

    assertThat(listWithoutNulls, hasSize(3));
}
```

## 3。使用 Apache Commons 集合删除空值`List`

现在让我们看一个使用 Apache Commons 集合库的简单解决方案，它使用了类似的函数风格:

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithCommonsCollections_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    CollectionUtils.filter(list, PredicateUtils.notNullPredicate());

    assertThat(list, hasSize(3));
}
```

请注意，这个解决方案还将**修改原始列表**。

## 4。使用 Lambdas (Java 8) 从一个`List`中删除空值

最后——让我们看看使用 Lambdas 过滤列表的 Java 8 解决方案**；过滤过程可以并行或串行完成:**

```java
@Test
public void givenListContainsNulls_whenFilteringParallel_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    List<Integer> listWithoutNulls = list.parallelStream()
      .filter(Objects::nonNull)
      .collect(Collectors.toList());
}

@Test
public void givenListContainsNulls_whenFilteringSerial_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    List<Integer> listWithoutNulls = list.stream()
      .filter(Objects::nonNull)
      .collect(Collectors.toList());
}

public void givenListContainsNulls_whenRemovingNullsWithRemoveIf_thenCorrect() {
    List<Integer> listWithoutNulls = Lists.newArrayList(null, 1, 2, null, 3, null);
    listWithoutNulls.removeIf(Objects::isNull);

    assertThat(listWithoutNulls, hasSize(3));
}
```

就是这样——一些快速且非常有用的解决方案，用于从列表中删除所有空元素。

## 5.结论

在本文中，我们能够探索使用 Java、Guava 或 Lambdas 从`List`中删除空值的不同方法。

所有这些例子和片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220830004449/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。