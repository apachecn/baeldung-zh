# JUnit 5 中的@TestInstance 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-testinstance-annotation>

## 1.介绍

测试类通常包含引用被测系统、模拟或者测试中使用的数据资源的成员变量。默认情况下，JUnit 4 和 5 在运行每个测试方法之前都会创建一个测试类的新实例。这在测试之间提供了一个清晰的状态分离。

在本教程中，我们将学习 [JUnit 5](/web/20220626073541/https://www.baeldung.com/junit-5) 如何允许我们使用`@TestInstance`注释来修改测试类的生命周期。我们还将看到这如何帮助我们管理大型资源或者测试之间更复杂的关系。

## 2.默认测试生命周期

让我们从默认的测试类生命周期开始，这是 JUnit 4 和 5 共有的:

```java
class AdditionTest {

    private int sum = 1;

    @Test
    void addingTwoReturnsThree() {
        sum += 2;
        assertEquals(3, sum);
    }

    @Test
    void addingThreeReturnsFour() {
        sum += 3;
        assertEquals(4, sum);
    }
} 
```

除了 JUnit 5 不需要的关键字`public`之外，这段代码很容易成为 JUnit 4 或 5 的测试代码。

这些测试通过是因为在调用每个测试方法之前创建了一个新的`AdditionTest` 实例。这意味着在执行每个测试之前，变量`sum`的值总是被设置为`1`。

如果只有一个测试对象的共享实例，变量`sum`将在每次测试后保持其状态。因此，第二次测试会失败。

## 3.@ `BeforeClass` 和 `@BeforeAll`标注

有时候，我们需要一个对象跨多个测试存在。假设我们想要读取一个大文件作为测试数据。因为在每次测试之前重复它可能很费时间，我们可能更喜欢读一次，并在整个测试夹具中保存它。

JUnit 4 通过其`@BeforeClass`注释解决了这个问题:

```java
private static String largeContent;

@BeforeClass
public static void setUpFixture() {
    // read the file and store in 'largeContent'
}
```

我们应该注意到**我们必须让用 JUnit 4 的`@BeforeClass` 注释的变量和方法成为静态的。**

JUnit 5 提供了一种不同的方法。它提供了用在静态函数上的`@BeforeAll`注释，以处理类的静态成员。

然而，如果测试实例生命周期被更改为`per-class`，那么`@BeforeAll` 也可以与实例函数和实例成员一起使用。

## 4.`@TestInstance`注解

`@TestInstance`注释让我们配置 JUnit 5 测试的生命周期。

**`@TestInstance`有两种模式。**一个是`LifeCycle.PER_METHOD`(默认)。另一个是`LifeCycle.PER_CLASS`。后者使我们能够要求 JUnit 只创建测试类的一个实例，并在测试之间重用它。

让我们用`@TestInstance`注释来注释我们的测试类，并使用`LifeCycle.PER_CLASS`模式:

```java
@TestInstance(LifeCycle.PER_CLASS)
class TweetSerializerUnitTest {

    private String largeContent;

    @BeforeAll
    void setUpFixture() {
        // read the file
    }

}
```

正如我们所见，没有一个变量或函数是静态的。当我们使用`PER_CLASS` 生命周期时，我们被允许为`@BeforeAll`使用一个**实例方法。**

我们还应该注意到，一个测试对实例变量的状态所做的更改现在对其他测试也是可见的。

## 5.`@TestInstance(PER_CLASS)`的用途

### 5.1.昂贵的资源

当在每次测试之前实例化一个类是非常昂贵的时候，这个注释是非常有用的。一个例子可能是建立一个数据库连接，或者加载一个大文件。

以前解决这个问题会导致静态变量和实例变量的复杂混合，现在使用共享的测试类实例会更清晰。

### 5.2.故意共享状态

共享状态在单元测试中通常是一种反模式，但在集成测试中却很有用。每个类的生命周期支持有意共享状态的顺序测试。为了避免后面的测试不得不重复前面测试的步骤，这可能是必要的，特别是如果让被测系统进入正确的状态很慢的话。

当共享状态时，为了按顺序执行所有的测试，JUnit 5 为我们提供了类型级的`[@TestMethodOrder](/web/20220626073541/https://www.baeldung.com/junit-5-test-order)`注释。然后我们可以在测试方法上使用`[@Order](/web/20220626073541/https://www.baeldung.com/junit-5-test-order)`注释，按照我们选择的顺序执行它们。

```java
@TestMethodOrder(OrderAnnotation.class)
class OrderUnitTest {

    @Test
    @Order(1)
    void firstTest() {
        // ...
    }

    @Test
    @Order(2)
    void secondTest() {
        // ...
    }

}
```

### 5.3.共享一些状态

共享测试类的同一个实例的挑战是，一些成员可能需要在测试之间被清理，而一些可能需要在整个测试期间被维护。

**我们可以用标注了`[@BeforeEach](/web/20220626073541/https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall)`或`@AfterEach`的方法重置测试之间需要清理的变量。**

## 6.结论

在本教程中，我们学习了`@TestInstance`注释，以及如何使用它来配置 JUnit 5 测试的生命周期。

我们还研究了为什么共享测试类的单个实例在处理共享资源或有意编写顺序测试方面是有用的。

和往常一样，本教程的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626073541/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-advanced)