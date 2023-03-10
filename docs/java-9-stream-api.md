# Java 9 流 API 改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-stream-api>

## 1。概述

在这篇快速的文章中，我们将关注 Java 9 中新的有趣的流 API 改进。

## 2。流 Takewhile/Dropwhile

关于这些方法的讨论在 [StackOverflow](https://web.archive.org/web/20220628093237/https://stackoverflow.com/) 上反复出现(最流行的是[这个](https://web.archive.org/web/20220628093237/https://stackoverflow.com/questions/20746429/limit-a-stream-by-a-predicate))。

假设我们想要生成一个`String`的`Stream`,方法是将前一个`Stream`的值增加一个字符，直到这个`Stream`中当前值的长度小于`10`。

在 Java 8 中我们将如何解决它？我们可以使用类似于 [`limit`](https://web.archive.org/web/20220628093237/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#limit(long)) 、 [`allMatch`](https://web.archive.org/web/20220628093237/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#allMatch(java.util.function.Predicate)) 的短路中间操作来实现其他目的，或者基于 [`Spliterator`](https://web.archive.org/web/20220628093237/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Spliterator.html) 编写我们自己的 [`takeWhile`实现](https://web.archive.org/web/20220628093237/https://stackoverflow.com/a/20765715/4922375)，从而使这个简单的问题变得复杂。

使用 Java 9，解决方案很简单:

```java
Stream<String> stream = Stream.iterate("", s -> s + "s")
  .takeWhile(s -> s.length() < 10); 
```

`takeWhile`操作采用应用于元素的`Predicate`来确定这些元素的最长前缀(如果流是有序的)或流元素的子集(如果流是无序的)。

为了向前推进，我们最好理解“最长前缀”和“一个`Stream's`子集”是什么意思:

*   **最长的前缀**是匹配给定谓词的流元素的连续序列。序列的第一个元素是该流的第一个元素，紧跟在序列的最后一个元素之后的元素与给定的谓词不匹配
*   **一个`Stream's`子集**是与给定谓词匹配的`Stream`的一些(但不是全部)元素的集合。

介绍完这些关键术语后，我们很容易理解另一个新的`dropWhile`操作。

它的作用与`takeWhile`完全相反。如果一个流是有序的，`dropWile`返回一个流，该流由这个`Stream` **的剩余元素组成，在丢弃了与给定谓词匹配的元素的最长前缀**之后。

否则，如果一个`Stream`是无序的，`dropWile`在丢弃了与给定谓词匹配的元素子集之后，返回一个由这个`Stream`T3 的剩余元素组成的流。

让我们用前面得到的`Stream`扔掉前五个元素:

```java
stream.dropWhile(s -> !s.contains("sssss"));
```

简单地说，`dropWhile`操作将删除元素，而元素的给定谓词返回`true`并停止删除第一个谓词的`false`。

## 3。流迭代

下一个新特性是**有限`Streams` 代**的重载`iterate`方法。不要与 [`finite`](https://web.archive.org/web/20220628093237/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#iterate(T,java.util.function.UnaryOperator)) 变体混淆，后者返回由某个函数产生的无限有序的`Stream`。

一个新的`iterate`稍微修改了这个方法，添加了一个应用于元素的谓词来确定流必须终止的时间。它的用法非常方便简洁:

```java
Stream.iterate(0, i -> i < 10, i -> i + 1)
  .forEach(System.out::println);
```

它可以与相应的`for`语句相关联:

```java
for (int i = 0; i < 10; ++i) {
    System.out.println(i);
}
```

## 4。空的流

有些情况下，我们需要将一个元素放入`Stream`中。有时候，这个元素可能是一个`null`，但是我们不希望我们的`Stream`包含这样的值。它导致编写一个`if`语句或者一个三元运算符来检查一个元素是否是一个`null.`

假设`collection`和`map` 变量已经被成功创建和填充，请看下面的例子:

```java
collection.stream()
  .flatMap(s -> {
      Integer temp = map.get(s);
      return temp != null ? Stream.of(temp) : Stream.empty();
  })
  .collect(Collectors.toList());
```

为了避免这样的样板代码，在`Stream`类中添加了 [`ofNullable`](https://web.archive.org/web/20220628093237/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#ofNullable(T)) 方法。使用这种方法，前面的样本可以简单地转换成:

```java
collection.stream()
  .flatMap(s -> Stream.ofNullable(map.get(s)))
  .collect(Collectors.toList());
```

## 5。结论

我们考虑了 Java 9 中 Stream API 的主要变化，以及这些改进将如何帮助我们用更少的努力编写更强有力的程序。

和往常一样，代码片段可以在 Github 上找到[。](https://web.archive.org/web/20220628093237/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-improvements)