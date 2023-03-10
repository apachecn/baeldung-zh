# 新的流，比较器和收集器在番石榴 21

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-21-new>

## **1。** ****简介****

本文是关于 Google Guava library 第 21 版推出的新特性的系列文章的第一篇。我们将讨论新增加的职业和以前版本的主要变化。

更具体地说，我们将讨论 `common.collect`包中的添加和变化。

Guava 21 在`common.collect`包中引入了一些新的有用的功能；让我们快速浏览一下这些新的实用程序，以及我们如何最大限度地利用它们。

## 2。`Streams`

我们都对 Java 8 中最新添加的`java.util.stream.Stream`感到兴奋。那么，番石榴现在很好地利用了流，并提供了 Oracle 可能错过的东西。

`Streams`是一个静态实用程序类，有一些处理 Java 8 流的非常需要的实用程序。

### 2.1。`Streams.stream()`

`Streams`类提供了使用`Iterable`、`Iterator`、`Optional`和`Collection`创建流的四种方法。

不过，不推荐使用`Collection`创建流，因为它是由 Java 8 提供的:

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
Stream<Integer> streamFromCollection = Streams.stream(numbers);
Stream<Integer> streamFromIterator = Streams.stream(numbers.iterator());
Stream<Integer> streamFromIterable = Streams.stream((Iterable<Integer>) numbers);
Stream<Integer> streamFromOptional = Streams.stream(Optional.of(1)); 
```

`Streams`类还提供了`OptionalDouble`、`OptionalLong`和`OptionalInt`口味。这些方法返回只包含该元素的流，否则为空流:

```java
LongStream streamFromOptionalLong = Streams.stream(OptionalLong.of(1));
IntStream streamFromOptionalInt = Streams.stream(OptionalInt.of(1));
DoubleStream streamFromOptionalDouble = Streams.stream(OptionalDouble.of(1.0));
```

### 2.2。`Streams.concat()`

这个类提供了连接多个同类流的方法。

```java
Stream<Integer> concatenatedStreams = Streams.concat(streamFromCollection, streamFromIterable,streamFromIterator);
```

`concat`功能有几种风格——`LongStream`、`IntStream`和`DoubleStream`。

### 2.3。`Streams.findLast()`

`Streams`使用`findLast()`方法找到流中的最后一个元素。

该方法或者返回最后一个元素，或者如果流中没有元素，则返回`Optional.empty()`:

```java
List<Integer> integers = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
Optional<Integer> lastItem = Streams.findLast(integers.stream());
```

`findLast()`方法适用于`LongStream`、`IntStream`和`DoubleStream`。

### 2.4。`Streams.mapWithIndex()`

通过使用`mapWithIndex()` 方法，流的每个元素携带关于它们各自位置(索引)的信息:

```java
mapWithIndex( Stream.of("a", "b", "c"), (str, index) -> str + ":" + index)
```

这将返回`Stream.of(“a:0″,”b:1″,”c:2”)`。

使用过载的`mapWithIndex()`可以用`IntStream`、`LongStream`和`DoubleStream`达到同样的效果。

### 2.5。`Streams.zip()`

为了使用一些函数映射两个流的对应元素，只需使用`Streams:`的 zip 方法

```java
Streams.zip(
  Stream.of("candy", "chocolate", "bar"),
  Stream.of("$1", "$2","$3"),
  (arg1, arg2) -> arg1 + ":" + arg2
);
```

这将返回`Stream.of(“candy:$1″,”chocolate:$2″,”bar:$3”);`

结果流将仅与两个输入流中较短的一个一样长；如果一个流更长，它的额外元素将被忽略。

## 3。`Comparators`

番石榴`Ordering`类已被弃用，在新版本中处于删除阶段。`Ordering`级的大部分功能已经在 JDK 8 中收录了。

Guava 引入了`Comparators`来提供`Ordering`的额外特性，Java 8 标准库还没有提供这些特性。

让我们快速浏览一下这些。

### 3.1。`Comparators.isInOrder()`

如果 Iterable 中的每个元素都大于或等于前一个元素，该方法返回 true，如`Comparator`所指定的:

```java
List<Integer> integers = Arrays.asList(1,2,3,4,4,6,7,8,9,10);
boolean isInAscendingOrder = Comparators.isInOrder(
  integers, new AscedingOrderComparator());
```

### 3.2。`Comparators.isInStrictOrder()`

非常类似于`isInOrder()`方法，但是它严格地保持条件，元素不能等于前一个，它必须大于。对于此方法，前面的代码将返回 false。

### 3.3。`Comparators.lexicographical()`

这个 API 返回一个新的`Comparator`实例——它按照字典顺序排序，两两比较对应的元素。在内部，它创建了一个新的`LexicographicalOrdering<S>()`实例。

## 4。`MoreCollectors`

`MoreCollectors`包含了一些 Java 8 `java.util.stream.Collectors`中没有的非常有用的`Collectors`，并且与`com.google.common`类型没有关联。

让我们来看几个。

### 4.1。`MoreCollectors.toOptional()`

这里，`Collector`将包含零个或一个元素的流转换为`Optional`:

```java
List<Integer> numbers = Arrays.asList(1);
Optional<Integer> number = numbers.stream()
  .map(e -> e * 2)
  .collect(MoreCollectors.toOptional());
```

如果流包含不止一个元素——收集器将抛出`IllegalArgumentException.` 

### 4.2。 `MoreCollectors.onlyElement()`

使用这个 API，`Collector`获取一个只包含一个元素的流，并返回该元素；如果流包含不止一个元素，它抛出`IllegalArgumentException`，或者如果流包含零个元素，它抛出`NoSuchElementException`。

## 5。`Interners.InternerBuilder`

这是 Guava 库中已经存在的`Interners`的内部构建器类。它提供了一些方便的方法来定义您喜欢的`Interner`的并发级别和类型(弱或强):

```java
Interners interners = Interners.newBuilder()
  .concurrencyLevel(2)
  .weak()
  .build();
```

## 6。结论

在这篇简短的文章中，我们探索了 Guava 21 的`common.collect package`中新添加的功能。

一如既往，这篇文章的代码可以在 Github 上找到[。](https://web.archive.org/web/20221128112341/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-21)