# Junit 5 中的@RepeatedTest 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-repeated-test>

## 1。概述

在这篇简短的文章中，我们将看看 JUnit 5 中引入的`**@RepeatedTest**`注释。它为我们提供了一个强大的方法来编写任何我们想要重复几次的测试。

如果你想了解更多关于 JUnit 5 的知识，请查看我们的其他文章[解释 JUnit 5 的基础知识](/web/20221128115712/https://www.baeldung.com/junit-5-preview)和[指南](/web/20221128115712/https://www.baeldung.com/junit-5)。

## 2。Maven 依赖性和设置

首先要注意的是，JUnit 5 需要 Java 8 才能运行。说到这里，我们来看看 Maven 的依赖性:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

这是我们编写测试时需要添加的主要 JUnit 5 依赖项。点击查看最新版本的神器[。](https://web.archive.org/web/20221128115712/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-engine%22)

## 3。一个简单的`@RepeatedTest`例子

创建重复测试很简单——只需在测试方法上添加`@RepeatedTest`注释:

```java
@RepeatedTest(3)
void repeatedTest(TestInfo testInfo) {
    System.out.println("Executing repeated test");

    assertEquals(2, Math.addExact(1, 1), "1 + 1 should equal 2");
}
```

注意，代替标准的`@Test`注释，我们在单元测试中使用了`@RepeatedTest`。**上面的测试将被执行三次**，就好像同一个测试被编写了三次一样。

测试报告(IDE 的 JUnit 选项卡中的报告文件或结果)将显示所有执行:

```java
repetition 1 of 3(repeatedTest(TestInfo))
repetition 2 of 3(repeatedTest(TestInfo))
repetition 3 of 3(repeatedTest(TestInfo))
```

## 4。`@RepeatedTest` 的生命周期支持

**`@RepeatedTest`的每次执行将表现得像一个常规的`@Test`** 一样，拥有完整的 JUnit 测试生命周期支持。这意味着，在每次执行期间，将调用`@BeforeEach`和`@AfterEach`方法。要演示这一点，只需在测试类中添加适当的方法:

```java
@BeforeEach
void beforeEachTest() {
    System.out.println("Before Each Test");
}

@AfterEach
void afterEachTest() {
    System.out.println("After Each Test");
    System.out.println("=====================");
}
```

如果我们运行之前的测试，结果将显示在控制台上:

```java
Before Each Test
Executing repeated test
After Each Test
=====================
Before Each Test
Executing repeated test
After Each Test
=====================
Before Each Test
Executing repeated test
After Each Test
=====================
```

正如我们所见， **`@BeforeEach`和`@AfterEach`方法是在每次执行**时被调用的。

## 5。配置测试名称

在第一个例子中，我们观察到测试报告的输出不包含任何标识符。这可以使用`name`属性进一步配置:

```java
@RepeatedTest(value = 3, name = RepeatedTest.LONG_DISPLAY_NAME)
void repeatedTestWithLongName() {
    System.out.println("Executing repeated test with long name");

    assertEquals(2, Math.addExact(1, 1), "1 + 1 should equal 2");
}
```

现在，输出将包含方法名以及重复索引:

```java
repeatedTestWithLongName() :: repetition 1 of 3(repeatedTestWithLongName())
repeatedTestWithLongName() :: repetition 2 of 3(repeatedTestWithLongName())
repeatedTestWithLongName() :: repetition 3 of 3(repeatedTestWithLongName())
```

另一个选择是使用`RepeatedTest.SHORT_DISPLAY_NAME`，这将产生测试的简称:

```java
repetition 1 of 3(repeatedTestWithShortName())
repetition 2 of 3(repeatedTestWithShortName())
repetition 3 of 3(repeatedTestWithShortName())
```

然而，如果我们需要使用我们定制的名称，这是非常可能的:

```java
@RepeatedTest(value = 3, name = "Custom name {currentRepetition}/{totalRepetitions}")
void repeatedTestWithCustomDisplayName(TestInfo testInfo) {
    assertEquals(2, Math.addExact(1, 1), "1 + 1 should equal 2");
}
```

`{currentRepetition}`和`{totalRepetitions}`是当前重复和总重复次数的占位符。这些值由 JUnit 在运行时自动提供，不需要额外的配置。输出与我们预期的差不多:

```java
Custom name 1/3(repeatedTestWithCustomDisplayName())
Custom name 2/3(repeatedTestWithCustomDisplayName())
Custom name 3/3(repeatedTestWithCustomDisplayName())
```

## 6。`RepetitionInfo`访问

除了`name`属性，JUnit 还提供了对测试代码中重复元数据的访问。这是通过向我们的测试方法添加一个`RepetitionInfo`参数来实现的:

```java
@RepeatedTest(3)
void repeatedTestWithRepetitionInfo(RepetitionInfo repetitionInfo) {
    System.out.println("Repetition #" + repetitionInfo.getCurrentRepetition());

    assertEquals(3, repetitionInfo.getTotalRepetitions());
}
```

输出将包含每次执行的当前重复索引:

```java
Repetition #1
Repetition #2
Repetition #3
```

`RepetitionInfo`由`RepetitionInfoParameterResolver`提供，仅在`@RepeatedTest.`的上下文中可用

## 7。结论

在这个快速教程中，我们探索了 JUnit 提供的`@RepeatedTest`注释，并学习了配置它的不同方法。

不要忘记在 GitHub 上查看这篇文章的完整源代码。