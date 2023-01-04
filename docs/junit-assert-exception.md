# 断言在 JUnit 4 和 5 中抛出异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-assert-exception>

## 1.介绍

在这个快速教程中，我们将了解如何使用 JUnit 库测试是否抛出了异常。

当然，我们会确保涵盖 JUnit 4 和 JUnit 5 版本。

## 延伸阅读:

## [AssertJ 异常断言](/web/20220830145516/https://www.baeldung.com/assertj-exception-assertion)

Learn how to use AssertJ for performing assertions on exceptions.[Read more](/web/20220830145516/https://www.baeldung.com/assertj-exception-assertion) →

## [JUnit 4 和 JUnit 5 中的断言](/web/20220830145516/https://www.baeldung.com/junit-assertions)

A look at assertions in both JUnit 4 and 5.[Read more](/web/20220830145516/https://www.baeldung.com/junit-assertions) →

## [使用 Mockito 模拟异常抛出](/web/20220830145516/https://www.baeldung.com/mockito-exceptions)

Learn to configure a method call to throw an exception in Mockito.[Read more](/web/20220830145516/https://www.baeldung.com/mockito-exceptions) →

## 2.JUnit 5

**JUnit 5 Jupiter 断言 API 引入了`assertThrows` 方法来断言异常。**

这需要预期异常的类型和一个`Executable`函数接口，我们可以通过一个 lambda 表达式传递测试中的代码:

```java
@Test
public void whenExceptionThrown_thenAssertionSucceeds() {
    Exception exception = assertThrows(NumberFormatException.class, () -> {
        Integer.parseInt("1a");
    });

    String expectedMessage = "For input string";
    String actualMessage = exception.getMessage();

    assertTrue(actualMessage.contains(expectedMessage));
}
```

如果预期的异常被抛出`,` `assertThrows`返回异常，这使我们能够断言消息。

此外，重要的是要注意，当被包含的代码抛出类型为`NumberFormatException`或其任何派生类型的异常时，该断言得到满足。

这意味着如果我们将`Exception`作为预期的异常类型传递，任何抛出的异常都会使断言成功，因为`Exception`是所有异常的超类型。

如果我们将上面的测试改为期望一个`RuntimeException`，这也将通过:

```java
@Test
public void whenDerivedExceptionThrown_thenAssertionSucceeds() {
    Exception exception = assertThrows(RuntimeException.class, () -> {
        Integer.parseInt("1a");
    });

    String expectedMessage = "For input string";
    String actualMessage = exception.getMessage();

    assertTrue(actualMessage.contains(expectedMessage));
}
```

`assertThrows()`方法支持对异常断言逻辑进行更细粒度的控制，因为我们可以在代码的特定部分使用。

## 3.JUnit 4

当使用 JUnit 4 时，我们可以简单地**使用`@Test`注释**的`expected` 属性来声明我们期望在带注释的测试方法中的任何地方抛出异常。

因此，当测试运行时，如果没有引发指定的异常，测试将失败，如果引发了异常，测试将通过:

```java
@Test(expected = NullPointerException.class)
public void whenExceptionThrown_thenExpectationSatisfied() {
    String test = null;
    test.length();
}
```

在这个例子中，我们已经声明我们期望我们的测试代码产生一个`NullPointerException`。

如果我们只对抛出异常感兴趣，这就足够了。

**当我们需要验证异常的一些其他属性时，我们可以使用`ExpectedException`规则。**

让我们看一个验证异常的`message`属性的例子:

```java
@Rule
public ExpectedException exceptionRule = ExpectedException.none();

@Test
public void whenExceptionThrown_thenRuleIsApplied() {
    exceptionRule.expect(NumberFormatException.class);
    exceptionRule.expectMessage("For input string");
    Integer.parseInt("1a");
}
```

在上面的例子中，我们首先声明了`ExpectedException`规则。然后在我们的测试中，我们断言试图解析一个`Integer`值的代码将产生一个带有消息“For input string”的`NumberFormatException`。

## 4.结论

在本文中，我们讨论了 JUnit 4 和 JUnit 5 的异常断言。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220830145516/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics)