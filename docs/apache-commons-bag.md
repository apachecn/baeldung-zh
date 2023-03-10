# Apache Commons 收藏包

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-bag>

[This article is part of a series:](javascript:void(0);)• Apache Commons Collections Bag (current article)[• Apache Commons Collections SetUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-setutils)
[• Apache Commons Collections OrderedMap](/web/20221208143854/https://www.baeldung.com/apache-commons-ordered-map)
[• Apache Commons Collections BidiMap](/web/20221208143854/https://www.baeldung.com/commons-collections-bidi-map)
[• A Guide to Apache Commons Collections CollectionUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-collection-utils)
[• Apache Commons Collections MapUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-map-utils)
[• Guide to Apache Commons CircularFifoQueue](/web/20221208143854/https://www.baeldung.com/commons-circular-fifo-queue)

## 1。简介

在这篇简短的文章中，我们将关注如何使用 Apache 的`Bag`集合。

## 延伸阅读:

## [Apache Commons BeanUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-beanutils)

Learn how to use Apache Commons BeanUtils for common bean operations.[Read more](/web/20221208143854/https://www.baeldung.com/apache-commons-beanutils) →

## Apache common me

A quick and practical guide to the Apache Commons IO open source library for Java covering many of its better-known features.[Read more](/web/20221208143854/https://www.baeldung.com/apache-commons-io) →

## [Apache Commons 文本简介](/web/20221208143854/https://www.baeldung.com/java-apache-commons-text)

Learn how to use Apache Commons Text for common String operations.[Read more](/web/20221208143854/https://www.baeldung.com/java-apache-commons-text) →

## 2。Maven 依赖关系

在开始之前，我们需要从 [Maven Central](https://web.archive.org/web/20221208143854/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22) 导入最新的依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

## 3。包包与系列

简单地说， `Bag`是一个允许存储多个项目及其重复计数的集合:

```java
public void whenAdded_thenCountIsKept() {
    Bag<Integer> bag = new HashBag<>(
      Arrays.asList(1, 2, 3, 3, 3, 1, 4));

    assertThat(2, equalTo(bag.getCount(1)));
} 
```

### 3.1。违反`Collection`合同

在阅读`Bag`的 API 文档时，我们可能会注意到一些方法被标记为违反了标准 Java 的集合契约。

例如，当我们从 Java 集合中使用一个`add()` API 时，我们会收到`true` ,即使该项目已经在集合中:

```java
Collection<Integer> collection = new ArrayList<>();
collection.add(1);
assertThat(collection.add(1), is(true));
```

当我们添加集合中已经可用的元素时，来自`Bag`实现的相同 API 将返回一个`false`:

```java
Bag<Integer> bag = new HashBag<>();
bag.add(1);

assertThat(bag.add(1), is(not(true)));
```

为了解决这些问题，Apache Collections 的库提供了一个名为`CollectionBag.`的装饰器，我们可以使用它来使我们的包集合符合 Java `Collection`契约:

```java
public void whenBagAddAPILikeCollectionAPI_thenTrue() {
    Bag<Integer> bag = CollectionBag.collectionBag(new HashBag<>());
    bag.add(1);

    assertThat(bag.add(1), is((true)));
}
```

## 4。箱包实施

现在让我们探索一下在 Apache 的 collections 库中`Bag`接口的各种实现。

### 4.1。`HashBag`

我们可以添加一个元素，并指示 API 该元素在我们的包集合中应该有多少个副本:

```java
public void givenAdd_whenCountOfElementsDefined_thenCountAreAdded() {
    Bag<Integer> bag = new HashBag<>();

    bag.add(1, 5); // adding 1 five times

    assertThat(5, equalTo(bag.getCount(1)));
}
```

我们还可以从包中删除元素的特定数量的副本或每个实例:

```java
public void givenMultipleCopies_whenRemove_allAreRemoved() {
    Bag<Integer> bag = new HashBag<>(
      Arrays.asList(1, 2, 3, 3, 3, 1, 4));

    bag.remove(3, 1); // remove one element, two still remain
    assertThat(2, equalTo(bag.getCount(3)));

    bag.remove(1); // remove all
    assertThat(0, equalTo(bag.getCount(1)));
}
```

### 4.2。`TreeBag`

`TreeBag`实现像任何其他树一样工作，另外维护`Bag`语义。

我们可以自然地用一个`TreeBag`对整数数组进行排序，然后查询集合中每个元素的实例数量:

```java
public void givenTree_whenDuplicateElementsAdded_thenSort() {
    TreeBag<Integer> bag = new TreeBag<>(Arrays.asList(7, 5,
      1, 7, 2, 3, 3, 3, 1, 4, 7));

    assertThat(bag.first(), equalTo(1));
    assertThat(bag.getCount(bag.first()), equalTo(2));
    assertThat(bag.last(), equalTo(7));
    assertThat(bag.getCount(bag.last()), equalTo(3));
}
```

`TreeBag`实现了一个`SortedBag`接口，这个接口的所有实现都可以使用装饰器`CollectionSortedBag`来遵守 Java 集合契约:

```java
public void whenTreeAddAPILikeCollectionAPI_thenTrue() {
    SortedBag<Integer> bag 
      = CollectionSortedBag.collectionSortedBag(new TreeBag<>());

    bag.add(1);

    assertThat(bag.add(1), is((true)));
}
```

### 4.3。`SynchronizedSortedBag`

`Bag`的另一个广泛使用的实现是`SynchronizedSortedBag`。准确地说，这是一个`SortedBag` 实现的同步装饰器。

我们可以使用这个装饰器和上一节中的`TreeBag`(一个`SortedBag`的实现)来同步对我们包的访问:

```java
public void givenSortedBag_whenDuplicateElementsAdded_thenSort() {
    SynchronizedSortedBag<Integer> bag = SynchronizedSortedBag
      .synchronizedSortedBag(new TreeBag<>(
        Arrays.asList(7, 5, 1, 7, 2, 3, 3, 3, 1, 4, 7)));

    assertThat(bag.first(), equalTo(1));
    assertThat(bag.getCount(bag.first()), equalTo(2));
    assertThat(bag.last(), equalTo(7));
    assertThat(bag.getCount(bag.last()), equalTo(3));
}
```

我们可以使用 API-`Collections.synchronizedSortedMap()` 和 `TreeMap –` 的组合来模拟我们在这里使用`SynchronizedSortedBag`所做的事情。

## 5。结论

在这个简短的教程中，我们已经了解了`Bag`接口及其各种实现。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-collections)

Next **»**[Apache Commons Collections SetUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-setutils)