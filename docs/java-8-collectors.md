# Java 8 的收集器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-collectors>

## 1。概述

在本教程中，我们将浏览 Java 8 的收集器，它们在处理`Stream`的最后一步使用。

要阅读更多关于`Stream` API 本身的内容，我们可以查看[这篇文章](/web/20220629010735/https://www.baeldung.com/java-8-streams)。

如果我们想看看如何利用收集器的能力进行并行处理，我们可以看看[这个项目。](https://web.archive.org/web/20220629010735/https://github.com/pivovarit/parallel-collectors)

## 延伸阅读:

## [Java 8 Stream API 教程](/web/20220629010735/https://www.baeldung.com/java-8-streams)

The article is an example-heavy introduction of the possibilities and operations offered by the Java 8 Stream API.[Read more](/web/20220629010735/https://www.baeldung.com/java-8-streams) →

## [Java 8 Collector 分组指南](/web/20220629010735/https://www.baeldung.com/java-groupingby-collector)

A guide to Java 8 groupingBy Collector with usage examples.[Read more](/web/20220629010735/https://www.baeldung.com/java-groupingby-collector) →

## [Java 9 中的新流收集器](/web/20220629010735/https://www.baeldung.com/java9-stream-collectors)

In this article, we explore new Stream collectors that were introduced in JDK 9[Read more](/web/20220629010735/https://www.baeldung.com/java9-stream-collectors) →

## 2。`Stream.collect()`法

`Stream.collect()`是 Java 8 的`Stream API`的终端方法之一。它允许我们执行可变的折叠操作(将元素重新打包到一些数据结构中，并应用一些额外的逻辑，将它们连接起来，等等)。)在一个`Stream`实例中保存的数据元素上。

该操作的策略通过`Collector`接口实现提供。

## 3。`Collectors`

所有预定义的实现都可以在`Collectors`类中找到。通常的做法是对它们使用以下静态导入来提高可读性:

```java
import static java.util.stream.Collectors.*;
```

我们也可以使用我们选择的单一导入收集器:

```java
import static java.util.stream.Collectors.toList;
import static java.util.stream.Collectors.toMap;
import static java.util.stream.Collectors.toSet;
```

在下面的例子中，我们将重用下面的列表:

```java
List<String> givenList = Arrays.asList("a", "bb", "ccc", "dd");
```

### 3.1。`Collectors.toList()`

`toList`收集器可以用来将所有的`Stream`元素收集到一个`List`实例中。需要记住的重要一点是，我们不能用这种方法假设任何特定的`List`实现。如果我们想对此有更多的控制，我们可以使用`toCollection` 来代替。

让我们创建一个代表元素序列的`Stream`实例，然后将它们收集到一个`List`实例中:

```java
List<String> result = givenList.stream()
  .collect(toList());
```

#### 3.1.1。`Collectors.toUnmodifiableList()`

Java 10 引入了一种便捷的方式将`Stream`元素累积到一个[不可修改的`List`T3 中:](https://web.archive.org/web/20220629010735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#unmodifiable)

```java
List<String> result = givenList.stream()
  .collect(toUnmodifiableList());
```

现在如果我们试图修改`result` `List`，我们会得到一个`UnsupportedOperationException`:

```java
assertThatThrownBy(() -> result.add("foo"))
  .isInstanceOf(UnsupportedOperationException.class);
```

### 3.2。`Collectors.toSet()`

`toSet`收集器可以用来将所有的`Stream`元素收集到一个`Set`实例中。需要记住的重要一点是，我们不能用这种方法假设任何特定的`Set`实现。如果我们想对此有更多的控制，我们可以使用`toCollection` 来代替。

让我们创建一个代表元素序列的`Stream`实例，然后将它们收集到一个`Set`实例中:

```java
Set<String> result = givenList.stream()
  .collect(toSet());
```

一个`Set`不包含重复元素。如果我们的集合包含彼此相等的元素，它们在结果`Set`中只出现一次:

```java
List<String> listWithDuplicates = Arrays.asList("a", "bb", "c", "d", "bb");
Set<String> result = listWithDuplicates.stream().collect(toSet());
assertThat(result).hasSize(4);
```

#### 3.2.1。`Collectors.toUnmodifiableSet()`

从 Java 10 开始，我们可以使用`toUnmodifiableSet()`收集器轻松创建一个[不可修改的`Set`T3:](https://web.archive.org/web/20220629010735/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html#unmodifiable)

```java
Set<String> result = givenList.stream()
  .collect(toUnmodifiableSet());
```

任何修改`result Set`的尝试都将以`UnsupportedOperationException`结束:

```java
assertThatThrownBy(() -> result.add("foo"))
  .isInstanceOf(UnsupportedOperationException.class);
```

### 3.3。`Collectors.toCollection()`

正如我们已经提到的，当使用`toSet` 和 `toList`收集器时，我们不能对它们的实现做任何假设。如果我们想使用定制的实现，我们需要使用`toCollection`收集器和我们选择的集合。

让我们创建一个代表元素序列的`Stream`实例，然后将它们收集到一个`LinkedList`实例中:

```java
List<String> result = givenList.stream()
  .collect(toCollection(LinkedList::new))
```

请注意，这不适用于任何不可变集合。在这种情况下，我们需要编写一个定制的`Collector`实现或者使用`collectingAndThen`。

### 3.4。`Collectors`。`toMap()`

`toMap` 收集器可以用来将`Stream`元素收集到一个`Map`实例中。为此，我们需要提供两个函数:

*   按键映射器
*   值映射器

我们将使用`keyMapper` 从 `Stream`元素中提取一个 `Map`键，并使用 `valueMapper` 提取一个与给定键相关的值。

让我们将这些元素收集到一个`Map`中，它将字符串存储为键，将它们的长度存储为值:

```java
Map<String, Integer> result = givenList.stream()
  .collect(toMap(Function.identity(), String::length))
```

`Function.identity()`只是定义一个接受并返回相同值的函数的快捷方式。

那么如果我们的集合包含重复的元素会发生什么呢？与`toSet`相反，`toMap`不静默地过滤重复项，这是可以理解的，因为它如何计算出为这个键选择哪个值呢？

```java
List<String> listWithDuplicates = Arrays.asList("a", "bb", "c", "d", "bb");
assertThatThrownBy(() -> {
    listWithDuplicates.stream().collect(toMap(Function.identity(), String::length));
}).isInstanceOf(IllegalStateException.class);
```

注意`toMap`甚至不评估值是否也相等。如果它看到重复的键，它会立即抛出一个`IllegalStateException`。

在这种密钥冲突的情况下，我们应该将`toMap`与另一个签名一起使用:

```java
Map<String, Integer> result = givenList.stream()
  .collect(toMap(Function.identity(), String::length, (item, identicalItem) -> item));
```

这里的第三个参数是一个`BinaryOperator`，在这里我们可以指定我们希望如何处理冲突。在这种情况下，我们将只选择这两个冲突值中的任何一个，因为我们知道相同的字符串也将总是具有相同的长度。

#### 3.4.1。`Collectors.toUnmodifiableMap()`

与`List` s 和`Set` s 类似，Java 10 引入了一种简单的方法将`Stream`元素收集到一个[不可修改的`Map`T5:](https://web.archive.org/web/20220629010735/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Map.html#unmodifiable)

```java
Map<String, Integer> result = givenList.stream()
  .collect(toMap(Function.identity(), String::length))
```

正如我们所看到的，如果我们试图在一个`result Map`中放入一个新的条目，我们将得到一个`UnsupportedOperationException`:

```java
assertThatThrownBy(() -> result.put("foo", 3))
  .isInstanceOf(UnsupportedOperationException.class);
```

### 3.5。`Collectors`。c*collectingandthen()*

`CollectingAndThen`是一个特殊的收集器，允许我们在收集结束后立即对结果执行另一个操作。

让我们将`Stream`元素收集到一个`List`实例中，然后将结果转换成一个`ImmutableList`实例:

```java
List<String> result = givenList.stream()
  .collect(collectingAndThen(toList(), ImmutableList::copyOf))
```

### 3.6。`Collectors`。j*oining()*

`Joining`收集器可用于连接`Stream<String>`元件。

我们可以通过以下方式将它们结合在一起:

```java
String result = givenList.stream() .collect(joining()); 
```

这将导致:

```java
"abbcccdd"
```

我们还可以指定自定义分隔符、前缀、后缀:

```java
String result = givenList.stream()
  .collect(joining(" "));
```

这将导致:

```java
"a bb ccc dd"
```

我们也可以写:

```java
String result = givenList.stream()
  .collect(joining(" ", "PRE-", "-POST"));
```

这将导致:

```java
"PRE-a bb ccc dd-POST"
```

### 3.7。`Collectors`。c *ounting()*

`Counting`是一个简单的收集器，允许对所有`Stream`元素进行计数。

现在我们可以写:

```java
Long result = givenList.stream()
  .collect(counting());
```

### 3.8。`Collectors`。s *总结 double/Long/Int()*

`SummarizingDouble/Long/Int`是一个收集器，它返回一个特殊的类，该类包含有关提取元素的 `Stream`中的数字数据的统计信息。

我们可以通过以下方式获得字符串长度的信息:

```java
DoubleSummaryStatistics result = givenList.stream()
  .collect(summarizingDouble(String::length));
```

在这种情况下，以下情况成立:

```java
assertThat(result.getAverage()).isEqualTo(2);
assertThat(result.getCount()).isEqualTo(4);
assertThat(result.getMax()).isEqualTo(3);
assertThat(result.getMin()).isEqualTo(1);
assertThat(result.getSum()).isEqualTo(8);
```

### 3.9。`Collectors.averagingDouble/Long/Int()`

`AveragingDouble/Long/Int`是一个收集器，它只返回提取元素的平均值。

我们可以通过以下方式获得平均字符串长度:

```java
Double result = givenList.stream()
  .collect(averagingDouble(String::length));
```

### 3.10。`Collectors`。s*umming double/Long/Int()*

`SummingDouble/Long/Int`是一个收集器，它只返回提取元素的总和。

我们可以通过以下方式获得所有字符串长度的总和:

```java
Double result = givenList.stream()
  .collect(summingDouble(String::length));
```

### 3.11。`Collectors.maxBy()/minBy()`

`MaxBy` / `MinBy`收集器根据提供的`Comparator`实例返回一个`Stream`的最大/最小元素。

我们可以通过以下方式选择最大的元素:

```java
Optional<String> result = givenList.stream()
  .collect(maxBy(Comparator.naturalOrder()));
```

我们可以看到返回值被包装在一个`Optional`实例中。这迫使用户重新考虑空集合的情况。

### 3.12。`Collectors`。`groupingBy()`

`GroupingBy` collector 用于根据一些属性对对象进行分组，然后将结果存储在一个`Map`实例中。

我们可以根据字符串长度对它们进行分组，并将分组结果存储在`Set`实例中:

```java
Map<Integer, Set<String>> result = givenList.stream()
  .collect(groupingBy(String::length, toSet()));
```

这将导致以下情况成立:

```java
assertThat(result)
  .containsEntry(1, newHashSet("a"))
  .containsEntry(2, newHashSet("bb", "dd"))
  .containsEntry(3, newHashSet("ccc")); 
```

我们可以看到,`groupingBy`方法的第二个参数是一个`Collector.`。此外，我们可以自由选择使用任何一个`Collector`。

### 3.13。`Collectors.partitioningBy()`

`PartitioningBy`是`groupingBy`的一个特例，它接受一个`Predicate`实例，然后将`Stream`元素收集到一个`Map`实例中，该实例将`Boolean`值存储为键，将集合存储为值。在“true”键下，我们可以找到与给定的`Predicate`匹配的元素集合，在“false”键下，我们可以找到与给定的`Predicate`不匹配的元素集合。

我们可以写:

```java
Map<Boolean, List<String>> result = givenList.stream()
  .collect(partitioningBy(s -> s.length() > 2))
```

这将生成包含以下内容的地图:

```java
{false=["a", "bb", "dd"], true=["ccc"]} 
```

### 3.14。`Collectors.teeing()`

让我们使用到目前为止所学的收集器，从给定的`Stream`中找出最大和最小的数字:

```java
List<Integer> numbers = Arrays.asList(42, 4, 2, 24);
Optional<Integer> min = numbers.stream().collect(minBy(Integer::compareTo));
Optional<Integer> max = numbers.stream().collect(maxBy(Integer::compareTo));
// do something useful with min and max
```

在这里，我们使用两个不同的收集器，然后结合这两个收集器的结果来创建一些有意义的东西。在 Java 12 之前，为了覆盖这样的用例，我们必须对给定的`Stream`进行两次操作，将中间结果存储到临时变量中，然后在之后组合这些结果。

幸运的是，Java 12 提供了一个内置的收集器，代表我们处理这些步骤；我们所要做的就是提供两个收集器和组合器功能。

**由于这种新的收集器[将](https://web.archive.org/web/20220629010735/https://en.wikipedia.org/wiki/Tee_(command))给定的水流转向两个不同的方向，故称之为`teeing:`**

```java
numbers.stream().collect(teeing(
  minBy(Integer::compareTo), // The first collector
  maxBy(Integer::compareTo), // The second collector
  (min, max) -> // Receives the result from those collectors and combines them
));
```

这个例子可以在 GitHub 的 [core-java-12](https://web.archive.org/web/20220629010735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-12) 项目中找到。

## 4。自定义收集器

如果我们想编写自己的收集器实现，我们需要实现收集器接口，并指定它的三个通用参数:

```java
public interface Collector<T, A, R> {...}
```

1.  **T**–可用于收集的对象类型
2.  **A**–可变累加器对象的类型
3.  **R**–最终结果的类型

让我们编写一个示例收集器，将元素收集到一个`ImmutableSet`实例中。我们从指定正确的类型开始:

```java
private class ImmutableSetCollector<T>
  implements Collector<T, ImmutableSet.Builder<T>, ImmutableSet<T>> {...}
```

由于我们需要一个可变集合来进行内部集合操作处理，所以我们不能使用`ImmutableSet`。相反，我们需要使用一些其他的可变集合，或者任何其他可以临时为我们积累对象的类。在这种情况下，我们将使用一个`ImmutableSet.Builder`，现在我们需要实现 5 个方法:

*   `Supplier<ImmutableSet.Builder<T>> **supplier**()`
*   `BiConsumer<ImmutableSet.Builder<T>, T> **accumulator**()`
*   `BinaryOperator<ImmutableSet.Builder<T>> **combiner**()`
*   `Function<ImmutableSet.Builder<T>, ImmutableSet<T>> **finisher**()`
*   `Set<Characteristics> **characteristics**()`

**`The supplier()`** 方法返回生成空累加器实例的`Supplier`实例。所以在这种情况下，我们可以简单地写:

```java
@Override
public Supplier<ImmutableSet.Builder<T>> supplier() {
    return ImmutableSet::builder;
} 
```

**`The accumulator()`** 方法返回一个函数，用于向现有的`accumulator`对象添加新元素。所以让我们使用`Builder`的`add`方法:

```java
@Override
public BiConsumer<ImmutableSet.Builder<T>, T> accumulator() {
    return ImmutableSet.Builder::add;
}
```

`**The combiner()**` 方法返回一个用于合并两个累加器的函数:

```java
@Override
public BinaryOperator<ImmutableSet.Builder<T>> combiner() {
    return (left, right) -> left.addAll(right.build());
}
```

`**The finisher()**`方法返回一个函数，用于将累加器转换为最终结果类型。所以在这种情况下，我们将只使用`Builder`的`build`方法:

```java
@Override
public Function<ImmutableSet.Builder<T>, ImmutableSet<T>> finisher() {
    return ImmutableSet.Builder::build;
}
```

`**The characteristics()**`方法用于为流提供一些附加信息，这些信息将用于内部优化。在这种情况下，我们不注意元素在`Set`中的顺序，因为我们将使用`Characteristics.UNORDERED`。要获得关于这个主题的更多信息，请查看`Characteristics` ' JavaDoc:

```java
@Override public Set<Characteristics> characteristics() {
    return Sets.immutableEnumSet(Characteristics.UNORDERED);
}
```

下面是完整的实现和用法:

```java
public class ImmutableSetCollector<T>
  implements Collector<T, ImmutableSet.Builder<T>, ImmutableSet<T>> {

@Override
public Supplier<ImmutableSet.Builder<T>> supplier() {
    return ImmutableSet::builder;
}

@Override
public BiConsumer<ImmutableSet.Builder<T>, T> accumulator() {
    return ImmutableSet.Builder::add;
}

@Override
public BinaryOperator<ImmutableSet.Builder<T>> combiner() {
    return (left, right) -> left.addAll(right.build());
}

@Override
public Function<ImmutableSet.Builder<T>, ImmutableSet<T>> finisher() {
    return ImmutableSet.Builder::build;
}

@Override
public Set<Characteristics> characteristics() {
    return Sets.immutableEnumSet(Characteristics.UNORDERED);
}

public static <T> ImmutableSetCollector<T> toImmutableSet() {
    return new ImmutableSetCollector<>();
}
```

最后，实际上:

```java
List<String> givenList = Arrays.asList("a", "bb", "ccc", "dddd");

ImmutableSet<String> result = givenList.stream()
  .collect(toImmutableSet());
```

## 5。结论

在本文中，我们深入探讨了 Java 8 的*收集器、*，并展示了如何实现一个收集器。一定要去[看看我的一个项目，它增强了 Java 并行处理的能力。](https://web.archive.org/web/20220629010735/https://github.com/pivovarit/parallel-collectors)

所有代码示例都可以在 [GitHub](https://web.archive.org/web/20220629010735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2) 上找到。更多有趣的文章可以在我的网站阅读[。](https://web.archive.org/web/20220629010735/http://4comprehension.com/)