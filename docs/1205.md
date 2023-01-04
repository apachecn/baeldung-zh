# 将 Java 流过滤为 1 个且仅 1 个元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filter-stream-unique-element>

## 1.概观

在本文中，我们将使用来自`[Collectors](/web/20220913124916/https://www.baeldung.com/java-8-collectors)`的两种方法来检索与给定元素流中的某个谓词相匹配的惟一元素。

对于这两种方法，我们将根据以下标准定义两种方法:

*   get 方法期望得到一个唯一的结果。否则，它抛出一个`[Exception](https://web.archive.org/web/20220913124916/https://baeldung.com/java-exceptions)`
*   find 方法接受结果可能会丢失，并返回一个带有值的`[Optional](/web/20220913124916/https://www.baeldung.com/java-optional)`(如果存在的话)

## 2.使用归约检索唯一结果

**`Collectors.reducing`执行其输入元素的缩减。**为此，它应用了一个被指定为`[BinaryOperator](/web/20220913124916/https://www.baeldung.com/java-bifunction-interface)`的函数。结果被描述为一个`Optional`。因此，我们可以定义我们的查找方法。

在我们的例子中，如果在[过滤](/web/20220913124916/https://www.baeldung.com/java-stream-filter-lambda)之后有两个或更多的元素，我们只需要丢弃结果:

```
public static <T> Optional<T> findUniqueElementMatchingPredicate_WithReduction(Stream<T> elements, Predicate<T> predicate) {
    return elements.filter(predicate)
      .collect(Collectors.reducing((a, b) -> null));
}
```

要编写 get 方法，我们需要进行以下更改:

*   如果我们检测到两个元素，我们可以直接抛出它们，而不是返回 null
*   最后，我们需要得到`Optional` : [的值，如果它是空的，我们也要抛出](/web/20220913124916/https://www.baeldung.com/java-optional-throw-exception)

此外，在这种情况下，**我们可以直接对** [`**Stream**`](/web/20220913124916/https://www.baeldung.com/java-8-streams) **:** 应用[减](/web/20220913124916/https://www.baeldung.com/java-stream-reduce)操作

```
public static <T> T getUniqueElementMatchingPredicate_WithReduction(Stream<T> elements, Predicate<T> predicate) {
    return elements.filter(predicate)
      .reduce((a, b) -> {
          throw new IllegalStateException("Too many elements match the predicate");
      })
      .orElseThrow(() -> new IllegalStateException("No element matches the predicate"));
}
```

## 3.使用`Collectors.collectingAndThen`检索唯一结果

**`Collectors.collectingAndThen`对采集操作的结果`List`应用函数。**

因此，为了定义 find 方法，我们需要使用`List`和:

*   如果`List`有零个或者两个以上的元素，返回`null`
*   如果`List`只有一个元素，返回它

以下是该操作的代码:

```
private static <T> T findUniqueElement(List<T> elements) {
    if (elements.size() == 1) {
        return elements.get(0);
    }
    return null;
}
```

因此，find 方法读取:

```
public static <T> Optional<T> findUniqueElementMatchingPredicate_WithCollectingAndThen(Stream<T> elements, Predicate<T> predicate) {
    return elements.filter(predicate)
      .collect(Collectors.collectingAndThen(Collectors.toList(), list -> Optional.ofNullable(findUniqueElement(list))));
}
```

为了使我们的私有方法适用于 get 情况，如果检索到的元素数量不正好是 1，我们需要抛出。让我们精确地区分没有结果和有太多结果的情况，就像我们对 reduction 所做的那样:

```
private static <T> T getUniqueElement(List<T> elements) {
    if (elements.size() > 1) {
        throw new IllegalStateException("Too many elements match the predicate");
    } else if (elements.size() == 0) {
        throw new IllegalStateException("No element matches the predicate");
    }
    return elements.get(0);
}
```

最后，假设我们将类命名为`FilterUtils`，我们可以编写 get 方法:

```
public static <T> T getUniqueElementMatchingPredicate_WithCollectingAndThen(Stream<T> elements, Predicate<T> predicate) {
    return elements.filter(predicate)
      .collect(Collectors.collectingAndThen(Collectors.toList(), FilterUtils::getUniqueElement));
}
```

## 4.性能基准

**让我们使用 [JMH](/web/20220913124916/https://www.baeldung.com/java-microbenchmark-harness) 在不同的方法之间进行一个快速的性能比较。**

首先，让我们将我们的方法应用于

*   一个 [`Stream`包含了从 1 到 100 万的所有`Integers`](/web/20220913124916/https://www.baeldung.com/java-intstream-convert)
*   一个 [`Predicate`](/web/20220913124916/https://www.baeldung.com/java-predicate-chain) 验证一个元素是否等于 751879

在这种情况下，将针对`Stream`的一个唯一元素来验证`Predicate`。让我们来看看`Benchmark`的定义:

```
@State(Scope.Benchmark)
public static class MyState {
    final Stream<Integer> getIntegers() { 
        return IntStream.range(1, 1000000).boxed();
    }

    final Predicate<Integer> PREDICATE = i -> i == 751879;
}

@Benchmark
public void evaluateFindUniqueElementMatchingPredicate_WithReduction(Blackhole blackhole, MyState state) {
    blackhole.consume(FilterUtils.findUniqueElementMatchingPredicate_WithReduction(state.INTEGERS.stream(), state.PREDICATE));
}

@Benchmark
public void evaluateFindUniqueElementMatchingPredicate_WithCollectingAndThen(Blackhole blackhole, MyState state) {
    blackhole.consume(FilterUtils.findUniqueElementMatchingPredicate_WithCollectingAndThen(state.INTEGERS.stream(), state.PREDICATE));
}

@Benchmark
public void evaluateGetUniqueElementMatchingPredicate_WithReduction(Blackhole blackhole, MyState state) {
    try {
        FilterUtils.getUniqueElementMatchingPredicate_WithReduction(state.INTEGERS.stream(), state.PREDICATE);
    } catch (IllegalStateException exception) {
        blackhole.consume(exception);
    }
}

@Benchmark
public void evaluateGetUniqueElementMatchingPredicate_WithCollectingAndThen(Blackhole blackhole, MyState state) {
    try {
        FilterUtils.getUniqueElementMatchingPredicate_WithCollectingAndThen(state.INTEGERS.stream(), state.PREDICATE);
    } catch (IllegalStateException exception) {
        blackhole.consume(exception);
    }
}
```

让我们运行它。我们正在测量每秒的运算次数。越高越好:

```
Benchmark                                                                          Mode  Cnt    Score    Error  Units
BenchmarkRunner.evaluateFindUniqueElementMatchingPredicate_WithCollectingAndThen  thrpt   25  140.581 ± 28.793  ops/s
BenchmarkRunner.evaluateFindUniqueElementMatchingPredicate_WithReduction          thrpt   25  100.171 ± 36.796  ops/s
BenchmarkRunner.evaluateGetUniqueElementMatchingPredicate_WithCollectingAndThen   thrpt   25  145.568 ±  5.333  ops/s
BenchmarkRunner.evaluateGetUniqueElementMatchingPredicate_WithReduction           thrpt   25  144.616 ± 12.917  ops/s
```

正如我们所看到的，在这种情况下，不同的方法表现非常相似。

让我们改变我们的`Predicate` 来检查`Stream`的一个元素是否等于 0。这个条件对于`List`的所有元素都是假的。我们现在可以再次运行基准测试:

```
Benchmark                                                                          Mode  Cnt    Score    Error  Units
BenchmarkRunner.evaluateFindUniqueElementMatchingPredicate_WithCollectingAndThen  thrpt   25  165.751 ± 19.816  ops/s
BenchmarkRunner.evaluateFindUniqueElementMatchingPredicate_WithReduction          thrpt   25  174.667 ± 20.909  ops/s
BenchmarkRunner.evaluateGetUniqueElementMatchingPredicate_WithCollectingAndThen   thrpt   25  188.293 ± 18.348  ops/s
BenchmarkRunner.evaluateGetUniqueElementMatchingPredicate_WithReduction           thrpt   25  196.689 ±  4.155  ops/s
```

在这里，性能图表是相当平衡的。

最后，让我们看看如果我们使用一个对大于 751879 的值返回 true 的`Predicate`会发生什么:有大量的`List`元素匹配这个`Predicate`。这导致了以下基准:

```
Benchmark                                                                          Mode  Cnt    Score    Error  Units
BenchmarkRunner.evaluateFindUniqueElementMatchingPredicate_WithCollectingAndThen  thrpt   25   70.879 ±  6.205  ops/s
BenchmarkRunner.evaluateFindUniqueElementMatchingPredicate_WithReduction          thrpt   25  210.142 ± 23.680  ops/s
BenchmarkRunner.evaluateGetUniqueElementMatchingPredicate_WithCollectingAndThen   thrpt   25   83.927 ±  1.812  ops/s
BenchmarkRunner.evaluateGetUniqueElementMatchingPredicate_WithReduction           thrpt   25  252.881 ±  2.710  ops/s
```

正如我们所看到的，简化的变体更有效。此外，直接在过滤后的`Stream`上使用`reduce`会更好，因为在找到两个匹配值后会直接抛出`Exception`。

简而言之，如果性能是一个问题:

*   **应优先使用还原**
*   如果我们期望找到很多潜在的匹配值，那么减少`Stream`的 get 方法要快得多

## 5.结论

在本教程中，我们看到了在过滤一个`Stream`后检索唯一结果的不同方法，然后比较了它们的效率。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220913124916/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-4)