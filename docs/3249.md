# Java final 关键字–对性能的影响

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-final-performance>

## 1.概观

使用`final`关键字的性能优势是 Java 开发人员中一个非常流行的争论话题。根据应用的不同，**`final`关键字可以有不同的用途和不同的性能含义**。

在本教程中，我们将探索在代码中使用`final`关键字是否有任何性能上的好处。我们将看看在变量、方法和类级别上使用`final`的性能含义。

除了性能，我们还将提到使用`final`关键字的设计方面。最后，我们将推荐我们是否应该使用它，以及为什么要使用它。

## 2.局部变量

当 [`final`](/web/20220628060709/https://www.baeldung.com/java-final) 应用于局部变量时，其**值必须被精确赋值一次**。

我们可以在最终变量声明或类构造函数中赋值。万一我们以后试图改变最终的变量值，编译器将抛出一个错误。

### 2.1.特性试验

让我们看看将`final`关键字应用于局部变量是否可以提高性能。

我们将利用 [JMH 工具](/web/20220628060709/https://www.baeldung.com/java-microbenchmark-harness)来测量基准测试方法的平均执行时间。在我们的基准方法中，我们将对非最终局部变量进行简单的字符串连接:

```
@Benchmark
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@BenchmarkMode(Mode.AverageTime)
public static String concatNonFinalStrings() {
    String x = "x";
    String y = "y";
    return x + y;
}
```

接下来，我们将重复相同的性能测试，但这次使用最终的局部变量:

```
@Benchmark
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@BenchmarkMode(Mode.AverageTime)
public static String concatFinalStrings() {
    final String x = "x";
    final String y = "y";
    return x + y;
}
```

JMH 将负责运行预热迭代，以便让 [JIT 编译器](/web/20220628060709/https://www.baeldung.com/ahead-of-time-compilation)优化生效。最后，让我们来看看以纳秒为单位的平均性能测量值:

```
Benchmark                              Mode  Cnt  Score   Error  Units
BenchmarkRunner.concatFinalStrings     avgt  200  2,976 ± 0,035  ns/op
BenchmarkRunner.concatNonFinalStrings  avgt  200  7,375 ± 0,119  ns/op
```

在我们的例子中，使用 final 局部变量使执行速度提高了 2.5 倍。

### 2.2.静态代码优化

字符串连接示例演示了**`final`关键字如何帮助编译器静态优化代码**。

使用非最终局部变量，编译器生成以下字节码来连接两个字符串:

```
NEW java/lang/StringBuilder
DUP
INVOKESPECIAL java/lang/StringBuilder.<init> ()V
ALOAD 0
INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
ALOAD 1
INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
INVOKEVIRTUAL java/lang/StringBuilder.toString ()Ljava/lang/String;
ARETURN
```

通过添加`final`关键字，我们帮助编译器断定字符串连接结果实际上永远不会改变。因此，编译器能够完全避免字符串连接，并静态优化生成的字节码:

```
LDC "xy"
ARETURN
```

我们应该注意到，在大多数情况下，将`final`添加到我们的局部变量不会像本例中那样带来显著的性能优势。

## 3.实例和类变量

我们可以将`final`关键字应用于实例或类变量。这样，我们可以确保它们的赋值只进行一次。我们可以在最终的实例变量声明时，在实例初始化块或构造函数中赋值。

通过将 [`static`](/web/20220628060709/https://www.baeldung.com/java-static) 关键字添加到类的成员变量来声明类变量。另外，**通过将`final` 关键字应用于一个类变量，我们定义了一个常量**。我们可以在常量声明时或者在静态初始化程序块中赋值:

```
static final boolean doX = false;
static final boolean doY = true;
```

让我们用使用这些`boolean`常量的条件编写一个简单的方法:

```
Console console = System.console();
if (doX) {
    console.writer().println("x");
} else if (doY) {
    console.writer().println("y");
}
```

接下来，让我们从`boolean`类变量中删除`final`关键字，并比较类生成的字节码:

*   使用非最终类变量的例子——76 行字节码
*   使用最终类变量(常量)的示例–39 行字节码

通过给类变量**添加`final`关键字，我们再次帮助编译器执行静态代码优化**。编译器将简单地用最终类变量的实际值替换它们的所有引用。

然而，我们应该注意，像这样的例子很少在现实生活中的 Java 应用程序中使用。将变量声明为`final `只会对实际应用程序的性能产生较小的积极影响。

## 4.有效最终

术语[实际上是最终变量](/web/20220628060709/https://www.baeldung.com/java-effectively-final)是在 Java 8 中引入的。如果一个变量没有被显式声明为 final，但它的值在初始化后从不改变，那么它实际上就是 final。

有效最终变量的主要[目的是使 lambdas 能够使用没有明确声明为最终变量的局部变量。然而，Java **编译器不会像对最终变量那样对最终变量**进行静态代码优化。](/web/20220628060709/https://www.baeldung.com/java-lambda-effectively-final-local-variables)

## 5.类别和方法

`final`关键字在应用于类和方法时有不同的用途。当我们将关键字`final`应用于一个类时，这个类就不能被子类化。当我们将它应用于一个方法时，该方法就不能被覆盖。

目前还没有将`final`应用于类和方法的性能优势的报道。此外，final 类和方法可能会给开发人员带来极大的不便，因为它们限制了我们重用现有代码的选择。因此，鲁莽地使用`final`会损害良好的面向对象设计原则。

创建 final 类或方法有一些合理的理由，比如加强不变性。然而，**性能优势并不是在类和方法级别**上使用`final`的好理由。

## 6.性能与简洁设计

除了性能，我们可能会考虑使用`final`的其他原因。`final`关键字有助于提高代码的可读性和可理解性。让我们来看几个例子，看看`final`如何传达设计选择:

*   最终类是防止扩展的设计决策——这可能是通向[不可变对象](/web/20220628060709/https://www.baeldung.com/java-immutable-object)的途径
*   方法被声明为 final，以防止子类不兼容
*   方法参数被声明为 final 以防止副作用
*   最终变量被设计成只读的

因此，我们应该**使用`final` 与其他开发者**交流设计选择。此外，应用于变量的`final`关键字可以作为编译器执行较小性能优化的有用提示。

## 7.结论

在本文中，我们研究了使用`final`关键字的性能优势。在示例中，我们展示了**将`final`关键字应用于变量可以对性能**产生微小的积极影响。然而，对类和方法应用`final`关键字不会带来任何性能上的好处。

我们证明了，与 final 变量不同，编译器实际上不使用 final 变量来执行静态代码优化。最后，除了性能，我们还研究了在不同层次上应用`final` 关键字的设计含义。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628060709/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)