# 质子包简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-protonpack>

## 1。概述

在本教程中，我们将看看 [Protonpack](https://web.archive.org/web/20220526042855/https://github.com/poetix/protonpack) 的主要特性，这是一个通过添加一些补充功能来扩展标准 [`Stream` API](https://web.archive.org/web/20220526042855/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#takeWhile(java.util.function.Predicate)) 的库。

请参考[这里的这篇文章](/web/20220526042855/https://www.baeldung.com/java-8-streams-introduction)，了解 Java `Stream` API 的基础知识。

## 2.Maven 依赖性

为了使用 Protonpack 库，我们需要在我们的`pom.xml`文件中添加一个依赖项:

```java
<dependency>
    <groupId>com.codepoetics</groupId>
    <artifactId>protonpack</artifactId>
    <version>1.15</version>
</dependency>
```

在 [Maven Central](https://web.archive.org/web/20220526042855/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.codepoetics%22%20AND%20a%3A%22protonpack%22) 上查看最新版本。

## 3.`StreamUtils`

这是扩展 Java 的标准`Stream` API 的主类。

这里讨论的所有方法都是[中间操作](/web/20220526042855/https://www.baeldung.com/java-8-streams-introduction#operations)，这意味着它们修改了一个`Stream` 但不触发它的处理。

### 3.1.`takeWhile()`和`takeUntil()`

`takeWhile()`从源流**中取值，只要它们满足提供的条件**:

```java
Stream<Integer> streamOfInt = Stream
  .iterate(1, i -> i + 1);
List<Integer> result = StreamUtils
  .takeWhile(streamOfInt, i -> i < 5)
  .collect(Collectors.toList());
assertThat(result).contains(1, 2, 3, 4);
```

相反，`takeUntil()`取值**，直到某个值满足提供的条件**，然后停止:

```java
Stream<Integer> streamOfInt = Stream
  .iterate(1, i -> i + 1);
List<Integer> result = StreamUtils
  .takeUntil(streamOfInt, i -> i >= 5)
  .collect(Collectors.toList());
assertThat(result).containsExactly(1, 2, 3, 4);
```

在 Java 9 以后的版本中，`takeWhile()`是标准 [`Stream` API](https://web.archive.org/web/20220526042855/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#takeWhile(java.util.function.Predicate)) 的一部分。

### 3.2.`zip()`

`zip()`将两个或三个流作为输入和组合器函数。方法**从每个流的相同位置获取一个值，并将其传递给组合器**。

这样做，直到其中一个流的值用完:

```java
String[] clubs = {"Juventus", "Barcelona", "Liverpool", "PSG"};
String[] players = {"Ronaldo", "Messi", "Salah"};
Set<String> zippedFrom2Sources = StreamUtils
  .zip(stream(clubs), stream(players), (club, player) -> club + " " + player)
  .collect(Collectors.toSet());

assertThat(zippedFrom2Sources)
  .contains("Juventus Ronaldo", "Barcelona Messi", "Liverpool Salah"); 
```

类似地，一个重载的`zip()`接受三个源流:

```java
String[] leagues = { "Serie A", "La Liga", "Premier League" };
Set<String> zippedFrom3Sources = StreamUtils
  .zip(stream(clubs), stream(players), stream(leagues), 
    (club, player, league) -> club + " " + player + " " + league)
  .collect(Collectors.toSet());

assertThat(zippedFrom3Sources).contains(
  "Juventus Ronaldo Serie A", 
  "Barcelona Messi La Liga", 
  "Liverpool Salah Premier League");
```

### 3.3。`zipWithIndex()`

`zipWithIndex() ` **取值并压缩每个值及其索引，以创建一个索引值流:**

```java
Stream<String> streamOfClubs = Stream
  .of("Juventus", "Barcelona", "Liverpool");
Set<Indexed<String>> zipsWithIndex = StreamUtils
  .zipWithIndex(streamOfClubs)
  .collect(Collectors.toSet());
assertThat(zipsWithIndex)
  .contains(Indexed.index(0, "Juventus"), Indexed.index(1, "Barcelona"), 
    Indexed.index(2, "Liverpool"));
```

### 3.4。`merge()`

`merge()`使用多个源流和一个组合器。它**从每个源流中获取相同索引位置的值，并将其传递给组合器**。

该方法的工作方式是从`seed`值开始，连续从每个流的相同索引中取 1 个值。

然后，将该值传递给组合器，并将得到的组合值反馈给组合器，以创建下一个值:

```java
Stream<String> streamOfClubs = Stream
  .of("Juventus", "Barcelona", "Liverpool", "PSG");
Stream<String> streamOfPlayers = Stream
  .of("Ronaldo", "Messi", "Salah");
Stream<String> streamOfLeagues = Stream
  .of("Serie A", "La Liga", "Premier League");

Set<String> merged = StreamUtils.merge(
  () ->  "",
  (valOne, valTwo) -> valOne + " " + valTwo,
  streamOfClubs,
  streamOfPlayers,
  streamOfLeagues)
  .collect(Collectors.toSet());

assertThat(merged)
  .contains("Juventus Ronaldo Serie A", "Barcelona Messi La Liga", 
    "Liverpool Salah Premier League", "PSG");
```

### 3.5。`mergeToList()`

`mergeToList()`接受多个流作为输入。它**将来自每个流的相同索引的值组合成一个`List`** :

```java
Stream<String> streamOfClubs = Stream
  .of("Juventus", "Barcelona", "PSG");
Stream<String> streamOfPlayers = Stream
  .of("Ronaldo", "Messi");

Stream<List<String>> mergedStreamOfList = StreamUtils
  .mergeToList(streamOfClubs, streamOfPlayers);
List<List<String>> mergedListOfList = mergedStreamOfList
  .collect(Collectors.toList());

assertThat(mergedListOfList.get(0))
  .containsExactly("Juventus", "Ronaldo");
assertThat(mergedListOfList.get(1))
  .containsExactly("Barcelona", "Messi");
assertThat(mergedListOfList.get(2))
  .containsExactly("PSG");
```

### 3.6。`interleave()`

`interleave()` **使用`selector`** 创建取自多个流的替代值。

该方法给`selector`一个包含每个流中一个值的集合，`selector`将选择一个值。

那么所选择的值将从集合中移除，并用所选择的值所源自的下一个值来替换。这种迭代一直持续到所有源的值都用完为止。

下一个例子使用`interleave() `通过`round-robin`策略创建交替值:

```java
Stream<String> streamOfClubs = Stream
  .of("Juventus", "Barcelona", "Liverpool");
Stream<String> streamOfPlayers = Stream
  .of("Ronaldo", "Messi");
Stream<String> streamOfLeagues = Stream
  .of("Serie A", "La Liga");

List<String> interleavedList = StreamUtils
  .interleave(Selectors.roundRobin(), streamOfClubs, streamOfPlayers, streamOfLeagues)
  .collect(Collectors.toList());

assertThat(interleavedList)
  .hasSize(7)
  .containsExactly("Juventus", "Ronaldo", "Serie A", "Barcelona", "Messi", "La Liga", "Liverpool"); 
```

请注意，上面的代码是出于教程的目的，因为循环法`selector `是由库提供的 [`Selectors.roundRobin()`](https://web.archive.org/web/20220526042855/https://github.com/poetix/protonpack/blob/master/src/main/java/com/codepoetics/protonpack/selectors/Selectors.java) 。

### 3.7。`skipUntil() `和`skipWhile()`

`skipUntil()`跳过值**，直到一个值满足条件**:

```java
Integer[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
List skippedUntilGreaterThan5 = StreamUtils
  .skipUntil(stream(numbers), i -> i > 5)
  .collect(Collectors.toList());

assertThat(skippedUntilGreaterThan5).containsExactly(6, 7, 8, 9, 10); 
```

相反，`skipWhile()` **在值满足条件**时跳过值:

```java
Integer[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
List skippedWhileLessThanEquals5 = StreamUtils
  .skipWhile(stream(numbers), i -> i <= 5 || )
  .collect(Collectors.toList());

assertThat(skippedWhileLessThanEquals5).containsExactly(6, 7, 8, 9, 10); 
```

关于`skipWhile() `很重要的一点是，它会在找到第一个不满足条件的值后继续流式传输:

```java
List skippedWhileGreaterThan5 = StreamUtils
  .skipWhile(stream(numbers), i -> i > 5)
  .collect(Collectors.toList());
assertThat(skippedWhileGreaterThan5).containsExactly(1, 2, 3, 4, 5, 6, 7, 8, 9, 10); 
```

在 Java 9 以后，标准`Stream` API 中的 [`dropWhile` `()`](https://web.archive.org/web/20220526042855/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#dropWhile(java.util.function.Predicate)) 提供与`skipWhile()`相同的功能。

### 3.8。`unfold()`

`unfold()`通过将自定义生成器应用于种子值，然后应用于每个生成的值，生成一个潜在的无限流——可以通过返回`Optional.empty():`来终止该流

```java
Stream<Integer> unfolded = StreamUtils
  .unfold(2, i -> (i < 100) 
    ? Optional.of(i * i) : Optional.empty());

assertThat(unfolded.collect(Collectors.toList()))
  .containsExactly(2, 4, 16, 256);
```

### 3.9。`windowed()`

`windowed()` **创建多个源流子集作为** `**List**. `的流，该方法以源流、`window size`和`skip value`为参数。

`List`长度等于`window` `size, `，而 s `kip value`确定子集相对于前一个子集的开始位置:

```java
Integer[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8 };

List<List> windowedWithSkip1 = StreamUtils
  .windowed(stream(numbers), 3, 1)
  .collect(Collectors.toList());
assertThat(windowedWithSkip1)
  .containsExactly(asList(1, 2, 3), asList(2, 3, 4), asList(3, 4, 5), asList(4, 5, 6), asList(5, 6, 7)); 
```

此外，最后一个窗口保证具有所需的大小，正如我们在下面的示例中看到的:

```java
List<List> windowedWithSkip2 = StreamUtils.windowed(stream(numbers), 3, 2).collect(Collectors.toList());
assertThat(windowedWithSkip2).containsExactly(asList(1, 2, 3), asList(3, 4, 5), asList(5, 6, 7)); 
```

### 3.10。`aggregate()`

有两种截然不同的方法。

第一个`aggregate() ` **根据给定的谓词**将相同值的元素组合在一起:

```java
Integer[] numbers = { 1, 2, 2, 3, 4, 4, 4, 5 };
List<List> aggregated = StreamUtils
  .aggregate(Arrays.stream(numbers), (int1, int2) -> int1.compareTo(int2) == 0)
  .collect(Collectors.toList());
assertThat(aggregated).containsExactly(asList(1), asList(2, 2), asList(3), asList(4, 4, 4), asList(5)); 
```

谓词以连续的方式接收值。因此，如果数字没有排序，上面会给出不同的结果。

另一方面，第二个`aggregate()`简单地用于**将来自源流的元素组合成期望大小的组**:

```java
List<List> aggregatedFixSize = StreamUtils
  .aggregate(stream(numbers), 5)
  .collect(Collectors.toList());
assertThat(aggregatedFixSize).containsExactly(asList(1, 2, 2, 3, 4), asList(4, 4, 5)); 
```

### 3.11。`aggregateOnListCondition()`

`aggregateOnListCondition()`根据谓词和当前活动组对值**进行分组。谓词被赋予当前活动的组作为一个`List`和下一个值。然后，它必须确定该组是否应该继续或开始一个新组。**

以下示例解决了将连续的整数值分组到一个组中的要求，其中每个组中的值之和不得大于 5:

```java
Integer[] numbers = { 1, 1, 2, 3, 4, 4, 5 };
Stream<List<Integer>> aggregated = StreamUtils
  .aggregateOnListCondition(stream(numbers), 
    (currentList, nextInt) -> currentList.stream().mapToInt(Integer::intValue).sum() + nextInt <= 5);
assertThat(aggregated)
  .containsExactly(asList(1, 1, 2), asList(3), asList(4), asList(4), asList(5));
```

## 4。`Streamable<T>`

`Stream `的实例不可重用。因此， **`Streamable`通过包装和公开与`Stream` :** 相同的方法来提供可重用的流

```java
Streamable<String> s = Streamable.of("a", "b", "c", "d");
List<String> collected1 = s.collect(Collectors.toList());
List<String> collected2 = s.collect(Collectors.toList());
assertThat(collected1).hasSize(4);
assertThat(collected2).hasSize(4);
```

## 5.`CollectorUtils`

`CollectorUtils`通过添加几个有用的收集器方法来补充标准`Collectors`。

### 5.1。`maxBy() `和`minBy()`

`maxBy()` **使用提供的投影逻辑**找到流中的最大值:

```java
Stream<String> clubs = Stream.of("Juventus", "Barcelona", "PSG");
Optional<String> longestName = clubs.collect(CollectorUtils.maxBy(String::length));
assertThat(longestName).contains("Barcelona");
```

相反，`minBy()` **使用提供的投影逻辑**找到最小值。

### 5.2。`unique()`

`unique()`收集器做一件非常简单的事情:**如果一个给定的流正好有 1 个元素，它返回唯一的值:**

```java
Stream<Integer> singleElement = Stream.of(1);
Optional<Integer> unique = singleElement.collect(CollectorUtils.unique());
assertThat(unique).contains(1); 
```

否则，`unique()`将抛出一个异常:

```java
Stream multipleElement = Stream.of(1, 2, 3);
assertThatExceptionOfType(NonUniqueValueException.class).isThrownBy(() -> {
    multipleElement.collect(CollectorUtils.unique());
}); 
```

## 6。结论

在本文中，我们了解了 Protonpack 库如何扩展 Java Stream API 以使其更易于使用。它添加了我们可能经常使用但在标准 API 中没有的有用方法。

从 Java 9 开始，Protonpack 提供的一些功能将在标准流 API 中可用。

像往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20220526042855/https://github.com/eugenp/tutorials/tree/master/libraries-6)