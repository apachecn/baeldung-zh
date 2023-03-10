# JUnitParams 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-params>

## 1。概述

在本文中，我们将探索`[JUnitParams](https://web.archive.org/web/20221207114201/https://github.com/Pragmatists/JUnitParams)` 库及其用法。简单地说，这个库在`JUnit`测试中提供了测试方法的简单参数化。

有些情况下，在多次测试之间唯一变化的是参数。`JUnit`本身有一个参数化支持，`JUnitParams` 在这个功能上有了很大的改进。

## 2。Maven 依赖关系

为了在我们的项目中使用 `JUnitParams`，我们需要将它添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>pl.pragmatists</groupId>
    <artifactId>JUnitParams</artifactId>
    <version>1.1.0</version>
</dependency>
```

这个库的最新版本可以在[这里](https://web.archive.org/web/20221207114201/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22pl.pragmatists%22%20AND%20a%3A%22JUnitParams%22)找到。

## 3。测试场景

让我们创建一个类来完成两个整数的安全加法。如果溢出，应该返回`Integer.MAX_VALUE`，如果下溢，应该返回`Integer.MIN_VALUE`:

```java
public class SafeAdditionUtil {

    public int safeAdd(int a, int b) {
        long result = ((long) a) + b;
        if (result > Integer.MAX_VALUE) {
            return Integer.MAX_VALUE;
        } else if (result < Integer.MIN_VALUE) {
            return Integer.MIN_VALUE;
        }
        return (int) result;
    }
}
```

## 4。构建简单的测试方法

我们需要测试不同输入值组合的方法实现，以确保实现适用于所有可能的场景。`JUnitParams` 提供了多种方式来实现参数化的测试创建。

让我们用最少的代码量来看看这个基本方法是如何实现的。之后，我们可以看到使用 JUnitParams 实现测试场景的其他可能方式是什么

```java
@RunWith(JUnitParamsRunner.class)
public class SafeAdditionUtilTest {

    private SafeAdditionUtil serviceUnderTest
      = new SafeAdditionUtil();

    @Test
    @Parameters({ 
      "1, 2, 3", 
      "-10, 30, 20", 
      "15, -5, 10", 
      "-5, -10, -15" })
    public void whenWithAnnotationProvidedParams_thenSafeAdd(
      int a, int b, int expectedValue) {

        assertEquals(expectedValue, serviceUnderTest.safeAdd(a, b));
    }

}
```

现在让我们看看这个测试类与常规的`JUnit`测试类有何不同。

我们注意到的第一件事是**在类注释-`JUnitParamsRunner`中有一个** **不同的测试运行者**。

转到测试方法，我们看到测试方法用带有一组输入参数的`@Parameters`注释进行了注释。它指出了将用于测试我们的服务方法的不同测试场景。

如果我们使用 Maven 运行测试，我们会看到**我们正在运行四个测试用例，而不是一个**。输出将类似于以下内容:

```java
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.baeldung.junitparams.SafeAdditionUtilTest
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.068 sec 
  - in com.baeldung.junitparams.SafeAdditionUtilTest

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
```

## 5。不同类型的参数化测试方法

如果我们有许多可能的场景需要测试，那么在注释中直接提供测试参数肯定不是最可读的方式。`JUnitParams` 提供了一套我们可以用来创建参数化测试的不同方法:

*   直接在`@Parameters`注释中(在上面的例子中使用)
*   使用注释中定义的命名测试方法
*   使用由测试方法名称映射的方法
*   注释中定义的命名测试类
*   使用 CSV 文件

让我们逐一探讨这些方法。

### 5.1。直接在`@Parameters`标注

我们已经在我们尝试的例子中使用了这种方法。我们需要记住的是，我们应该提供一个参数字符串数组。在参数字符串中，每个参数由逗号分隔。

例如，数组可以是`{ “1, 2, 3”, “-10, 30, 20”}`的形式，一组参数表示为`“1, 2, 3”`。

这种方法的局限性是我们只能提供原语和`String`作为测试参数。也不可能将对象作为测试方法参数提交。

### 5.2。参数方法

我们可以使用类中的另一个方法来提供测试方法参数。我们先来看一个例子:

```java
@Test
@Parameters(method = "parametersToTestAdd")
public void whenWithNamedMethod_thenSafeAdd(
  int a, int b, int expectedValue) {

    assertEquals(expectedValue, serviceUnderTest.safeAdd(a, b));
}

