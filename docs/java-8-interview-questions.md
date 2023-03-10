# Java 8 面试问题(+答案)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20220807183533/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20220807183533/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20220807183533/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20220807183533/https://www.baeldung.com/java-classes-initialization-questions)
• Java 8 Interview Questions(+ Answers) (current article)[• Memory Management in Java Interview Questions (+Answers)](/web/20220807183533/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20220807183533/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20220807183533/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20220807183533/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20220807183533/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20220807183533/https://www.baeldung.com/spring-interview-questions)

## 1。简介

在本教程中，我们将探讨一些在面试中可能会出现的与 JDK8 相关的问题。

Java 8 是一个平台版本，包含了新的语言特性和库类。这些新特性中的大部分都是为了实现更干净、更紧凑的代码，而有些则添加了 Java 以前从未支持过的新功能。

## 延伸阅读:

## [Java 面试中的内存管理问题(+答案)](/web/20220807183533/https://www.baeldung.com/java-memory-management-interview-questions)

A set of popular Memory Management-related interview questions and of course answers.[Read more](/web/20220807183533/https://www.baeldung.com/java-memory-management-interview-questions) →

## [Java 集合面试问题](/web/20220807183533/https://www.baeldung.com/java-collections-interview-questions)

A set of practical Collections-related Java interview questions[Read more](/web/20220807183533/https://www.baeldung.com/java-collections-interview-questions) →

## 2。Java 8 常识

### Q1。Java 8 中增加了哪些新特性？

Java 8 附带了几个新特性，但最重要的特性如下:

*   **Lambda 表达式**——一种新的语言特性，允许我们将动作视为对象
*   **方法引用**—使我们能够通过直接使用方法名引用方法来定义 Lambda 表达式
*   `**Optional**`—用于表达可选性的特殊包装类
*   **函数接口**——最多一个抽象方法的接口；可以使用 Lambda 表达式提供实现
*   默认方法让我们能够在抽象方法之外的接口中添加完整的实现
*   **Nashorn，JavaScript 引擎**—基于 Java 的引擎，用于执行和评估 JavaScript 代码
*   **`Stream`API**——一个特殊的迭代器类，允许我们以函数方式处理对象集合
*   **日期 API**—改进的、不可变的、受 JodaTime 启发的日期 API

除了这些新特性，编译器和 JVM 级别的许多特性增强都是在幕后完成的。

## 3。方法引用

### Q1。什么是方法引用？

方法引用是一个 Java 8 构造，可以用来引用一个方法而不用调用它。它用于将方法视为 Lambda 表达式。它们只是作为句法糖来减少一些 lambdas 的冗长。这种方式下的代码:

```java
(o) -> o.toString();
```

可以变成:

```java
Object::toString();
```

方法引用可以由分隔类名或对象名以及方法名的双冒号来标识。它有不同的变体，例如构造函数引用:

```java
String::new;
```

静态方法引用:

```java
String::valueOf;
```

绑定实例方法引用:

```java
str::toString;
```

未绑定的实例方法引用:

```java
String::toString;
```

我们可以通过跟随[这个链接](https://web.archive.org/web/20220807183533/https://www.codementor.io/eh3rrera/using-java-8-method-reference-du10866vx)和[这个](/web/20220807183533/https://www.baeldung.com/java-8-double-colon-operator)来阅读带有完整例子的方法引用的详细描述。

### Q2。String::Valueof 表达式是什么意思？

它是对`String`类的`valueOf`方法的静态方法引用。

## 4。`Optional`

### Q1。什么是`Optional`？怎么用？

`Optional`是 Java 8 中的一个新类，它封装了一个可选值，即要么存在要么不存在的值。它是一个对象的包装器，我们可以把它看作一个包含零个或一个元素的容器。

`Optional`有一个特殊的`Optional.empty()`值而不是被包装的`null`。因此，在许多情况下，它可以用来代替一个可空值来删除`NullPointerException`。

我们可以在这里阅读关于`Optional` [的专门文章。](/web/20220807183533/https://www.baeldung.com/java-optional)

正如其创建者所设计的那样，`Optional`的主要目的是成为一种返回类型的方法，以前它会返回`null`。这种方法需要我们编写样板代码来检查返回值，我们有时可能会忘记进行防御性检查。在 Java 8 中，`Optional`返回类型明确要求我们以不同的方式处理空或非空包装值。

例如，`Stream.min()`方法计算值流中的最小值。但是如果流是空的呢？如果不是因为`Optional`，这个方法将返回`null` 或者抛出一个异常。

但是，它返回一个`Optional`值，可能是`Optional.empty()`(第二种情况)。这使我们能够轻松处理这样的情况:

```java
int min1 = Arrays.stream(new int[]{1, 2, 3, 4, 5})
  .min()
  .orElse(0);
assertEquals(1, min1);

int min2 = Arrays.stream(new int[]{})
  .min()
  .orElse(0);
assertEquals(0, min2); 
```

值得注意的是`Optional`并不是 Scala 中`Option`那样的通用类。不建议我们在实体类中使用它作为字段值，因为它没有实现`Serializable`接口。

## 5。功能接口

### Q1。描述标准库中的一些功能接口

`java.util.function`包里有很多功能接口。比较常见的包括但不限于:

*   `Function`–它接受一个参数并返回一个结果
*   `Consumer`–它接受一个参数，不返回任何结果(表示副作用)
*   `Supplier`–它不接受任何参数并返回一个结果
*   `Predicate`–它接受一个参数并返回一个布尔值
*   `BiFunction`–它接受两个参数并返回一个结果
*   `BinaryOperator`–它类似于一个`BiFunction`，接受两个参数并返回一个结果。两个参数和结果都是相同的类型。
*   `UnaryOperator`–它类似于`Function`，接受单个参数并返回相同类型的结果

有关函数接口的更多信息，请参阅文章[“Java 8 中的函数接口”](/web/20220807183533/https://www.baeldung.com/java-8-functional-interfaces)

### Q2。什么是功能界面？定义功能接口的规则是什么？

函数接口是只有一个抽象方法的接口(`default`方法不算)，不多也不少。

如果需要这种接口的实例，可以使用 Lambda 表达式来代替。更正式的说法是:`Functional interfaces`为 lambda 表达式和方法引用提供目标类型。

这种表达式的参数和返回类型与单个抽象方法的参数和返回类型直接匹配。

例如，`Runnable`接口是一个函数接口，所以不用:

```java
Thread thread = new Thread(new Runnable() {
    public void run() {
        System.out.println("Hello World!");
    }
});
```

我们可以简单地做:

```java
Thread thread = new Thread(() -> System.out.println("Hello World!"));
```

函数接口通常用`@FunctionalInterface`注释来标注，这是信息性的，不影响语义。

## 6。默认方法

### Q1。什么是默认方法，我们什么时候使用它？

默认方法是具有实现的方法，可以在接口中找到。

我们可以使用默认方法向接口添加新功能，同时保持与已经实现该接口的类的向后兼容性:

```java
public interface Vehicle {
    public void move();
    default void hoot() {
        System.out.println("peep!");
    }
}
```

通常当我们向一个接口添加一个新的抽象方法时，所有的实现类都会中断，直到它们实现了新的抽象方法。在 Java 8 中，这个问题通过使用默认方法得到了解决。

例如，`Collection`接口没有`forEach`方法声明。因此，添加这样的方法只会破坏整个集合 API。

Java 8 引入了默认方法，这样,`Collection`接口可以有一个默认的`forEach`方法的实现，而不需要实现这个接口的类来实现相同的方法。

### Q2。下面的代码会编译吗？

```java
@FunctionalInterface
public interface Function2<T, U, V> {
    public V apply(T t, U u);

    default void count() {
        // increment counter
    }
}
```

是的，代码可以编译，因为它遵循了只定义一个抽象方法的函数接口规范。第二个方法`count`是一个默认方法，它不会增加抽象方法的数量。

## 7。λ表达式

### Q1。什么是 Lambda 表达式，它有什么用途？

简单来说，lambda 表达式是一个函数，我们可以将其作为对象引用和传递。

此外，lambda 表达式引入了 Java 中的函数式处理，并有助于编写紧凑、易读的代码。

因此，lambda 表达式是匿名类(如方法参数)的自然替代品。它们的主要用途之一是定义函数接口的内联实现。

### Q2。解释 Lambda 表达式的语法和特征

lambda 表达式由两部分组成，参数部分和表达式部分由向前箭头分隔:

```java
params -> expressions
```

任何 lambda 表达式都具有以下特征:

*   **可选类型声明**–当声明 lambda 左侧的参数时，我们不需要声明它们的类型，因为编译器可以从它们的值中推断出它们。所以`int param -> …`和`param ->…`都有效
*   **可选括号**–当只声明一个参数时，我们不需要把它放在括号里。这意味着`param -> …`和`(param) -> …`都是有效的，但是当声明一个以上的参数时，括号是必需的
*   **可选花括号**–当表达式部分只有一个语句时，就不需要花括号了。这意味着`param – > statement`和`param – > {statement;}`都是有效的，但是当有一个以上的语句时需要花括号
*   **可选的 return 语句**–当表达式返回一个值，并且这个值包含在花括号中，那么我们不需要 return 语句。这意味着`(a, b) – > {return a+b;}`和`(a, b) – > {a+b;}`都有效

要阅读更多关于 Lambda 表达式的内容，请点击[链接](https://web.archive.org/web/20220807183533/https://www.tutorialspoint.com/java8/java8_lambda_expressions.htm)和[链接](/web/20220807183533/https://www.baeldung.com/java-8-lambda-expressions-tips)。

## 8。Nashorn Javascript

### Q1。Java8 中的 Nashorn 是什么？

[Nashorn](/web/20220807183533/https://www.baeldung.com/java-nashorn) 是 Java 8 附带的 Java 平台的新 Javascript 处理引擎。在 JDK 7 之前，Java 平台出于同样的目的使用 Mozilla Rhino 作为 Javascript 处理引擎。

与它的前身相比，Nashorn 提供了更好的符合 ECMA 规范的 JavaScript 规范和更好的运行时性能。

### Q2。什么是 JJS？

在 Java 8 中，`jjs`是我们用来在控制台执行 Javascript 代码的新的可执行文件或命令行工具。

## 9。数据流

### Q1。什么是溪流？和集合有什么区别？

简单地说，流是一个迭代器，它的作用是接受一组应用于它所包含的每个元素的动作。

`stream`表示来自一个源(比如一个集合)的一系列对象，它支持聚合操作。它们旨在使收集过程简单明了。与集合相反，迭代的逻辑是在流内部实现的，所以我们可以使用像`map`和`flatMap`这样的方法来执行声明性处理。

此外，`Stream` API 是流畅的，允许流水线操作:

```java
int sum = Arrays.stream(new int[]{1, 2, 3})
  .filter(i -> i >= 2)
  .map(i -> i * 3)
  .sum();
```

与集合的另一个重要区别是，流本质上是延迟加载和处理的。

### Q2。中间运营和终端运营有什么区别？

我们将流操作合并到管道中来处理流。所有的操作不是中间的就是终端的。

中间操作是那些返回`Stream`本身的操作，允许对流进行进一步的操作。

这些操作总是懒惰的，也就是说，它们不在调用点处理流。只有当存在终端操作时，中间操作才能处理数据。一些中间操作是`filter`、`map`和`flatMap`。

相反，终端操作终止流水线并启动流处理。在终端操作调用期间，流通过所有中间操作。终端操作包括`forEach`、`reduce, Collect`和`sum`。

为了说明这一点，让我们看一个有副作用的例子:

```java
public static void main(String[] args) {
    System.out.println("Stream without terminal operation");

    Arrays.stream(new int[] { 1, 2, 3 }).map(i -> {
        System.out.println("doubling " + i);
        return i * 2;
    });

    System.out.println("Stream with terminal operation");
        Arrays.stream(new int[] { 1, 2, 3 }).map(i -> {
            System.out.println("doubling " + i);
            return i * 2;
    }).sum();
}
```

输出如下所示:

```java
Stream without terminal operation
Stream with terminal operation
doubling 1
doubling 2
doubling 3
```

正如我们所看到的，只有当终端操作存在时，才会触发中间操作。

### Q3。`Map`和`flatMap`流操作有什么区别？

`map`和`flatMap`的签名不同。一般来说，`map`操作将其返回值包装在其序数类型中，而`flatMap`没有。

例如，在`Optional`中，`map`操作将返回`Optional<String>`类型，而`flatMap`将返回`String`类型。

因此，在映射之后，我们需要展开(读“展平”)对象来检索值，而在平面映射之后，没有这样的需要，因为对象已经展平了。在`Stream`中，我们将相同的概念应用于映射和平面映射。

`map`和`flatMap`都是中间流操作，它们接收一个函数并将这个函数应用于流的所有元素。

不同的是，对于`map`，这个函数返回一个值，而对于`flatMap`，这个函数返回一个流。`flatMap`操作将这些流“展平”成一条。

这里有一个例子，我们获取了一个用户姓名和电话列表的地图，并将其“扁平化”为所有使用`flatMap`的用户的电话列表:

```java
Map<String, List<String>> people = new HashMap<>();
people.put("John", Arrays.asList("555-1123", "555-3389"));
people.put("Mary", Arrays.asList("555-2243", "555-5264"));
people.put("Steve", Arrays.asList("555-6654", "555-3242"));

List<String> phones = people.values().stream()
  .flatMap(Collection::stream)
    .collect(Collectors.toList());
```

### Q4。Java 8 中的流流水线是什么？

流管道是将操作链接在一起的概念。我们通过将流中可能发生的操作分为两类来实现这一点:中间操作和终端操作。

每个中间操作在运行时都返回流本身的一个实例。因此，我们可以设置任意数量的中间操作来处理数据，形成一个处理流水线。

然后必须有一个终端操作返回一个最终值并终止流水线。

## 10。Java 8 日期和时间 API

### Q1。告诉我们 Java 8 中新的日期和时间 API

Java 开发人员的一个长期问题是对普通开发人员所需的日期和时间操作的支持不足。

现有的类如`java.util.Date`和`SimpleDateFormatter`不是线程安全的，这给用户带来了潜在的并发问题。

糟糕的 API 设计在旧的 Java 数据 API 中也是一个现实。这里简单举个例子:`java.util.Date`中的年从 1900 开始，月从 1 开始，日从 0 开始，不是很直观。

这些问题和其他一些问题导致了第三方日期和时间库的流行，比如 Joda-Time。

为了解决这些问题并在 JDK 提供更好的支持，在包`java.time`下为 Java SE 8 设计了一个新的日期和时间 API，它没有这些问题。

## 11。结论

在本文中，我们探讨了几个带有 Java 8 偏见的重要技术面试问题。这绝不是一个详尽的列表，但它包含了我们认为最有可能在 Java 8 的每个新特性中出现的问题。

即使我们刚刚起步，对 Java 8 的无知也不是参加面试的好方法，尤其是当 Java 在简历中出现的时候。因此，重要的是，我们要花些时间来理解这些问题的答案，并尽可能做更多的研究。

祝你面试好运。

Next **»**[Memory Management in Java Interview Questions (+Answers)](/web/20220807183533/https://www.baeldung.com/java-memory-management-interview-questions)**«** Previous[Java Class Structure and Initialization Interview Questions](/web/20220807183533/https://www.baeldung.com/java-classes-initialization-questions)