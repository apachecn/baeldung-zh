# Java 8 中的函数接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-functional-interfaces>

## 1。简介

本教程是 Java 8 中不同函数接口的指南，以及它们的一般用例，和在标准 JDK 库中的用法。

## 延伸阅读:

## [可在 Java 中流式传输](/web/20221122150454/https://www.baeldung.com/java-iterable-to-stream)

The article explains how to convert an Iterable to Stream and why the Iterable interface doesn't support it directly.[Read more](/web/20221122150454/https://www.baeldung.com/java-iterable-to-stream) →

## [如何在 Java 8 流中使用 if/else 逻辑](/web/20221122150454/https://www.baeldung.com/java-8-streams-if-else-logic)

Learn how to apply if/else logic to Java 8 Streams.[Read more](/web/20221122150454/https://www.baeldung.com/java-8-streams-if-else-logic) →

## 2。Java 8 中的 lambdas

Java 8 以 lambda 表达式的形式带来了强大的新语法改进。lambda 是一个匿名函数，我们可以作为一流的语言公民来处理它。例如，我们可以将它传递给一个方法或从一个方法返回它。

在 Java 8 之前，我们通常会为每个需要封装单一功能的情况创建一个类。这意味着需要大量不必要的样板代码来定义作为原始函数表示的东西。

文章[“Lambda 表达式和函数接口:技巧和最佳实践”](/web/20221122150454/https://www.baeldung.com/java-8-lambda-expressions-tips)更详细地描述了使用 Lambda 的函数接口和最佳实践。本指南主要关注`java.util.function`包中的一些特殊功能接口。

## 3。功能接口

建议所有功能接口都有一个信息性的`@FunctionalInterface`注释。这清楚地传达了接口的目的，并且还允许编译器在带注释的接口不满足条件时生成错误。

**任何带有 SAM(单一抽象方法)的接口都是一个函数接口**，它的实现可能被当作 lambda 表达式。

注意 Java 8 的`default`方法不是`abstract`不算；一个函数接口可能仍然有多个`default`方法。我们可以通过查看`Function's` [文档](https://web.archive.org/web/20221122150454/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html)来观察这一点。

## 4。功能

lambda 最简单、最常见的情况是一个函数接口，它的方法接收一个值并返回另一个值。这个单个参数的函数由`Function`接口表示，该接口由它的参数类型和返回值参数化:

```java
public interface Function<T, R> { … }
```

标准库中`Function`类型的用法之一是`Map.computeIfAbsent`方法。此方法通过键从映射中返回一个值，但如果映射中不存在键，则计算一个值。为了计算一个值，它使用传递的函数实现:

```java
Map<String, Integer> nameMap = new HashMap<>();
Integer value = nameMap.computeIfAbsent("John", s -> s.length());
```

在这种情况下，我们将计算一个值，方法是将一个函数应用到一个键上，放入一个 map 中，并从一个方法调用中返回。W **e 可以用一个匹配被传递和返回的值类型**的方法引用来替换 lambda。

请记住，我们调用方法的对象实际上是方法的隐式第一个参数。这允许我们将实例方法`length`引用转换为`Function`接口:

```java
Integer value = nameMap.computeIfAbsent("John", String::length);
```

`Function`接口也有一个默认的`compose`方法，允许我们将几个函数合并成一个函数，并依次执行它们:

```java
Function<Integer, String> intToString = Object::toString;
Function<String, String> quote = s -> "'" + s + "'";

Function<Integer, String> quoteIntToString = quote.compose(intToString);

assertEquals("'5'", quoteIntToString.apply(5));
```

`quoteIntToString`函数是应用于`intToString`函数结果的`quote`函数的组合。

## 5。原始函数专门化

因为基本类型不能是泛型类型参数，所以对于最常用的基本类型`double`、 `int`、`long`以及它们在参数和返回类型中的组合，有各种版本的`Function`接口:

*   `IntFunction`、`LongFunction`、`DoubleFunction:` 参数是指定的类型，返回类型是参数化的
*   `ToIntFunction`、`ToLongFunction`、`ToDoubleFunction:` 返回类型为指定类型，参数为参数化
*   `DoubleToIntFunction`、`DoubleToLongFunction`、`IntToDoubleFunction`、`IntToLongFunction`、`LongToIntFunction`、`LongToDoubleFunction:` 具有被定义为原始类型的自变量和返回类型，如它们的名称所指定的

举个例子，没有现成的函数接口可以接受一个`short`并返回一个`byte`，但是没有什么可以阻止我们编写自己的接口:

```java
@FunctionalInterface
public interface ShortToByteFunction {

    byte applyAsByte(short s);

}
```

现在我们可以编写一个方法，使用由`ShortToByteFunction`定义的规则将数组`short`转换为数组`byte`:

```java
public byte[] transformArray(short[] array, ShortToByteFunction function) {
    byte[] transformedArray = new byte[array.length];
    for (int i = 0; i < array.length; i++) {
        transformedArray[i] = function.applyAsByte(array[i]);
    }
    return transformedArray;
}
```

下面是我们如何使用它将一个 shorts 数组转换成一个字节乘以 2 的数组:

```java
short[] array = {(short) 1, (short) 2, (short) 3};
byte[] transformedArray = transformArray(array, s -> (byte) (s * 2));

byte[] expectedArray = {(byte) 2, (byte) 4, (byte) 6};
assertArrayEquals(expectedArray, transformedArray);
```

## 6。二元函数专门化

为了用两个参数定义 lambdas，我们必须使用额外的接口，这些接口的名称中包含了关键字`Bi”`:`BiFunction`、`ToDoubleBiFunction`、`ToIntBiFunction`和`ToLongBiFunction`。

`BiFunction`的参数和返回类型都被通用化，而`ToDoubleBiFunction`和其他允许我们返回一个原始值。

在标准 API 中使用这个接口的一个典型例子是在`Map.replaceAll`方法中，它允许用一些计算值替换 map 中的所有值。

让我们使用一个`BiFunction`实现，它接收一个键和一个旧值来计算薪水的新值并返回它。

```java
Map<String, Integer> salaries = new HashMap<>();
salaries.put("John", 40000);
salaries.put("Freddy", 30000);
salaries.put("Samuel", 50000);

salaries.replaceAll((name, oldValue) -> 
  name.equals("Freddy") ? oldValue : oldValue + 10000);
```

## 7。供应商

`Supplier`函数接口是另一个不接受任何参数的`Function`专门化。我们通常用它来延迟生成值。例如，让我们定义一个函数来平方一个`double`值。它不会接收值本身，而是接收该值的`Supplier`:

```java
public double squareLazy(Supplier<Double> lazyValue) {
    return Math.pow(lazyValue.get(), 2);
}
```

这允许我们使用一个`Supplier`实现来延迟生成这个函数调用的参数。如果参数的生成需要相当长的时间，这将非常有用。我们将使用 Guava 的`sleepUninterruptibly`方法进行模拟:

```java
Supplier<Double> lazyValue = () -> {
    Uninterruptibles.sleepUninterruptibly(1000, TimeUnit.MILLISECONDS);
    return 9d;
};

Double valueSquared = squareLazy(lazyValue);
```

`Supplier`的另一个用例是定义序列生成的逻辑。为了演示它，让我们使用一个静态的`Stream.generate`方法来创建一个斐波纳契数列的`Stream`:

```java
int[] fibs = {0, 1};
Stream<Integer> fibonacci = Stream.generate(() -> {
    int result = fibs[1];
    int fib3 = fibs[0] + fibs[1];
    fibs[0] = fibs[1];
    fibs[1] = fib3;
    return result;
});
```

我们传递给`Stream.generate`方法的函数实现了`Supplier`函数接口。注意，作为一个有用的发生器，`Supplier`通常需要某种外部状态。在这种情况下，它的状态由最后两个斐波那契数列组成。

为了实现这种状态，我们使用一个数组而不是几个变量，因为在 lambda 中使用的所有外部变量都必须是有效的最终变量。

`Supplier`函数接口的其他专门化包括`BooleanSupplier`、`DoubleSupplier`、`LongSupplier`和`IntSupplier`，它们的返回类型都是对应的原语。

## 8。消费者

与`Supplier`相反，`Consumer`接受通用化的参数，不返回任何内容。这是一个表示副作用的函数。

例如，让我们通过在控制台中打印问候语来问候姓名列表中的每个人。传递给`List.forEach`方法的 lambda 实现了`Consumer`函数接口:

```java
List<String> names = Arrays.asList("John", "Freddy", "Samuel");
names.forEach(name -> System.out.println("Hello, " + name));
```

还有专门版本的`Consumer` — `DoubleConsumer`、`IntConsumer`和`LongConsumer` —它们接收原始值作为参数。更有趣的是`BiConsumer`界面。它的一个用例是遍历地图的条目:

```java
Map<String, Integer> ages = new HashMap<>();
ages.put("John", 25);
ages.put("Freddy", 24);
ages.put("Samuel", 30);

ages.forEach((name, age) -> System.out.println(name + " is " + age + " years old"));
```

另一组专门的`BiConsumer`版本由接受两个参数的`ObjDoubleConsumer`、`ObjIntConsumer`和`ObjLongConsumer,` 组成；其中一个参数是泛型，另一个是基本类型。

## 9。谓词

在数学逻辑中，谓词是一个接收值并返回布尔值的函数。

`Predicate`函数接口是一个`Function`的专门化，它接收一个通用化的值并返回一个布尔值。`Predicate` lambda 的一个典型用例是过滤一组值:

```java
List<String> names = Arrays.asList("Angela", "Aaron", "Bob", "Claire", "David");

List<String> namesWithA = names.stream()
  .filter(name -> name.startsWith("A"))
  .collect(Collectors.toList());
```

在上面的代码中，我们使用`Stream` API 过滤一个列表，只保留以字母“A”开头的名字。`Predicate` 实现封装了过滤逻辑。

和前面所有的例子一样，这个函数有`IntPredicate`、`DoublePredicate`和`LongPredicate`三个版本来接收原始值。

## 10。操作员

接口是接收和返回相同值类型的函数的特例。`UnaryOperator`接口接收一个参数。它在 Collections API 中的一个用例是用一些相同类型的计算值替换列表中的所有值:

```java
List<String> names = Arrays.asList("bob", "josh", "megan");

names.replaceAll(name -> name.toUpperCase());
```

`List.replaceAll`函数返回`void`,因为它就地替换了值。为了达到这个目的，用于转换列表值的 lambda 必须返回与接收到的结果类型相同的结果类型。这就是`UnaryOperator`在这里有用的原因。

当然，代替`name -> name.toUpperCase()`，我们可以简单地使用一个方法引用:

```java
names.replaceAll(String::toUpperCase);
```

`BinaryOperator`最有趣的用例之一是归约操作。假设我们希望将一个整数集合聚合成所有值的总和。有了`Stream` API，我们可以使用收集器`,`来完成这项工作，但是更通用的方法是使用`reduce`方法:

```java
List<Integer> values = Arrays.asList(3, 5, 8, 9, 12);

int sum = values.stream()
  .reduce(0, (i1, i2) -> i1 + i2); 
```

`reduce`方法接收一个初始累加器值和一个`BinaryOperator`函数。此函数的参数是一对相同类型的值；该函数本身还包含一个逻辑，用于将它们连接到同类型的单个值中。**传递的函数必须是关联的**，这意味着值聚合的顺序无关紧要，即应满足以下条件:

```java
op.apply(a, op.apply(b, c)) == op.apply(op.apply(a, b), c)
```

一个`BinaryOperator`操作符函数的关联属性允许我们轻松地并行化归约过程。

当然，也有`UnaryOperator`和`BinaryOperator`的专门化可以用原语值，即`DoubleUnaryOperator`、`IntUnaryOperator`、`LongUnaryOperator`、`DoubleBinaryOperator`、`IntBinaryOperator`、`LongBinaryOperator`。

## 11。遗留功能接口

并不是所有的功能接口都出现在 Java 8 中。Java 以前版本的许多接口都符合 a `FunctionalInterface`的约束，我们可以把它们当作 lambdas 来使用。突出的例子包括在并发 API 中使用的`Runnable`和`Callable`接口。在 Java 8 中，这些接口也用一个`@FunctionalInterface`注释标记。这使我们能够极大地简化并发代码:

```java
Thread thread = new Thread(() -> System.out.println("Hello From Another Thread"));
thread.start();
```

## 12。结论

在本文中，我们研究了 Java 8 API 中可以用作 lambda 表达式的不同函数接口。这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221122150454/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas)