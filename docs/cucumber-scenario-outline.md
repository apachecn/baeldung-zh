# 黄瓜和场景大纲

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cucumber-scenario-outline>

## 1。简介

Cucumber 是一个 BDD(行为驱动开发)测试框架。

使用框架**编写具有不同输入/输出排列的重复场景**可能非常耗时，难以维护，当然也令人沮丧。

Cucumber 提出了一个解决方案，通过使用`Scenario Outline coupled with Examples`的概念来减少这种工作。在下一节中，我们将尝试举一个例子，看看我们如何将这种努力减到最少。

如果你想了解更多关于这种方法和小黄瓜语言的内容，可以看看这篇文章。

## 2。添加黄瓜支架

要在一个简单的 Maven 项目中添加对 Cucumber 的支持，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>info.cukes</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>1.2.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>info.cukes</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>1.2.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-library</artifactId>
    <version>1.3</version>
    <scope>test</scope>
</dependency>
```

来自 Maven Central 的有用依赖链接: [cucumber-junit](https://web.archive.org/web/20221128121226/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22cucumber-junit%22) ， [cucumber-java](https://web.archive.org/web/20221128121226/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22cucumber-java%22) ， [hamcrest-library](https://web.archive.org/web/20221128121226/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hamcrest-library%22)

由于这些是测试库，它们不需要与实际的可部署库一起发布——这就是为什么它们都是`test`范围的。

## 3。一个简单的例子

让我们演示一下编写特色文件的臃肿方式和简洁方式。让我们首先定义我们想要为其编写测试的逻辑:

让我们首先定义我们想要为其编写测试的逻辑:

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
```

## 4。定义黄瓜测试

### 4.1。定义特征文件

```java
Feature: Calculator
  As a user
  I want to use a calculator to add numbers
  So that I don't need to add myself

  Scenario: Add two numbers -2 & 3
    Given I have a calculator
    When I add -2 and 3
    Then the result should be 1

  Scenario: Add two numbers 10 & 15
    Given I have a calculator
    When I add 10 and 15
    Then the result should be 25 
```

如此处所示，两种不同的数字组合被用来测试加法逻辑。除了数字，所有的场景都是一模一样的。

### 4.2。“胶水”代码

为了测试这些场景，我必须用相应的代码定义每个步骤，以便将语句转换成功能代码:

```java
public class CalculatorRunSteps {

    private int total;

    private Calculator calculator;

    @Before
    private void init() {
        total = -999;
    }

    @Given("^I have a calculator$")
    public void initializeCalculator() throws Throwable {
        calculator = new Calculator();
    }

    @When("^I add (-?\\d+) and (-?\\d+)$")
    public void testAdd(int num1, int num2) throws Throwable {
        total = calculator.add(num1, num2);
    }

    @Then("^the result should be (-?\\d+)$")
    public void validateResult(int result) throws Throwable {
        Assert.assertThat(total, Matchers.equalTo(result));
    }
}
```

### 4.3。一个跑步者类

为了集成特性和粘合代码，我们可以使用 JUnit runners:

```java
@RunWith(Cucumber.class)
@CucumberOptions(
  features = { "classpath:features/calculator.feature" },
  glue = {"com.baeldung.cucumber.calculator" })
public class CalculatorTest {}
```

## 5。使用场景大纲重写特性

我们在 4.1 节看到。定义特征文件是一项耗时的任务，并且更容易出错。使用`Scenario Outline:`可以将相同的特征文件减少到仅仅几行

```java
Feature: Calculator
  As a user
  I want to use a calculator to add numbers
  So that I don't need to add myself

  Scenario Outline: Add two numbers <num1> & <num2>
    Given I have a calculator
    When I add <num1> and <num2>
    Then the result should be <total>

  Examples: 
    | num1 | num2 | total |
    | -2 | 3 | 1 |
    | 10 | 15 | 25 |
    | 99 | -99 | 0 |
    | -1 | -10 | -11 |
```

当比较常规的`Scenario Definition`和`Scenario Outline`时，不再需要在步骤定义中硬编码值。在步长定义本身中，值被替换为参数`<parameter_name>`。

在场景大纲的结尾，使用`Examples`在管道分隔的表格格式中定义值。

定义`Examples`的示例如下所示:

```java
Examples:
  | Parameter_Name1 | Parameter_Name2 |
  | Value-1 | Value-2 |
  | Value-X | Value-Y |
```

## 6。结论

通过这篇简短的文章，我们展示了场景如何在本质上变得通用。并减少编写和维护这些场景的工作量。

本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128121226/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries)