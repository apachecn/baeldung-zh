# 确定 JUnit 测试的执行时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-test-execution-time>

## 1.概观

我们的构建经常为我们的项目运行大量的自动化测试用例。这些包括单元测试和集成测试。如果测试套件的执行花费了很长时间，我们可能希望优化我们的测试代码或者跟踪花费太长时间的测试。

在本教程中，我们将学习一些确定测试用例及测试套件执行时间的方法。

## 2.JUnit 示例

为了演示报告执行时间，让我们使用来自测试金字塔不同层的一些示例测试用例。我们将用`Thread.sleep()`来模拟测试用例的持续时间。

我们将在 JUnit 5 中实现我们的例子。然而，等效的工具和技术也适用于用 JUnit 4 编写的测试用例。

首先，这里有一个简单的单元测试:

```java
@Test
void someUnitTest() {

    assertTrue(doSomething());
}
```

其次，让我们进行一个需要更多时间来执行的集成测试:

```java
@Test
void someIntegrationTest() throws Exception {

    Thread.sleep(5000);
    assertTrue(doSomething());
}
```

最后，我们可以模拟一个缓慢的端到端用户场景:

```java
@Test
void someEndToEndTest() throws Exception {

    Thread.sleep(10000);
    assertTrue(doSomething());
}
```

在本文的其余部分，**我们将执行这些测试用例，并确定它们的执行时间**。

## 3.IDE JUnit 运行程序

找到 JUnit 测试的**执行时间的最快方法是使用我们的 IDE** 。因为大多数 ide 都带有嵌入式 JUnit runner，所以它们执行并报告测试结果。

两个最流行的 ide IntelliJ 和 Eclipse 都嵌入了 JUnit 运行程序。

### 2.1.IntelliJ JUnit Runner

