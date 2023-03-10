# 朱基托简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jukito>

## 1。概述

[Jukito](https://web.archive.org/web/20220626084030/https://github.com/ArcBees/Jukito) 是 [JUnit](https://web.archive.org/web/20220626084030/http://junit.org/junit5/) 、 [Guice](https://web.archive.org/web/20220626084030/https://github.com/google/guice) 和 [Mockito](https://web.archive.org/web/20220626084030/https://github.com/mockito/mockito) 的组合力量——用于简化同一接口的多个实现的测试。

在本文中，我们将看到作者如何成功地将这三个库结合起来，帮助我们减少大量样板代码，使我们的测试变得灵活和简单。

## 2。设置

首先，我们将在项目中添加以下依赖项:

```java
<dependency>
    <groupId>org.jukito</groupId>
    <artifactId>jukito</artifactId>
    <version>1.5</version>
    <scope>test</scope>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220626084030/https://search.maven.org/classic/#search%7Cga%7C1%7Cjukito) 找到最新版本。

## 3。一个接口的不同实现

为了开始理解 Jukito 的威力，我们将使用一个`Add`方法定义一个简单的`Calculator` 接口:

```java
public interface Calculator {
    public double add(double a, double b);
}
```

我们将实现以下接口:

```java
public class SimpleCalculator implements Calculator {

    @Override
    public double add(double a, double b) {
        return a + b;
    }
}
```

我们还需要另一个实现:

```java
public class ScientificCalculator extends SimpleCalculator {
}
```

现在，让我们使用 Jukito 来测试我们的两个实现:

```java
@RunWith(JukitoRunner.class)
public class CalculatorTest {

    public static class Module extends JukitoModule {

        @Override
        protected void configureTest() {
            bindMany(Calculator.class, SimpleCalculator.class, 
              ScientificCalculator.class);
        }
    }

    @Test
    public void givenTwoNumbers_WhenAdd_ThenSumBoth(@All Calculator calc) {
        double result = calc.add(1, 1);

        assertEquals(2, result, .1);
    }
}
```

在这个例子中，我们可以看到一个`JukitoModule`，它连接了所有指定的实现。

`@All`注释获取由`JukitoModule`生成的同一接口的所有绑定，并运行测试**，在运行时注入所有不同的实现**。

如果我们运行测试，我们可以看到实际上运行了两个测试，而不是一个:

```java
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
```

## 4。笛卡尔积

现在让我们为我们的`Add`方法的不同测试组合添加一个简单的嵌套类:

```java
public static class AdditionTest {
    int a;
    int b;
    int expected;

    // standard constructors/getters
}
```

这将扩大我们可以运行的测试数量，但是首先，我们需要在我们的`configureTest`方法中添加额外的绑定:

```java
bindManyInstances(AdditionTest.class, 
  new AdditionTest(1, 1, 2), 
  new AdditionTest(10, 10, 20), 
  new AdditionTest(18, 24, 42));
```

最后，我们向套件中添加了另一个测试:

```java
@Test
public void givenTwoNumbers_WhenAdd_ThenSumBoth(
  @All Calculator calc, 
  @All AdditionTest addTest) {

    double result = calc.add(addTest.a, addTest.b);

    assertEquals(addTest.expected, result, .1);
}
```

现在**`@All`注释将产生`Calculator`接口和`AdditionTest`实例的不同实现之间的不同组合的笛卡尔乘积。**

我们可以看看它现在产生的测试数量的增加:

```java
Tests run: 8, Failures: 0, Errors: 0, Skipped: 0
```

我们需要记住，对于笛卡尔乘积来说，测试执行的数量急剧增加。

所有测试的执行时间将随着执行次数的增加而线性增长。即:一个带有三个参数和一个`@All`注释以及每个参数四个绑定的测试方法将被执行 4 x 4 x 4 = 64 次。

同一测试方法有五个绑定将导致 5 x 5 x 5 = 125 次执行。

## 5。按姓名分组

我们将讨论的最后一个功能是按名称分组:

```java
bindManyNamedInstances(Integer.class, "even", 2, 4, 6);
bindManyNamedInstances(Integer.class, "odd", 1, 3, 5);
```

这里，我们向我们的`configureTest`方法添加了一些 integer 类的命名实例，以展示这些组可以做什么。

现在让我们添加一些测试:

```java
@Test
public void givenEvenNumbers_whenPrint_thenOutput(@All("even") Integer i) {
    System.out.println("even " + i);
}

@Test
public void givenOddNumbers_whenPrint_thenOutput(@All("odd") Integer i) {
    System.out.println("odd " + i);
}
```

上面的例子将打印六个字符串“偶数 2”、“偶数 4”、“偶数 6”、“奇数 1”、“奇数 3”、“奇数 5”。

请记住，在运行时不能保证它们的顺序。

## 6。结论

在这个快速教程中，我们看了一下 Jukito 如何通过提供足够的测试用例组合来允许使用整个测试套件。

完整的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220626084030/https://github.com/eugenp/tutorials/tree/master/testing-modules/mocks)