# 非静态方法中的@BeforeAll 和@AfterAll

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-beforeall-afterall-non-static>

## 1.概观

在这个简短的教程中，我们将使用`[Junit5](/web/20221128035106/https://www.baeldung.com/junit-5)`中可用的`@BeforeAll`和`@AfterAll`注释来实现非静态方法。

## 2.非静态方法中的`@BeforeAll`和`@AfterAll`

当进行单元测试时，我们可能偶尔想在非静态设置和拆卸方法中使用`@BeforeAll`和`@AfterAll`——例如，在@ `Nested`测试类中或者作为接口默认方法。

让我们用非静态的`@BeforeAll`和`@AfterAll`方法创建一个测试类:

```java
public class BeforeAndAfterAnnotationsUnitTest {

    String input;
    Long result;

    @BeforeAll
    public void setup() {
        input = "77";
    }

    @AfterAll
    public void teardown() {
        input = null;
        result = null;
    }

    @Test
    public void whenConvertStringToLong_thenResultShouldBeLong() {
        result = Long.valueOf(input);
        Assertions.assertEquals(77l, result);
    }​
}
```

如果我们运行上面的代码，它将抛出一个异常:

```java
org.junit.platform.commons.JUnitException:  ...
```

现在让我们看看如何避免这种情况。

## 3.`@TestInstance`注解

我们将使用 [`@TestInstance`](/web/20221128035106/https://www.baeldung.com/junit-testinstance-annotation) 注释来配置测试的生命周期。如果我们没有在我们的测试类中声明它，生命周期模式将默认为`PER_METHOD``.`所以，**为了防止我们的测试类抛出`JUnitException,`，我们需要用`@TestInstance(TestInstance.`** `**Lifecycle.PER_CLASS)**.`来注释它

让我们重做我们的测试类并添加`@TestInstance(TestInstance.` `Lifecycle.PER_CLASS):`

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class BeforeAndAfterAnnotationsUnitTest {

    String input;
    Long result;

    @BeforeAll
    public void setup() {
        input = "77";
    }

    @AfterAll
    public void teardown() {
        input = null;
        result = null;
    }

    @Test
    public void whenConvertStringToLong_thenResultShouldBeLong() {
        result = Long.valueOf(input);
        Assertions.assertEquals(77l, result);
    }
} 
```

在这种情况下，我们的测试运行成功。

## 4.结论

在这篇短文中，我们学习了如何在非静态方法中使用`@BeforeAll`和`@AfterAll`。首先，我们从一个简单的非静态例子开始，展示如果我们不包含`@TestInstance`注释会发生什么。然后，我们用`@TestInstance(TestInstance.Lifecycle.PER_CLASS)`注释了我们的测试，以防止抛出 *JUnitException* 。

和往常一样，所有这些例子的实现都在 GitHub 上[完成。](https://web.archive.org/web/20221128035106/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)