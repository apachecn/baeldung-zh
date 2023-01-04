# 在 JUnit 5 中使用黄瓜标签

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-cucumber-tags>

## 1.概观

在本教程中，我们将说明如何使用 Cucumber 标签表达式来操作测试的执行及其相关设置。

我们将看看如何分离我们的 API 和 UI 测试，并控制我们为每个测试运行的配置步骤。

## 2.具有 UI 和 API 组件的应用程序

我们的示例应用程序有一个简单的用户界面，用于在一系列值之间生成一个随机数:

[![](img/8a60a8b2607c8b73d61cde477ae611d3.png)](/web/20220523144727/https://www.baeldung.com/wp-content/uploads/2021/04/Screenshot-2021-04-08-at-13.43.49.png)

我们还有一个/ `status` Rest 端点返回一个 HTTP 状态代码。我们将使用`[Cucumber](/web/20220523144727/https://www.baeldung.com/cucumber-rest-api-testing) `和`[Junit 5](/web/20220523144727/https://www.baeldung.com/junit-5). `在验收测试中涵盖这两个功能

为了让 cucumber 与 Junit 5 一起工作，我们必须在我们的`pom`中声明[`cucumber`–`junit-platform-engine`](https://web.archive.org/web/20220523144727/https://search.maven.org/artifact/io.cucumber/cucumber-junit-platform-engine)作为它的依赖项:

```
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit-platform-engine</artifactId>
    <version>6.10.3</version>
</dependency>
```

## 3.黄瓜标签和条件挂钩

黄瓜标签可以帮助我们将场景组合在一起。假设我们对测试 UI 和 API 有不同的需求。例如，我们需要启动一个浏览器来测试 UI 组件，但是这对于调用`/status`端点来说并不是必需的。我们需要的是一种方法来确定运行哪些步骤以及何时运行。黄瓜标签可以帮助我们解决这个问题。

## 4.UI 测试

首先，让我们用一个标签将我们的`Features`或`Scenarios` 分组在一起。这里我们用一个`@ui` 标签来标记我们的 UI 特性:

```
@ui
Feature: UI - Random Number Generator

  Scenario: Successfully generate a random number
    Given we are expecting a random number between min and max
    And I am on random-number-generator page
    When I enter min 1
    And I enter max 10
    And I press Generate button
    Then I should receive a random number between 1 and 10
```

然后，基于这些标签，我们可以通过使用条件挂钩来操作我们为这组特性运行的内容。我们用单独的`@Before`和`@After` 方法来实现这一点，这些方法在我们的`ScenarioHooks`中用相关标签进行了注释:

```
@Before("@ui")
public void setupForUI() {
    uiContext.getWebDriver();
}
```

```
@After("@ui")
public void tearDownForUi(Scenario scenario) throws IOException {
    uiContext.getReport().write(scenario);
    uiContext.getReport().captureScreenShot(scenario, uiContext.getWebDriver());
    uiContext.getWebDriver().quit();
}
```

## 5.API 测试

类似于我们的 UI 测试，我们可以用`@api`标签来标记我们的 API 特性:

```
@api
Feature: Health check

  Scenario: Should have a working health check
    When I make a GET call on /status
    Then I should receive 200 response status code
    And should receive a non-empty body
```

我们也有带有`@api`标签的`@Before`和`@After`方法:

```
@Before("@api")
public void setupForApi() {
    RestAssuredMockMvc.mockMvc(mvc);
    RestAssuredMockMvc.config = RestAssuredMockMvc.config()
      .logConfig(new LogConfig(apiContext.getReport().getRestLogPrintStream(), true));
}

@After("@api")
public void tearDownForApi(Scenario scenario) throws IOException {
    apiContext.getReport().write(scenario);
}
```

当我们运行我们的`AcceptanceTestRunnerIT,`时，我们可以看到正在为相关测试执行适当的设置和拆卸步骤。

## 6.结论

在本文中，我们展示了如何通过使用 Cucumber 标签和条件挂钩来控制不同测试集的执行以及它们的安装/拆卸指令。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220523144727/https://github.com/eugenp/tutorials/tree/master/testing-modules/cucumber)