# Java 双功能接口指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bifunction-interface>

## 1.介绍

Java 8 引入了[函数式编程](/web/20220731202352/https://www.baeldung.com/java-8-functional-interfaces)，允许我们通过传入函数来参数化通用方法。

我们可能最熟悉单参数 Java 8 函数接口，如`Function`、`Predicate,`和`Consumer`。

**在本教程中，我们将看看使用两个参数**的函数接口。这种函数被称为二进制函数，在 Java 中用`BiFunction`函数接口表示。

## 2.单参数函数

让我们快速回顾一下如何使用单参数或一元函数，就像我们在 [streams](/web/20220731202352/https://www.baeldung.com/java-8-streams-introduction) 中所做的那样:

```java
List<String> mapped = Stream.of("hello", "world")
  .map(word -> word + "!")
  .collect(Collectors.toList());

assertThat(mapped).containsExactly("hello!", "world!");
```

正如我们所见，`map`使用了`Function`，它接受一个参数，并允许我们对该值执行操作，返回一个新值。

## 3.双参数运算

**Java 流库为我们提供了一个`reduce`函数，允许我们组合流**的元素。我们需要通过添加下一项来表达我们到目前为止积累的值是如何转化的。

`reduce`函数使用函数接口`BinaryOperator<T>`，它接受两个相同类型的对象作为输入。

让我们假设我们想要通过用破折号分隔符将新的项目放在前面来连接我们的流中的所有项目。在接下来的几节中，我们将看看实现这一点的几种方法。

### 3.1.使用λ

`BiFunction`的 lambda 实现以两个参数为前缀，用括号括起来:

```java
String result = Stream.of("hello", "world")
  .reduce("", (a, b) -> b + "-" + a);

assertThat(result).isEqualTo("world-hello-");
```

正如我们所看到的，这两个值，`a`和`b`是`Strings`。我们已经编写了一个 lambda，将它们组合在一起，得到想要的输出，第二个输出在前，中间有一个破折号。

我们应该注意到,`reduce`使用了一个起始值——在本例中是空字符串。因此，我们以上面代码的结尾破折号结束，因为流中的第一个值与它连接。

此外，我们应该注意到 Java 的类型推断允许我们在大多数情况下省略参数的类型。在上下文中 lambda 的类型不明确的情况下，我们可以为参数使用类型:

```java
String result = Stream.of("hello", "world")
  .reduce("", (String a, String b) -> b + "-" + a);
```

### 3.2.使用函数

如果我们想让上面的算法不在末尾加一个破折号呢？我们可以在 lambda 中编写更多的代码，但这可能会变得混乱。让我们提取一个函数来代替:

```java
private String combineWithoutTrailingDash(String a, String b) {
    if (a.isEmpty()) {
        return b;
    }
    return b + "-" + a;
}
```

然后称之为:

```java
String result = Stream.of("hello", "world") 
  .reduce("", (a, b) -> combineWithoutTrailingDash(a, b)); 

assertThat(result).isEqualTo("world-hello");
```

正如我们所见，lambda 调用我们的函数，这比将更复杂的实现内联更容易阅读。

### 3.3.使用方法引用

一些 ide 会自动提示我们将上面的 lambda 转换成方法引用，因为这样读起来更清晰。

让我们重写代码以使用方法引用:

```java
String result = Stream.of("hello", "world")
  .reduce("", this::combineWithoutTrailingDash);

assertThat(result).isEqualTo("world-hello");
```

方法引用通常使功能代码更易于理解。

## 4.使用`BiFunction`

到目前为止，我们已经演示了如何使用两个参数属于同一类型的函数。**`BiFunction`接口允许我们使用不同类型的参数**，返回值为第三种类型。

假设我们正在创建一个算法，通过对每一对元素执行运算，将两个大小相同的列表合并成第三个列表:

```java
List<String> list1 = Arrays.asList("a", "b", "c");
List<Integer> list2 = Arrays.asList(1, 2, 3);

List<String> result = new ArrayList<>();
for (int i=0; i < list1.size(); i++) {
    result.add(list1.get(i) + list2.get(i));
}

assertThat(result).containsExactly("a1", "b2", "c3");
```

### 4.1.推广这个函数

**我们可以使用一个`BiFunction`** 作为组合器来概括这个特殊函数:

```java
private static <T, U, R> List<R> listCombiner(
  List<T> list1, List<U> list2, BiFunction<T, U, R> combiner) {
    List<R> result = new ArrayList<>();
    for (int i = 0; i < list1.size(); i++) {
        result.add(combiner.apply(list1.get(i), list2.get(i)));
    }
    return result;
}
```

让我们看看这是怎么回事。有三种类型的参数:`T`用于第一个列表中的项目类型，`U`用于第二个列表中的类型，然后`R`用于组合函数返回的任何类型。

**我们通过调用这个函数的`apply`方法**来使用提供给这个函数的`BiFunction`来获得结果。

### 4.2.调用广义函数

我们的组合器是一个`BiFunction`，它允许我们注入一个算法，不管输入和输出是什么类型。让我们试一试:

```java
List<String> list1 = Arrays.asList("a", "b", "c");
List<Integer> list2 = Arrays.asList(1, 2, 3);

List<String> result = listCombiner(list1, list2, (a, b) -> a + b);

assertThat(result).containsExactly("a1", "b2", "c3");
```

我们也可以把它用于完全不同类型的输入和输出。

让我们注入一个算法来确定第一个列表中的值是否大于第二个列表中的值，并产生一个`boolean`结果:

```java
List<Double> list1 = Arrays.asList(1.0d, 2.1d, 3.3d);
List<Float> list2 = Arrays.asList(0.1f, 0.2f, 4f);

List<Boolean> result = listCombiner(list1, list2, (a, b) -> a > b);

assertThat(result).containsExactly(true, true, false);
```

### 4.3.一个`BiFunction`方法参考

让我们用提取的方法和方法引用重写上面的代码:

```java
List<Double> list1 = Arrays.asList(1.0d, 2.1d, 3.3d);
List<Float> list2 = Arrays.asList(0.1f, 0.2f, 4f);

List<Boolean> result = listCombiner(list1, list2, this::firstIsGreaterThanSecond);

assertThat(result).containsExactly(true, true, false);

private boolean firstIsGreaterThanSecond(Double a, Float b) {
    return a > b;
}
```

我们应该注意，这使得代码更容易阅读，因为方法`firstIsGreaterThanSecond`描述了作为方法引用注入的算法。

### 4.4.`BiFunction`方法引用使用`this`

假设我们想使用上述基于`BiFunction-`的算法来确定两个列表是否相等:

```java
List<Float> list1 = Arrays.asList(0.1f, 0.2f, 4f);
List<Float> list2 = Arrays.asList(0.1f, 0.2f, 4f);

List<Boolean> result = listCombiner(list1, list2, (a, b) -> a.equals(b));

assertThat(result).containsExactly(true, true, true);
```

我们实际上可以简化解决方案:

```java
List<Boolean> result = listCombiner(list1, list2, Float::equals);
```

这是因为`Float`中的`equals`函数与`BiFunction`具有相同的签名。它接受类型为`Float`的对象`this,` 的隐式第一个参数。类型为`Object`的第二个参数`other`是要比较的值。

## 5.作曲`BiFunctions`

如果我们可以使用方法引用来做和我们的数字列表比较例子一样的事情会怎么样呢？

```java
List<Double> list1 = Arrays.asList(1.0d, 2.1d, 3.3d);
List<Double> list2 = Arrays.asList(0.1d, 0.2d, 4d);

List<Integer> result = listCombiner(list1, list2, Double::compareTo);

assertThat(result).containsExactly(1, 1, -1);
```

这与我们的例子很接近，但是返回的是一个`Integer`，而不是原来的`Boolean`。这是因为`Double`中的`compareTo`方法返回了`Integer`。

我们可以通过**使用`andThen`组成一个函数**来添加我们需要的额外行为来实现我们的原始行为。这产生了一个`BiFunction`,它首先用两个输入做一件事，然后执行另一个操作。

接下来，让我们创建一个函数，将我们的方法引用`Double::compareTo`强制转换为`BiFunction`:

```java
private static <T, U, R> BiFunction<T, U, R> asBiFunction(BiFunction<T, U, R> function) {
    return function;
}
```

**lambda 或方法引用只有在被方法调用转换后才成为`BiFunction`。**我们可以使用这个辅助函数将我们的 lambda 显式转换成`BiFunction`对象。

现在，我们可以使用`andThen`在第一个函数上添加行为:

```java
List<Double> list1 = Arrays.asList(1.0d, 2.1d, 3.3d);
List<Double> list2 = Arrays.asList(0.1d, 0.2d, 4d);

List<Boolean> result = listCombiner(list1, list2,
  asBiFunction(Double::compareTo).andThen(i -> i > 0));

assertThat(result).containsExactly(true, true, false);
```

## 6.结论

在本教程中，我们已经根据提供的 Java 流库和我们自己的自定义函数探索了`BiFunction`和`BinaryOperator`。我们已经看到了如何使用 lambdas 和方法引用来传递`BiFunctions`,以及如何构造函数。

Java 库只提供单参数和双参数的函数接口。对于需要更多参数的情况，请看我们关于[奉承](/web/20220731202352/https://www.baeldung.com/java-currying)的文章以获得更多想法。

和往常一样，完整的代码样本可以在 GitHub 的[上找到。](https://web.archive.org/web/20220731202352/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-2)