IntelliJ 允许我们在[运行/调试配置](/web/20220626200250/https://www.baeldung.com/intellij-basics)的帮助下执行 JUnit 测试用例。一旦我们执行了测试，运行程序就会显示测试状态和执行时间:

[![](img/5318b10516cf9499e7eac51eb69d9e82.png)](/web/20220626200250/https://www.baeldung.com/wp-content/uploads/2019/11/Intellij-Junit-Test-Result.jpg)

由于我们执行了所有三个示例测试用例，我们可以看到总执行时间以及每个测试用例花费的时间。

我们可能还需要保存此类报告以供将来参考。 **IntelliJ 允许我们以 HTML 或 XML 格式导出此报告**。在上面的屏幕截图中，工具栏上突出显示了导出报告功能。

### 2.2.Eclipse JUnit Runner

Eclipse 还提供了一个嵌入式 JUnit runner 。我们可以在 test results 窗口中执行并找出单个测试用例或整个测试套件的执行时间:

[![](img/154569230f1327887e7afe46dd87fead.png)](/web/20220626200250/https://www.baeldung.com/wp-content/uploads/2019/11/Eclipse-JUnit-runner-results-tab.jpg)

但是，与 IntelliJ 测试运行程序相比，我们不能从 Eclipse 中导出报告。

## 3.Maven Surefire 插件

Maven Surefire 插件用于在构建生命周期的测试阶段执行单元测试。surefire 插件是默认 Maven 配置的一部分。但是，如果需要特定版本或[附加配置](https://web.archive.org/web/20220626200250/https://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html)，我们可以在`pom.xml`中声明:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M3</version>
    <configuration>
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-surefire-provider</artifactId>
            <version>1.3.2</version>
        </dependency>
    </dependencies>
</plugin>
```

用 Maven 测试时，有三种方法可以找到 JUnit 测试的执行时间。我们将在接下来的小节中研究每一个问题。

### 3.1.Maven 构建日志

Surefire 在构建日志中显示每个测试用例的执行状态和时间:

```java
[INFO] Running com.baeldung.execution.time.SampleExecutionTimeUnitTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 15.003 s 
- in com.baeldung.execution.time.SampleExecutionTimeUnitTest
```

这里，它显示了测试类中所有三个测试用例的组合执行时间。

### 3.2.Surefire 测试报告

surefire 插件还在。txt 和。xml 格式。这些一般都是**存放在项目**的目标目录下。Surefire 遵循两种文本报告的标准格式:

```java
----------------------------------------------
Test set: com.baeldung.execution.time.SampleExecutionTimeUnitTest
----------------------------------------------
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 15.003 s 
- in com.baeldung.execution.time.SampleExecutionTimeUnitTest
```

和 XML 报告:

```java
<?xml version="1.0" encoding="UTF-8"?>
<testsuite
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:noNamespaceSchemaLocation=
          "https://maven.apache.org/surefire/maven-surefire-plugin/xsd/surefire-test-report.xsd"
	name="com.baeldung.execution.time.SampleExecutionTimeUnitTest"
	time="15.003" tests="3" errors="0" skipped="0" failures="0">
	<testcase name="someEndToEndTest"
		classname="com.baeldung.execution.time.SampleExecutionTimeUnitTest"
		time="9.996" />
	<testcase name="someIntegrationTest"
		classname="com.baeldung.execution.time.SampleExecutionTimeUnitTest"
		time="5.003" />
	<testcase name="someUnitTest"
		classname="com.baeldung.execution.time.SampleExecutionTimeUnitTest"
		time="0.002" />
</testsuite>
```

虽然文本格式更适合可读性，但是 XML 格式是机器可读的，并且可以被导入到 HTML 和其他工具中可视化。

### 3.3.Surefire HTML 报告

我们可以使用`[maven-surefire-report-plugin](https://web.archive.org/web/20220626200250/https://search.maven.org/search?q=g:org.apache.maven.plugins%20AND%20a:maven-surefire-report-plugin&core=gav)`在浏览器中查看 HTML 格式的测试报告:

```java
<reporting>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-report-plugin</artifactId>
            <version>3.0.0-M3</version>
        </plugin>
    </plugins>
</reporting>
```

我们现在可以执行`mvn `命令来生成报告:

1.  `mvn surefire-report:report`–执行测试并生成 HTML 报告
2.  `mvn site`–将 CSS 样式添加到上一步生成的 HTML 中

[![](img/f0bf2e7bc11bed8d69017f3dd361d590.png)](/web/20220626200250/https://www.baeldung.com/wp-content/uploads/2019/11/Surefire-HTML-Report.jpg)

这个报告显示了一个类或者一个包中所有测试用例的执行时间，以及每个测试用例所花费的时间。

## 4.詹金斯测试结果

如果我们在 Jenkins 中运行 CI，我们可以导入 surefire 编写的 XML 文件。这允许 Jenkins 在测试失败时将构建标记为失败，并向我们展示测试趋势和结果。

当我们在 Jenkins 中查看测试结果时，我们还会看到执行时间。

对于我们的例子，在安装完 Jenkins 之后，我们将使用 Maven 和 surefire XML 报告文件来配置一个作业,以发布测试结果:

[![](img/30fbf3ab702e9261c022422e076bcd1d.png)](/web/20220626200250/https://www.baeldung.com/wp-content/uploads/2019/11/Jenkins-Post-Build-Action.jpg)

我们在工作中添加了一个后期构建操作来发布测试结果。Jenkins 现在将从给定的路径导入 XML 文件，并将该报告添加到构建执行摘要中:

[![](img/b6d714abf9fad4722d4d9cefa3b8f241.png)](/web/20220626200250/https://www.baeldung.com/wp-content/uploads/2019/11/Jenkins-Test-Results-1.jpg)

这也可以通过使用 Jenkins [管道构建](/web/20220626200250/https://www.baeldung.com/jenkins-pipelines)来实现。

## 5.结论

在本文中，我们讨论了确定 JUnit 测试执行时间的各种方法。最直接的方法是使用我们 IDE 的 JUnit runner。

然后，我们使用`maven-surefire-plugin`将测试报告归档为文本、XML 和 HTML 格式。

最后，我们向 CI 服务器提供了测试报告输出，这可以帮助我们分析不同的构建是如何执行的。

和往常一样，GitHub 上的[提供了本文的示例代码。](https://web.archive.org/web/20220626200250/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)