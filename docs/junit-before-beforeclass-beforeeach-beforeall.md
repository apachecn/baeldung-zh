# @ Before vs @ Before class vs @ Before each vs @ Before all

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall>

## 1。概述

在这个简短的教程中，我们将解释 JUnit 4 和 5 中的`@Before`、`@BeforeClass`、`@BeforeEach`和`@BeforeAll`注释之间的区别——以及如何使用它们的实际例子。

我们还将简要介绍它们的补充注释。

先说 JUnit 4。

## 延伸阅读:

## JUnit 5 指南

A quick and practical guide to JUnit 5[Read more](/web/20221208143856/https://www.baeldung.com/junit-5) →

## [JUnit 中的测试顺序](/web/20221208143856/https://www.baeldung.com/junit-5-test-order)

Learn how to run JUnit tests in a custom order.[Read more](/web/20221208143856/https://www.baeldung.com/junit-5-test-order) →

## [快速的 JUnit 与 TestNG 对比](/web/20221208143856/https://www.baeldung.com/junit-vs-testng)

A quick look at JUnit compared to TestNG - the other popular testing framework in the Java ecosystem.[Read more](/web/20221208143856/https://www.baeldung.com/junit-vs-testng) →

## 2。`@Before`

使用`@Before`注释的**方法在每次测试前运行。**当我们想在运行测试之前执行一些公共代码时，这很有用。

让我们初始化一个列表并添加一些值:

```java
@RunWith(JUnit4.class)
public class BeforeAndAfterAnnotationsUnitTest {

    // ...

    private List<String> list;

    @Before
    public void init() {
        LOG.info("startup");
        list = new ArrayList<>(Arrays.asList("test1", "test2"));
    }

    @After
    public void teardown() {
        LOG.info("teardown");
        list.clear();
    }
}
```

**注意，我们还添加了另一个用`@After`标注的方法，以便在每个测试执行后清除列表。**

现在让我们添加一些测试来检查列表的大小:

```java
@Test
public void whenCheckingListSize_thenSizeEqualsToInit() {
    LOG.info("executing test");
    assertEquals(2, list.size());

    list.add("another test");
}

@Test
public void whenCheckingListSizeAgain_thenSizeEqualsToInit() {
    LOG.info("executing another test");
    assertEquals(2, list.size());

    list.add("yet another test");
}
```

在这种情况下，**确保在运行每个测试**之前正确设置测试环境是至关重要的，因为列表在每个测试执行期间都会被修改。

如果我们看一下日志输出，我们可以检查到`init`和`teardown` 方法在每个测试中都运行了一次:

```java
... startup
... executing another test
... teardown
... startup
... executing test
... teardown
```

## 3。`@BeforeClass`

当我们想在每个测试之前执行一个昂贵的公共操作时，**最好在使用`@BeforeClass`运行所有测试之前只执行一次。**

一些常见的高开销操作的例子是创建数据库连接或启动服务器。

让我们创建一个简单的测试类来模拟数据库连接的创建:

```java
@RunWith(JUnit4.class)
public class BeforeClassAndAfterClassAnnotationsUnitTest {

    // ...

    @BeforeClass
    public static void setup() {
        LOG.info("startup - creating DB connection");
    }

    @AfterClass
    public static void tearDown() {
        LOG.info("closing DB connection");
    }
}
```

注意**这些方法必须是静态的**，所以它们将在运行类的测试之前被执行。

像以前一样，让我们也添加一些简单的测试:

```java
@Test
public void simpleTest() {
    LOG.info("simple test");
}

@Test
public void anotherSimpleTest() {
    LOG.info("another simple test");
}
```

这一次，如果我们看一下日志输出，我们可以检查到`setup`和`tearDown`方法只运行了一次:

```java
... startup - creating DB connection
... simple test
... another simple test
... closing DB connection
```

## 4。`@BeforeEach`和`@BeforeAll`

**`@BeforeEac` h 和`@BeforeAll`是`@Before`和`@BeforeClass`的 JUnit 5 等价物。为了避免混淆，这些注释被重新命名为更清晰的名称。**

让我们使用这些新注释复制我们以前的类，从`@BeforeEach`和`@AfterEach`注释开始:

```java
class BeforeEachAndAfterEachAnnotationsUnitTest {

    // ...

    private List<String> list;

    @BeforeEach 
    void init() {
        LOG.info("startup");
        list = new ArrayList<>(Arrays.asList("test1", "test2"));
    }

    @AfterEach
    void teardown() {
        LOG.info("teardown");
        list.clear();
    }

    // ...
}
```

如果我们检查日志，我们可以确认它的工作方式与使用`@Before`和`@After`注释的方式相同:

```java
... startup
... executing another test
... teardown
... startup
... executing test
... teardown
```

最后，让我们对另一个测试类做同样的事情，看看`@BeforeAll`和`@AfterAll`注释的作用:

```java
class BeforeAllAndAfterAllAnnotationsUnitTest {

    // ...

    @BeforeAll
    static void setup() {
        LOG.info("startup - creating DB connection");
    }

    @AfterAll
    static void tearDown() {
        LOG.info("closing DB connection");
    }

    // ...
}
```

输出与旧的注释相同:

```java
... startup - creating DB connection
... simple test
... another simple test
... closing DB connection
```

## 5。结论

在本文中，我们展示了 JUnit 中的`@Before`、`@BeforeClass`、`@BeforeEach`和`@BeforeAll`注释之间的区别，以及何时应该使用它们中的每一个。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics)