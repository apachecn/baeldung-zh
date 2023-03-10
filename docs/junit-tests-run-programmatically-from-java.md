# 从 Java 应用程序以编程方式运行 JUnit 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-tests-run-programmatically-from-java>

## 1。概述

在本教程中，我们将向**展示如何直接从 Java 代码**中运行 JUnit 测试——在一些场景中，这个选项会派上用场。

如果你是 JUnit 新手，或者如果你想升级到 JUnit 5，你可以查看一些我们关于这个主题的教程。

## 2。Maven 依赖关系

运行 JUnit 4 和 JUnit 5 测试，我们需要几个基本的依赖项:

```java
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.8.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-launcher</artifactId>
        <version>1.2.0</version>
    </dependency>
</dependencies>

// for JUnit 4
<dependency> 
    <groupId>junit</groupId> 
    <artifactId>junit</artifactId> 
    <version>4.12</version> 
    <scope>test</scope> 
</dependency>
```

最新版本的 [JUnit 4、](https://web.archive.org/web/20221206071026/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22junit%22%20AND%20a%3A%22junit%22) [JUnit 5](https://web.archive.org/web/20221206071026/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-engine%22) 和 [JUnit 平台启动器](https://web.archive.org/web/20221206071026/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.junit.platform%22%20AND%20a%3A%22junit-platform-launcher%22)可以在 Maven Central 上找到。

## 3。运行 JUnit 4 测试

### 3.1。测试场景

对于 JUnit 4 和 JUnit 5，我们将设置几个“占位符”测试类，这足以演示我们的示例:

```java
public class FirstUnitTest {

    @Test
    public void whenThis_thenThat() {
        assertTrue(true);
    }

    @Test
    public void whenSomething_thenSomething() {
        assertTrue(true);
    }

    @Test
    public void whenSomethingElse_thenSomethingElse() {
        assertTrue(true);
    }
}
```

```java
public class SecondUnitTest {

    @Test
    public void whenSomething_thenSomething() {
        assertTrue(true);
    }

    @Test
    public void whensomethingElse_thenSomethingElse() {
        assertTrue(true);
    }
}
```

当使用 JUnit 4 时，我们创建测试类，向每个测试方法添加`@Test`注释。

我们还可以添加其他有用的注释，比如`@Before`或`@After`，但这不在本教程的讨论范围之内。

### 3.2。运行单个测试类

为了从 Java 代码中运行 JUnit 测试，我们可以使用`JUnitCore`类(外加一个`TextListener`类，用于在`System.out`中显示输出):

```java
JUnitCore junit = new JUnitCore();
junit.addListener(new TextListener(System.out));
junit.run(FirstUnitTest.class);
```

在控制台上，我们将看到一条非常简单的消息，表明测试成功:

```java
Running one test class:
..
Time: 0.019
OK (2 tests)
```

### 3.3。运行多个测试类

如果我们希望**用 JUnit 4 指定多个测试类**，我们可以使用与单个类相同的代码，并简单地添加额外的类:

```java
JUnitCore junit = new JUnitCore();
junit.addListener(new TextListener(System.out));

Result result = junit.run(
  FirstUnitTest.class, 
  SecondUnitTest.class);

resultReport(result);
```

注意，结果存储在 JUnit 的 `Result` 类的实例中，我们使用一个简单的实用方法打印出来

```java
public static void resultReport(Result result) {
    System.out.println("Finished. Result: Failures: " +
      result.getFailureCount() + ". Ignored: " +
      result.getIgnoreCount() + ". Tests run: " +
      result.getRunCount() + ". Time: " +
      result.getRunTime() + "ms.");
} 
```

### 3.4。运行测试套件

如果我们需要分组一些测试类来运行它们，我们可以创建一个`TestSuite`。这只是一个空类，我们使用 JUnit 注释指定所有的类:

```java
@RunWith(Suite.class)
@Suite.SuiteClasses({
  FirstUnitTest.class,
  SecondUnitTest.class
})
public class MyTestSuite {
}
```

为了运行这些测试，我们将再次使用与之前相同的代码:

```java
JUnitCore junit = new JUnitCore();
junit.addListener(new TextListener(System.out));
Result result = junit.run(MyTestSuite.class);
resultReport(result);
```

### 3.5。运行重复测试

JUnit 的一个有趣的特性是，我们可以通过创建`RepeatedTest` 的实例来**重复测试。当我们测试随机值或进行性能检查时，这非常有用。**

在下一个例子中，我们将从`MergeListsTest`开始运行测试五次:

```java
Test test = new JUnit4TestAdapter(FirstUnitTest.class);
RepeatedTest repeatedTest = new RepeatedTest(test, 5);

JUnitCore junit = new JUnitCore();
junit.addListener(new TextListener(System.out));

junit.run(repeatedTest);
```

这里，我们使用`JUnit4TestAdapter`作为测试类的包装器。

我们甚至可以通过重复测试，以编程方式创建套件:

```java
TestSuite mySuite = new ActiveTestSuite();

JUnitCore junit = new JUnitCore();
junit.addListener(new TextListener(System.out));

mySuite.addTest(new RepeatedTest(new JUnit4TestAdapter(FirstUnitTest.class), 5));
mySuite.addTest(new RepeatedTest(new JUnit4TestAdapter(SecondUnitTest.class), 3));

junit.run(mySuite);
```

## 4。运行 JUnit 5 测试

### 4.1。测试场景

对于 JUnit 5，我们将使用与上一个演示相同的样本测试类——`FirstUnitTest`和`SecondUnitTest`,由于 JUnit 框架的不同版本，我们会有一些小的不同，比如用于`@Test`和断言方法的包。

### 4.2。运行单个测试类

为了从 Java 代码运行 JUnit 5 测试，我们将设置一个`LauncherDiscoveryRequest`的实例。它使用一个构建器类，我们必须在其中设置包选择器和测试类名过滤器，以获得我们想要运行的所有测试类。

然后，这个请求与一个启动器相关联，在执行测试之前，我们还将设置一个测试计划和一个执行监听器。

这两者都将提供关于要执行的测试和结果的信息:

```java
public class RunJUnit5TestsFromJava {
    SummaryGeneratingListener listener = new SummaryGeneratingListener();

    public void runOne() {
        LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
          .selectors(selectClass(FirstUnitTest.class))
          .build();
        Launcher launcher = LauncherFactory.create();
        TestPlan testPlan = launcher.discover(request);
        launcher.registerTestExecutionListeners(listener);
        launcher.execute(request);
    }
    // main method...
}
```

### 4.3。运行多个测试类

我们可以为运行多个测试类的请求设置选择器和过滤器。

让我们看看如何设置包选择器和测试类名过滤器，以获得我们想要运行的所有测试类:

```java
public void runAll() {
    LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
      .selectors(selectPackage("com.baeldung.junit5.runfromjava"))
      .filters(includeClassNamePatterns(".*Test"))
      .build();
    Launcher launcher = LauncherFactory.create();
    TestPlan testPlan = launcher.discover(request);
    launcher.registerTestExecutionListeners(listener);
    launcher.execute(request);
} 
```

### 4.4。测试输出

在`main()`方法中，我们调用我们的类，我们也使用监听器来获得结果细节。这次结果被存储为一个`TestExecutionSummary`。

提取其信息的最简单方法是打印到控制台输出流:

```java
public static void main(String[] args) {
    RunJUnit5TestsFromJava runner = new RunJUnit5TestsFromJava();
    runner.runAll();

    TestExecutionSummary summary = runner.listener.getSummary();
    summary.printTo(new PrintWriter(System.out));
}
```

这将为我们提供测试运行的详细信息:

```java
Test run finished after 177 ms
[         7 containers found      ]
[         0 containers skipped    ]
[         7 containers started    ]
[         0 containers aborted    ]
[         7 containers successful ]
[         0 containers failed     ]
[        10 tests found           ]
[         0 tests skipped         ]
[        10 tests started         ]
[         0 tests aborted         ]
[        10 tests successful      ]
[         0 tests failed          ]
```

## 5。结论

在本文中，我们展示了如何从 Java 代码中以编程方式运行 JUnit 测试，涵盖了 JUnit 4 以及这个测试框架的最新版本 JUnit 5。

和往常一样，这里显示的例子的实现可以在 GitHub 上获得，既适用于 [JUnit 5 例子](https://web.archive.org/web/20221206071026/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)，也适用于 [JUnit 4](https://web.archive.org/web/20221206071026/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4) 。