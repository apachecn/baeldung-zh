# Java 中的常量:模式和反模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constants-good-practices>

## 1.介绍

在本文中，我们将学习在 Java 中使用常量，重点是常见模式和反模式。

我们将从定义常数的一些基本约定开始。从这里开始，我们将讨论常见的反模式，然后再看常见的模式。

## 2.基础

常量是一个变量，它的值在被定义后不会改变。

让我们看看定义常数的基础:

```java
private static final int OUR_CONSTANT = 1;
```

我们将看到的一些模式将解决`[public](/web/20220926183451/https://www.baeldung.com/java-public-keyword)`或 [`private`](/web/20220926183451/https://www.baeldung.com/java-private-keyword) [访问修饰符](/web/20220926183451/https://www.baeldung.com/java-access-modifiers)决策。我们创建我们的常量`static`和`[final](/web/20220926183451/https://www.baeldung.com/java-final)`，并给它们一个合适的类型，无论是 Java 原语、类还是`enum`。**名字应该全是大写字母，单词之间用下划线隔开**，有时被称为尖叫蛇案。最后，我们提供价值本身。

## 3.反模式

首先，让我们从学习什么不该做开始。让我们看看在使用 Java 常量时可能会遇到的几种常见的反模式。

### 3.1.神奇的数字

**[幻数](/web/20220926183451/https://www.baeldung.com/cs/antipatterns-magic-numbers)是代码块中的数值:**

```java
if (number == 3.14159265359) {
    // ...
}
```

其他开发人员很难理解它们。此外，如果我们在整个代码中使用一个数字，就很难改变这个值。我们应该把这个数字定义为一个常数。

### 3.2.大型全局常量类

当我们开始一个项目时，创建一个名为`Constants`或`Utils`的类，目的是为那里的应用程序定义所有的常量，这可能是很自然的。对于较小的项目，这可能没问题，但是让我们考虑一下为什么这不是一个理想的解决方案的几个原因。

首先，让我们想象一下，我们的 constants 类中有一百个或更多的常量。如果不维护这个类，以跟上文档的进度，并偶尔将常量重构为逻辑分组，那么它将变得非常难以阅读。我们甚至可能以名称略有不同的重复常量告终。除了最小的项目，这种方法很可能会给我们带来可读性和可维护性的问题。

除了维护`Constants`类本身的逻辑之外，我们还通过鼓励这个全局常量类和应用程序的其他各部分之间过多的相互依赖，带来了其他的可维护性问题。

从更技术性的角度来看，**Java 编译器将常量的值放入我们使用它们的类的引用变量中**。因此，如果我们改变了 constants 类中的一个常量，并且只重新编译了那个类而没有重新编译引用类，我们可能会得到不一致的常量值。

### 3.3.常量接口反模式

常量接口模式是指我们定义一个包含特定功能的所有常量的接口，然后让需要这些功能的类来实现该接口。

让我们为计算器定义一个常量接口:

```java
public interface CalculatorConstants {
    double PI = 3.14159265359;
    double UPPER_LIMIT = 0x1.fffffffffffffP+1023;
    enum Operation {ADD, SUBTRACT, MULTIPLY, DIVIDE};
}
```

接下来，我们将实现我们的`CalculatorConstants`接口:

```java
public class GeometryCalculator implements CalculatorConstants {    
    public double operateOnTwoNumbers(double numberOne, double numberTwo, Operation operation) {
       // Code to do an operation
    }
}
```

反对使用常量接口的第一个理由是它违背了接口的目的。我们应该使用接口为我们的实现类将要提供的行为创建一个契约。当我们创建一个充满常量的接口时，我们没有定义任何行为。

其次，使用常量接口会给我们带来由字段隐藏引起的运行时问题。让我们通过在我们的`GeometryCalculator`类中定义一个`UPPER_LIMIT`常量来看看这是如何发生的:

```java
public static final double UPPER_LIMIT = 100000000000000000000.0;
```

一旦我们在我们的`GeometryCalculator`类中定义了这个常量，我们就在我们的类的`CalculatorConstants`接口中隐藏这个值。我们可能会得到意想不到的结果。

反对这种反模式的另一个理由是，它会导致名称空间污染。我们的`CalculatorConstants`现在将位于实现该接口的任何类及其任何子类的名称空间中。

## 4.模式

前面，我们看了定义常数的适当形式。让我们看看在应用程序中定义常量的其他一些好的实践。

### 4.1.一般良好做法

如果常量在逻辑上与一个类相关，我们可以在那里定义它们。如果我们将一组常量视为枚举类型的成员，我们可以使用`enum`来定义它们。

让我们在一个`Calculator`类中定义一些常量:

```java
public class Calculator {
    public static final double PI = 3.14159265359;
    private static final double UPPER_LIMIT = 0x1.fffffffffffffP+1023;
    public enum Operation {
        ADD,
        SUBTRACT,
        DIVIDE,
        MULTIPLY
    }

    public double operateOnTwoNumbers(double numberOne, double numberTwo, Operation operation) {
        if (numberOne > UPPER_LIMIT) {
            throw new IllegalArgumentException("'numberOne' is too large");
        }
        if (numberTwo > UPPER_LIMIT) {
            throw new IllegalArgumentException("'numberTwo' is too large");
        }
        double answer = 0;

        switch(operation) {
            case ADD:
                answer = numberOne + numberTwo;
                break;
            case SUBTRACT:
                answer = numberOne - numberTwo;
                break;
            case DIVIDE:
                answer = numberOne / numberTwo;
                break;
            case MULTIPLY:
                answer = numberOne * numberTwo;
                break;
        }

        return answer;
    }
}
```

在我们的例子中，我们已经为`UPPER_LIMIT`定义了一个常量，我们只打算在`Calculator`类中使用它，所以我们将它设置为`private`。我们希望其他类能够使用`PI`和`Operation`枚举，所以我们将它们设置为`public`。

让我们考虑一下对`Operation`使用`enum`的一些优点。第一个优点是它限制了可能的值。假设我们的方法接受一个字符串作为操作值，并期望提供四个常量字符串中的一个。我们可以很容易地预见一个场景，其中调用该方法的开发人员发送他们自己的字符串值。**有了`enum`，值就被限制在我们定义的范围内。**我们还可以看到枚举特别适合用在 [`switch`](/web/20220926183451/https://www.baeldung.com/java-switch) 语句中。

### 4.2.常量类

既然我们已经了解了一些通用的良好实践，让我们考虑一下常量类可能是个好主意的情况。假设我们的应用程序包含一个需要进行各种数学计算的类包。在这种情况下，我们在这个包中为我们将在计算类中使用的常量定义一个常量类可能是有意义的。

让我们创建一个`MathConstants`类:

```java
public final class MathConstants {
    public static final double PI = 3.14159265359;
    static final double GOLDEN_RATIO = 1.6180;
    static final double GRAVITATIONAL_ACCELERATION = 9.8;
    static final double EULERS_NUMBER = 2.7182818284590452353602874713527;

    public enum Operation {
        ADD,
        SUBTRACT,
        DIVIDE,
        MULTIPLY
    }

    private MathConstants() {

    }
}
```

我们首先要注意的是**我们的类是`final`以防止它被扩展**。此外，我们已经定义了一个`private`构造函数，所以它不能被实例化。最后，我们可以看到我们已经应用了本文前面讨论的其他良好实践。我们的常量`PI`是`public`，因为我们预期需要在我们的包之外访问它。其他常量我们保留为`package-private`，所以我们可以在我们的包中访问它们。我们制作了所有的常量`static`和`final`，并在一个尖叫的蛇盒中命名它们。操作是一组特定的值，所以我们使用了一个`enum`来定义它们。

我们可以看到，我们特定的包级常量类不同于大型的全局常量类，因为它被本地化到我们的包中，并且包含与该包的类相关的常量。

## 5.结论

在本文中，我们考虑了在 Java 中使用常量时一些最流行的模式和反模式的优缺点。在介绍反模式之前，我们从一些基本的格式规则开始。在学习了一些常见的反模式之后，我们看了一些常见的应用于常量的模式。

与往常一样，GitHub 上的[代码是可用的。](https://web.archive.org/web/20220926183451/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)