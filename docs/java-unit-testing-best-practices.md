# Java 中单元测试的最佳实践

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unit-testing-best-practices>

## 1.概观

单元测试是软件设计和实现的关键步骤。

它不仅提高了代码的效率和有效性，而且使代码更加健壮，减少了未来开发和维护中的退化。

在本教程中，我们将讨论 Java 单元测试的一些最佳实践。

## 2.什么是单元测试？

单元测试是一种测试源代码是否适合在生产中使用的方法。

我们通过创建各种测试用例来验证单个源代码单元的行为，从而开始编写单元测试。

然后**执行完整的测试套件来捕捉回归，要么在实现阶段，要么在为部署的各个阶段**构建包时，比如试运行和生产。

让我们看一个简单的场景。

首先，让我们创建`Circle`类并在其中实现`calculateArea`方法:

```java
public class Circle {

    public static double calculateArea(double radius) {
        return Math.PI * radius * radius;
    }
}
```

然后我们将为`Circle`类创建单元测试，以确保`calculateArea`方法按预期工作。

让我们在`src/main/test`目录中创建`CalculatorTest`类:

```java
public class CircleTest {

    @Test
    public void testCalculateArea() {
        //...
    }
}
```

在这种情况下，我们使用 [JUnit 的`@Test`注释](/web/20220811141445/https://www.baeldung.com/junit-5-test-annotation)以及构建工具，如 [Maven](/web/20220811141445/https://www.baeldung.com/maven-run-single-test) 或 [Gradle](/web/20220811141445/https://www.baeldung.com/junit-5-gradle) 来运行测试。

## 3.最佳实践

### 3.1.源代码

将测试类与主源代码分开是一个好主意。因此，**它们是与生产代码分开开发、执行和维护的。**

此外，它避免了在生产环境中运行测试代码的任何可能性。

我们可以按照 Maven 和 Gradle 等**构建工具的步骤，为测试实现寻找`src/main/test`目录。**

### 3.2.包命名约定

我们应该在`src/main/test`目录中为测试类创建一个类似的包结构，这样可以提高测试代码的可读性和可维护性。

简单地说，**测试类的包应该匹配源类**的包，它将测试源类的源代码单元。

例如，如果我们的`Circle`类存在于`com.baeldung.math`包中，那么`CircleTest`类也应该存在于`src/main/test`目录结构下的`com.baeldung.math`包中。

### 3.3.测试用例命名约定

测试名称应该是有洞察力的，用户只要看一眼名称本身就应该理解测试的行为和期望。

例如，我们的单元测试的名字是`testCalculateArea`，它对于任何关于测试场景和期望的有意义的信息都是模糊的。

因此，我们应该用`testCalculateAreaWithGeneralDoubleValueRadiusThatReturnsAreaInDouble`、 `testCalculateAreaWithLargeDoubleValueRadiusThatReturnsAreaAsInfinity`等动作和期望来命名一个测试。

然而，我们仍然可以改进名称以提高可读性。

**在`given_when_then`中命名测试用例通常有助于阐述单元测试的目的**:

```java
public class CircleTest {

    //...

    @Test
    public void givenRadius_whenCalculateArea_thenReturnArea() {
        //...
    }

    @Test
    public void givenDoubleMaxValueAsRadius_whenCalculateArea_thenReturnAreaAsInfinity() {
        //...
    }
}
```

我们还应该用**描述 [`Given`、`When`和`Then`](/web/20220811141445/https://www.baeldung.com/cs/bdd-guide) 格式的代码块。**此外，它**有助于将测试区分为三个部分:输入、动作和输出。**

首先，对应于`given`部分的代码块创建测试对象，模拟数据并安排输入。

接下来，`when` 部分的代码块代表一个特定的动作或测试场景。

同样，`then`部分指出了代码的输出，使用断言根据预期结果对其进行了验证。

### 3.4.预期与实际

一个测试用例应该在期望值和实际值之间有一个断言。

为了证实期望值与实际值的概念，我们可以看看 JUnit 的`Assert`类的 [`assertEquals`方法的定义:](/web/20220811141445/https://www.baeldung.com/junit-assertions#junit4-assertequals)

```java
public static void assertEquals(Object expected, Object actual)
```

让我们在一个测试案例中使用这个断言:

```java
@Test 
public void givenRadius_whenCalculateArea_thenReturnArea() {
    double actualArea = Circle.calculateArea(1d);
    double expectedArea = 3.141592653589793;
    Assert.assertEquals(expectedArea, actualArea); 
}
```

建议在变量名前面加上实际和预期的关键字，以提高测试代码的可读性。

### 3.5.偏好简单的测试用例

在前面的测试用例中，我们可以看到期望值是硬编码的。这样做是为了避免在测试用例中重写或重用实际的代码实现来获得预期的值。

不鼓励计算圆的面积来匹配`calculateArea`方法的返回值:

```java
@Test 
public void givenRadius_whenCalculateArea_thenReturnArea() {
    double actualArea = Circle.calculateArea(2d);
    double expectedArea = 3.141592653589793 * 2 * 2;
    Assert.assertEquals(expectedArea, actualArea); 
}
```

在这个断言中，我们使用相似的逻辑计算期望值和实际值，从而永远得到相似的结果。因此，我们的测试用例不会给代码的单元测试增加任何价值。

因此，我们应该**创建一个简单的测试用例，针对实际值断言硬编码的期望值。**

尽管有时需要在测试用例中编写逻辑，但是我们不应该做得太多。此外，正如通常所见，我们永远不应该在测试用例中实现生产逻辑来传递断言。

### 3.6.适当的断言

始终**使用适当的断言来验证预期结果与实际结果。**我们应该使用 [JUnit](/web/20220811141445/https://www.baeldung.com/junit) 的`Assert`类或类似框架如 [AssertJ](/web/20220811141445/https://www.baeldung.com/introduction-to-assertj) 中可用的各种方法。

例如，我们已经将`Assert.assertEquals`方法用于值断言。同样，我们可以使用`assertNotEquals`来检查期望值和实际值是否不相等。

其他方法如`assertNotNull`、`assertTrue`和`assertNotSame`在不同的断言中是有益的。

### 3.7.特定单元测试

我们应该创建单独的测试用例，而不是在同一个单元测试中添加多个断言。

当然，在同一个测试中验证多个场景有时很诱人，但是将它们分开是个好主意。然后，在测试失败的情况下，更容易确定哪个特定场景失败了，同样，修复代码也更简单。

因此，总是**编写一个单元测试来测试一个单独的特定场景。**

单元测试不会变得过于复杂而难以理解。此外，以后调试和维护单元测试会更容易。

### 3.8.测试生产场景

当我们在编写测试时考虑到真实的场景时，单元测试更有价值。

原则上，它有助于使单元测试更具相关性。此外，它被证明对于理解特定生产情况下的代码行为是必不可少的。

### 3.9.模拟外部服务

尽管单元测试集中在特定的较小的代码片段上，但是代码有可能在某些逻辑上依赖于外部服务。

因此，我们应该**模仿外部服务，仅仅针对不同的场景测试我们代码的逻辑和执行。**

我们可以使用各种框架如 [Mockito](/web/20220811141445/https://www.baeldung.com/mockito-series) 、 [EasyMock](/web/20220811141445/https://www.baeldung.com/easymock) 和 [JMockit](/web/20220811141445/https://www.baeldung.com/jmockit-101) 来模仿外部服务。

### 3.10.避免代码冗余

创建越来越多的**助手函数来生成常用的对象，并模仿数据或外部服务**进行类似的单元测试。

与其他建议一样，这增强了测试代码的可读性和可维护性。

### 3.11.释文

通常，测试框架出于各种目的提供注释，例如，执行设置、在运行测试之前执行代码以及在运行测试之后拆除代码。

各种注释如 JUnit 的 [`@Before`、`@BeforeClass` `and @After`](/web/20220811141445/https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall) 以及来自其他测试框架如 [TestNG](/web/20220811141445/https://www.baeldung.com/testng) 的注释都由我们支配。

我们应该**利用注释来为测试**准备系统，方法是在每次测试后创建数据、排列对象并删除所有内容，以保持测试用例相互隔离。

### 3.12.80%的测试覆盖率

对源代码进行更多的[测试覆盖总是有益的。然而，这不是唯一要实现的目标。我们应该做出明智的决定，选择一个对我们的实施、截止日期和团队都有利的更好的折衷方案。](/web/20220811141445/https://www.baeldung.com/cs/code-coverage)

根据经验，我们应该通过单元测试覆盖 80%的代码。

此外，我们可以使用像 [JaCoCo](/web/20220811141445/https://www.baeldung.com/jacoco) 和 [Cobertura](/web/20220811141445/https://www.baeldung.com/cobertura) 这样的工具以及 Maven 或 Gradle 来生成代码覆盖报告。

### 3.13.TDD 方法

[测试驱动开发(TDD)](/web/20220811141445/https://www.baeldung.com/java-test-driven-list) 是一种我们在实施之前和实施过程中创建测试用例的方法。该方法与设计和实现源代码的过程相结合。

好处包括从一开始就可测试的产品代码、易于重构的健壮实现和更少的回归。

### 3.14.自动化

我们可以通过自动化整个测试套件的执行来提高代码的可靠性，同时创建新的构建。

首先，这有助于避免各种发布环境中的不幸回归。它还确保了在错误代码发布之前的快速反馈。

因此，**单元测试执行应该是 [CI-CD 管道](/web/20220811141445/https://www.baeldung.com/ops/jenkins-pipelines)** 的一部分，并在出现故障时提醒利益相关者。

## 4.结论

在本文中，我们探索了 Java 中单元测试的一些最佳实践。遵循最佳实践可以在软件开发的许多方面提供帮助。