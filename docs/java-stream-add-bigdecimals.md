# 使用流 API 添加大小数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-add-bigdecimals>

## 1.概观

我们通常使用 Java [流 API](/web/20220628125819/https://www.baeldung.com/java-8-streams-introduction) 来处理数据集合。

一个很好的特性是支持对数字流的操作，比如`sum` 操作。然而，**我们不能以这种方式处理所有的数字类型。**

在本教程中，我们将看到如何对像`BigDecimal`这样的数字流执行`sum`操作。

## 2.我们通常如何对一个流求和

Stream API 提供了数字流，包括`IntStream, DoubleStream,` 和 `LongStream.`

让我们通过创建一个数字流来提醒自己这些是如何工作的。然后，我们用`[IntStream#sum](/web/20220628125819/https://www.baeldung.com/java-intstream-convert)`计算它的总数:

```java
IntStream intNumbers = IntStream.range(0, 3);
assertEquals(3, intNumbers.sum());
```

我们可以从一个`Double`列表开始做类似的事情。通过使用 streams，我们可以使用`mapToDouble`从一个对象流转换成一个`DoubleStream`:

```java
List<Double> doubleNumbers = Arrays.asList(23.48, 52.26, 13.5);
double result = doubleNumbers.stream()
    .mapToDouble(Double::doubleValue)
    .sum();
assertEquals(89.24, result, .1);
```

因此，如果我们能以同样的方式对一组`BigDecimal` 数字求和，那将是非常有用的。

**不幸的是，没有一个`BigDecimalStream.`** 所以，我们需要另一个解决方案。

## 3.使用 Reduce 将`BigDecimal`数字相加

不依赖`sum`，我们可以用 *[Stream.reduce](/web/20220628125819/https://www.baeldung.com/java-stream-reduce) :*

```java
Stream<Integer> intNumbers = Stream.of(5, 1, 100);
int result = intNumbers.reduce(0, Integer::sum);
assertEquals(106, result);
```

**这适用于任何可以逻辑相加的东西**，包括`BigDecimal`:

```java
Stream<BigDecimal> bigDecimalNumber = 
  Stream.of(BigDecimal.ZERO, BigDecimal.ONE, BigDecimal.TEN);
BigDecimal result = bigDecimalNumber.reduce(BigDecimal.ZERO, BigDecimal::add);
assertEquals(11, result);
```

`reduce`方法有两个参数:

*   `Identity` –相当于`0 `–是减少的起始值
*   `Accumulator function`–接受两个参数，目前的结果和流的下一个元素

## 4.结论

在本文中，我们研究了如何找到数字流中一些数字的和。然后我们发现了如何使用*减少*作为替代。

使用`reduce`允许我们对一组`BigDecimal`数求和。它可以应用于任何其他类型。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628125819/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)