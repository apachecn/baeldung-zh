# Java 流与 Vavr 流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-java-streams>

## 1。简介

在本文中，我们将看看`Stream`在 Java 和 Vavr 中的实现有何不同。

本文假设读者熟悉 Java 流 API 和 Vavr 库的基础知识。

## 2。比较

两种实现都代表了相同的懒惰序列概念，但在细节上有所不同。

**Java `Streams`构建时考虑到了强大的并行性**，为并行化提供了简单的支持。另一方面，Vavr 实现支持对数据序列的便捷处理，并且不提供对并行性的本地支持(但是可以通过将实例转换为 Java 实现来实现)。

这就是为什么 Java Streams 由 [`Spliterator`](/web/20221017183250/https://www.baeldung.com/java-spliterator) 实例支持——这是对更老的`Iterator` 的升级，Vavr 的实现由前述的`Iterator`支持(至少在一个最新的实现中)。

这两个实现都松散地绑定到它的后台数据结构，本质上是流遍历的数据源之上的门面，但是因为 Vavr 的实现是基于`Iterator-`的`,`，所以它不容忍源集合的并发修改。

Java 对流源的处理使得在终端流操作执行之前，行为良好的流源有可能被修改。

尽管存在基本的设计差异，但 Vavr 提供了一个非常健壮的 API，可以将其流(和其他数据结构)转换为 Java 实现。

## 3。附加功能

处理流及其元素的方法导致了我们在 Java 和 Vavr 中处理它们的方式的有趣差异

### 3.1。随机元素访问

为元素提供方便的 API 和访问方法是 Vavr 真正超越 Java API 的一个领域。例如，Vavr 有一些提供随机元素访问的方法:

*   `[get()](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#get-int-) `提供基于索引的流元素访问。
*   [`indexOf()`](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#indexOf-T-int-) 提供了与标准 Java `List.`中相同的索引位置功能
*   [`insert()`](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#insert-int-T-) 提供了将元素添加到流中指定位置的能力。
*   [`intersperse()`](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#intersperse-T-) 将提供的参数插入到流的所有元素之间。
*   [`find()`](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Traversable.html#find-java.util.function.Predicate-) 将从流内定位并返回一个项目。Java 提供了 [`noneMatched`](https://web.archive.org/web/20221017183250/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#noneMatch(java.util.function.Predicate)) ，它只是检查一个元素的存在。
*   `[update()](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#update-int-java.util.function.Function-) `将替换给定索引处的元素。这也接受一个函数来计算替换。
*   [`search` `()`](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/LinearSeq.html#search-T-) 将在已排序的流中定位项目(未排序的流将产生未定义的结果)

重要的是我们要记住，这个功能仍然是由一个数据结构支持的，这个数据结构具有线性搜索性能。

### 3.2。并行和并发修改

虽然 Vavr 的流不像 Java 的`parallel()`方法那样支持并行，但是有一种`[toJavaParallelStream](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/Value.html#toJavaParallelStream--) `方法可以提供 Vavr 流的基于 Java 的并行副本。

Vavr 流的一个相对薄弱的领域是基于 [`**Non-Interference**`的原理。](https://web.archive.org/web/20221017183250/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html#NonInterference)

简单地说，Java 流允许我们修改底层数据源，直到调用一个终端操作。只要没有在给定的 Java 流上调用终端操作，该流就可以获得对底层数据源的任何更改:

```java
List<Integer> intList = new ArrayList<>();
intList.add(1);
intList.add(2);
intList.add(3);
Stream<Integer> intStream = intList.stream(); //form the stream
intList.add(5); //modify underlying list
intStream.forEach(i -> System.out.println("In a Java stream: " + i)); 
```

我们会发现最后的添加反映在流的输出中。无论修改是在流管道内部还是外部，这种行为都是一致的:

```java
in a Java stream: 1
in a Java stream: 2
in a Java stream: 3
in a Java stream: 5 
```

我们发现 Vavr 流不能容忍这一点:

```java
Stream<Integer> vavrStream = Stream.ofAll(intList);
intList.add(5)
vavrStream.forEach(i -> System.out.println("in a Vavr Stream: " + i)); 
```

我们得到了什么:

```java
Exception in thread "main" java.util.ConcurrentModificationException
  at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
  at java.util.ArrayList$Itr.next(ArrayList.java:851)
  at io.vavr.collection.StreamModule$StreamFactory.create(Stream.java:2078)
```

按照 Java 的标准，Vavr 流不是“行为良好”的。Vavr 使用原始支持数据结构表现更好:

```java
int[] aStream = new int[]{1, 2, 4};
Stream<Integer> wrapped = Stream.ofAll(aStream);

aStream[2] = 5;
wrapped.forEach(i -> System.out.println("Vavr looped " + i));
```

给了我们:

```java
Vavr looped 1
Vavr looped 2
Vavr looped 5 
```

### 3.3。短路操作和`flatMap()`

与`map`操作一样，`flatMap,`是流处理中的中间操作——两个实现都遵循[中间流操作](https://web.archive.org/web/20221017183250/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html#StreamOps)的约定——在调用终端操作之前，底层数据结构的处理不应发生。

然而，JDK 8 和 9 的特点是[有一个错误](https://web.archive.org/web/20221017183250/https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8075939)，导致`flatMap`实现在与`findFirst `或`limit`之类的短路中间操作结合时破坏这个契约并急切地求值。

一个简单的例子:

```java
Stream.of(42)
  .flatMap(i -> Stream.generate(() -> { 
      System.out.println("nested call"); 
      return 42; 
  }))
  .findAny();
```

在上面的代码片段中，我们永远不会从`findAny`得到结果，因为`flatMap` 会被急切地计算，而不是简单地从嵌套的`Stream.`中取出一个元素

Java 10 中提供了对这个错误的修复。

Vavr 的`flatMap `没有相同的问题，并且在 O(1)中完成了功能相似的操作:

```java
Stream.of(42)
  .flatMap(i -> Stream.continually(() -> { 
      System.out.println("nested call"); 
      return 42; 
  }))
  .get(0); 
```

### 3.4。核心 Vavr 功能

在某些领域，Java 和 Vavr 之间没有一对一的比较；Vavr 增强了流媒体体验，其功能是 Java 无法比拟的(或者至少需要相当多的手动工作):

*   `[partition()](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#partition-java.util.function.Predicate-) `将给定一个谓词，将一个流的内容拆分成两个流。
*   `[permutation()](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#permutations--) `顾名思义，将计算流元素的排列(所有可能的唯一排序)。
*   `[combinations()](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#combinations-int-) `给出流的组合(即可能的项目选择)。
*   [`groupBy`](https://web.archive.org/web/20221017183250/https://static.javadoc.io/io.vavr/vavr/0.9.2/io/vavr/collection/Stream.html#groupBy-java.util.function.Function-) 将返回一个`Map `流，其中包含来自原始流的元素，由提供的分类器进行分类。
*   Vavr 中的 [`distinct`](https://web.archive.org/web/20221017183250/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#distinct()) 方法通过提供一个接受`compareTo` lambda 表达式的变体，对 Java 版本进行了改进。

虽然 Java SE 流对高级功能的支持有些缺乏创意，但奇怪的是， [Expression Language 3.0](https://web.archive.org/web/20221017183250/https://github.com/javaee/el-spec/blob/master/spec/pdf/EL3.0.PFD.pdf) 提供了比标准 JDK 流更多的功能支持。

## 4。流操作

Vavr 允许直接操纵流的内容:

*   **插入到现有的 Vavr 流中**

```java
Stream<String> vavredStream = Stream.of("foo", "bar", "baz");
vavredStream.forEach(item -> System.out.println("List items: " + item));
Stream<String> vavredStream2 = vavredStream.insert(2, "buzz");
vavredStream2.forEach(item -> System.out.println("List items: " + item));
```

*   **从流中删除一个项目**

```java
Stream<String> removed = inserted.remove("buzz"); 
```

*   **基于队列的操作**

通过 Vavr 的流由队列支持，它提供了恒定时间的`prepend`和`append `操作。

然而，**对 Vavr 流所做的更改不会传播回创建该流的数据源。**

## 5。结论

Vavr 和 Java 各有所长，我们已经展示了每个库对其设计目标的承诺——Java 是廉价的并行，Vavr 是方便的流操作。

由于 Vavr 支持在它自己的流和 Java 的流之间来回转换，人们可以在同一个项目中获得两个库的好处，而不会有太多的开销。

本教程的源代码可以在 Github 上的[处获得。](https://web.archive.org/web/20221017183250/https://github.com/eugenp/tutorials/tree/master/vavr-modules/java-vavr-stream)