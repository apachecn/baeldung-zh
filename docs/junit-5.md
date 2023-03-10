# JUnit 5 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5>

## 1。概述

JUnit 是 Java 生态系统中最流行的单元测试框架之一。JUnit 5 版本包含了许多激动人心的创新，**的目标是支持 Java 8 和更高版本的新特性**，以及支持许多不同风格的测试。

## 延伸阅读:

## 【JUnit 5 的并行测试执行

In this article, we'll cover how to execute parallel unit tests using JUnit 5.[Read more](/web/20220812054122/https://www.baeldung.com/junit-5-parallel-tests) →

## [使用 JUnit 5 和 Gradle](/web/20220812054122/https://www.baeldung.com/junit-5-gradle)

Learn how to set up and run JUnit 5 tests with Gradle.[Read more](/web/20220812054122/https://www.baeldung.com/junit-5-gradle) →

## [JUnit 5 参数化测试指南](/web/20220812054122/https://www.baeldung.com/parameterized-tests-junit-5)

Learn how to simplify test coverage in JUnit 5 with parameterized tests[Read more](/web/20220812054122/https://www.baeldung.com/parameterized-tests-junit-5) →

## 2。Maven 依赖关系

设置 [JUnit 5.x.0](https://web.archive.org/web/20220812054122/https://search.maven.org/search?q=a:junit-jupiter-engine) 非常简单；我们只需要将以下依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

此外，现在还直接支持在 Eclipse 中的 JUnit 平台以及 IntelliJ 上运行单元测试。当然，我们也可以使用 Maven 测试目标运行测试。

另一方面，IntelliJ 默认支持 JUnit 5。因此，在 IntelliJ 上运行 JUnit 5 相当容易。我们只需右击->运行，或者 Ctrl-Shift-F10。

需要注意的是，这个版本**需要 Java 8 才能工作**。

## 3。架构

JUnit 5 由来自三个不同子项目的几个不同模块组成。

### 3.1。JUnit 平台

该平台负责在 JVM 上启动测试框架。它在 JUnit 和它的客户(比如构建工具)之间定义了一个稳定而强大的接口。

该平台可以轻松地将客户端与 JUnit 集成在一起，以发现和执行测试。

它还定义了用于开发在 JUnit 平台上运行的测试框架的 test engine API。通过实现定制的 TestEngine，我们可以将第三方测试库直接插入 JUnit。

### 3.2。朱尼特木星

这个模块包括新的编程和扩展模型，用于在 JUnit 5 中编写测试。与 JUnit 4 相比，新的注释是:

*   `@TestFactory`–表示作为动态测试的测试工厂的方法
*   `@DisplayName` –定义测试类别或测试方法的自定义显示名称
*   `@Nested` –表示带注释的类是嵌套的、非静态的测试类
*   `@Tag` –声明过滤测试的标签
*   `@ExtendWith`–注册自定义扩展
*   `@BeforeEach –` 表示带注释的方法将在每个测试方法之前执行(之前为`@Before`
*   `@AfterEach` –表示注释方法将在每个测试方法之后执行(之前为`@After`)
*   `@BeforeAll` –表示带注释的方法将在当前类中的所有测试方法之前执行(之前为`@BeforeClass`)
*   `@AfterAll`–表示带注释的方法将在当前类中的所有测试方法之后执行(之前为`@AfterClass`)
*   `@Disable` –禁用测试类别或方法(之前为`@Ignore`)

### 3.3。JUnit Vintage

JUnit Vintage 支持在 JUnit 5 平台上运行基于 JUnit 3 和 JUnit 4 的测试。

## 4。基本注释

为了讨论新的注释，我们将这一部分分为以下负责执行的组:测试前、测试期间(可选)和测试后:

### 4.1。`@BeforeAll`和`@BeforeEach`

下面是在主要测试用例之前执行的简单代码示例:

```java
@BeforeAll
static void setup() {
    log.info("@BeforeAll - executes once before all test methods in this class");
}

@BeforeEach
void init() {
    log.info("@BeforeEach - executes before each test method in this class");
}
```

需要注意的是，带有`@BeforeAll` 注释的方法需要是静态的，否则代码将无法编译。

### 4.2。`@DisplayName`和`@Disabled`

现在让我们转向新的可选测试方法:

```java
@DisplayName("Single test successful")
@Test
void testSingleSuccessTest() {
    log.info("Success");
}

@Test
@Disabled("Not implemented yet")
void testShowSomething() {
}
```

正如我们所看到的，我们可以使用新的注释来更改显示名称或禁用带有注释的方法。

### 4.3。`@AfterEach`和`@AfterAll`

最后，让我们讨论与测试执行后的操作相关的方法:

```java
@AfterEach
void tearDown() {
    log.info("@AfterEach - executed after each test method.");
}

@AfterAll
static void done() {
    log.info("@AfterAll - executed after all test methods.");
}
```

请注意，带`@AfterAll`的方法也需要是静态方法。

## 5。断言和假设

JUnit 5 试图充分利用 Java 8 的新特性，尤其是 lambda 表达式。

### 5.1。断言

断言被移到了`org.junit.jupiter.api.Assertions,` ，并得到了显著的改进。如前所述，我们现在可以在断言中使用 lambdas:

```java
@Test
void lambdaExpressions() {
    List numbers = Arrays.asList(1, 2, 3);
    assertTrue(numbers.stream()
      .mapToInt(Integer::intValue)
      .sum() > 5, () -> "Sum should be greater than 5");
}
```

虽然上面的例子很简单，但是对断言消息使用 lambda 表达式的一个优点是它是延迟求值的，如果消息构造很昂贵，这可以节省时间和资源。

现在还可以用`assertAll(),` 对断言进行分组，这将报告组内任何失败的断言，用`MultipleFailuresError`:

```java
 @Test
 void groupAssertions() {
     int[] numbers = {0, 1, 2, 3, 4};
     assertAll("numbers",
         () -> assertEquals(numbers[0], 1),
         () -> assertEquals(numbers[3], 3),
         () -> assertEquals(numbers[4], 1)
     );
 }
```

这意味着现在可以更安全地做出更复杂的断言，因为我们将能够精确定位任何故障的确切位置。

### 5.2。假设

假设仅在满足特定条件时用于运行测试。这通常用于测试正常运行所需的外部条件，但这些条件与测试的内容没有直接关系。

我们可以用`assumeTrue()`、`assumeFalse()`和`assumingThat():`来声明一个假设

```java
@Test
void trueAssumption() {
    assumeTrue(5 > 1);
    assertEquals(5 + 2, 7);
}

@Test
void falseAssumption() {
    assumeFalse(5 < 1);
    assertEquals(5 + 2, 7);
}

@Test
void assumptionThat() {
    String someString = "Just a string";
    assumingThat(
        someString.equals("Just a string"),
        () -> assertEquals(2 + 2, 4)
    );
}
```

如果一个假设失败，抛出一个`TestAbortedException`并简单地跳过测试。

假设也理解 lambda 表达式。

## 6。异常测试

JUnit 5 中有两种异常测试方法，这两种方法我们都可以使用`assertThrows()`方法实现:

```java
@Test
void shouldThrowException() {
    Throwable exception = assertThrows(UnsupportedOperationException.class, () -> {
      throw new UnsupportedOperationException("Not supported");
    });
    assertEquals("Not supported", exception.getMessage());
}

@Test
void assertThrowsException() {
    String str = null;
    assertThrows(IllegalArgumentException.class, () -> {
      Integer.valueOf(str);
    });
}
```

第一个例子验证抛出异常的细节，第二个例子验证异常的类型。

## 7。测试套件

为了继续 JUnit 5 的新特性，我们将探索在一个测试套件中聚合多个测试类的概念，以便我们可以一起运行它们。JUnit 5 提供了两个注释，`@SelectPackages`和`@SelectClasses,`来创建测试套件。

请记住，在这个早期阶段，大多数 ide 都不支持这些特性。

让我们看看第一个:

```java
@Suite
@SelectPackages("com.baeldung")
@ExcludePackages("com.baeldung.suites")
public class AllUnitTest {}
```

`@SelectPackage`用于指定运行测试套件时要选择的包的名称。在我们的例子中，它将运行所有的测试。第二个注释`@SelectClasses`，用于指定运行测试套件时要选择的类:

```java
@Suite
@SelectClasses({AssertionTest.class, AssumptionTest.class, ExceptionTest.class})
public class AllUnitTest {}
```

例如，上面的类将创建一个包含三个测试类的套件。请注意，这些类不必在一个包中。

## 8。动态测试

我们要介绍的最后一个主题是 JUnit 5 的动态测试特性，它允许我们声明和运行在运行时生成的测试用例。与在编译时定义固定数量的测试用例的静态测试相反，动态测试允许我们在运行时动态定义测试用例。

动态测试可以由标注了`@TestFactory.`的工厂方法生成。让我们来看看代码:

```java
@TestFactory
Stream<DynamicTest> translateDynamicTestsFromStream() {
    return in.stream()
      .map(word ->
          DynamicTest.dynamicTest("Test translate " + word, () -> {
            int id = in.indexOf(word);
            assertEquals(out.get(id), translate(word));
          })
    );
}
```

这个例子非常简单易懂。我们想用两个`ArrayList`来翻译单词，分别命名为`in`和`out`。工厂方法必须返回一个`Stream`、`Collection`、`Iterable`或`Iterator`。在我们的例子中，我们选择了 Java 8 `Stream.`

请注意`@TestFactory`方法不能是私有的或静态的。测试的数量是动态的，它取决于`ArrayList` 的大小。

## 9。结论

在本文中，我们简要介绍了 JUnit 5 带来的变化。

我们探讨了 JUnit 5 架构的重大变化，涉及平台启动器、IDE、其他单元测试框架、与构建工具的集成等。此外，JUnit 5 与 Java 8 的集成程度更高，尤其是与 Lambdas 和 Stream 概念的集成。

本文中使用的例子可以在 [GitHub 项目](https://web.archive.org/web/20220812054122/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics)中找到。