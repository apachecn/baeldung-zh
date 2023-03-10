# TestNG 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/testng>

## 1。概述

在本文中，我们将介绍 TestNG 测试框架。

我们将关注:框架设置、编写简单的测试用例及配置、测试执行、测试报告生成以及并发测试执行。

## 2。设置

让我们从在我们的`pom.xml`文件中添加 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.1.0</version>
    <scope>test</scope>
</dependency>
```

最新版本可以在 [Maven 资源库](https://web.archive.org/web/20220524033319/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.testng%22%20AND%20a%3A%22testng%22)中找到。

当使用 Eclipse 时，可以从 [Eclipse Marketplace](https://web.archive.org/web/20220524033319/https://marketplace.eclipse.org/) 下载并安装 TestNG 插件。

## 3。编写测试用例

要使用 TestNG 编写一个测试，我们只需要用`org.testng.annotations.Test` 注释对测试方法进行注释:

```java
@Test
public void givenNumber_whenEven_thenTrue() {
    assertTrue(number % 2 == 0);
}
```

## 4。测试配置

在编写测试用例时，我们经常需要在测试执行前执行一些配置或初始化指令，以及在测试完成后进行一些清理。TestNG 在方法、类、组和套件级别提供了许多初始化和清理功能:

```java
@BeforeClass
public void setup() {
    number = 12;
}

@AfterClass
public void tearDown() {
    number = 0;
}
```

用`@BeforeClass` 标注的`setup()` 方法将在测试类的任何方法执行之前被调用，而`tearDown()` 将在测试类的所有方法执行之后被调用。

类似地，我们可以在方法、组、测试和套件级别对任何配置使用`@BeforeMethod, @AfterMethod, @Before/AfterGroup, @Before/AfterTest` 和`@Before/AfterSuite` 注释。

## 5。测试执行

我们可以用 Maven 的“test”命令运行测试用例，它将执行所有标注了`@Test` 的测试用例，并将它们放入默认的测试套件中。我们还可以使用`[maven-surefire-plugin](https://web.archive.org/web/20220524033319/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-surefire-plugin%22):`从 TestNG 测试套件 XML 文件中运行测试用例

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <suiteXmlFiles>
            <suiteXmlFile>
               src\test\resources\test_suite.xml
            </suiteXmlFile>
        </suiteXmlFiles>
    </configuration>
</plugin>
```

注意，如果我们有多个 XML 文件，覆盖了所有的测试用例，我们可以将它们全部添加到`suiteXmlFiles` 标签中:

```java
<suiteXmlFiles>
    <suiteXmlFile>
      src/test/resources/parametrized_test.xml
    </suiteXmlFile>
    <suiteXmlFile>
      src/test/resources/registration_test.xml
    </suiteXmlFile>
</suiteXmlFiles>
```

为了独立运行测试，我们需要在类路径中有 TestNG 库，以及编译好的测试类和 XML 配置文件:

```java
java org.testng.TestNG test_suite.xml
```

## 6。分组测试

测试可以分组运行，例如 50 个测试用例中的 15 个可以分组在一起执行，其他的保持原样。

在 TestNG 中，套件中的分组测试使用 XML 文件完成:

```java
<suite name="suite">
    <test name="test suite">
        <classes>
            <class name="com.baeldung.RegistrationTest" />
            <class name="com.baeldung.SignInTest" />
        </classes>
    </test>
</suite>
```

注意，两个测试类`RegistrationTest, SignInTest` 现在属于同一个 suite，一旦 suite 被执行，这个类中的测试用例将被执行。

除了测试套件，我们还可以在 TestNG 中创建测试组，而不是将测试类方法分组在一起。为此，在`@Test` 注释中添加`groups` 参数:

```java
@Test(groups = "regression")
public void givenNegativeNumber_sumLessthanZero_thenCorrect() {
    int sum = numbers.stream().reduce(0, Integer::sum);

    assertTrue(sum < 0);
}
```

让我们使用 XML 来执行这些组:

```java
<test name="test groups">
    <groups>
        <run>
            <include name="regression" />
        </run>
    </groups>
    <classes>
        <class
          name="com.baeldung.SummationServiceTest" />
    </classes>
</test>
```

这将执行在`SummationServiceTest` 类中用组`regression,` 标记的测试方法。

## 7。参数化测试

参数化单元测试用于在几种情况下测试相同的代码。在参数化单元测试的帮助下，我们可以建立一个从一些数据源获取数据的测试方法。主要思想是使单元测试方法可重用，并使用一组不同的输入进行测试。

在 TestNG 中，我们可以使用@ `Parameter` 或`@DataProvider` 注释来参数化测试。在使用 XML 文件时，用@ `Parameter:`标注测试方法

```java
@Test
@Parameters({"value", "isEven"})
public void
  givenNumberFromXML_ifEvenCheckOK_thenCorrect(int value, boolean isEven) {

    assertEquals(isEven, value % 2 == 0);
}
```

And provide the data using XML file:

```java
<suite name="My test suite">
    <test name="numbersXML">
        <parameter name="value" value="1"/>
        <parameter name="isEven" value="false"/>
        <classes>
            <class name="baeldung.com.ParametrizedTests"/>
        </classes>
    </test>
</suite>
```

使用 XML 文件中的数据是有用的，但是我们经常需要更复杂的数据。`@DataProvider`注释用于处理这些场景，可用于映射测试方法的复杂参数类型。`@DataProvider`对于原始数据类型:

```java
@DataProvider(name = "numbers")
public static Object[][] evenNumbers() {
    return new Object[][]{{1, false}, {2, true}, {4, true}};
}

@Test(dataProvider = "numbers")
public void 
  givenNumberFromDataProvider_ifEvenCheckOK_thenCorrect(Integer number, boolean expected) {    
    assertEquals(expected, number % 2 == 0);
}
```

`@DataProvider`对于物体:

```java
@Test(dataProvider = "numbersObject")
public void 
  givenNumberObjectFromDataProvider_ifEvenCheckOK_thenCorrect(EvenNumber number) {  
    assertEquals(number.isEven(), number.getValue() % 2 == 0);
}

@DataProvider(name = "numbersObject")
public Object[][] parameterProvider() {
    return new Object[][]{{new EvenNumber(1, false)},
      {new EvenNumber(2, true)}, {new EvenNumber(4, true)}};
}
```

使用它，任何需要测试的对象都可以被创建并在测试中使用。这对于集成测试用例非常有用。

## 8。忽略测试用例

我们有时不希望在开发过程中暂时执行某个测试用例。这可以通过在@ `Test` 注释中添加`enabled` `=false,` 来实现:

```java
@Test(enabled=false)
public void givenNumbers_sumEquals_thenCorrect() { 
    int sum = numbers.stream.reduce(0, Integer::sum);
    assertEquals(6, sum);
}
```

## 9。相关测试

让我们考虑一个场景，如果最初的测试用例失败，所有后续的测试用例都应该被执行，而不是被标记为跳过。TestNG 用`@Test` 注释的`dependsOnMethods` 参数提供了这个特性:

```java
@Test
public void givenEmail_ifValid_thenTrue() {
    boolean valid = email.contains("@");

    assertEquals(valid, true);
}

@Test(dependsOnMethods = {"givenEmail_ifValid_thenTrue"})
public void givenValidEmail_whenLoggedIn_thenTrue() {
    LOGGER.info("Email {} valid >> logging in", email);
}
```

注意，登录测试用例依赖于电子邮件验证测试用例。因此，如果电子邮件验证失败，登录测试将被跳过。

## 10。并发测试执行

TestNG 允许测试以并行或多线程模式运行，从而提供了一种测试这些多线程代码的方法。

您可以配置方法、类和套件在它们自己的线程中运行，从而减少总的执行时间。

### 10.1。并行的类和方法

要并行运行测试类，在 XML 配置文件的`suite`标签中提到`parallel` 属性，值为`classes:`

```java
<suite name="suite" parallel="classes" thread-count="2">
    <test name="test suite">
        <classes>
	    <class name="baeldung.com.RegistrationTest" />
            <class name="baeldung.com.SignInTest" />
        </classes>
    </test>
</suite>
```

注意，如果我们在 XML 文件中有多个`test` 标签，这些测试也可以并行运行，通过提及`parallel =” tests”.` 也可以并行执行单独的方法，提及`parallel =” methods”.`

### 10.2。测试方法的多线程执行

假设我们需要测试一个代码在多线程中运行时的行为。TestNG 允许在多线程中运行一个测试方法:

```java
public class MultiThreadedTests {

    @Test(threadPoolSize = 5, invocationCount = 10, timeOut = 1000)
    public void givenMethod_whenRunInThreads_thenCorrect() {
        int count = Thread.activeCount();

        assertTrue(count > 1);
    }
}
```

`threadPoolSize` 表示该方法将在前面提到的`n` 个线程中运行。`invocationCount` 和`timeOut`表示测试将执行多次，如果花费更多时间，测试将失败。

## 11.功能测试

TestNG 附带的特性也可以用于功能测试。与 Selenium 结合使用，它既可以用来测试 web 应用程序的功能，也可以用来测试带有 [HttpClient](https://web.archive.org/web/20220524033319/https://hc.apache.org/httpcomponents-client-ga/index.html) 的 web 服务。

更多关于 Selenium 和 TestNG 功能测试的细节可以在[这里](/web/20220524033319/https://www.baeldung.com/java-selenium-with-junit-and-testng)找到。在这篇[文章](/web/20220524033319/https://www.baeldung.com/integration-testing-a-rest-api)中还有一些关于集成测试的东西。

## 12.结论

在本文中，我们快速浏览了如何设置 TestNG 和执行一个简单的测试用例，生成报告，并发执行测试用例，以及一些关于函数式编程的内容。关于更多的特性，比如依赖测试、忽略测试用例、测试组和套件，你可以参考我们的 JUnit vs TestNG 文章[这里](/web/20220524033319/https://www.baeldung.com/junit-vs-testng)。

所有代码片段的实现都可以在 Github 上找到[。](https://web.archive.org/web/20220524033319/https://github.com/eugenp/tutorials/tree/master/testing-modules/testng)