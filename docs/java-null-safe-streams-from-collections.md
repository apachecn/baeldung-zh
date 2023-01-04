# 来自集合的 Java 空安全流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-null-safe-streams-from-collections>

## 1。概述

在本教程中，我们将看到如何从 Java 集合中创建空安全流。

首先，需要对 Java 8 的方法引用、Lambda 表达式、`Optional`和 Stream API 有所了解，才能完全理解这些材料。

如果您对这些话题中的任何一个都不熟悉，请先看看我们之前的文章:[Java 8 中的新特性](/web/20220628121151/https://www.baeldung.com/java-8-new-features)、[Java 8 可选指南](/web/20220628121151/https://www.baeldung.com/java-optional)和[Java 8 流简介](/web/20220628121151/https://www.baeldung.com/java-8-streams-introduction)。

## 2。Maven 依赖关系

在我们开始之前，对于某些场景，我们需要一个 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
```

`[commons-collections4](https://web.archive.org/web/20220628121151/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22)`库可以从 Maven Central 下载。

## 3。从收藏创建流

从任何类型的`Collection`创建 [`Stream`](/web/20220628121151/https://www.baeldung.com/java-8-streams-introduction) 的基本方法是根据所需的流类型调用集合上的`stream()`或`parallelStream()`方法:

```java
Collection<String> collection = Arrays.asList("a", "b", "c");
Stream<String> streamOfCollection = collection.stream(); 
```

我们的集合很可能在某个时候有一个外部源，当从集合中创建流时，我们可能会以类似于下面的方法结束:

```java
public Stream<String> collectionAsStream(Collection<String> collection) {
    return collection.stream();
} 
```

这可能会导致一些问题。当提供的集合指向一个`null`引用时，代码将在运行时抛出一个`NullPointerException`。

下一节将介绍我们如何防范这种情况。

## 4。使创建的收集流为空安全

### 4.1。添加检查以防止`Null`取消引用

为了防止意外的`null`指针异常，**当从集合创建流时，我们可以选择添加检查来防止`null`引用**:

```java
Stream<String> collectionAsStream(Collection<String> collection) {
    return collection == null 
      ? Stream.empty() 
      : collection.stream();
} 
```

然而，这种方法有几个问题。

首先，`null`检查妨碍了业务逻辑，降低了程序的整体可读性。

其次，在 Java SE 8 之后，使用`null`来表示值的缺失被认为是一种错误的方法:有一种更好的方法来对值的缺失和存在进行建模。

重要的是要记住，空的`Collection`和`null` `Collection`是不一样的。第一个提示我们的查询没有显示结果或元素，而第二个提示在这个过程中发生了某种错误。

### 4.2。使用来自`CollectionUtils`库的`emptyIfNull`方法

我们可以选择使用 Apache Commons 的 [`CollectionUtils`](https://web.archive.org/web/20220628121151/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/CollectionUtils.html) 库来确保我们的流是`null`安全的。这个库提供了一个`emptyIfNull`方法，该方法返回一个不可变的空集合，给定一个`null`集合作为参数，否则返回集合本身:

```java
public Stream<String> collectionAsStream(Collection<String> collection) {
    return emptyIfNull(collection).stream();
} 
```

这是一个非常简单的策略。但是，它依赖于外部库。如果一个软件开发策略限制了这样一个库的使用，那么这个解决方案就变成`null`无效。

### 4.3。使用 Java 8 的`Optional`

Java SE 8 的 [`Optional`](/web/20220628121151/https://www.baeldung.com/java-optional) 是一个单值容器，要么包含一个值，要么不包含。如果缺少一个值，则称`Optional`容器为空。

使用`Optional`可以被认为是从流中创建空安全集合的最佳总体策略。

让我们看看如何使用它，然后进行一个简短的讨论:

```java
public Stream<String> collectionToStream(Collection<String> collection) {
    return Optional.ofNullable(collection)
      .map(Collection::stream)
      .orElseGet(Stream::empty);
} 
```

*   **`Optional.ofNullable(collection)`** 从传入的集合中创建一个`Optional`对象。如果集合是`null.`，则创建一个空的`Optional`对象
*   **`map(Collection::stream)`** 提取包含在`Optional`对象中的值作为`map`方法(`Collection.stream()`)的参数
*   **`orElseGet(Stream::empty)`** 返回`Optional`对象为空时的回退值，即传入的集合为`null`。

因此，我们主动保护我们的代码不受意外的`null`指针异常的影响。

### 4.4.使用 Java 9 的`Stream` `OfNullable`

在 4.1 节检查我们以前的三元例子。考虑到一些元素可能是`null`而不是`Collection`，我们在`Stream`类中有了`ofNullable`方法。

我们可以将上面的示例转换为:

```java
Stream<String> collectionAsStream(Collection<String> collection) {  
  return collection.stream().flatMap(s -> Stream.ofNullable(s));
}
```

## 5。结论

在本文中，我们简要回顾了如何从给定的集合创建流。然后，我们继续探索三个关键策略，以确保从集合创建流时，所创建的流是空安全的。

最后，我们指出了在相关情况下使用每种策略的弱点。

和往常一样，本文附带的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628121151/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)