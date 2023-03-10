# 用黄瓜搭配格雷德

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cucumber-gradle>

## 1.介绍

黄瓜是一个支持行为驱动开发(BDD)的测试自动化工具。它运行用描述系统行为的纯文本 Gherkin 语法编写的规范。

在本教程中，我们将看到一些将 Cucumber 与 Gradle 集成的方法，以便作为项目构建的一部分运行 BDD 规范。

## 2。设置

首先，让我们使用 [Gradle 包装器](/web/20221129002507/https://www.baeldung.com/gradle-wrapper)建立一个 Gradle 项目。

接下来，我们将把 [`cucumber-java`](https://web.archive.org/web/20221129002507/https://search.maven.org/artifact/io.cucumber/cucumber-java) 依赖项添加到`build.gradle`:

```java
testImplementation 'io.cucumber:cucumber-java:6.10.4' 
```

这将正式的 Cucumber Java 实现添加到我们的项目中。

## 3.使用自定义任务运行

为了使用 Gradle 运行我们的规范，我们将**从 Cucumber** 创建一个使用命令行界面 Runner (CLI)的任务。

### 3.1.配置

让我们首先将所需的配置添加到项目的`build.gradle`文件中:

```java
configurations {
    cucumberRuntime {
        extendsFrom testImplementation
    }
}
```

接下来，我们将创建定制的`cucumberCli`任务:

```java
task cucumberCli() {
    dependsOn assemble, testClasses
    doLast {
        javaexec {
            main = "io.cucumber.core.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.main.output + sourceSets.test.output
            args = [
              '--plugin', 'pretty',
              '--plugin', 'html:target/cucumber-report.html', 
              '--glue', 'com.baeldung.cucumber', 
              'src/test/resources']
        }
    }
}
```

该任务被配置为运行在`src/test/resources`目录下的`.feature`文件中找到的所有测试场景。

`Main`类的`–glue`选项指定运行场景所需的步骤定义文件的位置。

`–plugin`选项指定测试报告的格式和位置。我们可以组合几个值来生成所需格式的报告，比如像我们的例子中的`pretty`和`HTML`。

还有其他几个选项可用。例如，有基于名称和标签过滤测试的选项。

### 3.2.方案

现在，让我们在`src/test/resources/features/account_credited.feature`文件中为我们的应用程序创建一个简单的场景:

```java
Feature: Account is credited with amount

  Scenario: Credit amount
    Given account balance is 0.0
    When the account is credited with 10.0
    Then account should have a balance of 10.0
```

接下来，我们将实现运行场景所需的相应步骤定义—`glue`:

```java
public class StepDefinitions {

    @Given("account balance is {double}")
    public void givenAccountBalance(Double initialBalance) {
        account = new Account(initialBalance);
    }

    // other step definitions 

}
```

### 3.3.运行任务

最后，让我们从命令行运行我们的`cucumberCli`任务:

```java
>> ./gradlew cucumberCli

> Task :cucumberCli

Scenario: Credit amount                      # src/test/resources/features/account_credited.feature:3
  Given account balance is 0.0               # com.baeldung.cucumber.StepDefinitions.account_balance_is(java.lang.Double)
  When the account is credited with 10.0     # com.baeldung.cucumber.StepDefinitions.the_account_is_credited_with(java.lang.Double)
  Then account should have a balance of 10.0 # com.baeldung.cucumber.StepDefinitions.account_should_have_a_balance_of(java.lang.Double)

1 Scenarios (1 passed)
3 Steps (3 passed)
0m0.381s
```

正如我们所看到的，我们的规范已经与 Gradle 集成，运行成功，输出显示在控制台上。此外，HTML 测试报告可在指定位置获得。

## 4.使用 JUnit 运行

我们可以使用 JUnit 运行 cucumber 场景，而不是在 Gradle 中创建定制任务。

让我们从包含`[cucumber-junit](https://web.archive.org/web/20221129002507/https://search.maven.org/artifact/io.cucumber/cucumber-junit)`依赖项开始:

```java
testImplementation 'io.cucumber:cucumber-junit:6.10.4'
```

当我们使用 JUnit 5 时，我们还需要添加 [`junit-vintage-engine`](https://web.archive.org/web/20221129002507/https://search.maven.org/artifact/org.junit.vintage/junit-vintage-engine) 依赖项:

```java
testImplementation 'org.junit.vintage:junit-vintage-engine:5.7.2'
```

接下来，我们将在测试源位置创建一个空的 runner 类:

```java
@RunWith(Cucumber.class)
@CucumberOptions(
  plugin = {"pretty", "html:target/cucumber-report.html"},
  features = {"src/test/resources"}
)
public class RunCucumberTest {
}
```

这里，我们在`@RunWith`注释中使用了 JUnit `Cucumber` runner。此外，所有 CLI 运行程序选项，如`features`和`plugin`，都可以通过 [`@CucumberOptions`](https://web.archive.org/web/20221129002507/https://github.com/cucumber/cucumber-jvm/blob/main/junit/src/main/java/io/cucumber/junit/CucumberOptions.java) 注释获得。

现在，执行**标准**

```java
>> ./gradlew test

> Task :test

RunCucumberTest > Credit amount PASSED

BUILD SUCCESSFUL in 2s
```

## 5.使用插件运行

最后一种方法是**使用第三方插件，该插件提供从 Gradle 版本运行规范**的能力。

在我们的例子中，我们将使用 [`gradle-cucumber-runner`](https://web.archive.org/web/20221129002507/https://github.com/tsundberg/gradle-cucumber-runner) 插件来运行 Cucumber JVM。实际上，这会将所有呼叫转发到我们之前使用的 CLI 运行程序。让我们将它包含在我们的项目中:

```java
plugins {
  id "se.thinkcode.cucumber-runner" version "0.0.8"
}
```

这将一个`cucumber`任务添加到我们的构建中，现在我们可以使用默认设置运行它:

```java
>> ./gradlew cucumber
```

值得注意的是**这不是一个官方的黄瓜插件**，也有其他的插件提供类似的功能。

## 6.结论

在本文中，我们演示了几种使用 Gradle 配置和运行 BDD 规范的方法。

最初，我们看了如何利用 CLI 运行程序创建自定义任务。然后，我们看了如何使用 Cucumber JUnit runner 来使用现有的 Gradle 任务执行规范。最后，我们使用了一个第三方插件来运行 Cucumber，而没有创建我们自己的定制任务。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129002507/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle/gradle-cucumber)