# Java 中列表和集合之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-list-to-set-and-set-to-list>

## 1。概述

在这个快速教程中，我们将看看**从普通 Java 开始，使用 Guava 和 [Apache Commons Collections](https://web.archive.org/web/20221211183031/https://commons.apache.org/proper/commons-collections/ "The Apache Common Collections lib") 库，最后使用 Java 10 在`List`和`Set,`** 之间的转换。

本文是 Baeldung 网站上的[“Java—回到基础”系列文章](/web/20221211183031/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [如何用 Java 找到列表中的元素](/web/20221211183031/https://www.baeldung.com/find-list-element-java)

Have a look at some quick ways to find an element in a list in Java[Read more](/web/20221211183031/https://www.baeldung.com/find-list-element-java) →

## [在 Java 中混洗收藏](/web/20221211183031/https://www.baeldung.com/java-shuffle-collection)

Learn how to shuffle various collections in Java.[Read more](/web/20221211183031/https://www.baeldung.com/java-shuffle-collection) →

## [在 Java 中检查两个列表是否相等](/web/20221211183031/https://www.baeldung.com/java-test-a-list-for-ordinality-and-equality)

A short article focused on the common problem of testing if two List instances contain the same elements in exactly the same order.[Read more](/web/20221211183031/https://www.baeldung.com/java-test-a-list-for-ordinality-and-equality) →

## 2。将`List`转换为`Set`

### 2.1。用普通 Java

让我们从使用 Java 将 **a `List`转换成`Set`开始:**

```
public void givenUsingCoreJava_whenListConvertedToSet_thenCorrect() {
    List<Integer> sourceList = Arrays.asList(0, 1, 2, 3, 4, 5);
    Set<Integer> targetSet = new HashSet<>(sourceList);
}
```

正如我们所看到的，转换过程是类型安全和简单的，因为每个集合的构造函数都接受另一个集合作为源。

### 2.2。有番石榴

让我们用番石榴做同样的转换:

```
public void givenUsingGuava_whenListConvertedToSet_thenCorrect() {
    List<Integer> sourceList = Lists.newArrayList(0, 1, 2, 3, 4, 5);
    Set<Integer> targetSet = Sets.newHashSet(sourceList);
}
```

### 2.3。使用 Apache Commons 集合

接下来让我们使用 Commons Collections API 在一个`List`和一个`Set`之间进行转换:

```
public void givenUsingCommonsCollections_whenListConvertedToSet_thenCorrect() {
    List<Integer> sourceList = Lists.newArrayList(0, 1, 2, 3, 4, 5);
    Set<Integer> targetSet = new HashSet<>(6);
    CollectionUtils.addAll(targetSet, sourceList);
}
```

### 2.4.使用 Java 10

一个额外的选择是使用 Java 10 中引入的 [`Set.copyOf`](https://web.archive.org/web/20221211183031/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html#copyOf(java.util.Collection)) 静态工厂方法:

```
public void givenUsingJava10_whenListConvertedToSet_thenCorrect() {
    List sourceList = Lists.newArrayList(0, 1, 2, 3, 4, 5);
    Set targetSet = Set.copyOf(sourceList);
}
```

注意，这样创建的`Set`是[不可修改的](https://web.archive.org/web/20221211183031/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html#unmodifiable)。

## 3。将`Set`转换为`List`

### 3.1。用普通 Java

现在让我们做反向转换，使用 Java 将**从`Set`转换为`List,` :**

```
public void givenUsingCoreJava_whenSetConvertedToList_thenCorrect() {
   Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
   List<Integer> targetList = new ArrayList<>(sourceSet);
}
```

### 3.2。有番石榴

我们可以用番石榴溶液做同样的事情:

```
public void givenUsingGuava_whenSetConvertedToList_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    List<Integer> targetList = Lists.newArrayList(sourceSet);
} 
```

这与 java 方法非常相似，只是重复的代码少一些。

### 3.3。使用 Apache Commons 集合

现在让我们看看在`Set` 和`List`之间转换的 Commons Collections 解决方案:

```
public void givenUsingCommonsCollections_whenSetConvertedToList_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    List<Integer> targetList = new ArrayList<>(6);
    CollectionUtils.addAll(targetList, sourceSet);
}
```

### 3.4.使用 Java 10

最后，我们可以使用 Java 10 中引入的 [`List.copyOf`](https://web.archive.org/web/20221211183031/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection)) :

```
public void givenUsingJava10_whenSetConvertedToList_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    List<Integer> targetList = List.copyOf(sourceSet);
}
```

我们需要记住，结果`List`是[不可修改的](https://web.archive.org/web/20221211183031/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#unmodifiable)。

## 4。结论

所有这些例子和代码片段的实现可以在 GitHub 上找到。 这是一个基于 Maven 的项目，所以应该很容易导入和运行。