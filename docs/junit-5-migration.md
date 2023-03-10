# 从 JUnit 4 迁移到 JUnit 5

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-migration>

## 1.概观

在本教程中，我们将学习如何从 JUnit 4 迁移到最新的 JUnit 5 版本，并概述两个版本的库之间的差异。

关于使用 JUnit 5 的一般指南，请参见我们的文章[这里](/web/20221105002446/https://www.baeldung.com/junit-5)。

## 2.JUnit 5 的优势

让我们从之前的版本 JUnit 4 开始，它有一些明显的限制:

*   一个 jar 库包含了整个框架。我们需要导入整个库，即使我们只需要一个特定的特性。在 JUnit 5 中，我们获得了更多的粒度，可以只导入必要的内容。
*   在 JUnit 4 中，一次只能有一个测试运行人员执行测试(例如`SpringJUnit4ClassRunner`或`Parameterized` )。 **JUnit 5 允许多个跑步者同时工作。**
*   JUnit 4 从未超越 Java 7，错过了 Java 8 的许多特性。JUnit 5 很好地利用了 Java 8 的特性。

JUnit 5 背后的想法是完全重写 JUnit 4，以消除这些缺点。

## 3.差异

JUnit 4 分为组成 JUnit 5 的模块:

*   JUnit 平台–这个模块涵盖了我们可能感兴趣的所有扩展框架:测试执行、发现和报告。
*   **JUnit Vintage—**这个模块允许向后兼容 JUnit 4 甚至 JUnit 3。

### 3.1.释文

JUnit 5 对其注释进行了重要的修改。**最重要的一点是，我们不能再使用`@Test`注释来指定期望值。**

JUnit 4 中的`expected`参数:

```java
@Test(expected = Exception.class)
public void shouldRaiseAnException() throws Exception {
    // ...
}
```

现在我们可以使用方法`assertThrows`:

```java
public void shouldRaiseAnException() throws Exception {
    Assertions.assertThrows(Exception.class, () -> {
        //...
    });
}
```

JUnit 4 中的`timeout` 属性:

```java
@Test(timeout = 1)
public void shouldFailBecauseTimeout() throws InterruptedException {
    Thread.sleep(10);
}
```

现在 JUnit 5 中的`assertTimeout`方法:

```java
@Test
public void shouldFailBecauseTimeout() throws InterruptedException {
    Assertions.assertTimeout(Duration.ofMillis(1), () -> Thread.sleep(10));
}
```

以下是 JUnit 5 中更改的一些其他注释:

*   `@Before`注释现在是`@BeforeEach`
*   `@After`注释现在是`@AfterEach`
*   `@BeforeClass`注释现在是`@BeforeAll`
*   `@AfterClass`注释现在是`@AfterAll`
*   `@Ignore`注释现在是`@Disabled`

### 3.2.断言

我们还可以在 JUnit 5 的 lambda 中编写断言消息，允许惰性评估跳过复杂的消息构造，直到需要为止:

```java
@Test
public void shouldFailBecauseTheNumbersAreNotEqual_lazyEvaluation() {
    Assertions.assertTrue(
      2 == 3, 
      () -> "Numbers " + 2 + " and " + 3 + " are not equal!");
}
```

此外，我们可以在 JUnit 5 中对断言进行分组:

```java
@Test
public void shouldAssertAllTheGroup() {
    List<Integer> list = Arrays.asList(1, 2, 4);
    Assertions.assertAll("List is not incremental",
        () -> Assertions.assertEquals(list.get(0).intValue(), 1),
        () -> Assertions.assertEquals(list.get(1).intValue(), 2),
        () -> Assertions.assertEquals(list.get(2).intValue(), 3));
}
```

### 3.3.假设

新的`Assumptions`类现在在`org.junit.jupiter.api.Assumptions`中。JUnit 5 完全支持 JUnit 4 中现有的假设方法，还添加了一组新方法，允许我们仅在特定场景下运行一些断言:

```java
@Test
public void whenEnvironmentIsWeb_thenUrlsShouldStartWithHttp() {
    assumingThat("WEB".equals(System.getenv("ENV")),
      () -> {
          assertTrue("http".startsWith(address));
      });
}
```

### 3.4.标记和过滤

在 JUnit 4 中，我们可以使用`@Category`注释对测试进行分组。在 JUnit 5 中， `@Category`注释被`@Tag`注释所取代:

```java
@Tag("annotations")
@Tag("junit5")
public class AnnotationTestExampleTest {
    /*...*/
}
```

我们可以使用`maven-surefire-plugin`来包含/排除特定的标签:

```java
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <properties>
                    <includeTags>junit5</includeTags>
                </properties>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 3.5.运行测试的新注释

在 JUnit 4 中，我们使用了`@RunWith `注释来将测试上下文与其他框架集成，或者改变测试用例中的整体执行流程。

有了 JUnit 5，我们现在可以使用`@ExtendWith`注释来提供类似的功能。

例如，要使用 JUnit 4 中的 Spring 特性:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  {"/app-config.xml", "/test-data-access-config.xml"})
public class SpringExtensionTest {
    /*...*/
}
```

在 JUnit 5 中，它是一个简单的扩展:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(
  { "/app-config.xml", "/test-data-access-config.xml" })
public class SpringExtensionTest {
    /*...*/
} 
```

### 3.6.新测试规则注释

在 JUnit 4 中，我们使用了`@Rule`和@ `ClassRule`注释来为测试添加特殊的功能。

在 JUnit 5 中。我们可以使用`@ExtendWith`注释重现相同的逻辑。

例如，假设我们在 JUnit 4 中有一个自定义规则，用于在测试前后编写日志跟踪:

```java
public class TraceUnitTestRule implements TestRule {

    @Override
    public Statement apply(Statement base, Description description) {
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                // Before and after an evaluation tracing here 
                ...
            }
        };
    }
}
```

我们在一个测试套件中实现了它:

```java
@Rule
public TraceUnitTestRule traceRuleTests = new TraceUnitTestRule(); 
```

在 JUnit 5 中，我们可以用更直观的方式编写相同的内容:

```java
public class TraceUnitExtension implements AfterEachCallback, BeforeEachCallback {

    @Override
    public void beforeEach(TestExtensionContext context) throws Exception {
        // ...
    }

    @Override
    public void afterEach(TestExtensionContext context) throws Exception {
        // ...
    }
}
```

使用 JUnit 5 的`AfterEachCallback` 和 `BeforeEachCallback`接口，在`org.junit.jupiter.api.extension` 包中可用，我们可以很容易地在测试套件中实现这个规则:

```java
@ExtendWith(TraceUnitExtension.class)
public class RuleExampleTest {

    @Test
    public void whenTracingTests() {
        /*...*/
    }
}
```

### 3.7.JUnit 5 复古

JUnit Vintage 通过在 JUnit 5 环境中运行 JUnit 3 或 JUnit 4 测试来帮助 JUnit 测试的迁移。

我们可以通过导入 JUnit Vintage 引擎来使用它:

```java
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>${junit5.vintage.version}</version>
    <scope>test</scope>
</dependency>
```

## 4.结论

JUnit 5 是 JUnit 4 框架的一个模块化的现代版本。在本文中，我们介绍了这两个版本之间的主要区别，并暗示了如何从一个版本迁移到另一个版本。

这篇文章的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221105002446/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics)