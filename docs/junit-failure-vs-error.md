# JUnit 中失败和错误的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-failure-vs-error>

## 1.介绍

在本教程中，我们将探索在 [JUnit](/web/20221205143713/https://www.baeldung.com/junit-5) 测试中失败和错误的区别。

简而言之，失败是未实现的断言，而错误是由于异常的测试执行。

## 2.示例代码

让我们考虑一个非常简单的例子，即一个计算器类，它有一个方法将两个`double`值相除:

```java
public static double divideNumbers(double dividend, double divisor) {  
    if (divisor == 0) { 
        throw new ArithmeticException("Division by zero!"); 
    } 
    return dividend / divisor; 
}
```

注意， **Java 实际上并没有为`double`除法独立抛出一个`ArithmeticException` T5——它返回`Infinity` 或`NaN`。**

## 3.失败示例

当用 JUnit 编写单元测试时，很可能会出现测试失败的情况。一种可能是我们的代码不符合它的测试标准。这意味着一个或多个测试用例由于**断言没有被满足而失败。**

在下面的示例中，断言将失败，因为除法的结果是 2 而不是 15。我们的断言和实际结果根本不相符:

```java
@Test
void whenDivideNumbers_thenExpectWrongResult() {
    double result = SimpleCalculator.divideNumbers(6, 3);
    assertEquals(15, result);
}
```

## 4.错误示例

另一种可能性是，我们在测试执行过程中遇到了**意外情况，很可能是由于异常**；例如，访问一个`null`引用将引发一个`RuntimeException`。

让我们看一个例子，测试将因错误而中止，因为我们试图除以零，我们通过在计算器代码中抛出一个异常来明确防止这种情况:

```java
@Test
void whenDivideByZero_thenThrowsException(){
    SimpleCalculator.divideNumbers(10, 0);
} 
```

现在，我们可以通过简单地将异常作为我们的[断言](/web/20221205143713/https://www.baeldung.com/junit-assert-exception)之一来修复这个测试。

```java
@Test
void whenDivideByZero_thenAssertException(){
    assertThrows(ArithmeticException.class, () -> SimpleCalculator.divideNumbers(10, 0));
}
```

然后，如果抛出异常，测试通过，但如果没有，那将是另一个失败。

## 5.结论

JUnit 测试中的失败和错误都表示不希望出现的情况，但是它们的语义是不同的。**失败表示无效的测试结果，错误表示意外的测试执行。**

另外，请在 [GitHub](https://web.archive.org/web/20221205143713/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics) 查看示例代码。