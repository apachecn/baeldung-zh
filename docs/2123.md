# 黄瓜春集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cucumber-spring-integration>

## 1。概述

Cucumber 是用 Ruby 编程语言编写的一个非常强大的测试框架，它遵循 BDD(行为驱动开发)方法。它使开发人员能够用非技术利益相关者可以验证的纯文本编写高级用例，并将它们转化为可执行的测试，用一种叫做 Gherkin 的语言编写。

我们已经在另一篇文章中讨论过这些问题。

并且[cumber-Spring 集成](https://web.archive.org/web/20220731203335/https://spring.io/blog/2013/08/04/webinar-replay-spring-with-cucumber-for-automation)旨在使测试自动化更容易。一旦我们将 Cucumber 测试集成到 Spring 中，我们应该能够将它们与 Maven 构建一起执行。

## 2。Maven 依赖关系

让我们通过定义 Maven 依赖关系开始使用 Cucumber-Spring 集成——从 Cucumber-JVM 依赖关系开始:

```
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>6.8.0</version>
    <scope>test</scope>
</dependency>
```

我们可以在这里找到黄瓜 JVM [的最新版本。](https://web.archive.org/web/20220731203335/https://mvnrepository.com/artifact/io.cucumber/cucumber-jvm)

接下来，我们将添加 JUnit 和 Cucumber 测试依赖项:

```
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>6.8.0</version>
    <scope>test</scope>
</dependency>
```

黄瓜 JUnit 的最新版本可以在这里找到。

最后，春天和黄瓜的依赖:

```
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>6.8.0</version>
    <scope>test</scope>
</dependency>
```

同样，我们可以通过[点击](https://web.archive.org/web/20220731203335/https://mvnrepository.com/artifact/io.cucumber/cucumber-spring)查看黄瓜春天的最新版本。

## 3。配置

现在我们来看看如何在 Spring 应用程序中集成 cumber。

首先，我们将创建一个 Spring Boot 应用程序——为此，我们将遵循 [Spring-Boot 应用程序文章](/web/20220731203335/https://www.baeldung.com/spring-boot-application-configuration)。然后，我们将创建一个 Spring REST 服务，并为它编写黄瓜测试。

### 3.1。休息控制器

首先，让我们创建一个简单的控制器:

```
@RestController
public class VersionController {
    @GetMapping("/version")
    public String getVersion() {
        return "1.0";
    }
}
```

### 3.2。黄瓜步骤定义

我们用 JUnit 运行 Cucumber 测试所需要的就是创建一个带有注释`@RunWith(Cucumber.class)`的空类:

```
@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberIntegrationTest {
}
```

我们可以看到注释`@CucumberOptions` ,其中我们指定了小黄瓜文件的位置，小黄瓜文件也被称为特征文件。至此，Cucumber 识别出了小黄瓜语言；你可以在介绍中提到的文章中读到更多关于小黄瓜的内容。

现在，让我们创建一个黄瓜特征文件:

```
Feature: the version can be retrieved
  Scenario: client makes call to GET /version
    When the client calls /version
    Then the client receives status code of 200
    And the client receives server version 1.0
```

场景是对 REST 服务 url `/version`进行 GET 调用，并验证响应。

接下来，我们需要创建一个所谓的粘合代码。这些方法将单个小黄瓜步骤与 Java 代码联系起来。

这里我们必须选择——我们可以在注释中使用[黄瓜表达式](https://web.archive.org/web/20220731203335/https://cucumber.io/docs/cucumber/cucumber-expressions/)或者正则表达式。在我们的例子中，我们将坚持使用正则表达式:

```
@When("^the client calls /version$")
public void the_client_issues_GET_version() throws Throwable{
    executeGet("http://localhost:8080/version");
}

@Then("^the client receives status code of (\\d+)$")
public void the_client_receives_status_code_of(int statusCode) throws Throwable {
    HttpStatus currentStatusCode = latestResponse.getTheResponse().getStatusCode();
    assertThat("status code is incorrect : "+ 
    latestResponse.getBody(), currentStatusCode.value(), is(statusCode));
}

@And("^the client receives server version (.+)$")
public void the_client_receives_server_version_body(String version) throws Throwable {
    assertThat(latestResponse.getBody(), is(version));
}
```

所以现在让我们将 Cucumber 测试与 Spring 应用程序上下文集成起来。为此，我们将创建一个新类，并用`@SpringBootTest`和`@CucumberContextConfiguration`对其进行注释:

```
@CucumberContextConfiguration
@SpringBootTest
public class SpringIntegrationTest {
    // executeGet implementation
}
```

现在所有的黄瓜定义都可以放入一个单独的 Java 类中，该类扩展了`SpringIntegrationTest`:

```
public class StepDefs extends SpringIntegrationTest {

    @When("^the client calls /version$")
    public void the_client_issues_GET_version() throws Throwable {
        executeGet("http://localhost:8080/version");
    }
}
```

我们现在都准备好试运行了。

最后，我们可以通过命令行快速运行，只需运行 **mvn 全新安装-Pintegration-lite-first**–Maven 将执行集成测试并在控制台中显示结果。

```
3 Scenarios ([32m3 passed[0m)
9 Steps ([32m9 passed[0m)
0m1.054s

Tests run: 12, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.283 sec - in
  com.baeldung.CucumberTest
2016-07-30 06:28:20.142  INFO 732 --- [Thread-2] AnnotationConfigEmbeddedWebApplicationContext :
  Closing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext:
  startup date [Sat Jul 30 06:28:12 CDT 2016]; root of context hierarchy

Results :

Tests run: 12, Failures: 0, Errors: 0, Skipped: 0 
```

## 4。结论

已经用 Spring 配置了 Cucumber，在 BDD 测试中使用 Spring 配置的组件将会很方便。这是一个在 Spring-Boot 应用程序中集成 cumber 测试的简单指南。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220731203335/https://github.com/eugenp/tutorials/tree/master/spring-cucumber)