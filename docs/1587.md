# Java 中的函数式编程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-functional-programming>

## 1.概观

在本教程中，我们将了解函数式编程范式的核心原则，以及如何在 Java 编程语言中实践它们。

我们还将介绍一些高级的函数式编程技术。

这将有助于我们评估从函数式编程中获得的好处，尤其是在 Java 中。

## 2.什么是函数式编程？

基本上，[函数式编程](/web/20221129014542/https://www.baeldung.com/cs/functional-programming)是**一种编写计算机程序的风格，将计算视为评估数学函数。**

在数学中，函数是将输入集与输出集联系起来的表达式。

重要的是，函数的输出只取决于它的输入。更有趣的是，我们可以将两个或更多的函数组合在一起，得到一个新函数。

### 2.1.λ演算

为了理解为什么数学函数的这些定义和属性在编程中如此重要，我们必须回到过去。

20 世纪 30 年代，数学家阿隆佐·邱奇开发了一个基于函数抽象来表达计算的正式系统。这个通用的计算模型被称为[λ演算](https://web.archive.org/web/20221129014542/https://plato.stanford.edu/entries/lambda-calculus/)。

Lambda 演算对编程语言理论的发展产生了巨大的影响，尤其是函数式编程语言。通常，函数式编程语言实现 lambda 演算。

由于 lambda 演算侧重于函数组合，函数式编程语言提供了在函数组合中组合软件的表达方式。

### 2.2.编程范例的分类

当然，函数式编程并不是实践中唯一的编程风格。概括地说，编程风格可以分为命令式和声明式编程范例。

命令式方法将程序定义为一系列改变程序状态的语句，直到程序到达最终状态。

过程编程是一种命令式编程，我们使用过程或子例程来构造程序。被称为[面向对象编程(OOP)](/web/20221129014542/https://www.baeldung.com/cs/oop-vs-functional) 的流行编程范例之一扩展了过程化编程概念。

相比之下，**声明性方法表达了计算的逻辑，而没有用一系列语句来描述它的控制流**。

简单地说，声明式方法的重点是定义程序必须实现什么，而不是它应该如何实现。函数式编程是声明式编程语言的一个子集。

这些类别有进一步的子类别，分类变得相当复杂，但是在本教程中我们不会深入讨论。

### 2.3.编程语言的分类

现在我们将试着理解编程语言是如何根据它们对函数式编程的支持来划分的。

纯函数式语言，比如 Haskell，只允许纯函数式程序。

其他语言既允许**函数式程序又允许**过程式程序，被认为是不纯函数式语言。许多语言都属于这一类，包括 Scala、Kotlin 和 Java。

理解今天大多数流行的编程语言是通用语言是很重要的，因此它们倾向于支持多种编程范例。

## 3.基本原则和概念

本节将介绍函数式编程的一些基本原理，以及如何在 Java 中采用它们。

请注意，我们将使用的许多特性并不总是 Java 的一部分，建议在 Java 8 或更高版本上有效地使用函数式编程。

### 3.1.一阶和高阶函数

如果一种编程语言将函数视为一等公民，则称其具有一等函数。

这意味着**函数被允许支持其他实体通常可用的所有操作。**这些包括将函数赋给变量，将它们作为参数传递给其他函数，并将它们作为其他函数的值返回。

这个特性使得在函数式编程中定义高阶函数成为可能。高阶函数能够接收函数作为参数，并返回一个函数作为结果。这进一步实现了函数编程中的一些技术，例如函数组合和 currying。

传统上，在 Java 中只能使用函数接口或匿名内部类等结构来传递函数。函数接口只有一个抽象方法，也称为单一抽象方法(SAM)接口。

假设我们必须为`Collections.sort`方法提供一个定制的比较器:

```
Collections.sort(numbers, new Comparator<Integer>() {
    @Override
    public int compare(Integer n1, Integer n2) {
        return n1.compareTo(n2);
    }
});
```

正如我们所看到的，这是一种冗长乏味的技术——当然不是鼓励开发人员采用函数式编程的东西。

幸运的是，Java 8 带来了许多新特性来简化这个过程，比如 lambda 表达式、方法引用和预定义的函数接口。

让我们看看 lambda 表达式如何帮助我们完成同样的任务:

```
Collections.sort(numbers, (n1, n2) -> n1.compareTo(n2));
```

这样肯定更简洁易懂。

然而，请注意，虽然这可能会给我们一种在 Java 中将函数作为一等公民使用的印象，但事实并非如此。

在 lambda 表达式的语法糖背后，Java 仍然将这些包装到函数接口中。所以， **Java 把一个 lambda 表达式当成了一个`Object`** ，是 Java 里真正的一等公民。

### 3.2.纯函数

纯函数的定义强调**纯函数应该只根据自变量返回值，并且应该没有副作用。**

这听起来与 Java 中的所有最佳实践背道而驰。

作为一种面向对象的语言，Java 推荐将封装作为核心编程实践。它鼓励隐藏对象的内部状态，只暴露访问和修改它所必需的方法。所以，这些方法不是严格意义上的纯函数。

当然，封装和其他面向对象的原则只是建议，在 Java 中没有约束力。

事实上，开发人员最近已经开始意识到定义不可变的状态和方法而没有副作用的价值。

假设我们想求出我们刚刚排序的所有数字的总和:

```
Integer sum(List<Integer> numbers) {
    return numbers.stream().collect(Collectors.summingInt(Integer::intValue));
}
```

这个方法只依赖于它接收的参数，所以它是确定性的。而且，不会产生任何副作用。

副作用可以是与方法的预期行为不同的任何东西。例如，**副作用可以简单到更新局部或全局状态**或者在返回值之前保存到数据库。(纯粹主义者也将日志记录视为副作用。)

所以，让我们看看我们如何处理合法的副作用。例如，出于真正的原因，我们可能需要将结果保存在数据库中。函数式编程中有一些技术可以在保留纯函数的同时处理副作用。

我们将在后面的章节中讨论其中的一些。

### 3.3.不变

不变性是函数式编程的核心原则之一，它**是指实体被实例化后不能被修改的属性。**

在函数式编程语言中，这是由语言级别的设计支持的。但是在 Java 中，我们必须自己决定创建不可变的数据结构。

请注意 **Java 本身提供了几个内置的不可变类型**，比如`String`。这主要是出于安全原因，因为我们在类加载中大量使用`String`,并在基于散列的数据结构中作为键。还有其他几个内置的不可变类型，如原始包装器和数学类型。

但是我们用 Java 创建的数据结构呢？当然，它们在默认情况下不是不可变的，我们必须做一些修改来实现不变性。

**对`final`关键词**的使用就是其中之一，但它并不止于此:

```
public class ImmutableData {
    private final String someData;
    private final AnotherImmutableData anotherImmutableData;
    public ImmutableData(final String someData, final AnotherImmutableData anotherImmutableData) {
        this.someData = someData;
        this.anotherImmutableData = anotherImmutableData;
    }
    public String getSomeData() {
        return someData;
    }
    public AnotherImmutableData getAnotherImmutableData() {
        return anotherImmutableData;
    }
}

public class AnotherImmutableData {
    private final Integer someOtherData;
    public AnotherImmutableData(final Integer someData) {
        this.someOtherData = someData;
    }
    public Integer getSomeOtherData() {
        return someOtherData;
    }
}
```

请注意，我们必须努力遵守一些规则:

*   不可变数据结构的所有字段必须是不可变的。
*   这也必须适用于所有嵌套类型和集合(包括它们包含的内容)。
*   根据需要，应该有一个或多个用于初始化的构造函数。
*   应该只有访问器方法，可能没有副作用。

每次都完全正确是不容易的，尤其是当数据结构开始变得复杂的时候。

然而，几个外部库可以使在 Java 中处理不可变数据变得更容易。例如， [Immutables](/web/20221129014542/https://www.baeldung.com/immutables) 和 [Project Lombok](/web/20221129014542/https://www.baeldung.com/intro-to-project-lombok) 提供了在 Java 中定义不可变数据结构的现成框架。

### 3.4.对透明性有关的

引用透明性可能是函数式编程中比较难理解的原则之一，但是这个概念非常简单。

如果用相应的值替换一个表达式对程序的行为没有影响，我们称之为引用透明。

这使得函数式编程中的一些强大技术成为可能，比如高阶函数和惰性求值。

为了更好地理解这一点，让我们举个例子:

```
public class SimpleData {
    private Logger logger = Logger.getGlobal();
    private String data;
    public String getData() {
        logger.log(Level.INFO, "Get data called for SimpleData");
        return data;
    }
    public SimpleData setData(String data) {
        logger.log(Level.INFO, "Set data called for SimpleData");
        this.data = data;
        return this;
    }
}
```

这是 Java 中一个典型的 POJO 类，但是我们很想知道它是否提供了引用透明性。

让我们观察以下陈述:

```
String data = new SimpleData().setData("Baeldung").getData();
logger.log(Level.INFO, new SimpleData().setData("Baeldung").getData());
logger.log(Level.INFO, data);
logger.log(Level.INFO, "Baeldung");
```

对`logger`的三个调用在语义上是等价的，但是在引用上是不透明的。

第一个调用不是引用透明的，因为它会产生副作用。如果我们像在第三个调用中那样用它的值替换这个调用，我们将会错过日志。

第二个调用也不是引用透明的，因为`SimpleData`是可变的。在程序的任何地方调用`data.setData`都很难用它的值来替换它。

因此，对于引用透明性，我们需要函数是纯的和不可变的。这是我们之前讨论的两个前提条件。

作为引用透明的一个有趣的结果，我们产生了上下文无关的代码。换句话说，我们可以在任何顺序和上下文中运行它们，这导致了不同的优化可能性。

## 4.函数式编程技术

我们前面讨论的函数式编程原则使我们能够使用多种技术从函数式编程中获益。

在这一节中，我们将介绍一些流行的技术，并理解如何在 Java 中实现它们。

### 4.1.功能组成

函数组合**是指将较简单的函数组合成复杂的函数。**

这主要是在 Java 中使用函数接口实现的，函数接口是 lambda 表达式和方法引用的目标类型。

通常，任何具有单一抽象方法的接口都可以作为功能接口。因此，我们可以很容易地定义一个功能接口。

但是 [Java 8 在`java.util.function`包下为我们默认提供了很多针对不同用例的功能接口](/web/20221129014542/https://www.baeldung.com/java-8-functional-interfaces)。

这些函数接口中的许多都根据`default`和`static`方法提供了对函数组合的支持。让我们挑选`Function`界面来更好地理解这一点。

是一个简单通用的函数接口，接受一个参数并产生一个结果。

它还提供了两个默认方法，`compose`和`andThen`，这将有助于我们进行函数组合:

```
Function<Double, Double> log = (value) -> Math.log(value);
Function<Double, Double> sqrt = (value) -> Math.sqrt(value);
Function<Double, Double> logThenSqrt = sqrt.compose(log);
logger.log(Level.INFO, String.valueOf(logThenSqrt.apply(3.14)));
// Output: 1.06
Function<Double, Double> sqrtThenLog = sqrt.andThen(log);
logger.log(Level.INFO, String.valueOf(sqrtThenLog.apply(3.14)));
// Output: 0.57
```

这两种方法都允许我们将多个函数组合成一个函数，但是提供不同的语义。当`compose`首先应用参数中传递的函数，然后应用被调用的函数时，`andThen`反过来做同样的事情。

其他几个函数接口都有**有趣的方法用于函数组合**，比如`Predicate`接口中的默认方法`and`、 `or`和`negate`。虽然这些函数接口接受单个参数，但也有[双元专门化](/web/20221129014542/https://www.baeldung.com/java-8-functional-interfaces#Specializations)，比如`BiFunction`和`BiPredicate`。

### 4.2.单子

许多函数式编程概念源自[范畴理论](https://web.archive.org/web/20221129014542/https://plato.stanford.edu/entries/category-theory/)，后者是**数学中函数的一般理论。**它提出了几个范畴的概念，如函子和自然变换。

对我们来说，知道这是在函数式编程中使用单子的基础是非常重要的。

形式上，单子是一种抽象，它允许一般地构造程序。因此，单子**允许我们包装一个值，应用一组转换，并在应用了所有转换的情况下取回该值。**

当然，任何单子都需要遵循三条定律——左恒等、右恒等和结合律——但这里我们不会深入讨论细节。

在 Java 中，有一些我们经常使用的单子，比如`Optional`和`Stream`:

```
Optional.of(2).flatMap(f -> Optional.of(3).flatMap(s -> Optional.of(f + s)))
```

为什么我们称`Optional`为单子？

这里`Optional`允许我们使用方法`of`包装一个值，并应用一系列转换。我们正在使用方法`flatMap`应用添加另一个包装值的转换。

我们可以证明`Optional`遵循单子三定律。然而，在某些情况下，`Optional`确实违反了单子定律。但是对于大多数实际情况来说，它应该足够好了。

如果我们理解了单子的基础，我们很快就会意识到 Java 中还有很多其他的例子，比如`Stream`和`CompletableFuture`。它们帮助我们实现不同的目标，但是它们都有一个标准的组成，在其中处理上下文操作或转换。

当然，**我们可以在 Java 中定义自己的单子类型来实现不同的目标**比如日志单子，报表单子或者审计单子。例如，monad 是处理函数式编程中副作用的函数式编程技术之一。

### 4.3.Currying

[Currying](/web/20221129014542/https://www.baeldung.com/java-currying) 是一种数学上的**技术，将一个带多个参数的函数转换成一系列带单个参数的函数。**

在函数式编程中，它为我们提供了一种强大的组合技术，在这种技术中，我们不需要调用带有所有参数的函数。

此外，一个 curried 函数直到接收到所有的参数时才会意识到它的作用。

在 Haskell 这样的纯函数式编程语言中，currying 得到了很好的支持。事实上，所有的功能都是默认设置的。

然而，在 Java 中，这并不简单:

```
Function<Double, Function<Double, Double>> weight = mass -> gravity -> mass * gravity;

Function<Double, Double> weightOnEarth = weight.apply(9.81);
logger.log(Level.INFO, "My weight on Earth: " + weightOnEarth.apply(60.0));

Function<Double, Double> weightOnMars = weight.apply(3.75);
logger.log(Level.INFO, "My weight on Mars: " + weightOnMars.apply(60.0));
```

这里我们定义了一个函数来计算我们在行星上的重量。虽然我们的质量保持不变，但重力因我们所在的星球而异。

我们**可以通过只传递重力来部分应用函数**来为一个特定的行星定义一个函数。此外，我们可以将这个部分应用的函数作为任意组合的参数或返回值来传递。

Currying **依赖语言提供两个基本特性:lambda 表达式和闭包。** Lambda 表达式是匿名函数，帮助我们将代码视为数据。我们之前已经看到了如何使用函数接口来实现它们。

一个 lambda 表达式可以在其词法范围上封闭，我们将其定义为闭包。

让我们看一个例子:

```
private static Function<Double, Double> weightOnEarth() {	
    final double gravity = 9.81;	
    return mass -> mass * gravity;
}
```

请注意我们在上面的方法中返回的 lambda 表达式是如何依赖于封闭变量的，我们称之为 closure。与其他函数式编程语言不同， **Java 有一个限制，即封闭范围必须是 [final 或有效的 final](/web/20221129014542/https://www.baeldung.com/java-effectively-final) 。**

作为一个有趣的结果，currying 还允许我们在 Java 中创建任意 arity 的函数接口。

### 4.4.递归

递归是函数式编程中另一项强大的技术，T2 允许我们将问题分解成更小的部分。递归的主要好处是帮助我们消除副作用，这是任何命令式循环的典型特征。

让我们看看如何使用递归计算一个数的阶乘:

```
Integer factorial(Integer number) {
    return (number == 1) ? 1 : number * factorial(number - 1);
}
```

这里我们递归地调用同一个函数，直到我们到达基本情况，然后开始计算我们的结果。

请注意，我们在计算每一步的结果之前或者在计算的开头进行递归调用。所以，这种类型的递归也被称为头部递归。

这种递归的一个缺点是，每一步都必须保持前面所有步骤的状态，直到我们到达基本情况。对于小数字来说，这不是一个问题，但是对于大数字来说，保持状态可能是低效的。

一个解决方案是称为尾部递归的略微不同的递归实现。这里我们确保递归调用是函数进行的最后一次调用。

让我们看看如何重写上面的函数来使用尾部递归:

```
Integer factorial(Integer number, Integer result) {
    return (number == 1) ? result : factorial(number - 1, result * number);
}
```

注意函数中累加器的使用，消除了在递归的每一步保持状态的需要。这种风格的真正好处是利用编译器优化，编译器可以决定放弃当前函数的堆栈框架，这种技术被称为尾调用消除。

虽然许多语言(如 Scala)支持尾调用消除，但 Java 仍然不支持这一点。这是 Java 待办事项的一部分，可能会以某种形式出现，作为在[项目下提出的更大变化的一部分。](https://web.archive.org/web/20221129014542/http://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html)

## 5.为什么函数式编程很重要

现在，我们可能想知道为什么我们要做这么多努力。对于 Java 背景的人来说，函数式编程所要求的转变并不是微不足道的。因此，在 Java 中采用函数式编程应该有一些真正有前途的优势。

在包括 Java 在内的任何语言中采用函数式编程的最大优势就是**纯函数和不可变状态。**如果我们回想一下，大多数编程挑战都源于副作用和各种可变状态。简单地去掉它们**使得我们的程序更容易阅读、推理、测试和维护。**

声明式编程**导致非常简洁和可读的程序。**作为声明式编程的子集，函数式编程提供了高阶函数、函数组合和函数链等多种构造。想想 Stream API 给 Java 8 带来的处理数据操作的好处。

但是除非完全准备好了，否则不要尝试转换。请注意，函数式编程不是一个我们可以立即使用并从中受益的简单设计模式。

函数式编程**更多的是改变了我们思考问题及其解决方案的方式**以及如何构建算法。

所以，在我们开始使用函数式编程之前，我们必须训练自己从函数的角度来思考我们的程序。

## 6.Java 合适吗？

很难否认函数式编程的好处，但是 Java 是它的合适选择吗？

从历史上看，Java 是作为一种更适合面向对象编程的通用编程语言而发展起来的。在 Java 8 之前，甚至连考虑使用函数式编程都是乏味的！但在 Java 8 之后，事情肯定发生了变化。

Java 中没有真正的函数类型，这违背了函数式编程的基本原则。伪装成 lambda 表达式的函数接口很大程度上弥补了这一点，至少在语法上是这样。

那么，Java 中的**类型本质上是可变的**，我们不得不写这么多样板文件来创建不可变的类型，这并没有什么帮助。

我们期望从函数式编程语言中得到 Java 中缺少或难以得到的其他东西。例如，Java 中参数的默认评估策略是 eager。但懒求值是函数式编程中更高效、更值得推荐的方式。

我们仍然可以使用操作符短路和函数接口在 Java 中实现惰性求值，但是它更复杂。

这个列表当然是不完整的，可能包括类型擦除的泛型支持，缺少对尾部调用优化的支持等等。然而，我们得到了一个宽泛的概念。

**Java 绝对不适合在函数式编程中从头开始一个程序。**

但是，如果我们已经有了一个用 Java 编写的现有程序，很可能是用面向对象编程编写的，该怎么办呢？没有什么能阻止我们获得函数式编程的一些好处，尤其是在 Java 8 中。

对于 Java 开发人员来说，这就是函数式编程的大部分好处所在。将面向对象编程与函数式编程的优点结合起来会大有裨益。

## 7.结论

在本文中，我们学习了函数式编程的基础知识。我们讨论了基本原则以及如何在 Java 中采用它们。

此外，我们讨论了函数式编程中的一些流行技术，并给出了 Java 示例。

最后，我们讨论了采用函数式编程的一些好处，并回答了 Java 是否同样适用。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221129014542/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-functional)