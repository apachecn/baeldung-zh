# JUnit 4 规则指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-4-rules>

## 1。概述

在本教程中，我们将看看由 [JUnit 4](https://web.archive.org/web/20221206001420/https://junit.org/junit4/) 库提供的规则特性。

在介绍发行版提供的最重要的基本规则之前，我们将首先介绍 JUnit 规则模型。此外，我们还将了解如何编写和使用我们自己的自定义 JUnit 规则。

要了解更多关于使用 JUnit 进行测试的信息，请查看我们全面的 [JUnit 系列](/web/20221206001420/https://www.baeldung.com/junit)。

**注意，如果你使用的是 JUnit 5，规则已经被[扩展模型](/web/20221206001420/https://www.baeldung.com/junit-5-extensions)所取代。**

## 2。JUnit 4 规则介绍

**JUnit 4 规则提供了一种灵活的机制，通过围绕测试用例执行运行一些代码来增强测试**。在某种意义上，这类似于在我们的测试类中有 [`@Before`和`@After`注释](/web/20221206001420/https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall)。

假设我们想在测试设置期间连接到一个外部资源，比如一个数据库，然后在测试完成后关闭这个连接。如果我们想要在多个测试中使用那个数据库，我们将会在每个测试中重复那个代码。

通过使用规则，我们可以将所有东西隔离在一个地方，并从多个测试类中轻松地重用代码。

## 3。使用 JUnit 4 规则

那么我们如何使用规则呢？我们可以通过以下简单的步骤来使用 JUnit 4 规则:

*   向我们的测试类添加一个`public`字段，并确保该字段的类型是`org.junit.rules.TestRule`接口的子类型
*   用`@Rule`注释对字段进行注释

在下一节中，我们将看到我们需要哪些项目依赖关系来开始。

## 4。Maven 依赖关系

首先，让我们添加例子中需要的项目依赖项。我们只需要主要的 JUnit 4 库:

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency> 
```

一如既往，我们可以从 [Maven Central](https://web.archive.org/web/20221206001420/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22junit%22%20AND%20a%3A%22junit%22) 获得最新版本。

## 5。分布中提供的规则

当然， **JUnit 提供了许多有用的预定义规则，作为库**的一部分。我们可以在`org.junit.rules`包中找到所有这些规则。

在这一节中，我们将看到一些如何使用它们的例子。

### 5.1。`TemporaryFolder`规则

测试时，我们经常需要访问一个临时文件或文件夹。然而，管理这些文件的创建和删除可能很麻烦。使用`TemporaryFolder`规则，我们可以管理当测试方法终止时应该被删除的文件和文件夹的创建:

```java
@Rule
public TemporaryFolder tmpFolder = new TemporaryFolder();

@Test
public void givenTempFolderRule_whenNewFile_thenFileIsCreated() throws IOException {
    File testFile = tmpFolder.newFile("test-file.txt");

    assertTrue("The file should have been created: ", testFile.isFile());
    assertEquals("Temp folder and test file should match: ", 
      tmpFolder.getRoot(), testFile.getParentFile());
}
```

正如我们所见，我们首先定义了`TemporaryFolder`规则`tmpFolder`。接下来，我们的测试方法在临时文件夹中创建一个名为`test-file.txt`的文件。然后，我们检查文件是否已经创建并存在于它应该存在的地方。真的好简单！

测试完成后，应删除临时文件夹和文件。然而，该规则并不检查删除是否成功。

本课程中还有一些其他有趣的方法值得一提:

*   ```java
    newFile()
    ```

    如果我们不提供任何文件名，那么这个方法会创建一个随机命名的新文件。

*   ```java
    newFolder(String... folderNames)
    ```

    为了递归地创建深层临时文件夹，我们可以使用这个方法。

*   ```java
    newFolder()
    ```

    同样，`newFolder()`方法创建一个随机命名的新文件夹。

值得一提的是，从 4.13 版本开始，`TemporaryFolder`规则允许验证删除的资源:

```java
@Rule 
public TemporaryFolder folder = TemporaryFolder.builder().assureDeletion().build();
```

如果一个资源不能被删除，测试将失败并显示一个`AssertionError`。

最后，在 [JUnit 5](/web/20221206001420/https://www.baeldung.com/junit-5) 中，我们可以使用[临时目录扩展](/web/20221206001420/https://www.baeldung.com/junit-5-temporary-directory)来实现相同的功能。

### 5.2。`ExpectedException`规则

顾名思义，**我们可以使用`ExpectedException`规则来验证某些代码抛出了预期的[异常](/web/20221206001420/https://www.baeldung.com/java-exceptions) :**

```java
@Rule
public final ExpectedException thrown = ExpectedException.none();

@Test
public void givenIllegalArgument_whenExceptionThrown_MessageAndCauseMatches() {
    thrown.expect(IllegalArgumentException.class);
    thrown.expectCause(isA(NullPointerException.class));
    thrown.expectMessage("This is illegal");

    throw new IllegalArgumentException("This is illegal", new NullPointerException());
}
```

正如我们在上面的例子中看到的，我们首先声明了`ExpectedException`规则。然后，在我们的测试中，我们断言抛出了一个`IllegalArgumentException`。

使用这个规则，我们还可以验证异常的一些其他属性，比如消息和原因。

关于用 JUnit 测试异常的深入指南，请查看我们关于如何[断言异常](/web/20221206001420/https://www.baeldung.com/junit-assert-exception)的优秀指南。

### 5.3。`TestName`规则

**简单地说，`TestName`规则提供了给定测试方法中的当前测试名称:**

```java
@Rule public TestName name = new TestName();

@Test
public void givenAddition_whenPrintingTestName_thenTestNameIsDisplayed() {
    LOG.info("Executing: {}", name.getMethodName());
    assertEquals("givenAddition_whenPrintingTestName_thenTestNameIsDisplayed", name.getMethodName());
}
```

在这个简单的例子中，当我们运行单元测试时，我们应该在输出中看到测试名称:

```java
INFO  c.baeldung.rules.JUnitRulesUnitTest - 
    Executing: givenAddition_whenPrintingTestName_thenTestNameIsDisplayed
```

### 5.4。`Timeout`规则

在下一个例子中，我们将看看`Timeout`规则。**这个规则提供了一个有用的替代方法，可以在一个单独的测试注释**上使用超时参数。

现在，让我们看看如何使用这个规则在测试类中的所有测试方法上设置一个全局超时:

```java
@Rule
public Timeout globalTimeout = Timeout.seconds(10);

@Test
public void givenLongRunningTest_whenTimout_thenTestFails() throws InterruptedException {
    TimeUnit.SECONDS.sleep(20);
}
```

在上面这个简单的例子中，**我们首先为所有的测试方法定义了一个 10 秒的全局超时**。然后，我们故意定义一个测试，它将花费超过 10 秒的时间。

当我们运行这个测试时，我们应该看到一个测试失败:

```java
org.junit.runners.model.TestTimedOutException: test timed out after 10 seconds
...
```

### 5.5。`ErrorCollector`规则

接下来，我们将看看`ErrorCollector`规则。**该规则允许在发现第一个问题后继续执行测试**。

让我们看看如何使用这个规则来收集所有的错误，并在测试终止时立即报告它们:

```java
@Rule 
public final ErrorCollector errorCollector = new ErrorCollector();

@Test
public void givenMultipleErrors_whenTestRuns_thenCollectorReportsErrors() {
    errorCollector.addError(new Throwable("First thing went wrong!"));
    errorCollector.addError(new Throwable("Another thing went wrong!"));

    errorCollector.checkThat("Hello World", not(containsString("ERROR!")));
}
```

在上面的例子中，我们向收集器添加了两个错误。当我们运行测试时，执行会继续，但是测试最终会失败。

在输出中，我们将看到报告的两个错误:

```java
java.lang.Throwable: First thing went wrong!
...
java.lang.Throwable: Another thing went wrong!
```

### 5.6。`Verifier`规则

**`Verifier`规则是一个抽象的基类，当我们希望从测试中验证一些额外的行为时可以使用它**。事实上，我们在上一节看到的`ErrorCollector`规则扩展了这个类。

现在让我们看一个定义我们自己的验证器的小例子:

```java
private List messageLog = new ArrayList();

@Rule
public Verifier verifier = new Verifier() {
    @Override
    public void verify() {
        assertFalse("Message Log is not Empty!", messageLog.isEmpty());
    }
}; 
```

这里，我们定义了一个新的`Verifier`并覆盖了`verify()`方法来添加一些额外的验证逻辑。在这个简单的例子中，我们只是检查例子中的消息日志是否为空。

现在，当我们运行单元测试并添加消息时，我们应该看到我们的验证器已经被应用了:

```java
@Test
public void givenNewMessage_whenVerified_thenMessageLogNotEmpty() {
    // ...
    messageLog.add("There is a new message!");
}
```

### 5.7。`DisableOnDebug`规则

**当我们调试**时，有时我们可能想要禁用一个规则。例如，在调试时禁用`Timeout`规则通常是可取的，以避免我们的测试在有时间正确调试之前超时和失败。

`DisableOnDebug`规则恰恰做到了这一点，并允许我们在调试时标记某些要禁用的规则:

```java
@Rule
public DisableOnDebug disableTimeout = new DisableOnDebug(Timeout.seconds(30));
```

在上面的例子中我们可以看到，为了使用这个规则，**我们简单地将我们想要禁用的规则传递给构造函数。**

这条规则的主要好处是，我们可以禁用规则，而无需在调试期间对测试类进行任何修改。

### 5.8。`ExternalResource`规则

通常，当编写集成测试时，我们可能希望在测试之前设置一个外部资源，然后再拆除它。幸运的是，JUnit 为此提供了另一个方便的基类。

**我们可以扩展抽象类`ExternalResource`来在测试前设置外部资源，比如文件或数据库连接。**事实上，我们之前看到的`TemporaryFolder`法则延伸了`ExternalResource`。

让我们快速看一下如何扩展这个类:

```java
@Rule
public final ExternalResource externalResource = new ExternalResource() {
    @Override
    protected void before() throws Throwable {
        // code to set up a specific external resource.
    };

    @Override
    protected void after() {
        // code to tear down the external resource
    };
};
```

在这个例子中，当我们定义一个外部资源时，我们只需要覆盖`before()`方法和`after()`方法，以便设置和删除我们的外部资源。

## 6。应用分类规则

到目前为止，我们看到的所有例子都适用于单个测试用例方法。**然而，有时我们可能想要在测试类级别应用一个规则**。我们可以通过使用`@ClassRule`注释来实现这一点。

这个注释的工作方式与`@Rule`非常相似，但是在整个测试中包含了一个规则— **主要的区别是我们用于类规则的字段必须是静态的:**

```java
@ClassRule
public static TemporaryFolder globalFolder = new TemporaryFolder();
```

## 7。定义自定义 JUnit 规则

正如我们所见，JUnit 4 提供了许多现成的有用规则。当然，我们可以定义自己的自定义规则。**要编写自定义规则，我们需要实现`TestRule`接口。**

让我们来看一个定义定制测试方法名称记录器规则的例子:

```java
public class TestMethodNameLogger implements TestRule {

    private static final Logger LOG = LoggerFactory.getLogger(TestMethodNameLogger.class);

    @Override
    public Statement apply(Statement base, Description description) {
        logInfo("Before test", description);
        try {
            return new Statement() {
                @Override
                public void evaluate() throws Throwable {
                    base.evaluate();
                }
            };
        } finally {
            logInfo("After test", description);
        }
    }

    private void logInfo(String msg, Description description) {
        LOG.info(msg + description.getMethodName());
    }
}
```

正如我们所见，`TestRule`接口包含一个名为`apply(Statement, Description)`的方法，我们必须覆盖它以返回一个`Statement`的实例。该语句表示我们在 JUnit 运行时中的测试。**当我们调用`evaluate()`方法时，它执行我们的测试。**

在这个例子中，我们记录了一个 before 和 after 消息，并包含了来自`Description`对象的单个测试的方法名。

## 8。使用规则链

**在最后一部分，我们将看看如何使用`RuleChain`规则:**来排序几个测试规则

```java
@Rule
public RuleChain chain = RuleChain.outerRule(new MessageLogger("First rule"))
    .around(new MessageLogger("Second rule"))
    .around(new MessageLogger("Third rule"));
```

在上面的例子中，我们创建了一个由三个规则组成的链，简单地打印出传递给每个`MessageLogger`构造函数的消息。

当我们运行我们的测试时，我们将看到这个链是如何按顺序应用的:

```java
Starting: First rule
Starting: Second rule
Starting: Third rule
Finished: Third rule
Finished: Second rule
Finished: First rule
```

## 9。结论

总而言之，在本教程中，我们已经详细研究了 JUnit 4 规则。

首先，我们从解释什么是规则以及如何使用规则开始。接下来，我们深入研究了作为 JUnit 发行版一部分的规则。

最后，我们看了如何定义我们自己的自定义规则，以及如何将规则链接在一起。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221206001420/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)