# Lambda 表达式和函数接口:提示和最佳实践

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-lambda-expressions-tips>

## 1。概述

既然 Java 8 已经被广泛使用，模式和最佳实践已经开始出现在它的一些主要特性中。在本教程中，我们将仔细研究函数接口和 lambda 表达式。

## 延伸阅读:

## 为什么 Lambdas 中使用的局部变量必须是最终的或有效的最终变量？

Learn why Java requires local variables to be effectively final when used in a lambda.[Read more](/web/20221001033555/https://www.baeldung.com/java-lambda-effectively-final-local-variables) →

## [Java 8——与 Lambdas 的强大对比](/web/20221001033555/https://www.baeldung.com/java-8-sort-lambda)

Elegant Sort in Java 8 - Lambda Expressions go right past syntactic sugar and bring powerful functional semantics into Java.[Read more](/web/20221001033555/https://www.baeldung.com/java-8-sort-lambda) →

## 2。首选标准功能接口

聚集在**[Java . util . function](https://web.archive.org/web/20221001033555/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html)**包中的函数接口，满足了大多数开发者为 lambda 表达式和方法引用提供目标类型的需求。这些接口中的每一个都是通用和抽象的，使得它们很容易适应几乎任何 lambda 表达式。开发人员应该在创建新的功能接口之前研究这个包。

让我们考虑一个接口` Foo`:

```java
@FunctionalInterface
public interface Foo {
    String method(String string);
}
```

另外，我们在某个类`UseFoo`中有一个方法`add() `，它把这个接口作为一个参数:

```java
public String add(String string, Foo foo) {
    return foo.method(string);
}
```

要执行它，我们应该写:

```java
Foo foo = parameter -> parameter + " from lambda";
String result = useFoo.add("Message ", foo);
```

如果我们仔细观察，我们会发现`Foo`只不过是一个接受一个参数并产生一个结果的函数。Java 8 已经在 [java.util.function](https://web.archive.org/web/20221001033555/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html) 包的`[Function<T,R>](https://web.archive.org/web/20221001033555/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html)`中提供了这样一个接口。

现在我们可以完全移除接口`Foo`，并将代码改为:

```java
public String add(String string, Function<String, String> fn) {
    return fn.apply(string);
}
```

要执行此操作，我们可以编写:

```java
Function<String, String> fn = 
  parameter -> parameter + " from lambda";
String result = useFoo.add("Message ", fn);
```

## 3。使用`@FunctionalInterface`标注

现在让我们用`[@FunctionalInterface](https://web.archive.org/web/20221001033555/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/FunctionalInterface.html).`来注释我们的函数接口起初，这个注释似乎没有用。即使没有它，只要它只有一个抽象方法，我们的接口也将被视为功能性的。

然而，让我们想象一个有几个接口的大项目；手动控制一切很难。一个被设计为功能性的接口，可能会由于添加了另一个抽象方法而被意外地改变，使其不能作为功能性接口使用。

通过使用`@FunctionalInterface`注释，编译器将触发一个错误，以响应任何破坏函数接口的预定义结构的尝试。它也是一个非常方便的工具，可以让其他开发人员更容易理解我们的应用程序架构。

所以我们可以用这个:

```java
@FunctionalInterface
public interface Foo {
    String method();
}
```

而不仅仅是:

```java
public interface Foo {
    String method();
}
```

## 4。不要在函数接口中过度使用默认方法

我们可以很容易地将默认方法添加到函数接口中。只要只有一个抽象方法声明，函数接口契约就可以接受这一点:

```java
@FunctionalInterface
public interface Foo {
    String method(String string);
    default void defaultMethod() {}
}
```

如果抽象方法具有相同的签名，函数接口可以由其他函数接口扩展:

```java
@FunctionalInterface
public interface FooExtended extends Baz, Bar {}

@FunctionalInterface
public interface Baz {	
    String method(String string);	
    default String defaultBaz() {}		
}

@FunctionalInterface
public interface Bar {	
    String method(String string);	
    default String defaultBar() {}	
}
```

正如常规接口一样，**用相同的默认方法扩展不同的功能接口可能会有问题**。

例如，让我们将`defaultCommon()`方法添加到`Bar`和`Baz`接口中:

```java
@FunctionalInterface
public interface Baz {
    String method(String string);
    default String defaultBaz() {}
    default String defaultCommon(){}
}

@FunctionalInterface
public interface Bar {
    String method(String string);
    default String defaultBar() {}
    default String defaultCommon() {}
}
```

在这种情况下，我们将得到一个编译时错误:

```java
interface FooExtended inherits unrelated defaults for defaultCommon() from types Baz and Bar...
```

要解决这个问题，应该在`FooExtended`接口中覆盖`defaultCommon()`方法。我们可以提供该方法的自定义实现；然而，**我们也可以重用来自父接口**的实现:

```java
@FunctionalInterface
public interface FooExtended extends Baz, Bar {
    @Override
    default String defaultCommon() {
        return Bar.super.defaultCommon();
    }
}
```

需要注意的是，我们必须小心。向接口添加太多默认方法不是一个很好的架构决策。这应被视为一种折衷，仅在需要升级现有接口而不破坏向后兼容性时使用。

## 5。用 Lambda 表达式实例化函数接口

编译器将允许我们使用一个内部类来实例化一个函数接口；然而，这可能导致非常冗长的代码。我们应该更喜欢使用 lambda 表达式:

```java
Foo foo = parameter -> parameter + " from Foo";
```

在内部类上:

```java
Foo fooByIC = new Foo() {
    @Override
    public String method(String string) {
        return string + " from Foo";
    }
}; 
```

**lambda 表达式方法可用于旧库中任何合适的接口。**可用于`Runnable`、`Comparator`等接口； **h** **然而，这** **并不意味着我们应该回顾我们的整个旧代码库并改变一切。**

## 6。避免将函数接口作为参数重载方法

我们应该使用不同名称的方法来避免冲突:

```java
public interface Processor {
    String process(Callable<String> c) throws Exception;
    String process(Supplier<String> s);
}

public class ProcessorImpl implements Processor {
    @Override
    public String process(Callable<String> c) throws Exception {
        // implementation details
    }

    @Override
    public String process(Supplier<String> s) {
        // implementation details
    }
}
```

乍一看，这似乎是合理的，但是任何执行`ProcessorImpl`的方法的尝试:

```java
String result = processor.process(() -> "abc");
```

以包含以下消息的错误结束:

```java
reference to process is ambiguous
both method process(java.util.concurrent.Callable<java.lang.String>) 
in com.baeldung.java8.lambda.tips.ProcessorImpl 
and method process(java.util.function.Supplier<java.lang.String>) 
in com.baeldung.java8.lambda.tips.ProcessorImpl match
```

要解决这个问题，我们有两个选择。**第一个选项是使用不同名称的方法:**

```java
String processWithCallable(Callable<String> c) throws Exception;

String processWithSupplier(Supplier<String> s);
```

**第二个选项是手动执行铸造，**这不是首选:

```java
String result = processor.process((Supplier<String>) () -> "abc");
```

## 7。不要将 Lambda 表达式视为内部类

尽管在前面的例子中，我们用 lambda 表达式替换了 inner class，但这两个概念在一个重要方面是不同的:作用域。

当我们使用内部类时，它创建了一个新的作用域。我们可以通过实例化同名的新局部变量来隐藏封闭范围内的局部变量。我们还可以在内部类中使用关键字`**this**`作为对其实例的引用。

然而，Lambda 表达式使用封闭范围。我们不能隐藏 lambda 主体内部封闭范围的变量。在这种情况下，关键字`**this**`是对封闭实例的引用。

例如，在类`UseFoo,` 中，我们有一个实例变量`value:`

```java
private String value = "Enclosing scope value";
```

然后在该类的某个方法中，放置以下代码并执行该方法:

```java
public String scopeExperiment() {
    Foo fooIC = new Foo() {
        String value = "Inner class value";

        @Override
        public String method(String string) {
            return this.value;
        }
    };
    String resultIC = fooIC.method("");

    Foo fooLambda = parameter -> {
        String value = "Lambda value";
        return this.value;
    };
    String resultLambda = fooLambda.method("");

    return "Results: resultIC = " + resultIC + 
      ", resultLambda = " + resultLambda;
}
```

如果我们执行`scopeExperiment()`方法，我们将得到以下结果:`Results: resultIC = Inner class value, resultLambda = Enclosing scope value`

正如我们所看到的，通过在 IC 中调用`this.value`，我们可以从一个局部变量的实例中访问它。在 lambda 的例子中，`this.value`调用让我们可以访问在`UseFoo`类中定义的变量`value,` ，但是不能访问在 lambda 主体中定义的变量`value` 。

## 8。保持 Lambda 表达式简短明了

如果可能的话，我们应该使用单行结构，而不是一大段代码。记住， **lambdas 应该是一个** **的表达，而不是叙述。**尽管语法简洁， **lambdas 应该明确表达它们提供的功能。**

这主要是风格上的建议，因为性能不会有很大的变化。然而，一般来说，理解和使用这样的代码要容易得多。

这可以通过多种方式实现；让我们仔细看看。

### 8.1。避免 Lambda 主体中的代码块

在理想情况下，lambdas 应该用一行代码编写。使用这种方法，lambda 是一个不言自明的结构，它声明应该用什么数据执行什么操作(在 lambda 带有参数的情况下)。

如果我们有一个很大的代码块，lambda 的功能并不清晰。

考虑到这一点，请执行以下操作:

```java
Foo foo = parameter -> buildString(parameter);
```

```java
private String buildString(String parameter) {
    String result = "Something " + parameter;
    //many lines of code
    return result;
}
```

而不是:

```java
Foo foo = parameter -> { String result = "Something " + parameter; 
    //many lines of code 
    return result; 
};
```

需要注意的是，我们不应该把这条“单线λ”规则当成教条。如果我们在 lambda 的定义中有两三行代码，那么将这些代码提取到另一个方法中可能没有价值。

### 8.2。避免指定参数类型

在大多数情况下，编译器能够借助 **[类型推断](https://web.archive.org/web/20221001033555/https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html)** 来解析 lambda 参数的类型。因此，向参数添加类型是可选的，可以省略。

我们可以这样做:

```java
(a, b) -> a.toLowerCase() + b.toLowerCase();
```

而不是这个:

```java
(String a, String b) -> a.toLowerCase() + b.toLowerCase();
```

### 8.3。避免在单个参数周围使用括号

Lambda 语法只需要在多个参数周围加上括号，或者根本没有参数。这就是为什么让我们的代码稍微短一点是安全的，并且当只有一个参数时排除括号。

所以我们可以这样做:

```java
a -> a.toLowerCase();
```

而不是这个:

```java
(a) -> a.toLowerCase();
```

### 8.4。避免返回语句和大括号

**大括号**和`**return**`语句在单行 lambda 体中是可选的。这意味着为了清楚和简明起见，可以省略它们。

我们可以这样做:

```java
a -> a.toLowerCase();
```

而不是这个:

```java
a -> {return a.toLowerCase()};
```

### 8.5。使用方法引用

甚至在我们之前的例子中，lambda 表达式经常只是调用已经在别处实现的方法。在这种情况下，使用 Java 8 的另一个特性 **[方法引用](https://web.archive.org/web/20221001033555/https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)** 非常有用。

λ表达式将是:

```java
a -> a.toLowerCase();
```

我们可以替换为:

```java
String::toLowerCase;
```

这并不总是更短，但它使代码更具可读性。

## 9。使用“最终有效”变量

访问 lambda 表达式中的非最终变量会导致编译时错误， **b** **但这并不意味着我们应该将每个目标变量都标记为`final.`**

根据“ **[有效最终](https://web.archive.org/web/20221001033555/https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html)** 的概念，编译器将每个变量视为`final`，只要它只被赋值一次。

在 lambdas 中使用这样的变量是安全的，因为编译器将控制它们的状态，并在试图更改它们后立即触发编译时错误。

例如，以下代码将不会编译:

```java
public void method() {
    String localVariable = "Local";
    Foo foo = parameter -> {
        String localVariable = parameter;
        return localVariable;
    };
}
```

编译器会通知我们:

```java
Variable 'localVariable' is already defined in the scope.
```

这种方法可以简化 lambda 执行线程安全的过程。

## 10。保护对象变量免受突变

lambdas 的主要用途之一是用于并行计算，这意味着它们在线程安全方面非常有用。

“有效最终”范式在这里很有帮助，但并不是在所有情况下都是如此。Lambdas 不能从封闭范围中更改对象的值。但是在可变对象变量的情况下，状态可以在 lambda 表达式中改变。

考虑以下代码:

```java
int[] total = new int[1];
Runnable r = () -> total[0]++;
r.run();
```

这段代码是合法的，因为 `total` 变量仍然是“有效的最终变量”，但是它引用的对象在 lambda 执行后会有相同的状态吗？不要！

保留这个例子作为提醒，以避免可能导致意外突变的代码。

## 11。结论

在本文中，我们探讨了 Java 8 的 lambda 表达式和函数接口中的一些最佳实践和缺陷。尽管这些新功能很有用，很强大，但它们只是工具。每个开发人员在使用它们时都应该注意。

在[GitHub 项目](https://web.archive.org/web/20221001033555/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas)中可以获得该示例的完整**源代码**。这是一个 Maven 和 Eclipse 项目，因此可以导入并按原样使用。