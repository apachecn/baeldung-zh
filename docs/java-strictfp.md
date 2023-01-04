# Java 中的 strictfp 关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-strictfp>

## 1.介绍

默认情况下，Java 中的浮点计算是平台相关的。因此，浮点结果的精度取决于使用的硬件。

在本教程中，我们将学习如何在 Java 中使用`strictfp`来确保平台无关的浮点计算。

## 2.`strictfp` 用法

我们可以使用`strictfp`关键字作为类、非抽象方法或接口的非访问修饰符:

```java
public strictfp class ScientificCalculator {
    ...

    public double sum(double value1, double value2) {
        return value1 + value2;
    }

    public double diff(double value1, double value2) { 
        return value1 - value2; 
    }
}

public strictfp void calculateMarksPercentage() {
    ...
}

public strictfp interface Circle {
    double computeArea(double radius);
}
```

当我们用`strictfp, `声明一个接口或类时，它的所有成员方法和其他嵌套类型都继承了它的行为。

然而，请**注意，我们不允许在变量、构造函数或抽象方法上使用`strictfp`关键字。**

此外，如果我们有一个标记了它的超类，它不会让我们的子类继承那个行为。

## 3.什么时候用？

每当我们非常关心所有浮点计算的确定性行为时，Java `strictfp`关键字就派上了用场:

```java
@Test
public void whenMethodOfstrictfpClassInvoked_thenIdenticalResultOnAllPlatforms() {
    ScientificCalculator calculator = new ScientificCalculator();
    double result = calculator.sum(23e10, 98e17);
    assertThat(result, is(9.800000230000001E18));

    result = calculator.diff(Double.MAX_VALUE, 1.56);
    assertThat(result, is(1.7976931348623157E308));
}
```

由于`ScientificCalculator`类使用了这个关键字，上面的测试用例将在所有硬件平台上通过。请注意，如果我们不使用**，JVM 可以自由使用目标平台硬件上任何额外的精度。**

它的一个流行的真实世界用例是一个执行高度敏感的医学计算的系统。

## 4.结论

在这个快速教程中，我们讨论了在 Java 中何时以及如何使用`strictfp`关键字。

和往常一样，GitHub 上的[提供了所有展示的代码示例。](https://web.archive.org/web/20220628115736/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-modifiers)