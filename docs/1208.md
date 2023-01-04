# 如何向流中添加单个元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-append-prepend>

## 1。概述

在这篇简短的文章中，我们将看看如何**向 Java 8 `Stream`** 添加元素，这不像向普通集合添加元素那样直观。

## 2。前置

通过调用静态的`Stream.` concat `()`方法，我们可以很容易地将给定的元素添加到`Stream`中:

```
@Test
public void givenStream_whenPrependingObject_thenPrepended() {
    Stream<Integer> anStream = Stream.of(1, 2, 3, 4, 5);

    Stream<Integer> newStream = Stream.concat(Stream.of(99), anStream);

    assertEquals(newStream.findFirst().get(), (Integer) 99);
}
```

## 3。追加

同样，要将一个元素追加到一个`Stream,`的末尾，我们只需要反转参数。

请记住， **`Streams`可以表示无限序列**,所以有些情况下您可能永远也找不到新元素:

```
@Test
public void givenStream_whenAppendingObject_thenAppended() {
    Stream<String> anStream = Stream.of("a", "b", "c", "d", "e");

    Stream<String> newStream = Stream.concat(anStream, Stream.of("A"));

    List<String> resultList = newStream.collect(Collectors.toList());

    assertEquals(resultList.get(resultList.size() - 1), "A");
}
```

## 4。在特定指数

`Stream (java.util.stream)`在 Java 中是支持顺序和并行聚合操作的元素序列。没有向特定索引添加值的功能，因为它不是为这种事情设计的。然而，有几种方法可以实现它。

一种方法是**使用终端操作将流的值收集到`ArrayList`T4，然后简单地使用`**add(int index, E element)**`方法。请记住，这会给你想要的结果，但你也会**失去`Stream`的懒惰，因为你需要在插入新元素之前消耗它。****

另一种方法是使用`Spliterator`并按顺序迭代元素到目标索引(使用`Spliterator`类的迭代器)。)概念是将流分成两个流(A 和 B)。流 A 从索引 0 开始到目标索引，流 B 是剩余的元素。然后将元素添加到流 A 中，最后连接流 A 和 b。

在这种情况下，由于流没有被贪婪地消耗，它仍然是部分懒惰的:

```
private static  Stream insertInStream(Stream stream, T elem, int index) {
    Spliterator spliterator = stream.spliterator();
    Iterator iterator = Spliterators.iterator(spliterator);

    return Stream.concat(Stream.concat(Stream.generate(iterator::next)
      .limit(index), Stream.of(elem)), StreamSupport.stream(spliterator, false));
}
```

现在，让我们测试我们的代码，以确保一切按预期运行:

```
@Test
public void givenStream_whenInsertingObject_thenInserted() {
    Stream<Double> anStream = Stream.of(1.1, 2.2, 3.3);
    Stream<Double> newStream = insertInStream(anStream, 9.9, 3);

    List<Double> resultList = newStream.collect(Collectors.toList());

    assertEquals(resultList.get(3), (Double) 9.9);
}
```

## 5。结论

在这篇短文中，我们已经看到了如何将单个元素添加到`Stream,`的开头、结尾或给定位置。

请记住，尽管预先添加一个元素对任何`Stream,`都有效，但是将它添加到末尾或特定的索引处只对有限的流有效。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20221126220041/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)