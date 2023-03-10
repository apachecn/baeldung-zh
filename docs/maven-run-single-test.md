# 用 Maven 运行单个测试或方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-run-single-test>

## 1.概观

通常，我们在使用 [Maven surefire 插件](/web/20221208143917/https://www.baeldung.com/maven-surefire-plugin)的 [Maven](/web/20221208143917/https://www.baeldung.com/maven-guide) 构建期间执行测试。

本教程将探索如何使用这个插件来运行一个单独的测试类或测试方法。

## 2.问题简介

Maven surefire 插件很容易使用。它只有一个目标:`test`。

因此，在默认配置下，**我们可以通过命令`mvn test`执行项目中的所有测试。**

有时，我们可能想要执行一个单一的测试类或者甚至一个单一的测试方法。

在本教程中，我们将把 [JUnit 5](/web/20221208143917/https://www.baeldung.com/junit-5) 作为测试提供者的例子来说明如何实现它。

## 3.示例项目

为了以更简单的方式显示测试结果，让我们创建几个简单的测试类:

```java
class TheFirstUnitTest {

    // declaring logger ... 

    @Test
    void whenTestCase_thenPass() {
        logger.info("Running a dummyTest");
    }
}

class TheSecondUnitTest {

    // declaring logger ... 

    @Test
    void whenTestCase1_thenPrintTest1_1() {
        logger.info("Running When Case1: test1_1");
    }

    @Test
    void whenTestCase1_thenPrintTest1_2() {
        logger.info("Running When Case1: test1_2");
    }

    @Test
    void whenTestCase1_thenPrintTest1_3() {
        logger.info("Running When Case1: test1_3");
    }

    @Test
    void whenTestCase2_thenPrintTest2_1() {
        logger.info("Running When Case2: test2_1");
    }
} 
```

在`TheFirstUnitTest`类中，我们只有一个测试方法。然而，`TheSecondUnitTest`包含了四种测试方法。我们所有的方法名称都遵循“`when…then…`”模式。

为了简单起见，我们让每个测试方法输出一行，表明该方法正在被调用。

现在，如果我们运行`mvn test`，所有的测试都将被执行:

```java
$ mvn test
...
[INFO] Scanning for projects...
...
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.baeldung.runasingletest.TheSecondUnitTest
16:58:16.444 [main] INFO ...TheSecondUnitTest - Running When Case2: test2_1
16:58:16.448 [main] INFO ...TheSecondUnitTest - Running When Case1: test1_1
16:58:16.449 [main] INFO ...TheSecondUnitTest - Running When Case1: test1_2
16:58:16.450 [main] INFO ...TheSecondUnitTest - Running When Case1: test1_3
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.065 s - in com.baeldung.runasingletest.TheSecondUnitTest
[INFO] Running com.baeldung.runasingletest.TheFirstUnitTest
16:58:16.453 [main] INFO ...TheFirstUnitTest - Running a dummyTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0 s - in com.baeldung.runasingletest.TheFirstUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
 ... 
```

接下来，让我们告诉 Maven 只执行指定的测试。

## 4.执行单个测试类

Maven surefire 插件提供了一个`test`参数，我们可以用它来指定我们想要执行的测试类或方法。

**如果我们想要执行一个单独的测试类，我们可以执行命令`mvn test -Dtest=”TestClassName”`。**

例如，我们可以将`-Dtest=”TheFirstUnitTest”`传递给`mvn`命令，只执行`TheFirstUnitTest`类:

```java
$ mvn test -Dtest="TheFirstUnitTest"
...
[INFO] Scanning for projects...
...
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.baeldung.runasingletest.TheFirstUnitTest
17:10:35.351 [main] INFO com.baeldung.runasingletest.TheFirstUnitTest - Running a dummyTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.053 s - in com.baeldung.runasingletest.TheFirstUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
 ...
```

如输出所示，只执行我们传递给`test`参数的测试类。

## 5.执行单一测试方法

另外，**我们可以通过将`-Dtest=”TestClassName#TestMethodName” `传递给`mvn`命令来请求 Maven surefire 插件执行一个测试方法。**

现在让我们执行`TheSecondUnitTest`类中的`whenTestCase2_thenPrintTest2_1()`方法:

```java
$ mvn test -Dtest="TheSecondUnitTest#whenTestCase2_thenPrintTest2_1"    
...
[INFO] Scanning for projects...
...
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.baeldung.runasingletest.TheSecondUnitTest
17:22:07.063 [main] INFO ...TheSecondUnitTest - Running When Case2: test2_1
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.057 s - in com.baeldung.runasingletest.TheSecondUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
...
```

正如我们所看到的，这一次，我们只执行了指定的测试方法。

## 6.关于`test`参数的更多信息

到目前为止，我们已经展示了如何通过相应地提供`test`参数值来执行单个测试类或测试方法。

实际上，Maven surefire 插件允许我们以不同的格式设置`test`参数的值，以灵活地执行测试。

接下来，我们将展示一些常用的格式:

*   按名称执行多个测试类:`-Dtest=”TestClassName1, TestClassName2, TestClassName3…”`
*   按名称模式执行多个测试类:`-Dtest=”*ServiceUnitTest”`或`-Dtest=”The*UnitTest, Controller*Test”`
*   按名称指定多种测试方法:`-Dtest=”ClassName#method1+method2″`
*   通过名称模式指定多个方法名:`-Dtest=”ClassName#whenSomethingHappens_*”`

最后，我们再看一个例子。

假设我们只想执行`TheSecondUnitTest`类中所有的`whenTestCase1…`方法。

因此，按照我们上面讨论的模式，我们希望`-Dtest=”TheSecondUnitTest#whenTestCase1*”`将完成这项工作:

```java
$ mvn test -Dtest="TheSecondUnitTest#whenTestCase1*"
...
[INFO] Scanning for projects...
...
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.baeldung.runasingletest.TheSecondUnitTest
17:51:04.973 [main] INFO ...TheSecondUnitTest - Running When Case1: test1_1
17:51:04.979 [main] INFO ...TheSecondUnitTest - Running When Case1: test1_2
17:51:04.980 [main] INFO ...TheSecondUnitTest - Running When Case1: test1_3
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.055 s - in com.baeldung.runasingletest.TheSecondUnitTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
...
```

正如我们所期望的，只有三个匹配指定名称模式的测试方法被执行。

## 7.结论

Maven surefire 插件提供了一个`test`参数，允许我们非常灵活地选择想要执行的测试。

在本文中，我们讨论了一些常用的`test`参数的值格式。

此外，我们已经通过例子说明了如何用 Maven 只运行指定的测试类或测试方法。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-surefire-plugin)