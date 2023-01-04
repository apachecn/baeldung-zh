# JUnit 5 TestWatcher API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-testwatcher>

## 1。概述

当进行单元测试时，我们可能会周期性地希望处理测试方法执行的结果。在这个快速教程中，**我们将看看如何使用由[JUnit](https://web.archive.org/web/20221127014133/http://junit.org/junit5/)提供的 TestWatcher API 来完成这个任务。**

要深入了解 JUnit 测试指南，请查看我们优秀的 JUnit 5 指南[。](/web/20221127014133/https://www.baeldung.com/junit-5)

## 2。`TestWatcher` API

**简而言之， [`TestWatcher`](https://web.archive.org/web/20221127014133/https://junit.org/junit5/docs/5.5.1/api/org/junit/jupiter/api/extension/TestWatcher.html) 接口为希望处理测试结果**的[扩展](/web/20221127014133/https://www.baeldung.com/junit-5-extensions)定义了 API。我们可以认为这个 API 的一种方式是提供钩子来获取单个测试用例的状态。

但是，在我们深入一些真实的例子之前，**让我们后退一步，简要总结一下`TestWatcher`接口**中的方法:

*   ```java
    testAborted​(ExtensionContext context, Throwable cause)
    ```

    为了处理中止测试的结果，我们可以覆盖`testAborted`方法。顾名思义，这个方法是在测试中止后调用的。

*   ```java
    testDisabled​(ExtensionContext context, Optional reason)
    ```

    当我们想要处理一个被禁用的测试方法的结果时，我们可以覆盖`testDisabled`方法。该方法还可以包括测试被禁用的原因。

*   ```java
    testFailed(ExtensionContext context, Throwable cause)
    ```

    如果我们想在测试失败后做一些额外的处理，我们可以简单地实现`testFailed`方法中的功能。该方法可能包括测试失败的原因。

*   ```java
    testSuccessful(ExtensionContext context)
    ```

    最后但同样重要的是，当我们希望处理成功测试的结果时，我们只需覆盖`testSuccessful`方法。

我们应该注意，所有的方法都包含了`ExtensionContext`。这封装了当前测试执行的上下文。

## 3。Maven 依赖关系

首先，让我们添加例子中需要的项目依赖项。
除了主要的 JUnit 5 库`junit-jupiter-engine`，我们还需要`junit-jupiter-api`库:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

一如既往，我们可以从 [Maven Central](https://web.archive.org/web/20221127014133/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-api%22) 获得最新版本。

## 4。一个`TestResultLoggerExtension`例子

既然我们已经对`TestWatcher` API 有了基本的了解，我们就来看一个实际的例子。

**让我们首先创建一个简单的扩展来记录结果并提供我们测试的摘要**。在这种情况下，为了创建扩展，我们需要定义一个实现`TestWatcher`接口的类:

```java
public class TestResultLoggerExtension implements TestWatcher, AfterAllCallback {
    private List<TestResultStatus> testResultsStatus = new ArrayList<>();

    private enum TestResultStatus {
        SUCCESSFUL, ABORTED, FAILED, DISABLED;
    }

    //...
}
```

**和所有扩展接口一样，`TestWatcher`接口也扩展了主`Extension`接口**，它只是一个标记接口。在这个例子中，我们还实现了`AfterAllCallback`接口。

在我们的扩展中，我们有一个列表`TestResultStatus`，这是一个简单的枚举，我们将使用它来表示测试结果的状态。

### 4.1。处理测试结果

现在，让我们看看如何处理单个单元测试方法的结果:

```java
@Override
public void testDisabled(ExtensionContext context, Optional<String> reason) {
    LOG.info("Test Disabled for test {}: with reason :- {}", 
      context.getDisplayName(),
      reason.orElse("No reason"));

    testResultsStatus.add(TestResultStatus.DISABLED);
}

@Override
public void testSuccessful(ExtensionContext context) {
    LOG.info("Test Successful for test {}: ", context.getDisplayName());

    testResultsStatus.add(TestResultStatus.SUCCESSFUL);
} 
```

我们首先填充扩展的主体，**覆盖`testDisabled()`和`testSuccessful()`方法**。

在我们简单的例子中，我们输出测试的名称，并将测试的状态添加到`testResultsStatus`列表中。

**我们将以这种方式继续其他两种方法— `testAborted()`和`testFailed()` :**

```java
@Override
public void testAborted(ExtensionContext context, Throwable cause) {
    LOG.info("Test Aborted for test {}: ", context.getDisplayName());

    testResultsStatus.add(TestResultStatus.ABORTED);
}

@Override
public void testFailed(ExtensionContext context, Throwable cause) {
    LOG.info("Test Failed for test {}: ", context.getDisplayName());

    testResultsStatus.add(TestResultStatus.FAILED);
}
```

### 4.2。总结测试结果

在我们例子的最后一部分，**我们将覆盖`afterAll()`方法**:

```java
@Override
public void afterAll(ExtensionContext context) throws Exception {
    Map<TestResultStatus, Long> summary = testResultsStatus.stream()
      .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

    LOG.info("Test result summary for {} {}", context.getDisplayName(), summary.toString());
}
```

快速回顾一下，`afterAll`方法是在所有测试方法运行之后执行的。在输出一个非常基本的总结之前，我们使用这种方法对测试结果列表中的不同`TestResultStatus`进行分组。

关于生命周期回调的深入指南，请查看我们优秀的 JUnit 5 扩展指南。

## 5。运行测试

在倒数第二节中，我们将看到使用简单的日志扩展的测试结果。

既然我们已经定义了我们的扩展，我们将首先使用标准的`@ExtendWith`注释注册它:

```java
@ExtendWith(TestResultLoggerExtension.class)
class TestWatcherAPIUnitTest {

    @Test
    void givenFalseIsTrue_whenTestAbortedThenCaptureResult() {
        Assumptions.assumeTrue(false);
    }

    @Disabled
    @Test
    void givenTrueIsTrue_whenTestDisabledThenCaptureResult() {
        Assert.assertTrue(true);
    }

    //...
```

接下来，我们用单元测试填充我们的测试类，添加禁用的、中止的和成功的测试。

### 5.1。查看输出

当我们运行单元测试时，我们应该看到每个测试的输出:

```java
INFO  c.b.e.t.TestResultLoggerExtension - 
    Test Successful for test givenTrueIsTrue_whenTestAbortedThenCaptureResult()
...
Test result summary for TestWatcherAPIUnitTest {ABORTED=1, SUCCESSFUL=1, DISABLED=2}
```

当然，当所有的测试方法完成后，我们还会看到打印的摘要。

## 6。抓到你了

在最后一节中，让我们回顾一下在使用`TestWatcher`接口时应该注意的一些微妙之处:

*   不允许 TestWatcher 扩展影响测试的执行；这意味着**如果从`TestWatcher`抛出一个异常，它将不会传播到正在运行的测试**
*   目前，该 API 仅用于报告`@Test`方法和`@TestTemplate`方法的结果
*   默认情况下，如果没有向`testDisabled`方法提供原因，那么它将包含测试方法的完全限定名，后跟“`is @Disabled`

## 7。结论

总之，在本教程中，我们已经展示了如何利用 JUnit 5 `TestWatcher` API 来处理测试方法执行的结果。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221127014133/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-advanced)