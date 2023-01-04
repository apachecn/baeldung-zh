# 黄瓜 Java 8 支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cucumber-java-8-support>

## 1。概述

在这个快速教程中，我们将学习如何对 Cucumber 使用 Java 8 lambda 表达式。

## 2。Maven 配置

首先，我们需要将以下依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>info.cukes</groupId>
    <artifactId>cucumber-java8</artifactId>
    <version>1.2.5</version>
    <scope>test</scope>
</dependency>
```

在 [Maven Central](https://web.archive.org/web/20221128115024/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22cucumber-java8%22) 上可以找到`cucumber-java8`的依赖关系。

## 3。使用λ的步骤定义

接下来，我们将讨论如何使用 Java 8 lambda 表达式编写我们的步骤定义:

```
public class ShoppingStepsDef implements En {

    private int budget = 0;

    public ShoppingStepsDef() {
        Given("I have (\\d+) in my wallet", (Integer money) -> budget = money);

        When("I buy .* with (\\d+)", (Integer price) -> budget -= price);

        Then("I should have (\\d+) in my wallet", (Integer finalBudget) -> 
          assertEquals(budget, finalBudget.intValue()));
    }
}
```

我们以一个简单的购物功能为例:

```
Given("I have (\\d+) in my wallet", (Integer money) -> budget = money);
```

注意如何:

*   在这一步中，我们设置初始预算，我们有一个类型为`Integer`的参数`money`
*   因为我们使用了一个语句，所以不需要花括号

## 4。测试场景

最后，让我们看看我们的测试场景:

```
Feature: Shopping

    Scenario: Track my budget 
        Given I have 100 in my wallet
        When I buy milk with 10
        Then I should have 90 in my wallet

    Scenario: Track my budget 
        Given I have 200 in my wallet
        When I buy rice with 20
        Then I should have 180 in my wallet
```

和测试配置:

```
@RunWith(Cucumber.class)
@CucumberOptions(features = { "classpath:features/shopping.feature" })
public class ShoppingIntegrationTest {
    // 
}
```

关于 Cucumber 配置的更多细节，请查看 [Cucumber 和场景概要](/web/20221128115024/https://www.baeldung.com/cucumber-scenario-outline)教程。

## 5。结论

我们学习了如何对 Cucumber 使用 Java 8 lambda 表达式。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128115024/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries)