# 快速比较 JUnit 和 TestNG

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-vs-testng>

## 1。概述

JUnit 和 TestNG 无疑是 Java 生态系统中最流行的两个单元测试框架。虽然 JUnit 启发了 TestNG 本身，但它提供了自己独特的特性，并且与 JUnit 不同，它适用于功能性和更高级别的测试。

在本帖中，**我们将讨论和比较这些框架，包括它们的特性和常见用例**。

## 2。测试设置

在编写测试用例时，我们经常需要在测试执行前执行一些配置或初始化指令，以及在测试完成后进行一些清理。让我们在两个框架中评估这些。

**JUnit 在两个层次上提供初始化和清理，在每个方法和类之前和之后。**我们在方法级别有`@BeforeEach`、 `@AfterEach`注释，在类级别有`@BeforeAll` 和 `@AfterAll`:

```java
public class SummationServiceTest {

    private static List<Integer> numbers;

    @BeforeAll
    public static void initialize() {
        numbers = new ArrayList<>();
    }

    @AfterAll
    public static void tearDown() {
        numbers = null;
    }

    @BeforeEach
    public void runBeforeEachTest() {
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
    }

    @AfterEach
    public void runAfterEachTest() {
        numbers.clear();
    }

    @Test
    public void givenNumbers_sumEquals_thenCorrect() {
        int sum = numbers.stream().reduce(0, Integer::sum);
        assertEquals(6, sum);
    }
}
```

注意，这个例子使用的是 JUnit 5。在之前的 JUnit 4 版本中，我们需要使用`@Before` 和`@After` 注释，它们相当于`@BeforeEach`和` @AfterEach. `。同样，`@BeforeAll` 和 `@AfterAll` 是 JUnit 4 的`@BeforeClass` 和`@AfterClass.`的替代品

与 JUnit 类似， **TestNG 也提供方法和类级别的初始化和清理**。虽然`@BeforeClass` 和`@AfterClass` 在类级别保持不变，但是方法级别的注释是@ `BeforeMethod` 和`@AfterMethod:`

```java
@BeforeClass
public void initialize() {
    numbers = new ArrayList<>();
}

@AfterClass
public void tearDown() {
    numbers = null;
}

@BeforeMethod
public void runBeforeEachTest() {
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
}

@AfterMethod
public void runAfterEachTest() {
    numbers.clear();
}
```

**TestNG 还为套件和组级别的配置提供了`@BeforeSuite, @AfterSuite, @BeforeGroup and @AfterGroup` 注释:**

```java
@BeforeGroups("positive_tests")
public void runBeforeEachGroup() {
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
}

@AfterGroups("negative_tests")
public void runAfterEachGroup() {
    numbers.clear(); 
}
```

此外，如果我们需要 TestNG XML 配置文件中的`<test>` 标签中包含的测试用例之前或之后的任何配置，我们可以使用`@BeforeTest` 和@ `AfterTest` :

```java
<test name="test setup">
    <classes>
        <class name="SummationServiceTest">
            <methods>
                <include name="givenNumbers_sumEquals_thenCorrect" />
            </methods>
        </class>
    </classes>
</test>
```

请注意，`@BeforeClass`和`@AfterClass`方法的声明在 JUnit 中必须是静态的。相比之下，TestNG 方法声明没有这些约束。

## 3。忽略测试

**两个框架都支持忽略测试用例**，尽管他们做的很不一样。JUnit 提供了`@Ignore` 注释:

```java
@Ignore
@Test
public void givenNumbers_sumEquals_thenCorrect() {
    int sum = numbers.stream().reduce(0, Integer::sum);
    Assert.assertEquals(6, sum);
}
```

而 TestNG 使用带有布尔值`true`或`false`的参数`@Test` :

```java
@Test(enabled=false)
public void givenNumbers_sumEquals_thenCorrect() {
    int sum = numbers.stream.reduce(0, Integer::sum);
    Assert.assertEquals(6, sum);
}
```

## 4。一起运行测试

在`JUnit`和 TestNG 中，将测试作为一个集合一起运行是可能的，但是它们以不同的方式运行。

**我们可以使用`@Suite,` `@SelectPackages`和`@SelectClasses`注释对测试用例进行分组，并在`JUnit 5`中作为套件运行。**套件是一组测试用例的集合，我们可以将它们组合在一起，作为一个单独的测试来运行。

如果我们想要将不同包的测试用例分组在一个`Suite `中一起运行，我们需要`@SelectPackages`注释:

```java
@Suite
@SelectPackages({ "org.baeldung.java.suite.childpackage1", "org.baeldung.java.suite.childpackage2" })
public class SelectPackagesSuiteUnitTest {

}
```

如果我们想要特定的测试类一起运行，`JUnit 5`通过`@SelectClasses`提供了灵活性:

```java
@Suite
@SelectClasses({Class1UnitTest.class, Class2UnitTest.class})
public class SelectClassesSuiteUnitTest {

}
```

之前使用`JUnit 4`，我们使用`@RunWith`和`@Suite `注释实现了分组和一起运行多个测试:

```java
@RunWith(Suite.class)
@Suite.SuiteClasses({ RegistrationTest.class, SignInTest.class })
public class SuiteTest {

}
```

在 TestNG 中，我们可以使用一个 XML 文件对测试进行分组:

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

这表示`RegistrationTest` 和`SignInTest` 将一起运行。

除了对类进行分组，TestNG 还可以使用@ `Test(groups=”groupName”)`注释对方法进行分组:

```java
@Test(groups = "regression")
public void givenNegativeNumber_sumLessthanZero_thenCorrect() {
    int sum = numbers.stream().reduce(0, Integer::sum);
    Assert.assertTrue(sum < 0);
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

这将执行标记为组`regression`的测试方法。

## 5。测试异常

使用注释测试异常的特性在 JUnit 和 TestNG 中都可用。

让我们首先用抛出异常的方法创建一个类:

```java
public class Calculator {
    public double divide(double a, double b) {
        if (b == 0) {
            throw new DivideByZeroException("Divider cannot be equal to zero!");
        }
        return a/b;
    }
}
```

在`JUnit 5`中，我们可以使用`assertThrows ` API 来测试异常:

```java
@Test
public void whenDividerIsZero_thenDivideByZeroExceptionIsThrown() {
    Calculator calculator = new Calculator();
    assertThrows(DivideByZeroException.class, () -> calculator.divide(10, 0));
}
```

在`JUnit 4, `中，我们可以通过在测试 API 上使用`@Test(expected = DivideByZeroException.class) `来实现这一点。

使用 TestNG，我们也可以实现同样的功能:

```java
@Test(expectedExceptions = ArithmeticException.class) 
public void givenNumber_whenThrowsException_thenCorrect() { 
    int i = 1 / 0;
}
```

这个特性意味着从一段代码中抛出什么异常，这是测试的一部分。

## 6。参数化测试

参数化单元测试有助于在几种情况下测试相同的代码。在参数化单元测试的帮助下，我们可以建立一个从一些数据源获取数据的测试方法。主要思想是使单元测试方法可重用，并使用一组不同的输入进行测试。

**在`JUnit 5`中，我们有测试方法的优势，直接从配置的源中消耗数据参数。**默认情况下，JUnit 5 提供了一些`source`注释，比如:

*   `@ValueSource:`我们可以将它与类型为`Short, Byte, Int, Long, Float, Double, Char,` 和 `String:`的值数组一起使用

```java
@ParameterizedTest
@ValueSource(strings = { "Hello", "World" })
void givenString_TestNullOrNot(String word) {
    assertNotNull(word);
}
```

*   `@EnumSource – `将`Enum `常量作为参数传递给测试方法:

```java
@ParameterizedTest
@EnumSource(value = PizzaDeliveryStrategy.class, names = {"EXPRESS", "NORMAL"})
void givenEnum_TestContainsOrNot(PizzaDeliveryStrategy timeUnit) {
    assertTrue(EnumSet.of(PizzaDeliveryStrategy.EXPRESS, PizzaDeliveryStrategy.NORMAL).contains(timeUnit));
}
```

*   `@MethodSource – p`评估生成流的外部方法:

```java
static Stream<String> wordDataProvider() {
    return Stream.of("foo", "bar");
}

@ParameterizedTest
@MethodSource("wordDataProvider")
void givenMethodSource_TestInputStream(String argument) {
    assertNotNull(argument);
}
```

*   `@CsvSource –` 使用 CSV 值作为参数的来源:

```java
@ParameterizedTest
@CsvSource({ "1, Car", "2, House", "3, Train" })
void givenCSVSource_TestContent(int id, String word) {
	assertNotNull(id);
	assertNotNull(word);
}
```

类似地，如果我们需要从类路径中读取一个 CSV 文件，我们还有其他来源，如`@CsvFileSource `和`@ArgumentSource`来指定一个定制的、可重用的`ArgumentsProvider.`

在`JUnit 4`中，测试类必须用`@RunWith`进行注释，使其成为参数化类，并使用`@Parameter`来表示单元测试的参数值。

**在 TestNG 中，我们可以使用@ `Parameter` 或`@DataProvider` 注释来参数化测试。**在使用 XML 文件时，用@ `Parameter:`标注测试方法

```java
@Test
@Parameters({"value", "isEven"})
public void 
  givenNumberFromXML_ifEvenCheckOK_thenCorrect(int value, boolean isEven) {
    Assert.assertEquals(isEven, value % 2 == 0);
}
```

并在 XML 文件中提供数据:

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

虽然使用 XML 文件中的信息简单而有用，但在某些情况下，您可能需要提供更复杂的数据。

为此，我们可以使用`@DataProvider`注释，它允许我们为测试方法映射复杂的参数类型。

下面是一个对原始数据类型使用`@DataProvider`的例子:

```java
@DataProvider(name = "numbers")
public static Object[][] evenNumbers() {
    return new Object[][]{{1, false}, {2, true}, {4, true}};
}

@Test(dataProvider = "numbers")
public void givenNumberFromDataProvider_ifEvenCheckOK_thenCorrect
  (Integer number, boolean expected) {
    Assert.assertEquals(expected, number % 2 == 0);
}
```

和`@DataProvider `对于物体:

```java
@Test(dataProvider = "numbersObject")
public void givenNumberObjectFromDataProvider_ifEvenCheckOK_thenCorrect
  (EvenNumber number) {
    Assert.assertEquals(number.isEven(), number.getValue() % 2 == 0);
}

@DataProvider(name = "numbersObject")
public Object[][] parameterProvider() {
    return new Object[][]{{new EvenNumber(1, false)},
      {new EvenNumber(2, true)}, {new EvenNumber(4, true)}};
}
```

同样，可以使用 data provider 创建和返回任何要测试的特定对象。这在与 Spring 等框架集成时很有用。

注意，在 TestNG 中，由于`@DataProvider` 方法不需要是静态的，我们可以在同一个测试类中使用多个数据提供者方法。

## 7。测试超时

超时测试意味着，如果在某个特定的时间段内执行没有完成，测试用例应该失败。JUnit 和 TestNG 都支持超时测试。在`JUnit 5`中，我们可以将超时测试写成`:`

```java
@Test
public void givenExecution_takeMoreTime_thenFail() throws InterruptedException {
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(10000));
}
```

在`JUnit 4`和 TestNG 中，我们可以使用@ `Test(timeout=1000)`进行相同的测试

```java
@Test(timeOut = 1000)
public void givenExecution_takeMoreTime_thenFail() {
    while (true);
}
```

## 8。相关测试

TestNG 支持依赖测试。这意味着在一组测试方法中，如果初始测试失败，那么所有后续的依赖测试都将被跳过，而不是像 JUnit 那样被标记为失败。

让我们来看一个场景，我们需要验证电子邮件，如果成功，将继续登录:

```java
@Test
public void givenEmail_ifValid_thenTrue() {
    boolean valid = email.contains("@");
    Assert.assertEquals(valid, true);
}

@Test(dependsOnMethods = {"givenEmail_ifValid_thenTrue"})
public void givenValidEmail_whenLoggedIn_thenTrue() {
    LOGGER.info("Email {} valid >> logging in", email);
}
```

## 9。测试执行顺序

在 JUnit 4 或 TestNG 中，测试方法的执行没有明确的隐含顺序。方法只是在 Java 反射 API 返回时被调用。从 JUnit 4 开始，它使用了更确定但不可预测的顺序。

为了有更多的控制，我们将用`@FixMethodOrder` 注释来注释测试类，并提到一个方法排序器:

```java
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SortedTests {

    @Test
    public void a_givenString_whenChangedtoInt_thenTrue() {
        assertTrue(
          Integer.valueOf("10") instanceof Integer);
    }

    @Test
    public void b_givenInt_whenChangedtoString_thenTrue() {
        assertTrue(
          String.valueOf(10) instanceof String);
    }

}
```

`MethodSorters.NAME_ASCENDING` 参数按照方法名的字典顺序对方法进行排序。除了这个分类器，我们还有`MethodSorter.DEFAULT and MethodSorter.JVM as well.`

同时 TestNG 还提供了几种控制测试方法执行顺序的方法。我们在`@Test` 注释中提供了`priority` 参数:

```java
@Test(priority = 1)
public void givenString_whenChangedToInt_thenCorrect() {
    Assert.assertTrue(
      Integer.valueOf("10") instanceof Integer);
}

@Test(priority = 2)
public void givenInt_whenChangedToString_thenCorrect() {
    Assert.assertTrue(
      String.valueOf(23) instanceof String);
}
```

注意，priority 调用基于优先级的测试方法，但是不保证在调用下一个优先级之前完成一个级别的测试。

有时在 TestNG 中编写功能测试用例时，我们可能会有一个相互依赖的测试，其中每个测试运行的执行顺序必须相同。为了实现这一点，我们应该使用@ `Test` 注释的`dependsOnMethods`参数，就像我们在前面部分看到的那样。

## 10.自定义测试名称

默认情况下，每当我们运行一个测试时，测试类和测试方法的名称都会打印在控制台或 IDE 中。`JUnit 5`提供了一个独特的特性，我们可以使用`@DisplayName`注释为类和测试方法提供定制的描述性名称。

这个注释没有提供任何测试好处，但是它也为非技术人员带来了易于阅读和理解的测试结果:

```java
@ParameterizedTest
@ValueSource(strings = { "Hello", "World" })
@DisplayName("Test Method to check that the inputs are not nullable")
void givenString_TestNullOrNot(String word) {
    assertNotNull(word);
}
```

每当我们运行测试时，输出将显示显示名称而不是方法名称。

现在，在`TestNG`中没有办法提供自定义名称。

## 11。结论

JUnit 和 TestNG 都是 Java 生态系统中的现代测试工具。

在本文中，我们快速浏览了使用这两种测试框架编写测试的各种方法。

所有代码片段的实现可以在 [TestNG](https://web.archive.org/web/20220526060647/https://github.com/eugenp/tutorials/tree/master/testing-modules/testng) 和 [junit-5 Github 项目](https://web.archive.org/web/20220526060647/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit5-migration)中找到。