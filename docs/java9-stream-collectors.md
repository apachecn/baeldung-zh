# Java 9 中的新流收集器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java9-stream-collectors>

## 1。概述

Java 8 中增加了 [`Collectors`](https://web.archive.org/web/20221115122557/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html) ，这有助于将输入元素累积到可变容器中，如`Map`、`List`和`Set`。

在本文中，我们将探讨 Java 9 中添加的两个新收集器**:与`Collectors.groupingBy`结合使用的`Collectors.filtering`和** `**Collectors.flatMapping**` 提供了元素的智能集合。

## 2。`Filtering Collector`

`Collectors.filtering` 类似于`Stream filter()`；它用于`filtering`输入元素，但用于不同的场景。`Stream's` `filter`用于流链，而`filtering`是一个`Collector`，它被设计成与`groupingBy`一起使用。

使用`Stream's` `filter`，首先过滤数值，然后进行分组。这样，被过滤掉的值就消失了，没有任何痕迹。如果我们需要一个轨迹，那么我们需要先分组，然后应用过滤，这实际上是`Collectors.filtering`所做的。

`Collectors.filtering`采用过滤输入元素的函数和收集过滤元素的收集器:

```java
@Test
public void givenList_whenSatifyPredicate_thenMapValueWithOccurences() {
    List<Integer> numbers = List.of(1, 2, 3, 5, 5);

    Map<Integer, Long> result = numbers.stream()
      .filter(val -> val > 3)
      .collect(Collectors.groupingBy(i -> i, Collectors.counting()));

    assertEquals(1, result.size());

    result = numbers.stream()
      .collect(Collectors.groupingBy(i -> i,
        Collectors.filtering(val -> val > 3, Collectors.counting())));

    assertEquals(4, result.size());
}
```

## 3。`FlatMapping Collector`

`Collectors.flatMapping`与`Collectors.mapping`相似，但有一个更精细的目标。两个收集器都接受一个函数和一个收集元素的收集器，但是`flatMapping`函数接受一个`Stream`元素，然后由收集器累积。

让我们看看下面的模型类:

```java
class Blog {
    private String authorName;
    private List<String> comments;

    // constructor and getters
} 
```

`Collectors.flatMapping`让我们跳过中间收集，直接写入映射到由`Collectors.groupingBy`定义的组的单个容器:

```java
@Test
public void givenListOfBlogs_whenAuthorName_thenMapAuthorWithComments() {
    Blog blog1 = new Blog("1", "Nice", "Very Nice");
    Blog blog2 = new Blog("2", "Disappointing", "Ok", "Could be better");
    List<Blog> blogs = List.of(blog1, blog2);

    Map<String,  List<List<String>>> authorComments1 = blogs.stream()
     .collect(Collectors.groupingBy(Blog::getAuthorName, 
       Collectors.mapping(Blog::getComments, Collectors.toList())));

    assertEquals(2, authorComments1.size());
    assertEquals(2, authorComments1.get("1").get(0).size());
    assertEquals(3, authorComments1.get("2").get(0).size());

    Map<String, List<String>> authorComments2 = blogs.stream()
      .collect(Collectors.groupingBy(Blog::getAuthorName, 
        Collectors.flatMapping(blog -> blog.getComments().stream(), 
        Collectors.toList())));

    assertEquals(2, authorComments2.size());
    assertEquals(2, authorComments2.get("1").size());
    assertEquals(3, authorComments2.get("2").size());
}
```

`Collectors.mapping`将所有分组的作者评论映射到收集器的容器，即`List`，而这个中间集合被`flatMapping`移除，因为它给出了要映射到收集器容器的评论列表的直接流。

## 4。结论

本文说明了`Java9`中引入的新`Collectors`的用法，即 **`Collectors.filtering()`和`Collectors.flatMapping()`与`Collectors.groupingBy()`** 结合使用。

这些收集器也可以和`Collectors.partitioningBy()` 一起使用，但是它只根据条件创建了两个分区，并且没有利用收集器的真正能力；因此不在本教程中讨论。

本教程中代码片段的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221115122557/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-improvements)