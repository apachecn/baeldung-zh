# 在 Java 中合并流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-merge-streams>

## 1。概述

在这篇简短的文章中，我们解释了合并 Java `Streams`的不同方式——这不是一个非常直观的操作。

## 2。使用普通 Java

JDK 8 `Stream`类有一些有用的静态实用方法。让我们仔细看看`concat()`方法。

### 2.1。合并两个`Streams`

组合 2 个`Stream`的最简单方法是使用静态`Stream.concat()`方法:

```java
@Test
public void whenMergingStreams_thenResultStreamContainsElementsFromBoth() {
    Stream<Integer> stream1 = Stream.of(1, 3, 5);
    Stream<Integer> stream2 = Stream.of(2, 4, 6);

    Stream<Integer> resultingStream = Stream.concat(stream1, stream2);

    assertEquals(
      Arrays.asList(1, 3, 5, 2, 4, 6),
      resultingStream.collect(Collectors.toList()));
} 
```

### 2.2。合并多个`Stream` s

当我们需要合并超过 2 个时，事情就变得有点复杂了。一种可能是连接前两个流，然后将结果与下一个流连接，依此类推。

下一段代码展示了这一点:

```java
@Test
public void given3Streams_whenMerged_thenResultStreamContainsAllElements() {
    Stream<Integer> stream1 = Stream.of(1, 3, 5);
    Stream<Integer> stream2 = Stream.of(2, 4, 6);
    Stream<Integer> stream3 = Stream.of(18, 15, 36);

    Stream<Integer> resultingStream = Stream.concat(
      Stream.concat(stream1, stream2), stream3);

    assertEquals(
      Arrays.asList(1, 3, 5, 2, 4, 6, 18, 15, 36),
      resultingStream.collect(Collectors.toList()));
} 
```

正如我们所看到的，这种方法对于更多的流变得不可行。当然，我们可以创建中间变量或辅助方法来使它更具可读性，但这里有一个更好的选择:

```java
@Test
public void given4Streams_whenMerged_thenResultStreamContainsAllElements() {
    Stream<Integer> stream1 = Stream.of(1, 3, 5);
    Stream<Integer> stream2 = Stream.of(2, 4, 6);
    Stream<Integer> stream3 = Stream.of(18, 15, 36);
    Stream<Integer> stream4 = Stream.of(99);

    Stream<Integer> resultingStream = Stream.of(
      stream1, stream2, stream3, stream4)
      .flatMap(i -> i);

    assertEquals(
      Arrays.asList(1, 3, 5, 2, 4, 6, 18, 15, 36, 99),
      resultingStream.collect(Collectors.toList()));
} 
```

这里发生的是:

*   我们首先创建一个新的包含 4 个`Streams,`的`Stream`，这导致了一个`Stream<Stream<Integer>>`
*   然后我们用恒等函数把它转换成 T1

## 3。使用 StreamEx

[StreamEx](https://web.archive.org/web/20220930105634/https://github.com/amaembo/streamex) 是一个开源的 Java 库，它扩展了 Java 8 流的可能性。它使用`StreamEx`类作为 JDK 的`Stream`接口的增强。

### 3.1。合并`Stream` s

StreamEx 库允许我们使用`append()`实例方法合并流:

```java
@Test
public void given4Streams_whenMerged_thenResultStreamContainsAllElements() {
    Stream<Integer> stream1 = Stream.of(1, 3, 5);
    Stream<Integer> stream2 = Stream.of(2, 4, 6);
    Stream<Integer> stream3 = Stream.of(18, 15, 36);
    Stream<Integer> stream4 = Stream.of(99);

    Stream<Integer> resultingStream = StreamEx.of(stream1)
      .append(stream2)
      .append(stream3)
      .append(stream4);

    assertEquals(
      Arrays.asList(1, 3, 5, 2, 4, 6, 18, 15, 36, 99),
      resultingStream.collect(Collectors.toList()));
} 
```

因为它是一个实例方法，所以我们可以很容易地将它链接起来并追加多个流。

注意，如果我们将`resultingStream`变量输入到`StreamEx`类型，我们也可以通过使用`toList()`从流中创建一个`List`。

### 3.2。使用`prepend()` 合并流

StreamEx 还包含一个在元素之前添加元素的方法，称为`prepend()`:

```java
@Test
public void given3Streams_whenPrepended_thenResultStreamContainsAllElements() {
    Stream<String> stream1 = Stream.of("foo", "bar");
    Stream<String> openingBracketStream = Stream.of("[");
    Stream<String> closingBracketStream = Stream.of("]");

    Stream<String> resultingStream = StreamEx.of(stream1)
      .append(closingBracketStream)
      .prepend(openingBracketStream);

    assertEquals(
      Arrays.asList("[", "foo", "bar", "]"),
      resultingStream.collect(Collectors.toList()));
} 
```

## 4。使用 Jooλ

[jOOλ](https://web.archive.org/web/20220930105634/https://github.com/jOOQ/jOOL) 是一个兼容 JDK 8 的库，它为 JDK 提供了有用的扩展。这里最重要的流抽象叫做`Seq`。注意，这是一个连续有序的流，所以调用`parallel()`没有任何作用。

### 4.1。合并流

就像 StreamEx 库一样，jOOλ有一个`append()`方法:

```java
@Test
public void given2Streams_whenMerged_thenResultStreamContainsAllElements() {
    Stream<Integer> seq1 = Stream.of(1, 3, 5);
    Stream<Integer> seq2 = Stream.of(2, 4, 6);

    Stream<Integer> resultingSeq = Seq.ofType(seq1, Integer.class)
      .append(seq2);

    assertEquals(
      Arrays.asList(1, 3, 5, 2, 4, 6),
      resultingSeq.collect(Collectors.toList()));
} 
```

此外，如果我们将`resultingSeq`变量键入 jOOλ `Seq`类型，还有一个方便的`toList()`方法。

### 4.2。用`prepend()`和合并溪流

正如所料，由于存在一个`append()`方法，jOOλ中也有一个`prepend()`方法:

```java
@Test
public void given3Streams_whenPrepending_thenResultStreamContainsAllElements() {
    Stream<String> seq = Stream.of("foo", "bar");
    Stream<String> openingBracketSeq = Stream.of("[");
    Stream<String> closingBracketSeq = Stream.of("]");

    Stream<String> resultingStream = Seq.ofType(seq, String.class)
      .append(closingBracketSeq)
      .prepend(openingBracketSeq);

    Assert.assertEquals(
      Arrays.asList("[", "foo", "bar", "]"),
      resultingStream.collect(Collectors.toList()));
} 
```

## 5。结论

我们看到，使用 JDK 8 合并流相对简单。当我们需要进行大量的合并时，为了可读性，使用 StreamEx 或 jOOλ库可能是有益的。

你可以在 GitHub 上找到源代码[。](https://web.archive.org/web/20220930105634/https://github.com/eugenp/tutorials/tree/master/libraries)