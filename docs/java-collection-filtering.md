# 如何在 Java 中过滤集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collection-filtering>

## 1.概观

在这个简短的教程中，**我们将看看在 Java** 中过滤集合的不同方法——也就是说，找到满足特定条件的所有项目。

这是几乎所有 Java 应用程序中都存在的一项基本任务。

由于这个原因，为此目的提供功能的库的数量是很大的。

特别是，在本教程中，我们将涵盖:

*   Java 8 Streams 的`filter()`函数
*   Java 9 `filtering`收集器
*   相关的`Eclipse Collections`API
*   阿帕奇的`CollectionUtils filter()`方法
*   番石榴的`Collections2 filter()`方法

## 2.使用流

自从 Java 8 推出以来， [Streams](/web/20220630021051/https://www.baeldung.com/java-8-streams-introduction) 在我们必须处理大量数据的大多数情况下都扮演了重要角色。

因此，这是大多数情况下的首选方法，因为它是用 Java 构建的，不需要额外的依赖。

### 2.1.使用`Streams`过滤收藏

为了简单起见，**在所有的例子中，我们的目标是创建一个方法，只从`Integer` 值的`Collection`** 中检索偶数。

因此，我们可以将用于评估每个项目的条件表示为'`value % 2 == 0`'。

在所有情况下，我们必须将这个条件定义为一个`Predicate`对象:

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> streamsPredicate = item -> item % 2 == 0;

    return baseCollection.stream()
      .filter(streamsPredicate)
      .collect(Collectors.toList());
}
```

重要的是要注意到**我们在本教程中分析的每个库都提供了自己的`Predicate `实现**，但是它们都被定义为函数接口，因此允许我们使用 Lambda 函数来声明它们。

在这种情况下，我们使用了 Java 提供的预定义的`Collector `，它将元素累积到一个`List`中，但是我们也可以使用其他的，正如在[上一篇文章](/web/20220630021051/https://www.baeldung.com/java-8-collectors)中所讨论的。

### 2.2.Java 9 中集合分组后的过滤

Streams 允许我们使用 [`groupingBy collector`](/web/20220630021051/https://www.baeldung.com/java-groupingby-collector) 来聚合项目。

然而，如果我们像在上一节中那样进行过滤，在这个收集器发挥作用之前，一些元素可能会在早期阶段被丢弃。

出于这个原因，Java 9 引入了**`filtering`收集器，目的是在分组后处理子集合。**

按照我们的示例，假设我们希望在过滤掉奇数之前，根据每个整数的位数对集合进行分组:

```java
public Map<Integer, List<Integer>> findEvenNumbersAfterGrouping(
  Collection<Integer> baseCollection) {

    Function<Integer, Integer> getQuantityOfDigits = item -> (int) Math.log10(item) + 1;

    return baseCollection.stream()
      .collect(groupingBy(
        getQuantityOfDigits,
        filtering(item -> item % 2 == 0, toList())));
}
```

简而言之，如果我们使用这个收集器，我们可能会得到一个空的值条目，而如果我们在分组之前进行过滤，收集器根本不会创建这样的条目。

当然，我们会根据我们的需求来选择方法。

## 3.使用`Eclipse Collections`

我们还可以利用其他一些第三方库来实现我们的目标，要么是因为我们的应用不支持 Java 8，要么是因为我们想利用 Java 没有提供的一些强大功能。

这就是`Eclipse Collections`的情况，它是一个努力跟上新范例的库，不断发展并接受所有最新 Java 版本引入的变化。

我们可以从探索我们的 [Eclipse 集合介绍文章](/web/20220630021051/https://www.baeldung.com/eclipse-collections)开始，对这个库提供的功能有一个更广泛的了解。

### 3.1.属国

让我们从向项目的`pom.xml`添加以下依赖项开始:

```java
<dependency>
    <groupId>org.eclipse.collections</groupId>
    <artifactId>eclipse-collections</artifactId>
    <version>9.2.0</version>
</dependency>
```

[`eclipse-collections`](https://web.archive.org/web/20220630021051/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.eclipse.collections%22%20AND%20a%3A%22eclipse-collections%22) 包括所有必要的数据结构接口和 API 本身。

### 3.2.使用`Eclipse Collections`过滤收藏

现在让我们在它的一个数据结构上使用 eclipse 的过滤功能，比如它的`MutableList`:

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> eclipsePredicate
      = item -> item % 2 == 0;

    Collection<Integer> filteredList = Lists.mutable
      .ofAll(baseCollection)
      .select(eclipsePredicate);

    return filteredList;
}
```

或者，我们可以使用`Iterate`的`select()` 静态方法来定义`filteredList`对象:

```java
Collection<Integer> filteredList
 = Iterate.select(baseCollection, eclipsePredicate);
```

## 4.使用 Apache 的`CollectionUtils`

为了开始使用 Apache 的`CollectionUtils`库，我们可以查看[这个简短的教程](/web/20220630021051/https://www.baeldung.com/apache-commons-collection-utils)，在那里我们讨论了它的用法。

然而，在本教程中，我们将关注它的` filter()`实现。

### 4.1.属国

首先，我们需要在`pom.xml`文件中包含以下依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
```

### 4.2.使用`CollectionUtils`过滤收藏

我们现在准备使用`CollectonUtils`方法:

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> apachePredicate = item -> item % 2 == 0;

    CollectionUtils.filter(baseCollection, apachePredicate);
    return baseCollection;
}
```

我们必须考虑到这个方法通过删除每个不匹配条件的条目来修改`baseCollection`。

这意味着**基本集合必须是可变的，否则它将抛出一个异常**。

## 5.使用番石榴的`Collections2`

像以前一样，我们可以阅读我们以前的帖子[‘在番石榴中过滤和转换集合’](/web/20220630021051/https://www.baeldung.com/guava-filter-and-transform-a-collection)以获得关于这个主题的更多信息。

### 5.1.属国

让我们从在我们的`pom.xml`文件中添加[这个依赖关系](https://web.archive.org/web/20220630021051/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

### 5.2.使用`Collections2`过滤收藏

正如我们所看到的，这种方法与上一节中遵循的方法非常相似:

```java
public Collection<Integer> findEvenNumbers(Collection<Integer> baseCollection) {
    Predicate<Integer> guavaPredicate = item -> item % 2 == 0;

    return Collections2.filter(baseCollection, guavaPredicate);
}
```

同样，这里我们定义了一个特定于番石榴的`Predicate`对象。

在这种情况下，Guava 不修改`baseCollection`，它生成一个新的，所以我们可以使用一个不可变的集合作为输入。

## 6.结论

总之，我们已经看到在 Java 中有许多不同的过滤集合的方法。

尽管流通常是首选的方法，但是了解并记住其他库提供的功能是有好处的。

尤其是当我们需要支持旧的 Java 版本时。然而，如果是这种情况，我们需要记住在整个教程中使用的最新 Java 特性，比如 lambdas 应该用匿名类替换。

像往常一样，我们可以在我们的 [Github repo](https://web.archive.org/web/20220630021051/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2) 中找到本教程中显示的所有例子。