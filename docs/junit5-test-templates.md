# 使用 JUnit 5 为测试用例编写模板

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit5-test-templates>

## 1.概观

JUnit 5 库提供了比以前版本更多的新特性。一个这样的特性是[测试模板](https://web.archive.org/web/20221128055004/https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-templates)。简而言之，测试模板是 JUnit 5 的参数化和[重复](/web/20221128055004/https://www.baeldung.com/junit-5-repeated-test)测试的强大概括。

在本教程中，我们将学习如何使用 JUnit 5 创建一个测试模板。

## 2。Maven 依赖关系

让我们从将依赖项添加到我们的`pom.xml`开始。

我们需要添加主要的 JUnit 5 `[junit-jupiter-engine](https://web.archive.org/web/20221128055004/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22junit-jupiter-engine%22%20AND%20g%3Aorg.junit.jupiter)`依赖项:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
</dependency>
```

除此之外，我们还需要添加 [`junit-jupiter-api`](https://web.archive.org/web/20221128055004/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22junit-jupiter-api%22%20AND%20g%3Aorg.junit.jupiter) 的依赖关系:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.8.1</version>
</dependency>
```

同样，我们可以向我们的`build.gradle`文件添加必要的依赖项:

```java
testCompile group: 'org.junit.jupiter', name: 'junit-jupiter-engine', version: '5.8.1'
testCompile group: 'org.junit.jupiter', name: 'junit-jupiter-api', version: '5.8.1'
```

## 3.问题陈述

在查看测试模板之前，让我们简单看一下 JUnit 5 的[参数化测试](/web/20221128055004/https://www.baeldung.com/parameterized-tests-junit-5)。参数化测试允许我们将不同的参数注入到测试方法中。因此，当**使用参数化测试，** **我们可以用不同的参数多次执行一个测试方法。**

让我们假设我们现在想要多次运行我们的测试方法——不仅使用不同的参数，而且每次都在不同的调用上下文下运行。

换句话说，**我们希望测试方法被多次执行，每次调用都使用不同的配置组合**，比如:

*   使用不同的参数
*   以不同的方式准备测试类实例——也就是说，向测试实例中注入不同的依赖项
*   在不同的条件下运行测试，比如如果环境是“`QA`”，则启用/禁用调用的子集
*   以不同的生命周期回调行为运行——也许我们希望在调用子集之前和之后建立和拆除数据库

在这种情况下，使用参数化测试很快被证明是有限的。幸运的是，JUnit 5 以测试模板的形式为这个场景提供了一个强大的解决方案。

## 4.测试模板

测试模板本身不是测试用例。相反，顾名思义，它们只是给定测试用例的模板。它们是参数化和重复测试的强大概括。

对于调用上下文提供者提供给它们的每个调用上下文，测试模板被调用一次。

现在让我们看一个测试模板的例子。如上所述，主要参与者是:

*   测试目标方法
*   测试模板方法
*   用模板方法注册的一个或多个调用上下文提供程序
*   由每个调用上下文提供者提供的一个或多个调用上下文

### 4.1.测试目标方法

对于这个例子，我们将使用一个简单的`UserIdGeneratorImpl.generate`方法作为我们的测试目标。

让我们定义一下`UserIdGeneratorImpl`类:

```java
public class UserIdGeneratorImpl implements UserIdGenerator {
    private boolean isFeatureEnabled;

    public UserIdGeneratorImpl(boolean isFeatureEnabled) {
        this.isFeatureEnabled = isFeatureEnabled;
    }

    public String generate(String firstName, String lastName) {
        String initialAndLastName = firstName.substring(0, 1).concat(lastName);
        return isFeatureEnabled ? "bael".concat(initialAndLastName) : initialAndLastName;
    }
}
```

作为我们测试目标的`generate`方法将`firstName`和`lastName`作为参数，并生成一个用户 id。用户 id 的格式因功能开关是否启用而异。

让我们看看这个是什么样子的:

```java
Given feature switch is disabled When firstName = "John" and lastName = "Smith" Then "JSmith" is returned
Given feature switch is enabled When firstName = "John" and lastName = "Smith" Then "baelJSmith" is returned
```

接下来，让我们编写测试模板方法。

### 4.2.测试模板方法

这是我们的测试目标方法`UserIdGeneratorImpl.generate`的测试模板:

```java
public class UserIdGeneratorImplUnitTest {
    @TestTemplate
    @ExtendWith(UserIdGeneratorTestInvocationContextProvider.class)
    public void whenUserIdRequested_thenUserIdIsReturnedInCorrectFormat(UserIdGeneratorTestCase testCase) {
        UserIdGenerator userIdGenerator = new UserIdGeneratorImpl(testCase.isFeatureEnabled());

        String actualUserId = userIdGenerator.generate(testCase.getFirstName(), testCase.getLastName());

        assertThat(actualUserId).isEqualTo(testCase.getExpectedUserId());
    }
}
```

让我们仔细看看测试模板方法。

首先，**我们通过用 JUnit 5 `@TestTemplate`注释**标记它来创建我们的测试模板方法。

接下来，**我们使用`@ExtendWith`注释**注册一个上下文提供者、`UserIdGeneratorTestInvocationContextProvider,`、**。我们可以向测试模板注册多个上下文提供者。然而，在这个例子中，我们注册了一个提供者。**

同样，模板方法接收一个`UserIdGeneratorTestCase`的实例作为参数。这只是一个测试用例的输入和预期结果的包装类:

```java
public class UserIdGeneratorTestCase {
    private boolean isFeatureEnabled;
    private String firstName;
    private String lastName;
    private String expectedUserId;

    // Standard setters and getters
}
```

最后，我们调用测试目标方法，并断言结果是预期的

现在，是时候定义我们的调用上下文提供者`.`

### 4.3.调用上下文提供程序

我们需要用我们的测试模板注册至少一个`TestTemplateInvocationContextProvider`。**每个注册的`TestTemplateInvocationContextProvider`提供了一个`TestTemplateInvocationContext`实例**的`Stream`。

之前，使用`@ExtendWith`注释，我们将`UserIdGeneratorTestInvocationContextProvider`注册为我们的调用提供者。

现在让我们来定义这个类:

```java
public class UserIdGeneratorTestInvocationContextProvider implements TestTemplateInvocationContextProvider {
    //...
}
```

我们的调用上下文实现了`TestTemplateInvocationContextProvider`接口，它有两个方法:

*   `supportsTestTemplate`
*   `provideTestTemplateInvocationContexts`

让我们从实现`supportsTestTemplate`方法开始:

```java
@Override
public boolean supportsTestTemplate(ExtensionContext extensionContext) {
    return true;
}
```

JUnit 5 执行引擎首先调用`supportsTestTemplate`方法来验证提供者是否适用于给定的`ExecutionContext`。在这种情况下，我们干脆返回`true`。

现在，让我们实现`provideTestTemplateInvocationContexts`方法:

```java
@Override
public Stream<TestTemplateInvocationContext> provideTestTemplateInvocationContexts(
  ExtensionContext extensionContext) {
    boolean featureDisabled = false;
    boolean featureEnabled = true;

    return Stream.of(
      featureDisabledContext(
        new UserIdGeneratorTestCase(
          "Given feature switch disabled When user name is John Smith Then generated userid is JSmith",
          featureDisabled,
          "John",
          "Smith",
          "JSmith")),
      featureEnabledContext(
        new UserIdGeneratorTestCase(
          "Given feature switch enabled When user name is John Smith Then generated userid is baelJSmith",
          featureEnabled,
          "John",
          "Smith",
          "baelJSmith"))
    );
}
```

`provideTestTemplateInvocationContexts`方法的目的是提供一个`TestTemplateInvocationContext`实例的`Stream`。对于我们的例子，它返回两个实例，由方法`featureDisabledContext`和`featureEnabledContext`提供。因此，我们的测试模板将运行两次。

接下来，让我们看看这些方法返回的两个`TestTemplateInvocationContext`实例。

### 4.4.调用上下文实例

调用上下文是`TestTemplateInvocationContext`接口的实现，并实现以下方法:

*   `getDisplayName`–提供测试显示名称
*   `getAdditionalExtensions`–返回调用上下文的附加扩展

让我们定义返回第一个调用上下文实例的`featureDisabledContext`方法:

```java
private TestTemplateInvocationContext featureDisabledContext(
  UserIdGeneratorTestCase userIdGeneratorTestCase) {
    return new TestTemplateInvocationContext() {
        @Override
        public String getDisplayName(int invocationIndex) {
            return userIdGeneratorTestCase.getDisplayName();
        }

        @Override
        public List<Extension> getAdditionalExtensions() {
            return asList(
              new GenericTypedParameterResolver(userIdGeneratorTestCase), 
              new BeforeTestExecutionCallback() {
                  @Override
                  public void beforeTestExecution(ExtensionContext extensionContext) {
                      System.out.println("BeforeTestExecutionCallback:Disabled context");
                  }
              }, 
              new AfterTestExecutionCallback() {
                  @Override
                  public void afterTestExecution(ExtensionContext extensionContext) {
                      System.out.println("AfterTestExecutionCallback:Disabled context");
                  }
              }
            );
        }
    };
}
```

首先，对于`featureDisabledContext`方法返回的调用上下文，我们注册的[扩展](/web/20221128055004/https://www.baeldung.com/junit-5-extensions)是:

*   `GenericTypedParameterResolver`–一个[参数解析器](/web/20221128055004/https://www.baeldung.com/junit-5-parameters)扩展
*   `BeforeTestExecutionCallback`–在测试执行之前立即运行的生命周期回调扩展
*   测试执行后立即运行的生命周期回调扩展

然而，对于由`featureEnabledContext`方法返回的第二个调用上下文，让我们注册一组不同的扩展(保留`GenericTypedParameterResolver`):

```java
private TestTemplateInvocationContext featureEnabledContext(
  UserIdGeneratorTestCase userIdGeneratorTestCase) {
    return new TestTemplateInvocationContext() {
        @Override
        public String getDisplayName(int invocationIndex) {
            return userIdGeneratorTestCase.getDisplayName();
        }

        @Override
        public List<Extension> getAdditionalExtensions() {
            return asList(
              new GenericTypedParameterResolver(userIdGeneratorTestCase), 
              new DisabledOnQAEnvironmentExtension(), 
              new BeforeEachCallback() {
                  @Override
                  public void beforeEach(ExtensionContext extensionContext) {
                      System.out.println("BeforeEachCallback:Enabled context");
                  }
              }, 
              new AfterEachCallback() {
                  @Override
                  public void afterEach(ExtensionContext extensionContext) {
                      System.out.println("AfterEachCallback:Enabled context");
                  }
              }
            );
        }
    };
}
```

对于第二个调用上下文，我们注册的扩展是:

*   `GenericTypedParameterResolver`–参数解析器扩展
*   `DisabledOnQAEnvironmentExtension`–如果环境属性(从`application.properties`文件加载)为“`qa`，则禁用测试的执行条件
*   在每个测试方法执行之前运行的生命周期回调扩展
*   `AfterEachCallback`–在每次测试方法执行后运行的生命周期回调扩展

从上面的例子中，可以清楚地看到:

*   相同的测试方法在多个调用上下文下运行
*   每个调用上下文使用自己的一组扩展，这些扩展在数量和性质上都不同于其他调用上下文中的扩展

因此，一个测试方法每次都可以在完全不同的调用上下文中被调用多次。通过注册多个上下文提供者，我们可以提供更多的调用上下文层来运行测试。

## 5.结论

在本文中，我们了解了 JUnit 5 的测试模板是如何对参数化和重复测试进行强大的概括的。

首先，我们看了参数化测试的一些限制。接下来，我们讨论了测试模板如何通过允许在不同的上下文中为每次调用运行测试来克服这些限制。

最后，我们看了一个创建新测试模板的例子。我们对这个例子进行了分解，以理解模板如何与调用上下文提供者和调用上下文协同工作。

与往常一样，本文中使用的示例的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128055004/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit5-annotations)