private Object[] parametersToTestAdd() {
    return new Object[] { 
        new Object[] { 1, 2, 3 }, 
        new Object[] { -10, 30, 20 }, 
        new Object[] { Integer.MAX_VALUE, 2, Integer.MAX_VALUE }, 
        new Object[] { Integer.MIN_VALUE, -8, Integer.MIN_VALUE } 
    };
}
```

关于方法`parametersToAdd(),`的测试方法被注释，它通过运行被引用的方法获取参数。

提供者方法的规范应该返回一个由`Object`组成的数组作为结果。如果具有给定名称的方法不可用，测试用例将失败，并显示以下错误:

```java
java.lang.RuntimeException: Could not find method: bogusMethodName so no params were used.
```

### 5.3。由测试方法名映射的方法

如果我们没有在`@Parameters`注释中指定任何东西，`JUnitParams` 试图根据测试方法名加载一个测试数据提供者方法。方法名被构造成`“parametersFor”+ <test method name>:`

```java
@Test
@Parameters
public void whenWithnoParam_thenLoadByNameSafeAdd(
  int a, int b, int expectedValue) {

    assertEquals(expectedValue, serviceUnderTest.safeAdd(a, b));
}

private Object[] parametersForWhenWithnoParam_thenLoadByNameSafe() {
    return new Object[] { 
        new Object[] { 1, 2, 3 }, 
        new Object[] { -10, 30, 20 }, 
        new Object[] { Integer.MAX_VALUE, 2, Integer.MAX_VALUE }, 
        new Object[] { Integer.MIN_VALUE, -8, Integer.MIN_VALUE } 
    };
}
```

在上面的例子中，测试方法的名称是`whenWithnoParam_shouldLoadByNameAbdSafeAdd()`。

因此，当测试方法被执行时，它寻找名为`parametersForWhenWithnoParam_shouldLoadByNameAbdSafeAdd()`的数据提供者方法。

由于该方法存在，它将从其中加载数据并运行测试。**如果没有与所需名称匹配的方法，测试失败**，如上例所示。

### 5.4。注释中定义的命名测试类

类似于我们在前面的例子中引用数据提供者方法的方式，我们可以引用一个单独的类来为我们的测试提供数据:

```java
@Test
@Parameters(source = TestDataProvider.class)
public void whenWithNamedClass_thenSafeAdd(
  int a, int b, int expectedValue) {

    assertEquals(expectedValue, serviceUnderTest.safeAdd(a, b));
}
public class TestDataProvider {

    public static Object[] provideBasicData() {
        return new Object[] { 
            new Object[] { 1, 2, 3 }, 
            new Object[] { -10, 30, 20 }, 
            new Object[] { 15, -5, 10 }, 
            new Object[] { -5, -10, -15 } 
        };
    }

    public static Object[] provideEdgeCaseData() {
        return new Object[] { 
            new Object[] { 
              Integer.MAX_VALUE, 2, Integer.MAX_VALUE }, 
            new Object[] { 
              Integer.MIN_VALUE, -2, Integer.MIN_VALUE }, 
        };
    }
}
```

假设方法名以“provide”开头，我们可以在一个类中拥有任意数量的测试数据提供者。如果是这样，执行器选择这些方法并返回数据。

如果没有类方法满足这个要求，即使这些方法返回一个`Object`的数组，这些方法也会被忽略。

### 5.5。使用 CSV 文件

我们可以使用一个外部的 CSV 文件来加载测试数据。如果可能的测试用例的数量非常大，或者测试用例经常被改变，这是很有帮助的。这些改变可以在不影响测试代码的情况下完成。

假设我们有一个测试参数为 `JunitParamsTestParameters.csv`的 CSV 文件:

```java
1,2,3
-10, 30, 20
15, -5, 10
-5, -10, -15
```

现在让我们看看如何使用这个文件来加载测试方法中的测试参数:

```java
@Test
@FileParameters("src/test/resources/JunitParamsTestParameters.csv")
public void whenWithCsvFile_thenSafeAdd(
  int a, int b, int expectedValue) {

    assertEquals(expectedValue, serviceUnderTest.safeAdd(a, b));
}
```

这种方法的一个限制是不能传递复杂的对象。只有原语和`String`是有效的。

## 6。结论

在本教程中，我们简单地看了如何利用`JUnitParams` 的功能。

我们还讨论了该库为我们的测试方法提供测试参数的不同方法——远远超出了 JUnit 本身的能力。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221207114201/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)