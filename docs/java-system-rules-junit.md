# 系统规则库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-rules-junit>

## 1.概观

有时当编写单元测试时，我们可能需要测试直接与 [`System`](/web/20221208143839/https://www.baeldung.com/java-lang-system) 类交互的代码。通常在应用程序中，比如直接调用`[System.exit](/web/20221208143839/https://www.baeldung.com/java-system-exit)`或使用`System.in`读取参数的命令行工具。

在本教程中，**我们将看看一个名为[系统规则](https://web.archive.org/web/20221208143839/https://stefanbirkner.github.io/system-rules/)的简洁外部库的最常见特性，它为测试使用`System`类**的代码提供了一组 [JUnit 规则](/web/20221208143839/https://www.baeldung.com/junit-4-rules)。

## 2.Maven 依赖性

首先，让我们将[系统规则依赖关系](https://web.archive.org/web/20221208143839/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22system-rules%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.19.0</version>
</dependency>
```

我们还将添加从 [Maven Central](https://web.archive.org/web/20221208143839/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22system-lambda%22) 获得的[系统λ](https://web.archive.org/web/20221208143839/https://github.com/stefanbirkner/system-lambda)依赖性:

```java
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-lambda</artifactId>
    <version>1.1.0</version>
</dependency>
```

**由于系统规则不直接支持 [**JUnit5**](/web/20221208143839/https://www.baeldung.com/junit-5) ，我们增加了最后一个依赖项。**这提供了在测试中使用的系统 Lambda 包装方法。有一个基于[扩展](/web/20221208143839/https://www.baeldung.com/junit-5-extensions)的替代方案叫做[系统存根](/web/20221208143839/https://www.baeldung.com/java-system-stubs)。

## 3.使用系统属性

简单概括一下，Java 平台使用一个 [`Properties`对象](/web/20221208143839/https://www.baeldung.com/java-properties)来提供关于本地系统和配置的信息。我们可以很容易地打印出属性:

```java
System.getProperties()
  .forEach((key, value) -> System.out.println(key + ": " + value));
```

正如我们所看到的，属性包括诸如当前用户、Java 运行时的当前版本和文件路径名分隔符等信息:

```java
java.version: 1.8.0_221
file.separator: /
user.home: /Users/baeldung
os.name: Mac OS X
...
```

我们也可以通过使用`System.setProperty`方法来设置我们自己的系统属性。在处理我们测试中的系统属性时应该小心，因为这些属性是 JVM 全局的。

例如，如果我们设置了一个系统属性，我们应该确保在测试完成或者出现故障时将属性恢复到其原始值。这有时会导致繁琐的设置和代码崩溃。然而，如果我们忽略了这一点，它可能会在我们的测试中导致意想不到的副作用。

在下一节中，我们将看到如何在测试完成后以简洁和简单的方式提供、清理并确保恢复系统属性值。

## 4.提供系统属性

假设我们有一个系统属性`log_dir` ，它包含我们的日志应该被写入的位置，并且我们的应用程序在启动时设置这个位置:

```java
System.setProperty("log_dir", "/tmp/baeldung/logs");
```

### 4.1.提供单一属性

现在让我们考虑从我们的单元测试中，我们想要提供一个不同的值。我们可以使用`ProvideSystemProperty`规则来做到这一点:

```java
public class ProvidesSystemPropertyWithRuleUnitTest {

    @Rule
    public final ProvideSystemProperty providesSystemPropertyRule = new ProvideSystemProperty("log_dir", "test/resources");

    @Test
    public void givenProvideSystemProperty_whenGetLogDir_thenLogDirIsProvidedSuccessfully() {
        assertEquals("log_dir should be provided", "test/resources", System.getProperty("log_dir"));
    }
    // unit test definition continues
} 
```

**使用`ProvideSystemProperty`规则，我们可以为给定的系统属性设置一个任意值，以供我们的测试使用。**在这个例子中，我们将`log_dir`属性设置为我们的`test/resources`目录，并且从我们的单元测试中，简单地断言测试属性值已经被成功提供。

如果我们在测试类完成时打印出`log_dir`属性的值:

```java
@AfterClass
public static void tearDownAfterClass() throws Exception {
    System.out.println(System.getProperty("log_dir"));
} 
```

我们可以看到财产的价值已经恢复到原来的价值:

```java
/tmp/baeldung/logs
```

### 4.2.提供多种属性

如果我们需要提供多个属性，我们可以使用`and`方法将我们测试所需的尽可能多的属性值链接在一起:

```java
@Rule
public final ProvideSystemProperty providesSystemPropertyRule = 
    new ProvideSystemProperty("log_dir", "test/resources").and("another_property", "another_value")
```

### 4.3.从文件中提供属性

同样，我们也可以使用`ProvideSystemProperty`规则从文件或类路径资源中提供属性:

```java
@Rule
public final ProvideSystemProperty providesSystemPropertyFromFileRule = 
  ProvideSystemProperty.fromResource("/test.properties");

@Test
public void givenProvideSystemPropertyFromFile_whenGetName_thenNameIsProvidedSuccessfully() {
    assertEquals("name should be provided", "baeldung", System.getProperty("name"));
    assertEquals("version should be provided", "1.0", System.getProperty("version"));
}
```

在上面的例子中，我们假设在类路径中有一个`test.properties`文件:

```java
name=baeldung
version=1.0
```

### 4.4.用 JUnit5 和 Lambdas 提供属性

正如我们前面提到的，我们也可以使用库的 System Lambda 版本来实现与 JUnit5 兼容的测试。

让我们看看如何使用这个版本的库来实现我们的测试:

```java
@BeforeAll
static void setUpBeforeClass() throws Exception {
    System.setProperty("log_dir", "/tmp/baeldung/logs");
}

@Test
void givenSetSystemProperty_whenGetLogDir_thenLogDirIsProvidedSuccessfully() throws Exception {
    restoreSystemProperties(() -> {
        System.setProperty("log_dir", "test/resources");
        assertEquals("log_dir should be provided", "test/resources", System.getProperty("log_dir"));
    });

    assertEquals("log_dir should be provided", "/tmp/baeldung/logs", System.getProperty("log_dir"));
}
```

在这个版本中，我们可以使用`restoreSystemProperties`方法来执行给定的语句。**在这个语句中，我们可以设置并提供我们需要的系统属性**的值。我们可以看到，在这个方法执行完毕后，`log_dir`的值和之前的`/tmp/baeldung/logs`是一样的。

不幸的是，没有内置的支持来使用`restoreSystemProperties` 方法从文件中提供属性。

## 5.清除系统属性

有时，我们可能希望在测试开始时清除一组系统属性，并在测试结束时恢复它们的原始值，而不管测试是通过还是失败。

为此，我们可以使用`ClearSystemProperties`规则:

```java
@Rule
public final ClearSystemProperties userNameIsClearedRule = new ClearSystemProperties("user.name");

@Test
public void givenClearUsernameProperty_whenGetUserName_thenNull() {
    assertNull(System.getProperty("user.name"));
}
```

系统属性`user.name`是预定义的系统属性之一，它包含用户帐户名。正如在上面的单元测试中所预期的那样，我们清除了这个属性，并从我们的测试中检查它是否为空。

**方便的是，我们也可以将多个属性名传递给`ClearSystemProperties`构造函数。**

## 6.嘲讽`System.in`

有时，我们可能会创建从`System.in`读取数据的交互式命令行应用程序。

在本节中，我们将使用一个非常简单的示例，它从标准输入中读取名字和姓氏，并将它们连接在一起:

```java
private String getFullname() {
    try (Scanner scanner = new Scanner(System.in)) {
        String firstName = scanner.next();
        String surname = scanner.next();
        return String.join(" ", firstName, surname);
    }
}
```

系统规则包含了`TextFromStandardInputStream`规则，我们可以用它来指定调用`System.in`时应该提供的行:

```java
@Rule
public final TextFromStandardInputStream systemInMock = emptyStandardInputStream();

@Test
public void givenTwoNames_whenSystemInMock_thenNamesJoinedTogether() {
    systemInMock.provideLines("Jonathan", "Cook");
    assertEquals("Names should be concatenated", "Jonathan Cook", getFullname());
}
```

**我们通过使用`providesLines`方法来实现这一点，该方法使用一个 [varargs](/web/20221208143839/https://www.baeldung.com/java-varargs) 参数来指定多个值。**

在这个例子中，我们在调用`getFullname`方法之前提供了两个值，其中引用了`System.in`。我们提供的两个行值将在每次调用`scanner.next()`时返回。

让我们看看如何使用 System Lambda 在 JUnit 5 版本的测试中实现同样的效果:

```java
@Test
void givenTwoNames_whenSystemInMock_thenNamesJoinedTogether() throws Exception {
    withTextFromSystemIn("Jonathan", "Cook").execute(() -> {
        assertEquals("Names should be concatenated", "Jonathan Cook", getFullname());
    });
}
```

**在这个变体中，我们使用类似命名的`withTextFromSystemIn `方法，它让我们指定所提供的`System.in`值。**

重要的是，在这两种情况下，测试结束后，`System.in`的原始值将被恢复。

## 7.测试`System.out`和`System.err`

在之前的教程中，我们看到了如何使用系统规则进行单元测试`[System.out.println()](/web/20221208143839/https://www.baeldung.com/java-testing-system-out-println#using-system-rules).`

方便的是，我们可以应用几乎相同的方法来测试与标准错误流交互的代码。这次我们使用`SystemErrRule`:

```java
@Rule
public final SystemErrRule systemErrRule = new SystemErrRule().enableLog();

@Test
public void givenSystemErrRule_whenInvokePrintln_thenLogSuccess() {
    printError("An Error occurred Baeldung Readers!!");

    Assert.assertEquals("An Error occurred Baeldung Readers!!", 
      systemErrRule.getLog().trim());
}

private void printError(String output) {
    System.err.println(output);
}
```

不错！使用`SystemErrRule`，我们可以拦截对`System.err`的写入。首先，我们通过调用规则上的`enableLog`方法开始记录写入`System.err`的所有内容。然后我们简单地调用`getLog`将文本写入 System.err，因为我们调用了`enableLog`。

现在，让我们实现测试的 JUnit5 版本:

```java
@Test
void givenTapSystemErr_whenInvokePrintln_thenOutputIsReturnedSuccessfully() throws Exception {

    String text = tapSystemErr(() -> {
        printError("An error occurred Baeldung Readers!!");
    });

    Assert.assertEquals("An error occurred Baeldung Readers!!", text.trim());
}
```

在这个版本中，**我们使用了`tapSystemErr`方法，它执行语句并让我们捕获传递给`System.err`的内容。**

## 8.处理`System.exit`

命令行应用程序通常通过调用`System.exit`来终止。**如果我们想要测试这样一个应用程序，当我们的测试遇到调用`System.exit`的代码时，它很可能会在完成之前异常终止。**

幸运的是，系统规则提供了一个简洁的解决方案，使用`ExpectedSystemExit`规则来处理这个问题:

```java
@Rule
public final ExpectedSystemExit exitRule = ExpectedSystemExit.none();

@Test
public void givenSystemExitRule_whenAppCallsSystemExit_thenExitRuleWorkssAsExpected() {
    exitRule.expectSystemExitWithStatus(1);
    exit();
}

private void exit() {
    System.exit(1);
}
```

使用`ExpectedSystemExit`规则允许我们从测试中指定预期的`System.exit()`调用。在这个简单的例子中，我们还使用`expectSystemExitWithStatus`方法检查预期的状态代码。

**我们可以在 JUnit 5 版本中使用`catchSystemExit`方法**实现类似的东西:

```java
@Test
void givenCatchSystemExit_whenAppCallsSystemExit_thenStatusIsReturnedSuccessfully() throws Exception {
    int statusCode = catchSystemExit(() -> {
        exit();
    });
    assertEquals("status code should be 1:", 1, statusCode);
}
```

## 9.结论

总而言之，在本教程中，我们已经详细探索了系统规则库。

首先，我们从解释如何测试使用系统属性的代码开始。然后我们看了如何测试标准输出和标准输入。最后，我们看了如何处理从测试中调用 `System.exit`的代码。

系统规则库还支持从我们的测试中提供环境变量和特殊的安全管理器。请务必查看完整的[文档](https://web.archive.org/web/20221208143839/https://stefanbirkner.github.io/system-rules/)了解详情。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries-2)