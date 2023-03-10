# Stream.reduce()指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-reduce>

## 1.概观

[Stream API](/web/20221208143903/https://www.baeldung.com/java-8-streams-introduction) 提供了丰富的中间函数、归约函数和终端函数，也支持并行化。

更具体地说，**归约流操作允许我们通过对序列中的元素重复应用组合操作，从一系列元素**中产生一个结果。

在本教程中，**我们将看看通用的 [`Stream.reduce()`](https://web.archive.org/web/20221208143903/https://docs.oracle.com/javase/tutorial/collections/streams/reduction.html) 操作**，并在一些具体的用例中看到它。

## 延伸阅读:

## [用 Java 流对数字求和](/web/20221208143903/https://www.baeldung.com/java-stream-sum)

A quick and practical guide to summing numbers with Java Stream API.[Read more](/web/20221208143903/https://www.baeldung.com/java-stream-sum) →

## [Java 8 流简介](/web/20221208143903/https://www.baeldung.com/java-8-streams-introduction)

A quick and practical introduction to Java 8 Streams.[Read more](/web/20221208143903/https://www.baeldung.com/java-8-streams-introduction) →

## [Java 双功能接口指南](/web/20221208143903/https://www.baeldung.com/java-bifunction-interface)

Learn some common patterns for Java functional interfaces that take two parameters.[Read more](/web/20221208143903/https://www.baeldung.com/java-bifunction-interface) →

## 2。关键概念:恒等式、累加器和合并器

在我们深入研究如何使用`Stream.reduce()`操作之前，让我们将操作的参与者元素分解成单独的块。这样，我们将更容易理解每个人所扮演的角色。

*   `Identity`–一个元素，是归约操作的初始值，如果流为空，则为默认结果
*   `Accumulator`–采用两个参数的函数:归约运算的部分结果和流的下一个元素
*   `Combiner`–当归约被并行化或累加器参数类型与累加器实现类型不匹配时，用于组合归约操作的部分结果的函数

## 3。使用`Stream.reduce()`

为了更好地理解 identity、accumulator 和 combiner 元素的功能，让我们看一些基本的例子:

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
int result = numbers
  .stream()
  .reduce(0, (subtotal, element) -> subtotal + element);
assertThat(result).isEqualTo(21);
```

在这种情况下，**的`Integer`值为 0 就是身份。**存储归约操作的初始值，以及`Integer`值流为空时的默认结果。

同样，**λ表达式**:

```java
subtotal, element -> subtotal + element
```

**是累加器**，因为它取`Integer`值和流中下一个元素的部分和。

为了使代码更加简洁，我们可以使用方法引用来代替 lambda 表达式:

```java
int result = numbers.stream().reduce(0, Integer::sum);
assertThat(result).isEqualTo(21);
```

当然，我们可以在包含其他类型元素的流上使用`reduce()`操作。

例如，我们可以在一组`String`元素上使用`reduce()`,并将它们合并成一个结果:

```java
List<String> letters = Arrays.asList("a", "b", "c", "d", "e");
String result = letters
  .stream()
  .reduce("", (partialString, element) -> partialString + element);
assertThat(result).isEqualTo("abcde");
```

类似地，我们可以切换到使用方法引用的版本:

```java
String result = letters.stream().reduce("", String::concat);
assertThat(result).isEqualTo("abcde");
```

让我们使用`reduce()`操作来连接`letters`数组的大写元素:

```java
String result = letters
  .stream()
  .reduce(
    "", (partialString, element) -> partialString.toUpperCase() + element.toUpperCase());
assertThat(result).isEqualTo("ABCDE");
```

此外，我们可以在并行化的流中使用`reduce()`(稍后会详细介绍):

```java
List<Integer> ages = Arrays.asList(25, 30, 45, 28, 32);
int computedAges = ages.parallelStream().reduce(0, (a, b) -> a + b, Integer::sum);
```

当流并行执行时，Java 运行时将流分成多个子流。在这种情况下，**我们需要使用一个函数将子流的结果合并成一个单独的结果。** **这是组合器**的作用——在上面的代码片段中，它是`Integer::sum`方法引用。

有趣的是，这段代码无法编译:

```java
List<User> users = Arrays.asList(new User("John", 30), new User("Julie", 35));
int computedAges = 
  users.stream().reduce(0, (partialAgeResult, user) -> partialAgeResult + user.getAge()); 
```

在这种情况下，我们有一个`User`对象流，累加器参数的类型是`Integer`和`User.`，然而，累加器实现是`Integers,` 的和，所以编译器不能推断出`user`参数的类型。

我们可以通过使用合并器来解决这个问题:

```java
int result = users.stream()
  .reduce(0, (partialAgeResult, user) -> partialAgeResult + user.getAge(), Integer::sum);
assertThat(result).isEqualTo(65);
```

**简单来说，如果我们使用顺序流，并且累加器参数的类型与其实现的类型匹配，我们就不需要使用合并器。**

## 4。并行减少

正如我们之前所学的，我们可以在并行化的流上使用`reduce()`。

当我们使用并行流时，我们应该确保在流上执行的`reduce()`或任何其他聚合操作:

*   关联:结果不受操作数顺序的影响
*   无干扰:操作不影响数据源
*   无状态和确定性:操作没有状态，对于给定的输入产生相同的输出

我们应该满足所有这些条件，以防止不可预测的结果。

正如所料，在并行流上执行的操作，包括`reduce()`，是并行执行的，因此利用了多核硬件架构。

由于明显的原因，**并行化的流比顺序的流有更好的性能。**即使如此，如果应用于流的操作不昂贵，或者流中的元素数量很少，那么它们可能是多余的。

当然，当我们需要处理大型流并执行昂贵的聚合操作时，并行流是正确的选择。

让我们创建一个简单的[JMH](/web/20221208143903/https://www.baeldung.com/java-microbenchmark-harness)(Java 微基准测试工具)基准测试，并比较在顺序流和并行流上使用`reduce()`操作时各自的执行时间:

```java
@State(Scope.Thread)
private final List<User> userList = createUsers();

@Benchmark
public Integer executeReduceOnParallelizedStream() {
    return this.userList
      .parallelStream()
      .reduce(
        0, (partialAgeResult, user) -> partialAgeResult + user.getAge(), Integer::sum);
}

@Benchmark
public Integer executeReduceOnSequentialStream() {
    return this.userList
      .stream()
      .reduce(
        0, (partialAgeResult, user) -> partialAgeResult + user.getAge(), Integer::sum);
} 
```

**在上述 JMH 基准测试中，我们比较了平均执行时间。**我们简单地创建一个包含大量`User`对象的`List`。接下来，我们在一个顺序流和一个并行流上调用`reduce()`，并检查后者的执行速度是否比前者快(以秒/操作为单位)。

这些是我们的基准测试结果:

```java
Benchmark                                                   Mode  Cnt  Score    Error  Units
JMHStreamReduceBenchMark.executeReduceOnParallelizedStream  avgt    5  0,007 ±  0,001   s/op
JMHStreamReduceBenchMark.executeReduceOnSequentialStream    avgt    5  0,010 ±  0,001   s/op
```

## 5。减少时抛出并处理异常

在上面的例子中，`reduce()`操作没有抛出任何异常。当然，也有可能。

例如，假设我们需要将一个流的所有元素除以一个提供的因子，然后将它们相加:

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
int divider = 2;
int result = numbers.stream().reduce(0, a / divider + b / divider); 
```

只要`divider`变量不为零，这就可以工作。但如果是零，`reduce()`会抛出一个 [`ArithmeticException`](https://web.archive.org/web/20221208143903/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ArithmeticException.html) 异常:被零除。

我们可以很容易地捕捉异常并对其做一些有用的事情，比如记录它、从它恢复等等，这取决于用例，通过使用 [try/catch](/web/20221208143903/https://www.baeldung.com/java-exceptions) 块:

```java
public static int divideListElements(List<Integer> values, int divider) {
    return values.stream()
      .reduce(0, (a, b) -> {
          try {
              return a / divider + b / divider;
          } catch (ArithmeticException e) {
              LOGGER.log(Level.INFO, "Arithmetic Exception: Division by Zero");
          }
          return 0;
      });
}
```

虽然这种方法可行，**我们用`try/catch`块**污染了 lambda 表达式。我们不再像以前那样有简洁的一行程序了。

为了解决这个问题，我们可以使用[提取函数重构技术](https://web.archive.org/web/20221208143903/https://refactoring.com/catalog/extractFunction.html) **并将`try/catch`块提取到一个单独的方法**:

```java
private static int divide(int value, int factor) {
    int result = 0;
    try {
        result = value / factor;
    } catch (ArithmeticException e) {
        LOGGER.log(Level.INFO, "Arithmetic Exception: Division by Zero");
    }
    return result
} 
```

现在,`divideListElements()`方法的实现再次变得清晰而流畅:

```java
public static int divideListElements(List<Integer> values, int divider) {
    return values.stream().reduce(0, (a, b) -> divide(a, divider) + divide(b, divider));
} 
```

假设`divideListElements()`是一个由抽象`NumberUtils`类实现的实用方法，我们可以创建一个单元测试来检查`divideListElements()`方法的行为:

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
assertThat(NumberUtils.divideListElements(numbers, 1)).isEqualTo(21); 
```

当提供的`Integer`值的`List`包含 0 时，让我们也测试一下`divideListElements()`方法:

```java
List<Integer> numbers = Arrays.asList(0, 1, 2, 3, 4, 5, 6);
assertThat(NumberUtils.divideListElements(numbers, 1)).isEqualTo(21); 
```

最后，让我们测试分频器也为 0 时的方法实现:

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
assertThat(NumberUtils.divideListElements(numbers, 0)).isEqualTo(0);
```

## 6.复杂的自定义对象

我们**也可以将`Stream.reduce() `用于包含非原语字段的自定义对象。为此，我们需要为数据类型`.`提供一个相关的 i `dentity`、 `accumulator` 和 `combiner`**

假设我们的`User `是一个评论网站的一部分。我们每个`User`都可以拥有一个`Rating`，它是由许多`Review`平均得到的

首先，让我们从我们的`Review `对象开始。

每个`Review` 应该包含一个简单的评论和分数:

```java
public class Review {

    private int points;
    private String review;

    // constructor, getters and setters
}
```

接下来，我们需要定义我们的`Rating,` ，它将在`points`字段旁边保存我们的评论。随着我们添加更多评论，该字段将相应增加或减少:

```java
public class Rating {

    double points;
    List<Review> reviews = new ArrayList<>();

    public void add(Review review) {
        reviews.add(review);
        computeRating();
    }

    private double computeRating() {
        double totalPoints = 
          reviews.stream().map(Review::getPoints).reduce(0, Integer::sum);
        this.points = totalPoints / reviews.size();
        return this.points;
    }

    public static Rating average(Rating r1, Rating r2) {
        Rating combined = new Rating();
        combined.reviews = new ArrayList<>(r1.reviews);
        combined.reviews.addAll(r2.reviews);
        combined.computeRating();
        return combined;
    }

}
```

我们还添加了一个`average `函数来计算基于两个输入`Rating`的平均值。这将很好地用于我们的`combiner `和`accumulator `组件。

接下来，让我们定义一个`User`列表，每个列表都有自己的评论集:

```java
User john = new User("John", 30);
john.getRating().add(new Review(5, ""));
john.getRating().add(new Review(3, "not bad"));
User julie = new User("Julie", 35);
john.getRating().add(new Review(4, "great!"));
john.getRating().add(new Review(2, "terrible experience"));
john.getRating().add(new Review(4, ""));
List<User> users = Arrays.asList(john, julie); 
```

既然 John 和 Julie 已经考虑在内，让我们使用`Stream.reduce()`来计算两个用户的平均评分。

**作为一个`identity`，如果我们的输入列表为空**，让我们返回一个新的`Rating`:

```java
Rating averageRating = users.stream()
  .reduce(new Rating(), 
    (rating, user) -> Rating.average(rating, user.getRating()), 
    Rating::average);
```

如果我们算一下，我们应该会发现平均分数是 3.6:

```java
assertThat(averageRating.getPoints()).isEqualTo(3.6);
```

## 7。结论

在本文中，**我们学习了如何使用`Stream.reduce()`操作。**

此外，我们还学习了如何对顺序流和并行流执行缩减，以及如何在缩减的同时处理异常。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143903/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)