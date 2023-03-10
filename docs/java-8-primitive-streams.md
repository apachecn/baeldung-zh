# Java 8 中的原始类型流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-primitive-streams>

## 1。简介

流 API 是 Java 8 中添加的关键特性之一。

简而言之，通过提供一个声明性的 API，API 允许我们更方便、更有效地处理集合和其他元素序列。

## 2。原始流

流主要处理对象的集合，而不是原始类型。

幸运的是，为了提供一种方法来处理三种最常用的原语类型——`int, long`和`double`——标准库包括了三种原语专用的实现:`IntStream`、`LongStream,`和`DoubleStream`。

原语流之所以受到限制，主要是因为装箱开销以及为其他原语创建专用流在许多情况下并不那么有用。

## 3。算术运算

让我们从一些常用算术运算的有趣方法开始，比如`min`、`max`、`sum`和`average:`

```java
int[] integers = new int[] {20, 98, 12, 7, 35};
int min = Arrays.stream(integers)
  .min()
  .getAsInt(); // returns 7
```

现在让我们一步一步地看上面的代码片段，以了解发生了什么。

我们通过使用`java.util.Arrays.stream(int[])`来创建我们的`IntStream`，然后使用`min()`方法来获得最小的整数作为`java.util.OptionalInt`，最后调用`getAsInt()`来获得`int`值。

创建`IntStream`的另一种方法是使用`IntStream.of(int…)`。`max()`方法将返回最大的整数:

```java
int max = IntStream.of(20, 98, 12, 7, 35)
  .max()
  .getAsInt(); // returns 98
```

接下来——要得到整数的和，我们只需调用`sum()`方法，我们不需要使用`getAsInt()`,因为它已经将结果作为`int`值返回:

```java
int sum = IntStream.of(20, 98, 12, 7, 35).sum(); // returns 172
```

我们调用`average()`方法来获得整数值的平均值，正如我们所看到的，我们应该使用`getAsDouble()`,因为它返回一个`double`类型的值。

```java
double avg = IntStream.of(20, 98, 12, 7, 35)
  .average()
  .getAsDouble(); // returns 34.4
```

## 4。范围

我们还可以基于一个范围创建一个`IntStream`:

```java
int sum = IntStream.range(1, 10)
  .sum(); // returns 45
int sum = IntStream.rangeClosed(1, 10)
  .sum(); // returns 55
```

如上面的代码片段所示，有两种方法可以创建一系列整数值`range()`和`rangeClosed()`。

不同的是`range()`的结尾是独占的，而在`rangeClosed()`中是包含的。

范围方法仅适用于`IntStream`和`LongStream`。

我们可以使用 range 作为 for-each 循环的一种奇特形式:

```java
IntStream.rangeClosed(1, 5)
  .forEach(System.out::println);
```

将它们用作 for-each 循环替换的好处是，我们还可以利用并行执行:

```java
IntStream.rangeClosed(1, 5)
  .parallel()
  .forEach(System.out::println);
```

尽管这些花哨的循环很有帮助，但对于简单的迭代，使用传统的 for 循环比使用函数循环更好，因为在某些情况下更简单、可读性更好、性能更好。

## 5。装箱和拆箱

有时候，我们需要将原始值转换成它们的包装等价物。

在那些情况下，我们可以使用`boxed()` 方法:

```java
List<Integer> evenInts = IntStream.rangeClosed(1, 10)
  .filter(i -> i % 2 == 0)
  .boxed()
  .collect(Collectors.toList());
```

我们还可以从包装类流转换到原始流:

```java
// returns 78
int sum = Arrays.asList(33,45)
  .stream()
  .mapToInt(i -> i)
  .sum();
```

我们总是可以使用`mapToXxx`和`flatMapToXxx`方法来创建原始流。

## 6。结论

Java Streams 是对该语言的一个非常强大的补充。在这里，我们仅仅触及了原始流的表面，但是，您已经可以使用它们进行生产了。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220811145435/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)